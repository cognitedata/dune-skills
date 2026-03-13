# Wizard / Multi-Step Layout Template

```tsx
import { Button } from '@/components/ui/button';

interface WizardStep {
  label: string;
  content: React.ReactNode;
}

export function WizardLayout({ steps }: { steps: WizardStep[] }) {
  const [currentStep, setCurrentStep] = useState(0);

  if (steps.length === 0) return null;

  const isFirst = currentStep === 0;
  const isLast = currentStep === steps.length - 1;

  return (
    <div className="flex min-h-screen flex-col">
      {/* Step indicator */}
      <div className="border-b border-border bg-background px-6 py-4">
        {/* Desktop: full step indicator */}
        <nav className="mx-auto hidden max-w-2xl sm:block" 
             aria-label="Progress">
          <ol className="flex items-center justify-between">
            {steps.map((step, index) => (
              <li key={index} className="flex items-center">
                <span
                  className={`flex h-8 w-8 items-center justify-center 
                    rounded-full text-sm font-medium
                    ${index < currentStep
                      ? 'bg-primary text-primary-foreground'
                      : index === currentStep
                      ? 'border-2 border-primary text-primary'
                      : 'border border-border text-muted-foreground'
                    }`}
                >
                  {index + 1}
                </span>
                <span className={`ml-2 text-sm
                  ${index <= currentStep 
                    ? 'text-foreground font-medium' 
                    : 'text-muted-foreground'
                  }`}>
                  {step.label}
                </span>
                {index < steps.length - 1 && (
                  <div className={`mx-4 h-px w-12
                    ${index < currentStep 
                      ? 'bg-primary' 
                      : 'bg-border'
                    }`} 
                  />
                )}
              </li>
            ))}
          </ol>
        </nav>

        {/* Mobile: compact indicator */}
        <p className="text-sm text-muted-foreground sm:hidden">
          Step {currentStep + 1} of {steps.length}: {steps[currentStep].label}
        </p>
      </div>

      {/* Step content — centered, constrained */}
      <main className="flex-1 px-6 py-8">
        <div className="mx-auto max-w-2xl space-y-6">
          {steps[currentStep].content}
        </div>
      </main>

      {/* Navigation footer */}
      <div className="sticky bottom-0 border-t border-border 
                      bg-background px-6 py-4">
        <div className="mx-auto flex max-w-2xl justify-between">
          <Button
            variant="secondary"
            onClick={() => setCurrentStep((s) => s - 1)}
            disabled={isFirst}
          >
            Back
          </Button>
          <Button
            onClick={() => {
              if (isLast) handleSubmit();
              else setCurrentStep((s) => s + 1);
            }}
          >
            {isLast ? 'Submit' : 'Continue'}
          </Button>
        </div>
      </div>
    </div>
  );
}
```

Key points:
- Step indicator uses semantic tokens (bg-primary, text-primary-foreground)
- Desktop: horizontal numbered steps with connecting lines
- Mobile: compact "Step X of Y" text
- Content centered at `max-w-2xl` (same as form-page)
- Sticky footer with Back/Continue navigation
- Back button disabled on first step
- Last step shows "Submit" instead of "Continue"
- `aria-label="Progress"` on the step nav for accessibility
