# Detail Page Layout Template

```tsx
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import {
  Breadcrumb, BreadcrumbItem, BreadcrumbLink,
  BreadcrumbSeparator,
} from '@/components/ui/breadcrumb';

export function DetailPage({ report }: { report: Report }) {
  return (
    <div className="space-y-6">
      {/* Breadcrumb */}
      <Breadcrumb>
        <BreadcrumbItem>
          <BreadcrumbLink href="/reports">Reports</BreadcrumbLink>
        </BreadcrumbItem>
        <BreadcrumbSeparator />
        <BreadcrumbItem>
          <BreadcrumbLink>{report.name}</BreadcrumbLink>
        </BreadcrumbItem>
      </Breadcrumb>

      {/* Record header */}
      <div className="flex items-start justify-between">
        <div className="space-y-1">
          <h1 className="text-2xl font-semibold text-foreground">
            {report.name}
          </h1>
          <div className="flex items-center gap-2">
            <Badge variant="default">{report.status}</Badge>
            <span className="text-sm text-muted-foreground">
              Last updated {report.updatedAt}
            </span>
          </div>
        </div>
        <div className="flex items-center gap-2">
          <Button variant="secondary">Edit report</Button>
          <Button variant="destructive">Delete report</Button>
        </div>
      </div>

      {/* Two-column layout: main (2/3) + sidebar (1/3) */}
      <div className="grid grid-cols-1 gap-6 lg:grid-cols-3">
        {/* Main content — spans 2 columns on desktop */}
        <div className="space-y-6 lg:col-span-2">
          <Card>
            <CardHeader>
              <CardTitle>Details</CardTitle>
            </CardHeader>
            <CardContent>
              {/* Primary content */}
            </CardContent>
          </Card>
        </div>

        {/* Sidebar — 1 column on desktop, stacks below on mobile */}
        <div className="space-y-6">
          <Card>
            <CardHeader>
              <CardTitle>Metadata</CardTitle>
            </CardHeader>
            <CardContent>
              {/* Secondary content */}
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  );
}
```

Key points:
- `max-w-4xl` can wrap the page for constrained width
- Main/sidebar split: `grid-cols-3` with `col-span-2` + `col-span-1`
- On mobile: stacks to single column automatically
- Breadcrumb provides location context
- Action buttons in header use specific verbs
