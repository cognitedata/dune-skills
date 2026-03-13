# Sidebar + Content Layout Template

Reference: [Storybook Example: Sidebar Left](https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--sidebar-left-layout)

```tsx
import {
  SidebarProvider,
  Sidebar,
  SidebarContent,
  SidebarGroup,
  SidebarGroupLabel,
  SidebarGroupContent,
  SidebarMenu,
  SidebarMenuItem,
  SidebarMenuButton,
} from '@/components/ui/sidebar';
import { Breadcrumb, BreadcrumbItem, BreadcrumbLink, BreadcrumbSeparator } from '@/components/ui/breadcrumb';

export function AppLayout({ children }: { children: React.ReactNode }) {
  return (
    <SidebarProvider>
      <div className="flex min-h-screen w-full">
        <Sidebar>
          <SidebarContent>
            <SidebarGroup>
              <SidebarGroupLabel>Navigation</SidebarGroupLabel>
              <SidebarGroupContent>
                <SidebarMenu>
                  <SidebarMenuItem>
                    <SidebarMenuButton isActive>
                      Dashboard
                    </SidebarMenuButton>
                  </SidebarMenuItem>
                  <SidebarMenuItem>
                    <SidebarMenuButton>
                      Reports
                    </SidebarMenuButton>
                  </SidebarMenuItem>
                </SidebarMenu>
              </SidebarGroupContent>
            </SidebarGroup>
          </SidebarContent>
        </Sidebar>

        <main className="flex-1 min-w-0 bg-background">
          <div className="px-6 py-8 space-y-6">
            <Breadcrumb>
              <BreadcrumbItem>
                <BreadcrumbLink href="/">Home</BreadcrumbLink>
              </BreadcrumbItem>
              <BreadcrumbSeparator />
              <BreadcrumbItem>
                <BreadcrumbLink>Current page</BreadcrumbLink>
              </BreadcrumbItem>
            </Breadcrumb>

            {children}
          </div>
        </main>
      </div>
    </SidebarProvider>
  );
}
```

Key points:
- `min-w-0` on main prevents flex child overflow
- `flex-1` makes content fill remaining space
- Sidebar component handles collapse behavior automatically
- `bg-background` on main for theme-aware background
- Standard padding: `px-6 py-8`
- Standard section spacing: `space-y-6`
