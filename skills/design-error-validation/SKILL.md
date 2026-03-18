---
name: design-error-validation
description: Implements form validation, loading states, error handling, and confirmation dialogs with Aura components and tokens. Use when building or modifying forms, handling API responses, or implementing async operations.
allowed-tools: Read, Glob, Grep, Edit, Write
---

## Role
You are implementing how the application responds to user 
actions. These patterns determine whether users trust the 
application. Follow these for every form, API call, 
and user action.

All UI elements use Aura components and tokens. Error states 
use the destructive token family, warnings use warning tokens, 
success uses success tokens.

For all Storybook URLs, see `./storybook-links.md`.


<when-to-reference>
Consult this skill whenever you are:
- Building or modifying a form
- Handling API responses
- Implementing loading or progress indicators
- Adding confirmation dialogs
- Handling any async operation
</when-to-reference>

<patterns>

<pattern name="form-validation" priority="critical">
<instruction>
- Validate on blur, not on every keystroke
- Show errors inline, adjacent to the field
- Preserve user input on failure (never clear the form)
- Move focus to first error field on submission failure
- Announce errors to screen readers via aria-live
</instruction>

<status-token-mapping>
| State | Background | Text | Border |
|-------|-----------|------|--------|
| Error | bg-destructive | text-destructive-foreground | border-destructive |
| Warning | bg-warning | text-warning-foreground | — |
| Success | bg-success | text-success-foreground | — |
| Disabled | bg-disabled | text-disabled-foreground | — |
</status-token-mapping>

<message-patterns>
| Field type | Correct message | Incorrect message |
|-----------|----------------|-------------------|
| Required | "Report name is required." | "Required" |
| Email | "Email must include an @ symbol." | "Invalid" |
| Password | "At least 8 characters." | "Too short" |
| Number | "Value must be between 1 and 100." | "Invalid" |
| Date | "End date must be after start date." | "Invalid date" |

See **design-content-guidelines** skill for full message patterns.
</message-patterns>

<field-validation-states>
Every form field must support the validation states applicable 
to its type. Use this table to determine which states to implement:

| Field type | required | format | length | range | uniqueness |
|-----------|----------|--------|--------|-------|------------|
| Text Input | yes | — | optional | — | optional |
| Email Input | yes | yes | — | — | optional |
| Password | yes | yes | yes | — | — |
| Number Input | yes | — | — | yes | — |
| Date Picker | yes | — | — | yes | — |
| Textarea | yes | — | yes | — | — |
| Select | yes | — | — | — | — |
| Combobox | yes | — | — | — | — |
| Checkbox | — | — | — | — | — |
| File Upload | yes | yes | — | yes (size) | — |

"yes" = must implement. "optional" = implement if relevant.
</field-validation-states>

<complete-field-example>
A complete form field with all states (default, focused, 
error, success, disabled):

import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { HelperText } from '@/components/ui/helper-text';

{/* Default / Focused state */}
<div className="space-y-2">
  <Label htmlFor="report-name">
    Report name <span className="text-destructive">*</span>
  </Label>
  <Input
    id="report-name"
    value={name}
    onChange={(e) => setName(e.target.value)}
    onBlur={validateName}
    aria-describedby="report-name-helper"
    aria-invalid={!!nameError}
    disabled={isSubmitting}
  />
  {nameError ? (
    <HelperText id="report-name-helper" variant="error">
      {nameError}
    </HelperText>
  ) : (
    <HelperText id="report-name-helper">
      A descriptive name for your report.
    </HelperText>
  )}
</div>

Key implementation details:
- Label with required indicator (asterisk in text-destructive)
- Input with aria-describedby linking to HelperText
- Input with aria-invalid reflecting error state
- Validation on blur via onBlur handler
- HelperText swaps between hint (default) and error message
- Disabled state during form submission
</complete-field-example>
</pattern>

<pattern name="loading-states" priority="critical">
<instruction>
Any action taking more than 300ms must show a loading 
indicator using Aura components.
</instruction>

<variants>
| Context | Pattern | Aura component |
|---------|---------|---------------|
| Page load | Skeleton screen | Skeleton |
| Button action | Button disabled + spinner | Button loading state |
| Data refresh | Overlay spinner | Spinner on existing content |
| Long operation | Progress bar + message | Progress |
</variants>

<example type="correct">
{/* Button loading during async action */}
<Button 
  variant="default" 
  onClick={handleSave}
  disabled={isSaving}
>
  {isSaving ? (
    <>
      <Loader2 className="mr-2 h-4 w-4 animate-spin" />
      Saving...
    </>
  ) : (
    'Save changes'
  )}
</Button>

{/* Skeleton while page loads */}
{isLoading ? (
  <div className="space-y-4">
    <Skeleton className="h-8 w-48" />
    <Skeleton className="h-4 w-full" />
    <Skeleton className="h-4 w-3/4" />
  </div>
) : (
  <DataTable data={reports} columns={columns} />
)}
</example>

<example type="incorrect">
{/* No loading state */}
<Button onClick={handleSave}>Save changes</Button>

{/* Blank screen while loading */}
{isLoading ? null : <DataTable data={reports} />}
</example>
</pattern>

<pattern name="error-states" priority="critical">
<instruction>
Every API failure must show a user-facing message using 
Aura Alert component. Never fail silently.
See **design-content-guidelines** skill for message wording.
</instruction>

<full-example>
import { Alert, AlertDescription } from '@/components/ui/alert';
import { AlertCircle } from 'lucide-react';

{error && (
  <Alert variant="destructive">
    <AlertCircle className="h-4 w-4" />
    <AlertDescription>
      {error}
    </AlertDescription>
    <Button 
      variant="secondary" 
      size="sm" 
      onClick={retry}
      className="ml-auto"
    >
      Try again
    </Button>
  </Alert>
)}
</full-example>
</pattern>

<pattern name="success-feedback" priority="high">
<instruction>
Use Sonner toast for brief confirmations.
</instruction>

<example type="correct">
import { toast } from 'sonner';

// After save
toast.success('Report saved successfully.');

// After delete
toast.success('Report deleted.');

// After bulk action
toast.success(`${count} items archived.`);
</example>

<example type="incorrect">
// No feedback
await saveReport(data);
navigate('/reports');

// Vague
toast.success('Done!');
</example>
</pattern>

<pattern name="confirmation-dialogs" priority="critical">
<instruction>
Destructive actions must show Dialog with specific 
action verb. See **design-content-guidelines** skill for copy.
</instruction>

<full-example>
import {
  Dialog, DialogContent, DialogDescription,
  DialogFooter, DialogHeader, DialogTitle,
} from '@/components/ui/dialog';

<Dialog open={showDelete} onOpenChange={setShowDelete}>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Delete this report?</DialogTitle>
      <DialogDescription>
        This will permanently remove "{report.name}" and 
        all associated data. This cannot be undone.
      </DialogDescription>
    </DialogHeader>
    <DialogFooter>
      <Button 
        variant="secondary" 
        onClick={() => setShowDelete(false)}
      >
        Cancel
      </Button>
      <Button 
        variant="destructive" 
        onClick={handleDelete}
      >
        Delete report
      </Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
</full-example>

<example type="incorrect">
{/* Yes/No, no description, wrong variant */}
<Dialog open={show}>
  <DialogContent>
    <DialogTitle>Confirm</DialogTitle>
    <DialogDescription>Are you sure?</DialogDescription>
    <DialogFooter>
      <Button variant="secondary">No</Button>
      <Button variant="default">Yes</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
</example>
</pattern>

</patterns>

<edge-cases>
1. Destructive action with undo? — Still confirm. Mention 
   undo in body: "You can undo within 30 seconds."
2. Bulk delete? — One confirmation: "Delete 12 reports?"
3. Auto-save? — Subtle "Saved" indicator, not toast each time.
4. Error in multi-step flow? — Don't lose progress. Show 
   error on current step. Let user retry.
</edge-cases>

<visual-checks>
<instruction>
For error handling and validation verification, test these user flows visually:

**Form validation flow testing:**
1. **Field-level validation**: Trigger inline errors (empty required fields, invalid email format)
2. **Form submission errors**: Submit invalid form, verify error summary and field highlighting
3. **Error message clarity**: Check error text is specific and actionable (not generic "Invalid input")
4. **Error recovery**: Fix errors and verify they clear properly, success states appear

**Loading and async state verification:**
- **Button loading states**: Submit forms, verify loading spinner/text appears on submit button
- **Page-level loading**: Navigate between sections, verify loading indicators during data fetch
- **Progressive loading**: Check skeleton screens, partial content loading gracefully
- **Timeout handling**: Verify timeout errors show appropriate retry options

**Success feedback verification:**
- **Toast notifications**: Verify success messages appear and auto-dismiss appropriately
- **Inline success states**: Form fields show check marks or green states after validation
- **Page transitions**: Success actions redirect or update UI as expected
- **Confirmation dialogs**: Test destructive action confirmations work correctly

**Error state accessibility:**
- **Screen reader announcements**: Errors announced when they appear
- **Focus management**: Focus moves to first error field after form submission
- **Color independence**: Errors identifiable without relying on color alone
- **Error summary**: List of all errors accessible at top of form

**Edge case handling:**
- **Network errors**: Offline/network failure states display helpful messaging
- **Validation edge cases**: Test boundary values, special characters, long inputs
- **Multi-step flows**: Error in one step doesn't break entire workflow
- **Concurrent actions**: Multiple simultaneous actions don't create conflicting states
</instruction>

<focus-areas>
Priority visual checks for error handling:
1. **Form error flow** — Complete form validation cycle from error to success
2. **Loading state coverage** — All async actions show appropriate loading indicators
3. **Success feedback** — Users receive clear confirmation of successful actions
4. **Error accessibility** — Error states are perceivable by all users
</focus-areas>
</visual-checks>
