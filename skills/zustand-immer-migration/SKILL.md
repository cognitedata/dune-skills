---
name: zustand-immer-migration
description: Migrate React component state (useState, useReducer, custom hooks) to a Zustand+Immer store with IndexedDB persistence. Use when the user wants to extract state from React components into a standalone store, decouple state from the component lifecycle, convert useState/useReducer to Zustand, add Immer middleware, or persist state to IndexedDB.
---

# Zustand + Immer Store Migration

Migrate React `useState`/`useReducer`/custom-hook state into a Zustand store with
Immer middleware so that state is decoupled from the component lifecycle, accessible
from plain TS (controllers, services), and supports concurrent writers.

## When to Use

- State is shared across many components via prop drilling or context
- Non-React code (controllers, services, async flows) needs to read/write state
- Multiple async operations update state concurrently
- State needs persistence (IndexedDB via `idb-keyval`)
- You hit race conditions from stale closures in `useState` callbacks

## Dependencies

```bash
pnpm add zustand immer idb-keyval
```

## File Layout

```
src/stores/
├── fooStore.ts              # Vanilla Zustand store (no React)
└── useFooStore.ts           # React selector hooks (thin wrappers)
```

**Key principle**: The store file has zero React imports. React hooks live in a
separate file so the store can be called from anywhere.

## Step 1: Create the Vanilla Store

Use `createStore` (from `zustand/vanilla`), NOT `create` (from `zustand`).
This keeps the store framework-agnostic.

```typescript
import { createStore } from "zustand/vanilla";
import { immer } from "zustand/middleware/immer";
import { createJSONStorage, persist } from "zustand/middleware";
import { del, get, set } from "idb-keyval";

// IndexedDB adapter for persist middleware
const indexedDBStorage = createJSONStorage(() => ({
  getItem: async (name: string) =>
    ((await get(name)) as string | undefined) ?? null,
  setItem: async (name: string, value: string) => {
    await set(name, value);
  },
  removeItem: async (name: string) => {
    await del(name);
  },
}));

interface FooState {
  items: Record<string, Item>;
  activeOps: Record<string, ActiveOp>;  // ephemeral, exclude from persistence
  _hasHydrated: boolean;

  // Actions
  addItem: (item: Item) => void;
  removeItem: (id: string) => void;

  // Async actions receive external deps as params (SDK, services)
  // so the store stays serialisable
  doSomethingAsync: (params: {
    id: string;
    sdk: SomeClient;
  }) => Promise<Result>;
}

export const fooStore = createStore<FooState>()(
  persist(
    immer((set, get) => ({
      items: {},
      activeOps: {},
      _hasHydrated: false,

      addItem: (item) => {
        set((s) => { s.items[item.id] = item; });
      },

      removeItem: (id) => {
        set((s) => { delete s.items[id]; });
      },

      doSomethingAsync: async ({ id, sdk }) => {
        set((s) => {
          s.activeOps[id] = { status: "running", startTime: Date.now() };
        });
        try {
          const result = await sdk.doWork(id);
          set((s) => {
            s.items[id].result = result;
            delete s.activeOps[id];
          });
          return result;
        } catch (err) {
          set((s) => { delete s.activeOps[id]; });
          throw err;
        }
      },
    })),
    {
      name: "my-store-key",      // IndexedDB key
      storage: indexedDBStorage,
      partialize: (state): Partial<FooState> => ({
        items: state.items,
      }),
      onRehydrateStorage: () => () => {
        fooStore.setState({ _hasHydrated: true });
      },
    },
  ),
);
```

### Key Patterns

| Pattern | Guidance |
|---------|----------|
| **Dates** | Store as ISO strings to avoid serialisation issues. Convert at the boundary (selector hooks). |
| **Non-serialisable deps** | Pass SDK, services, runtimes as action *parameters*, not store state. |
| **Ephemeral state** | Track in-flight ops in a separate record; exclude from `partialize`. |
| **Module-level caches** | Keep service instance caches (e.g. `Map<string, Service>`) outside the store as module variables. |
| **Async rehydration** | IndexedDB is async; store starts empty then rehydrates. Use `_hasHydrated` flag if consumers need to wait. |
| **Concurrency** | Immer drafts are synchronous. Each `set()` call is atomic. Multiple async actions on *different* keys are safe. Multiple writers to the *same* key require careful ordering. |

## Step 2: Create React Selector Hooks

```typescript
import { useCallback, useMemo } from "react";
import { useStore } from "zustand";
import { useShallow } from "zustand/react/shallow";
import { fooStore, type FooState } from "./fooStore";

// Narrow selectors minimise re-renders
export function useItem(id: string | undefined) {
  return useStore(
    fooStore,
    useCallback(
      (s: FooState) => (id ? s.items[id] : undefined),
      [id],
    ),
  );
}

export function useIsActive(id: string | undefined): boolean {
  return useStore(
    fooStore,
    useCallback(
      (s: FooState) => (id ? id in s.activeOps : false),
      [id],
    ),
  );
}

// For derived lists, use useShallow on the source record, then useMemo
export function useFilteredItems(filter?: Filter) {
  const allItems = useStore(
    fooStore,
    useShallow((s: FooState) => s.items),
  );
  return useMemo(() => {
    let items = Object.values(allItems);
    if (filter?.type) items = items.filter((i) => i.type === filter.type);
    return items;
  }, [allItems, filter?.type]);
}

export function useHasHydrated(): boolean {
  return useStore(
    fooStore,
    useCallback((s: FooState) => s._hasHydrated, []),
  );
}
```

## Step 3: Migrate Callers

### From React components

Replace `useState`/context reads with selector hooks:

```typescript
// Before
const { items, addItem } = useMyContext();

// After
const items = useFilteredItems();
const addItem = fooStore.getState().addItem;  // stable, no hook needed
```

### From non-React code (controllers, services, async flows)

Call store actions directly — no hooks, no refs, no component routing:

```typescript
// Before: routed through a UI component ref
await componentRef.current?.doThing(id);

// After: call store directly
await fooStore.getState().doSomethingAsync({ id, sdk });
```

### Bridging legacy interfaces

If existing components expect the old hook return shape, create a compat hook
that wraps the store and returns the legacy interface. Migrate components off the
compat hook incrementally.

```typescript
export function useFooCompat() {
  const storeItems = useStore(fooStore, (s) => s.items);
  const [selectedId, setSelectedId] = useState<string | null>(null);

  // Convert store types to legacy types at the boundary
  const legacyItems = useMemo(
    () => Object.values(storeItems).map(toLegacyItem),
    [storeItems],
  );

  return { items: legacyItems, selectedId, setSelectedId, /* ... */ };
}
```

**Compat hooks are temporary.** Move them toward deletion as soon as practical:

1. **Avoid external piping.** If a parent component creates the compat object
   and passes it as a prop to a child, the child should own the hook instead.
   Passing converted data through props creates circular flows (parent reads
   store → converts → passes to child → child syncs back to store).

2. **Prefer internal adapter hooks.** When a consumer component needs the
   legacy interface but the rest of the app already uses the store, create a
   small adapter hook *inside* the component's module rather than lifting it
   to the parent. The adapter reads the store via selector hooks, converts
   types (e.g. ISO strings → Date objects) at the boundary, and exposes the
   legacy interface. `currentConversationId` and other UI state that was
   previously managed by the compat hook can instead come from a prop or
   external selector.

3. **Delete the compat file** once all consumers call the store directly or
   use their own internal adapter.

## Step 4: Clean Up

After migration:

- Delete the old hooks/context if no longer imported
- Remove prop drilling and context providers that existed only to pass state
- Remove imperative refs (`useImperativeHandle`, `forwardRef`) that existed to
  trigger actions on child components
- Remove `flushSync`, `requestAnimationFrame` polling, and other workarounds for
  stale-closure / async-state issues

## Devtools

Add `devtools` middleware in development for time-travel debugging in the Redux
DevTools browser extension. Wrap the outermost middleware:

```typescript
import { devtools } from "zustand/middleware";

const withDevtools = import.meta.env.DEV
  ? (fn: Parameters<typeof devtools>[0]) =>
      devtools(fn, { name: "FooStore" })
  : (fn: Parameters<typeof devtools>[0]) => fn;

export const fooStore = createStore<FooState>()(
  withDevtools(
    persist(
      immer((set, get) => ({ /* ... */ })),
      { /* persist options */ },
    ),
  ),
);
```

## Testing

Vanilla stores are trivially testable — no React rendering needed:

```typescript
import { createFooStore } from "./fooStore";

test("addItem inserts into the store", () => {
  const store = createFooStore("test-key");
  store.getState().addItem({ id: "a", name: "Alpha" });

  expect(store.getState().items["a"]).toEqual({ id: "a", name: "Alpha" });
});

test("removeItem deletes from the store", () => {
  const store = createFooStore("test-key");
  store.getState().addItem({ id: "a", name: "Alpha" });
  store.getState().removeItem("a");

  expect(store.getState().items["a"]).toBeUndefined();
});
```

Export a `createFooStore(name)` factory (in addition to the singleton) so each
test gets a fresh instance with no shared state.

## Gotchas

1. **`createStore` vs `create`**: Use `createStore` (vanilla) for the store file;
   use `useStore` in React hooks. Do NOT use `create` unless the store is only ever
   consumed from React.

2. **`createStore<T>()(...)` double-call**: The first call applies the type
   parameter; the second passes the middleware-wrapped initialiser. This is
   required by TypeScript for correct middleware type inference — it is not a bug.

3. **Middleware ordering**: Middleware is nested inside-out.
   `persist(immer(fn))` means Immer runs first (closest to your code),
   persist wraps the result. Reversing them breaks Immer draft detection.
   Rule of thumb: **immer innermost, persist around it, devtools outermost**.

4. **`partialize` return type**: Annotate the return as `Partial<FooState>`.
   Zustand's `PersistOptions` accepts `PersistedState extends Partial<S>`, so
   `Partial<FooState>` works out of the box with no casts.

5. **Immer + async**: `set()` inside an async action uses a *new* draft each time.
   Do not hold references to draft objects across `await` boundaries.

6. **Shallow equality**: `useStore` uses `Object.is` by default. For object/array
   selectors, use `useShallow` or a custom equality function to avoid spurious
   re-renders.

7. **IndexedDB is async**: The store starts with default state and rehydrates after
   ~5-10 ms. Components that render before rehydration see empty state. Gate on
   `_hasHydrated` if this matters.
