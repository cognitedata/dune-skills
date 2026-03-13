# Full-Width Dashboard Layout Template

Reference: [Storybook Example: Dashboard](https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--dashboard-layout)
Reference: [Storybook Example: Comprehensive Dashboard](https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--comprehensive-dashboard)

```tsx
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';

export function DashboardPage() {
  return (
    <div className="space-y-6">
      {/* Page header with filters */}
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-semibold text-foreground">
          Dashboard
        </h1>
        <div className="flex items-center gap-3">
          {/* Filter controls */}
        </div>
      </div>

      {/* Metric cards — responsive grid */}
      <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-4">
        <MetricCard title="Total reports" value="1,234" />
        <MetricCard title="Active users" value="56" />
        <MetricCard title="Completion rate" value="87%" />
        <MetricCard title="Avg. response time" value="2.3s" />
      </div>

      {/* Charts — responsive grid */}
      <div className="grid grid-cols-1 gap-6 lg:grid-cols-2">
        <Card>
          <CardHeader>
            <CardTitle>Trend over time</CardTitle>
          </CardHeader>
          <CardContent>
            {/* Chart component */}
          </CardContent>
        </Card>
        <Card>
          <CardHeader>
            <CardTitle>Distribution</CardTitle>
          </CardHeader>
          <CardContent>
            {/* Chart component */}
          </CardContent>
        </Card>
      </div>

      {/* Data table — full width */}
      <Card>
        <CardHeader>
          <CardTitle>Recent activity</CardTitle>
        </CardHeader>
        <CardContent>
          {/* DataTable component with empty state */}
        </CardContent>
      </Card>
    </div>
  );
}

function MetricCard({ title, value }: { title: string; value: string }) {
  return (
    <Card>
      <CardContent className="pt-6">
        <p className="text-sm text-muted-foreground">{title}</p>
        <p className="text-2xl font-bold text-foreground">{value}</p>
      </CardContent>
    </Card>
  );
}
```

Key points:
- `max-w-7xl` can wrap the whole page if centered content is desired
- Metric grid: `grid-cols-1 sm:grid-cols-2 lg:grid-cols-4`
- Chart grid: `grid-cols-1 lg:grid-cols-2`
- Use `gap-4` for card grids, `gap-6` for chart/section grids
- All colors use semantic tokens (text-foreground, text-muted-foreground)
