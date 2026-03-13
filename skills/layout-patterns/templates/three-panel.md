# Three-Panel Layout Template

Reference: [Storybook Pattern: Layout Compositions](https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--compositions)

```tsx
import { Button } from '@/components/ui/button';
import { Drawer, DrawerContent, DrawerTrigger } from '@/components/ui/drawer';

export function ThreePanelLayout() {
  const [showRight, setShowRight] = useState(true);

  return (
    <div className="flex h-screen overflow-hidden">
      {/* Left panel — fixed width, scrollable */}
      <aside className="hidden w-64 shrink-0 border-r border-border 
                        bg-card overflow-y-auto md:block">
        <div className="p-4 space-y-2">
          <h2 className="text-sm font-medium text-muted-foreground 
                         uppercase tracking-wider">
            Navigation
          </h2>
          {/* Tree view / nav items */}
        </div>
      </aside>

      {/* Center panel — flexible, takes remaining space */}
      <main className="flex-1 min-w-0 overflow-y-auto">
        <div className="px-6 py-4 space-y-4">
          <div className="flex items-center justify-between">
            <h1 className="text-xl font-semibold text-foreground">
              Main content
            </h1>
            <Button
              variant="ghost"
              size="sm"
              onClick={() => setShowRight(!showRight)}
              className="hidden md:inline-flex"
            >
              {showRight ? 'Hide details' : 'Show details'}
            </Button>
          </div>
          {/* Primary content */}
        </div>
      </main>

      {/* Right panel — toggleable on tablet, drawer on mobile */}
      {showRight && (
        <aside className="hidden w-80 shrink-0 border-l border-border 
                          bg-card overflow-y-auto lg:block">
          <div className="p-4 space-y-4">
            <h2 className="text-lg font-medium text-foreground">
              Properties
            </h2>
            {/* Detail/properties content */}
          </div>
        </aside>
      )}

      {/* Mobile: left panel as drawer */}
      <Drawer>
        <DrawerTrigger asChild className="md:hidden fixed bottom-4 left-4">
          <Button variant="secondary" size="sm">
            Navigation
          </Button>
        </DrawerTrigger>
        <DrawerContent>
          <div className="p-4 space-y-2">
            {/* Same nav content as left aside */}
          </div>
        </DrawerContent>
      </Drawer>
    </div>
  );
}
```

Key points:
- Left panel: `w-64 shrink-0` fixed width, hidden below `md:`
- Center panel: `flex-1 min-w-0` fills remaining space
- Right panel: `w-80 shrink-0` fixed width, hidden below `lg:`
- `overflow-y-auto` on each panel for independent scrolling
- `overflow-hidden` on container prevents page-level scroll
- Toggle button to show/hide right panel on tablet
- Drawer component for left nav on mobile
- `border-border` and `bg-card` for panel theming

Responsive behavior:
- Desktop (1440px+): all 3 panels visible
- Tablet (768-1439px): left + center visible, right toggleable
- Mobile (<768px): center only, left as drawer
