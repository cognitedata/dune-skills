# Settings Page Layout Template

```tsx
// Note: Aura documents this as 'Segmented Control' in Storybook. The React API exports it as Tabs.
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';

export function SettingsPage() {
  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-semibold text-foreground">
        Settings
      </h1>

      {/* Desktop: side nav + content. Tablet/Mobile: tabs */}
      <div className="hidden lg:grid lg:grid-cols-4 lg:gap-6">
        {/* Left navigation — desktop only */}
        <nav className="space-y-1">
          <SettingsNavItem active>General</SettingsNavItem>
          <SettingsNavItem>Notifications</SettingsNavItem>
          <SettingsNavItem>Security</SettingsNavItem>
          <SettingsNavItem>Integrations</SettingsNavItem>
        </nav>

        {/* Settings content — 3/4 width */}
        <div className="lg:col-span-3 space-y-6">
          <Card>
            <CardHeader>
              <CardTitle>General settings</CardTitle>
            </CardHeader>
            <CardContent className="space-y-4">
              {/* Settings form fields */}
            </CardContent>
          </Card>
          <div className="flex justify-end">
            <Button>Save settings</Button>
          </div>
        </div>
      </div>

      {/* Tablet/Mobile: tabs instead of side nav */}
      <div className="lg:hidden">
        <Tabs defaultValue="general">
          <TabsList>
            <TabsTrigger value="general">General</TabsTrigger>
            <TabsTrigger value="notifications">Notifications</TabsTrigger>
            <TabsTrigger value="security">Security</TabsTrigger>
            <TabsTrigger value="integrations">Integrations</TabsTrigger>
          </TabsList>
          <TabsContent value="general" className="mt-6 space-y-6">
            <Card>
              <CardContent className="pt-6 space-y-4">
                {/* Same settings form */}
              </CardContent>
            </Card>
          </TabsContent>
          {/* Other tab contents */}
        </Tabs>
      </div>
    </div>
  );
}

function SettingsNavItem({ 
  children, active = false 
}: { children: React.ReactNode; active?: boolean }) {
  return (
    <button
      className={`w-full rounded-md px-3 py-2 text-left text-sm
        ${active 
          ? 'bg-accent text-accent-foreground font-medium' 
          : 'text-muted-foreground hover:bg-hover hover:text-foreground'
        }`}
    >
      {children}
    </button>
  );
}
```

Key points:
- Desktop: left nav (1/4) + content (3/4) using `grid-cols-4`
- Tablet/Mobile: switches to Segmented Control component
- `hidden lg:grid` / `lg:hidden` for responsive swap
- Active nav item uses `bg-accent` semantic token
