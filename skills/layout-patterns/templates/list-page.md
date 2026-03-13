# List Page Layout Template

Reference: [Storybook Example: Card Grid](https://cognitedata.github.io/aura/storybook/?path=/story/foundations-layout--card-grid-layout)

```tsx
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table';
import { Pagination } from '@/components/ui/pagination';
import { Skeleton } from '@/components/ui/skeleton';

export function ListPage() {
  return (
    <div className="space-y-6">
      {/* Page header */}
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-semibold text-foreground">
          Reports
        </h1>
        <Button>Create report</Button>
      </div>

      {/* Filters toolbar */}
      <div className="flex items-center gap-3">
        <Input
          placeholder="Search reports..."
          className="max-w-sm"
          aria-label="Search reports"
        />
        {/* Additional filter controls */}
      </div>

      {/* Data table */}
      {isLoading ? (
        <div className="space-y-3">
          <Skeleton className="h-10 w-full" />
          <Skeleton className="h-10 w-full" />
          <Skeleton className="h-10 w-full" />
          <Skeleton className="h-10 w-full" />
        </div>
      ) : data.length === 0 ? (
        /* Empty state */
        <div className="flex flex-col items-center justify-center 
                        rounded-lg border border-dashed border-border 
                        py-16 text-center">
          <p className="text-lg font-medium text-foreground">
            No reports yet
          </p>
          <p className="mt-1 text-sm text-muted-foreground">
            Create your first report to get started.
          </p>
          <Button className="mt-4">Create report</Button>
        </div>
      ) : (
        <div className="rounded-lg border border-border">
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Name</TableHead>
                <TableHead>Status</TableHead>
                <TableHead>Created</TableHead>
                <TableHead className="text-right">Actions</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {data.map((item) => (
                <TableRow key={item.id}>
                  <TableCell className="font-medium">
                    {item.name}
                  </TableCell>
                  <TableCell>{item.status}</TableCell>
                  <TableCell>{item.createdAt}</TableCell>
                  <TableCell className="text-right">
                    {/* Action menu */}
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </div>
      )}

      {/* Pagination */}
      {data.length > 0 && (
        <div className="flex justify-center">
          <Pagination />
        </div>
      )}
    </div>
  );
}
```

Key points:
- Header with title + primary action button (top-right)
- Search input constrained with `max-w-sm`, has `aria-label`
- Skeleton loading state matches table row heights
- Empty state with explanation + CTA button
- Table wrapped in `border border-border rounded-lg`
- Pagination centered below table

Responsive notes:
- On mobile, consider hiding non-essential table columns
- For card-based variant, use `grid-cols-1 sm:grid-cols-2 lg:grid-cols-3`
  with Card components instead of Table
