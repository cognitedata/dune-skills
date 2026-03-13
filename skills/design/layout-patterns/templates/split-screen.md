# Split-Screen (2-Panel) Layout Template

Reference: [Storybook Pattern: Column Spans](https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--column-spans)

```tsx
// Note: Aura documents this as 'Segmented Control' in Storybook. The React API exports it as Tabs.
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';

export function SplitScreenLayout() {
  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-semibold text-foreground">
        Compare reports
      </h1>

      {/* Desktop/Tablet: side-by-side panels */}
      <div className="hidden sm:grid sm:grid-cols-2 sm:gap-6">
        <div className="min-w-0 space-y-4">
          <h2 className="text-lg font-medium text-foreground">
            Left panel
          </h2>
          <div className="rounded-lg border border-border 
                          bg-card p-4 min-h-[400px]">
            {/* Left panel content */}
          </div>
        </div>
        <div className="min-w-0 space-y-4">
          <h2 className="text-lg font-medium text-foreground">
            Right panel
          </h2>
          <div className="rounded-lg border border-border 
                          bg-card p-4 min-h-[400px]">
            {/* Right panel content */}
          </div>
        </div>
      </div>

      {/* Mobile: tabs to switch between panels */}
      <div className="sm:hidden">
        <Tabs defaultValue="left">
          <TabsList className="w-full">
            <TabsTrigger value="left" className="flex-1">
              Left panel
            </TabsTrigger>
            <TabsTrigger value="right" className="flex-1">
              Right panel
            </TabsTrigger>
          </TabsList>
          <TabsContent value="left" className="mt-4">
            <div className="rounded-lg border border-border 
                            bg-card p-4 min-h-[400px]">
              {/* Left panel content */}
            </div>
          </TabsContent>
          <TabsContent value="right" className="mt-4">
            <div className="rounded-lg border border-border 
                            bg-card p-4 min-h-[400px]">
              {/* Right panel content */}
            </div>
          </TabsContent>
        </Tabs>
      </div>
    </div>
  );
}
```

Key points:
- `grid-cols-2` for equal-width panels
- `min-w-0` on each panel prevents flex/grid overflow
- On mobile (`sm:hidden`): Segmented Control component switches between panels
- Panels use `bg-card` and `border-border` for consistent theming
- Use `gap-6` between panels for comfortable spacing

Variants:
- **Resizable split**: Add a drag handle between panels using a 
  custom resize component
- **Asymmetric split**: Use `grid-cols-5` with `col-span-3` + 
  `col-span-2` for a 60/40 split
