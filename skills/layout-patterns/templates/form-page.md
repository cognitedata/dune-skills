# Form Page Layout Template

```tsx
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { HelperText } from '@/components/ui/helper-text';
import { Separator } from '@/components/ui/separator';

export function FormPage() {
  return (
    <div className="space-y-6">
      {/* Back navigation */}
      <Button variant="ghost" size="sm" onClick={goBack}>
        ← Back to reports
      </Button>

      {/* Form container — centered, constrained width */}
      <div className="mx-auto max-w-2xl space-y-8">
        <h1 className="text-2xl font-semibold text-foreground">
          Create report
        </h1>

        {/* Section 1 */}
        <section className="space-y-4">
          <h2 className="text-lg font-medium text-foreground">
            Basic information
          </h2>
          <div className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="name">
                Report name <span className="text-destructive">*</span>
              </Label>
              <Input
                id="name"
                placeholder="Q2 Production summary"
                aria-describedby="name-helper"
              />
              <HelperText id="name-helper">
                A descriptive name for your report.
              </HelperText>
            </div>
          </div>
        </section>

        <Separator />

        {/* Section 2 */}
        <section className="space-y-4">
          <h2 className="text-lg font-medium text-foreground">
            Configuration
          </h2>
          <div className="space-y-4">
            {/* More form fields */}
          </div>
        </section>
      </div>

      {/* Sticky footer */}
      <div className="sticky bottom-0 border-t border-border 
                      bg-background px-6 py-4">
        <div className="mx-auto flex max-w-2xl justify-end gap-3">
          <Button variant="secondary" onClick={handleCancel}>
            Cancel
          </Button>
          <Button onClick={handleSave}>
            Create report
          </Button>
        </div>
      </div>
    </div>
  );
}
```

Key points:
- `max-w-2xl` (672px) centers the form for comfortable reading width
- `space-y-8` between sections, `space-y-4` within sections
- Sticky footer stays visible during scrolling
- Footer uses same `max-w-2xl` to align with form content
- Cancel button always available (error prevention)
- Button labels follow [Verb] [object] pattern
