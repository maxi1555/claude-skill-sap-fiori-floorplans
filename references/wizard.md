# Wizard

The Fiori floorplan for **guided multi-step creation or editing** of a single object. Use when the task is long, unfamiliar, or branching, and the user benefits from being walked through it step by step.

> **Also consult** `references/global-patterns.md` (object handling, messaging, accessibility) and `references/action-placement.md` (button styling for Next/Review/Create/Cancel). The Wizard has stricter action rules than other floorplans because action styling drives comprehension of "where am I in the flow".

## When to use

- Task is **long** (multi-part form that would be overwhelming as one flat page).
- Task is **unfamiliar** to the user (onboarding, first-time purchase configuration, initial setup).
- Task is **non-linear** — later steps depend on earlier choices (branching).
- User benefits from a **review/summary** before commit.

Fiori is explicit on the size:
- **Minimum 3 steps, maximum 8.** A 2-step wizard is just a dialog with "Next". 9+ steps is a signal you're modelling the wrong thing; group or rethink.

## When NOT to use

- Routine daily task for an expert user → flat form on an Object Page.
- Simple 2-step flow → a single page with two sections, not a Wizard.
- Create-in-place (small record added to a list) → inline row in a table or a creation dialog.
- Anything read-only → Wizard is only for create/edit.
- Anything that doesn't need a pre-commit review step → plain form.

## Two Wizard types (Fiori)

- **Standard wizard** — total number of steps is known up front, progression is linear. Use when the flow is fixed.
- **Branching wizard** — total number is not known; user's choices in one step determine which steps come next. A dotted line on the progress bar indicates "more steps will follow". Use when the form has conditional paths.

## Visual styles

- **Numbered circles** (default): each step displays its ordinal number.
- **Icon circles**: if your steps have strong symbolic identities, icons can be clearer than numbers. If you use icons, assign one to **every** step — not some-icon-some-number.

## What you get for free

- **Progress bar** at the top showing completed/current/upcoming steps.
- **Step navigation** by clicking completed/active steps in the progress bar.
- **Automatic scrolling** to the content of the newly selected step.
- **Responsive grouping** on narrow screens: steps group together and overlap; tapping opens a popover or dialog to select the step to navigate to.

## Critical: Wizard does NOT provide built-in Next/Previous buttons

This is the single thing developers get wrong most often. The `ui5-wizard` component deliberately does not provide a "move to next step" button. The app is expected to place its own button (or anchor, or whatever) **inside the step content** that, when activated, sets `selected={true}` on the next step and `disabled={false}`. That's how the wizard advances.

So your step content is roughly:

```tsx
<WizardStep titleText="Step 1" selected={step === 0} disabled={step !== 0}>
  {/* step 1 form */}
  <Button
    design="Emphasized"
    disabled={!canProceedStep1}
    onClick={() => setStep(1)}
  >
    Step 2
  </Button>
</WizardStep>
```

The button label should be `Step N` where N is the next step number, except on the last step before Review, where it's `Review`. On the Review (summary) step, the button is the finalizing action (`Create`, `Save`, etc.). The footer holds `Cancel`.

## Fiori design rules

### Step labels

- Every step has a **`titleText`** (short verb+noun: "Enter customer details", "Select products", "Confirm").
- Optional **`subtitleText`** (one-line clarifying sentence).
- Labels should be scannable from the progress bar.

### Per-step validation

- Validate fields when the user tries to advance. If invalid, keep them on the current step.
- Use inline field validation (red border + message) and a message popover summary.
- The advance button is `disabled` until mandatory fields in the current step are valid.

### Summary/Review step

- **Mandatory.** Always include a final review step before commit.
- Shows all entered data read-only.
- Main action (`Create`, `Save`, `Submit`) in the footer.
- "Edit" button or back-arrow returns to the wizard to adjust values. This does **not** reset the wizard.

### Cancel behavior

- Cancel is in the footer.
- Because the wizard is a lightweight create/edit, use a concise confirmation: "Discard this <object>?" with a Discard (emphasized) / Keep Editing option — **not** the heavy data-loss dialog used elsewhere.

### Save as Draft

- If the form is long (several steps with lots of fields), offer **Save as Draft** in the footer alongside Cancel.
- Draft handling uses the object's draft mechanism; when re-entering the wizard, show a dialog informing the user a draft exists.

### Header behavior

- The wizard floorplan uses the dynamic page, but the **wizard's header does not snap on scroll**. Don't expect snapping behavior here.

### FCL compatibility

- Wizard works in full-screen and inside a Flexible Column Layout.
- Inside FCL, the wizard always occupies the **rightmost column**; there's no forward navigation from the wizard (it's a leaf).

## v2 component API cheat sheet

**`Wizard`:**
- `onStepChange` — event when the selected step changes.
- `contentLayout` — `"MultipleSteps"` (default) or `"SingleStep"` (only the active step is visible).

**`WizardStep`:**
- `titleText` — step label in progress bar.
- `subtitleText` — optional helper text under title.
- `icon` — icon name (imported) instead of a number.
- `selected` — boolean, **controlled**: which step is active.
- `disabled` — boolean, **controlled**: whether the step can be navigated to.
- `branching` — boolean, for branching wizards; appends a dotted indicator after this step.

To advance, **you** set `selected={true}` on the target step and `disabled={false}` on it (and unselect the current). Do this from an `onClick` on a button inside the current step.

## Production scaffold (branching, with validation, draft, and review)

```tsx
'use client';

import {
  DynamicPage,
  DynamicPageTitle,
  Wizard,
  WizardStep,
  Title,
  Label,
  Text,
  Form,
  FormItem,
  FormGroup,
  Input,
  Select,
  Option,
  CheckBox,
  Bar,
  Button,
  ButtonDesign,
  MessageStrip,
  Dialog,
  MessageBox,
  MessageBoxTypes,
  MessageBoxActions,
} from '@ui5/webcomponents-react';

import { useState } from 'react';

type Plan = 'Basic' | 'Pro' | 'Enterprise';

type FormState = {
  customerName: string;
  email: string;
  plan: Plan | '';
  needsSSO: boolean;
  ssoProvider: string;
  billingAddress: string;
  billingEmail: string;
};

type Props = {
  onCreate: (state: FormState) => Promise<void>;
  onSaveDraft: (state: Partial<FormState>) => Promise<void>;
  onCancel: () => void;
  hasExistingDraft?: boolean;
};

export default function OnboardCustomerWizard({ onCreate, onSaveDraft, onCancel, hasExistingDraft }: Props) {
  // Dynamic step list because of branching: Enterprise + SSO adds an SSO step.
  const [step, setStep] = useState(0);
  const [form, setForm] = useState<FormState>({
    customerName: '',
    email: '',
    plan: '',
    needsSSO: false,
    ssoProvider: '',
    billingAddress: '',
    billingEmail: '',
  });
  const [submitting, setSubmitting] = useState(false);
  const [confirmCancel, setConfirmCancel] = useState(false);

  const update = (patch: Partial<FormState>) => setForm((s) => ({ ...s, ...patch }));

  const ssoStepNeeded = form.plan === 'Enterprise' && form.needsSSO;

  // Build dynamic step list based on branching answers
  const steps = [
    'details',
    'plan',
    ...(ssoStepNeeded ? ['sso'] : []),
    'billing',
    'review',
  ] as const;
  const totalSteps = steps.length;

  const validators: Record<(typeof steps)[number], () => boolean> = {
    details: () => !!form.customerName.trim() && /\S+@\S+\.\S+/.test(form.email),
    plan: () => form.plan !== '',
    sso: () => !ssoStepNeeded || !!form.ssoProvider.trim(),
    billing: () => !!form.billingAddress.trim() && /\S+@\S+\.\S+/.test(form.billingEmail),
    review: () => true,
  };

  const currentKey = steps[step];
  const canProceed = validators[currentKey]();

  const next = () => setStep((s) => Math.min(s + 1, totalSteps - 1));
  const prev = () => setStep((s) => Math.max(s - 1, 0));

  const handleCreate = async () => {
    setSubmitting(true);
    try {
      await onCreate(form);
    } finally {
      setSubmitting(false);
    }
  };

  const handleCancel = () => {
    // Fiori guidance: for wizards, use a concise "Discard this customer?" dialog, not the full data-loss dialog
    setConfirmCancel(true);
  };

  return (
    <DynamicPage
      titleArea={<DynamicPageTitle header={<Title>Onboard New Customer</Title>} />}
      footerArea={
        <Bar
          design="FloatingFooter"
          endContent={
            <>
              {currentKey !== 'review' && step > 0 && (
                <Button design="Transparent" onClick={prev}>
                  Previous
                </Button>
              )}
              {currentKey === 'review' ? (
                <Button design="Emphasized" disabled={submitting} onClick={handleCreate}>
                  {submitting ? 'Creating…' : 'Create'}
                </Button>
              ) : null}
              {/* Save as Draft — only show while editing, not on review */}
              {currentKey !== 'review' && (
                <Button design="Transparent" onClick={() => onSaveDraft(form)}>
                  Save as Draft
                </Button>
              )}
              <Button design="Transparent" onClick={handleCancel}>
                Cancel
              </Button>
            </>
          }
        />
      }
    >
      {hasExistingDraft && (
        <MessageStrip design="Information" hideCloseButton style={{ margin: 'var(--sapContent_Space_S)' }}>
          A draft for this customer already exists. Your changes will update the draft.
        </MessageStrip>
      )}

      <Wizard
        onStepChange={(e) => {
          // Allow clicking a previously-completed step in the progress bar
          const idx = steps.findIndex((k) => (e.detail.step as HTMLElement).dataset.key === k);
          if (idx !== -1 && idx < step) setStep(idx);
        }}
      >
        <WizardStep
          data-key="details"
          titleText="Customer Details"
          subtitleText="Contact person and name"
          selected={currentKey === 'details'}
          disabled={currentKey !== 'details'}
        >
          <Form labelSpan="S12 M4 L4 XL4" layout="S1 M2 L2 XL2">
            <FormGroup headerText="Contact">
              <FormItem labelContent={<Label required>Customer name</Label>}>
                <Input
                  value={form.customerName}
                  onInput={(e) => update({ customerName: (e.target as HTMLInputElement).value })}
                />
              </FormItem>
              <FormItem labelContent={<Label required>Email</Label>}>
                <Input
                  type="Email"
                  value={form.email}
                  onInput={(e) => update({ email: (e.target as HTMLInputElement).value })}
                />
              </FormItem>
            </FormGroup>
          </Form>

          {/* Wizard doesn't provide a "next" button — we do, inside the step. */}
          <div style={{ marginTop: 'var(--sapContent_Space_M)' }}>
            <Button design={ButtonDesign.Emphasized} disabled={!canProceed} onClick={next}>
              {steps[1] === 'review' ? 'Review' : `Step 2`}
            </Button>
          </div>
        </WizardStep>

        <WizardStep
          data-key="plan"
          titleText="Choose Plan"
          subtitleText="Tier and SSO"
          selected={currentKey === 'plan'}
          disabled={currentKey !== 'plan'}
          branching={form.plan === 'Enterprise'}
        >
          <Form labelSpan="S12 M4 L4 XL4" layout="S1 M2 L2 XL2">
            <FormGroup headerText="Plan">
              <FormItem labelContent={<Label required>Plan</Label>}>
                <Select
                  onChange={(e) =>
                    update({ plan: (e.detail.selectedOption as HTMLElement).dataset.value as Plan })
                  }
                >
                  <Option data-value="" selected={form.plan === ''}>Select…</Option>
                  <Option data-value="Basic" selected={form.plan === 'Basic'}>Basic</Option>
                  <Option data-value="Pro" selected={form.plan === 'Pro'}>Pro</Option>
                  <Option data-value="Enterprise" selected={form.plan === 'Enterprise'}>Enterprise</Option>
                </Select>
              </FormItem>
              {form.plan === 'Enterprise' && (
                <FormItem labelContent={<Label>Needs Single Sign-On?</Label>}>
                  <CheckBox
                    checked={form.needsSSO}
                    onChange={(e) => update({ needsSSO: (e.target as HTMLInputElement).checked })}
                  />
                </FormItem>
              )}
            </FormGroup>
          </Form>

          <div style={{ marginTop: 'var(--sapContent_Space_M)' }}>
            <Button design={ButtonDesign.Emphasized} disabled={!canProceed} onClick={next}>
              {steps[step + 1] === 'review' ? 'Review' : `Step ${step + 2}`}
            </Button>
          </div>
        </WizardStep>

        {ssoStepNeeded && (
          <WizardStep
            data-key="sso"
            titleText="SSO Setup"
            subtitleText="Identity provider configuration"
            selected={currentKey === 'sso'}
            disabled={currentKey !== 'sso'}
          >
            <Form labelSpan="S12 M4 L4 XL4" layout="S1 M2 L2 XL2">
              <FormGroup headerText="Identity provider">
                <FormItem labelContent={<Label required>Provider</Label>}>
                  <Input
                    value={form.ssoProvider}
                    placeholder="e.g. Okta, Azure AD, Auth0"
                    onInput={(e) => update({ ssoProvider: (e.target as HTMLInputElement).value })}
                  />
                </FormItem>
              </FormGroup>
            </Form>

            <div style={{ marginTop: 'var(--sapContent_Space_M)' }}>
              <Button design={ButtonDesign.Emphasized} disabled={!canProceed} onClick={next}>
                {steps[step + 1] === 'review' ? 'Review' : `Step ${step + 2}`}
              </Button>
            </div>
          </WizardStep>
        )}

        <WizardStep
          data-key="billing"
          titleText="Billing"
          subtitleText="Invoicing details"
          selected={currentKey === 'billing'}
          disabled={currentKey !== 'billing'}
        >
          <Form labelSpan="S12 M4 L4 XL4" layout="S1 M2 L2 XL2">
            <FormGroup headerText="Billing address">
              <FormItem labelContent={<Label required>Address</Label>}>
                <Input
                  value={form.billingAddress}
                  onInput={(e) => update({ billingAddress: (e.target as HTMLInputElement).value })}
                />
              </FormItem>
              <FormItem labelContent={<Label required>Billing email</Label>}>
                <Input
                  type="Email"
                  value={form.billingEmail}
                  onInput={(e) => update({ billingEmail: (e.target as HTMLInputElement).value })}
                />
              </FormItem>
            </FormGroup>
          </Form>

          <div style={{ marginTop: 'var(--sapContent_Space_M)' }}>
            <Button design={ButtonDesign.Emphasized} disabled={!canProceed} onClick={next}>
              Review
            </Button>
          </div>
        </WizardStep>

        <WizardStep
          data-key="review"
          titleText="Review"
          subtitleText="Confirm and create"
          selected={currentKey === 'review'}
          disabled={currentKey !== 'review'}
        >
          <MessageStrip design="Information" hideCloseButton>
            Review the details below. Use "Previous" or click any step in the progress bar to change values.
          </MessageStrip>

          <div style={{ padding: 'var(--sapContent_Space_M)', display: 'grid', gap: '0.5rem' }}>
            <Text><strong>Name:</strong> {form.customerName}</Text>
            <Text><strong>Email:</strong> {form.email}</Text>
            <Text><strong>Plan:</strong> {form.plan}</Text>
            {ssoStepNeeded && <Text><strong>SSO provider:</strong> {form.ssoProvider}</Text>}
            <Text><strong>Billing address:</strong> {form.billingAddress}</Text>
            <Text><strong>Billing email:</strong> {form.billingEmail}</Text>
          </div>

          {/* Note: the finalizing "Create" button lives in the footerArea,
              not inside the step content. This is the one step where footer holds the primary action. */}
        </WizardStep>
      </Wizard>

      {confirmCancel && (
        <MessageBox
          open
          type={MessageBoxTypes.Warning}
          actions={[MessageBoxActions.Delete, MessageBoxActions.Cancel]}
          titleText="Discard this customer?"
          onClose={(e) => {
            if (e.detail.action === MessageBoxActions.Delete) {
              setConfirmCancel(false);
              onCancel();
            } else {
              setConfirmCancel(false);
            }
          }}
        >
          Any entered information will be discarded.
        </MessageBox>
      )}
    </DynamicPage>
  );
}
```

### Notes on the scaffold

- **Branching**: when the user picks the Enterprise plan and checks SSO, a new "SSO Setup" step is dynamically inserted. `branching` on the Plan step shows the dotted indicator per Fiori guidance.
- **Next buttons live inside each step** — this is the Wizard's expected pattern. The label is `Step N` normally, `Review` on the step before Review.
- **Create button lives in `footerArea`** — it's the finalizing action, and only the Review step's primary action. Alongside it, Save as Draft and Cancel.
- **Data-loss dialog**: a `MessageBox` with Delete/Cancel actions and the "Discard this customer?" wording, per Fiori's light-weight wizard cancel pattern.
- **Existing draft notice**: `MessageStrip` at the top of the page.

## Fiori text templates for Wizard

Use these exact patterns. They match Fiori elements conventions.

**Cancel confirmation (quick-confirmation popover):**
- During Create wizard: `Discard this [customer / sales order / contract]?` — button `Discard`.
- During Edit wizard: `Discard all changes?` — button `Discard`.

**Finalize button labels (on the Review step's footer):**
- Create wizard: `Create` (emphasized) — the primary.
- Edit wizard: `Save` (emphasized) — the primary.
- Alternative path: `Save as Draft` (transparent) — secondary.
- Negative path: `Cancel` (transparent) — triggers the discard popover above.

**Step navigation labels (inside step content):**
- Normal step: `Step 2 of 5` or just `Next`. Using a step number is clearer.
- Last substantive step (right before Review): `Review`.
- Review step has no Next — just the footer's Create/Save.

**Success MessageToast after finalize:**
- Create: `[Customer name] created.`
- Save: `[Customer name] saved.`
- Draft save: `Draft saved.`
- Discard: `Changes discarded.`

**Existing-draft strip at top of wizard (when resuming):**
- `You have an unsaved draft of this [customer] from [date, time].` — with a `Discard draft` link on the right.

**Validation errors inside a step:**
- Per-field: use `valueState="Error"` with a concrete message (`Format must be YYYY-MM-DD`).
- At Next click, if any field is invalid, block navigation and focus the first invalid field. Don't silently allow skipping.

## Anti-patterns

- Using a Wizard for a 2-step "click, confirm" flow. → Use a dialog or a plain form.
- Using a Wizard for a routine daily task (PO creation for a buyer who does 50 a day). → Use a flat form on an Object Page.
- Omitting the Review step. → Mandatory.
- Using the heavy "Data Loss" dialog on Cancel. → Use the concise "Discard this <object>?" pattern.
- Adding Next/Previous buttons *outside* the wizard. → Put them inside the step content (Next) and the footer (Previous is optional; the progress bar handles most navigation).
- Using icons on only some steps. → All or none.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents`
- `@ui5/webcomponents-fiori` (Wizard is in the Fiori package)
- `@ui5/webcomponents-icons`
