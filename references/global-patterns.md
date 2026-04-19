# Global Patterns — Fiori's cross-cutting best practices

This file captures SAP's official Fiori-for-Web **global patterns** — the cross-cutting UX rules that apply across every floorplan: navigation, action placement, object handling (create/edit/delete/draft), messaging, empty & loading states, keyboard & accessibility. When in doubt on any floorplan, these patterns take precedence over individual control choices.

Source: `sap.com/design-system/fiori-design-web/.../foundations/best-practices/global-patterns`.

## Table of contents

1. Navigation
2. Action placement — the Fiori action-type taxonomy
3. Object handling (create, edit, delete, draft)
4. Message handling
5. Empty states (IllustratedMessage)
6. Loading / busy states
7. Keyboard & accessibility

---

## 1. Navigation

### The mental model

SAP Fiori uses a **hub-and-spoke + application-network** model. The hub is the SAP Fiori launchpad (the home page that hosts all tile apps). Every app is a spoke. Apps can link to each other, forming an application network within the hub.

For a standalone Next.js app outside an SAP Fiori launchpad, your ShellBar is the closest equivalent to the launchpad shell: it's always at the top, its logo navigates home, and its back button goes to the previous page.

### Rules

- **Shell bar is always at the top of each app.** Never nest it, never hide it during normal flows.
- **Logo click → navigate to the launchpad home page** (or, in a standalone app, to the app's landing page).
- **Back button → previous page.** Exception: if the app was opened via a **deep link and there are no prior entries in history**, show no back button.
- **Forward navigation** happens via links, line items, buttons, or other UI elements. **No visual distinction** between in-app navigation and cross-app navigation — users simply follow a link. Consistency within an app matters: if links in a table open a quick view on one row, they should open a quick view on every row.
- **Switching display ↔ edit mode is NOT a navigation step.** No new browser-history entry; no back button added; a shared URL always resolves to display mode.
- **In split-screen / master-detail layouts**, the back button above the master list returns to whatever launched the app (launchpad or originating app), not to a within-app previous state.
- **Mobile behavior** for split screens: either master or detail shows full-screen. Selecting a master item creates a real history entry, and back returns to the master list.

### Breadcrumbs

- Use breadcrumbs inside the title of a page to show the app-internal hierarchy. Don't duplicate items that already exist in the ShellBar's menu.
- Breadcrumb items are clickable to navigate up the hierarchy.

### Typical implementation in Next.js + ui5-webcomponents-react

```tsx
// shell.tsx — one ShellBar across the whole app, router-driven navigation
<ShellBar
  primaryTitle="Customer Central"
  logo={<img src="/logo.svg" alt="Company logo" />}
  profile={<Avatar><img src="/avatar.png" alt="User avatar" /></Avatar>}
  onLogoClick={() => router.push('/')}
>
  <ShellBarItem icon={settingsIcon} text="Settings" onClick={() => router.push('/settings')} />
</ShellBar>
```

```tsx
// object-page.tsx — breadcrumbs inside the title, not in the shell
<ObjectPageTitle
  breadcrumbs={
    <Breadcrumbs>
      <BreadcrumbsItem onClick={() => router.push('/customers')}>Customers</BreadcrumbsItem>
      <BreadcrumbsItem>{customer.name}</BreadcrumbsItem>
    </Breadcrumbs>
  }
  header={<Title>{customer.name}</Title>}
/>
```

---

## 2. Action placement — the Fiori action-type taxonomy

This is the most important global pattern for a Fiori developer to get right, and also the one most often botched. A page that misplaces its actions just doesn't feel like Fiori.

### The four action types

Every action on a Fiori page belongs to one of these types:

| Type | Purpose | Button style (header/footer) | Button style (content toolbar) |
|---|---|---|---|
| **Primary** | The most important action on the page | `Emphasized` | `Emphasized` (only if no page-level primary exists), else `Default` (ghost) |
| **Secondary** | Supporting, non-destructive actions | `Default` | `Transparent` |
| **Semantic** | Positive / negative workflow actions (Accept, Reject) | `Positive` / `Negative` | `Positive` / `Negative` |
| **Negative path** | Leave without changes (Cancel, Close) | `Transparent` | `Transparent` |

### The rules (official Fiori guidelines)

1. **Only one primary action per page** (across header and footer toolbars combined).
2. **Never mix Emphasized and Semantic buttons** in the same toolbar. Pick one visual system per toolbar.
3. **If a page-level primary exists**, content toolbars must use `Default` (ghost) for their own most-important button — never a second Emphasized.
4. **Workflow / finalizing actions (Save, Post, Submit, Accept, Reject) always go in the footer toolbar.** Never in the header, with very few exceptions (apps with only 1–2 actions and no other header actions).
5. **Icon OR text, never both** — with one exception: inline row actions in tables.
6. **Always use text** for primary, secondary, semantic, and negative-path actions. Use icon-only only when the icon's meaning is universally understood (add, delete, share).
7. **Header toolbar actions are right-aligned, grouped left→right**:
   1. Business actions (Edit, Delete)
   2. Manage content (Filter)
   3. Manage layout (Full screen)
   4. Generic (Share)
8. **Footer toolbar actions are right-aligned, grouped left→right**:
   1. Forward path (Post, Accept)
   2. Alternative path (Reject)
   3. Negative path (Cancel, Close)
9. **Primary goes leftmost** (within its toolbar) **even if it logically belongs to a group further right**.
10. **If both header and footer toolbars are present**, the footer is more in focus and usually hosts the primary action.

### Action priority (overflow behavior)

Fiori has a priority system for overflow:

- **Low** — first to overflow when space runs out.
- **High** (default) — overflow only after all Low-priority actions have overflowed.
- **AlwaysOverflow** — always in the overflow menu.
- **NeverOverflow** — never moved to overflow; labels truncate instead.

**Emphasized and primary-action buttons must not be moved to overflow.** Mark them `NeverOverflow` if you build your own overflow handling.

### What belongs where — the quick lookup

| Action | Toolbar |
|---|---|
| Save / Submit / Post / Approve / Reject | Footer |
| Cancel (the workflow) | Footer (with quick-confirmation popover) |
| Delete (when only one Delete applies) | Footer |
| Edit (toggle page to edit mode) | Title `actionsBar` *or* Footer |
| Share / Export / Print / Copy Link | Title `actionsBar` |
| Create (in a list context) | Table toolbar |
| Delete Selected / Approve Selected | Table toolbar (disabled when selection is empty) |
| Edit this row / Approve this row | Row inline action (terminal Cell) |
| Sort / Filter / Group by column | Column header popover (NOT the table toolbar) |
| Help / Settings / Sign Out | ShellBar |

### Style recipes (verified v2)

```tsx
// Footer toolbar with one primary + secondary + negative
<Bar
  design="FloatingFooter"
  endContent={
    <>
      <Button design="Emphasized">Save</Button>       {/* primary */}
      <Button design="Transparent">Cancel</Button>   {/* negative path */}
    </>
  }
/>

// Semantic footer (approval workflow) — never Emphasized + Positive together
<Bar
  design="FloatingFooter"
  endContent={
    <>
      <Button design="Positive">Accept</Button>       {/* semantic */}
      <Button design="Negative">Reject</Button>       {/* semantic */}
      <Button design="Transparent">Cancel</Button>    {/* negative path */}
    </>
  }
/>

// Title actionsBar — non-finalizing global actions
<Toolbar design="Transparent">
  <ToolbarButton design="Transparent" icon={shareIcon} text="Share" />
  <ToolbarButton design="Transparent" icon={pdfIcon} text="Export PDF" />
</Toolbar>

// Table toolbar — Create is primary-for-the-table (not page); use Default if page has its own primary
<Toolbar>
  <Title level="H4">Orders ({rows.length})</Title>
  <ToolbarSpacer />
  <ToolbarButton design="Default" icon={addIcon} text="Create" />
  <ToolbarButton design="Transparent" icon={deleteIcon} text="Delete" disabled={!hasSelection} />
</Toolbar>
```

---

## 3. Object handling — create, edit, delete, draft

### Floorplan choice per object complexity

| Object shape | Use |
|---|---|
| Simple object, up to 8 fields | Dialog (partial edit) |
| Simple object, more fields | Object Page |
| Complex object with subpages | Object Page + subpages (choose a flow below) |
| Creating, multi-step, branching | Wizard |
| Create brand new | Verb: **"Create"** |
| Attach existing | Verb: **"Add"** |

### Three flows for complex objects with subpages

Pick **one** for a given object and stick to it. Never mix them.

**Partial flow** (recommended when only some sections are editable)
- Users edit a single section or item in place. A local Edit button turns the section editable.
- The layout stays the same; a footer toolbar is not needed.
- Variant: partial edit with dialog (for very few fields).

**Local flow** (each page saves independently)
- Main page and subpages each have their own footer toolbar with Save/Cancel.
- Navigating between them with unsaved changes triggers a data-loss dialog (unless draft handling is on).
- Good when different users/teams work on different parts of the object, or the object is heterogeneous.

**Global flow** (whole flow saves as one unit)
- Only the main page has Save/Cancel. Subpages auto-save in the background (temporary save).
- Data-loss dialog appears only when leaving the whole flow without saving.
- Good when one user owns the whole edit session.

### Draft handling

Draft handling is Fiori's way of preventing data loss during long edits and locking objects against concurrent editors.

- **Drafts auto-save every 20 seconds** while the user is editing. A visible Save button is still required to commit to the active entity.
- **Exclusive draft scenario**: while a user has a draft open, the object is locked for others until the lock expires. When it expires, another user can discard the draft and take over.
- **Bookmark resume**: if a draft exists and the user hits a bookmark for the active entity, show a dialog:
  > We saved a draft of your changes to [entity] on [timestamp]. Do you want to resume editing or discard the changes?
  Buttons: **Resume** (reopen the draft in edit mode), **Discard** (show active entity in display mode), **Back** (return without opening).

### Data-loss patterns

When the user tries to leave without saving:

| Action | Pattern |
|---|---|
| Cancel from Create mode | **Quick confirmation popover**: "Discard this draft?" / button "Discard" |
| Cancel from Edit mode | **Quick confirmation popover**: "Discard all changes?" / button "Discard" |
| Navigate away (browser back, breadcrumb, ShellBar) during edit, no draft handling | **Data-loss dialog** (MessageBox): "You have unsaved changes. [Save / Discard / Cancel]" |
| Delete a changed object | **Dialog**: "Another user edited this [entity] without saving. Delete anyway?" |

The **quick confirmation popover** is less disruptive than a dialog; use it only for Cancel-flow confirmations. Use a dialog for Delete, for navigation-away, and for any other irreversible action.

### Delete

- Always show a confirmation **dialog** before deleting (not a popover).
- Message: "Delete [object name]? [Cancel / Delete]"
- After successful delete, show a **MessageToast** success: "[object name] deleted."
- On Object Page, Delete is usually shown in display mode only — users don't typically edit and delete an object at the same time.
- In list/table context, Delete Selected is a table-toolbar action; it's disabled until something is selected.

### Create and Edit naming

| Scenario | Button label |
|---|---|
| Create a new object (e.g. new sales order) | **Create** |
| Assign an existing object (attach a contact) | **Add** |
| Finalize create | **Create** (emphasized) |
| Finalize save | **Save** (emphasized) |
| Subpage finalize in local flow | **Save and Next** (if sequential editing is expected), else **Save** |

### Object Page: display ↔ edit toggle

- Display mode is the default. Clicking Edit switches to edit mode. This is NOT a browser-history entry; a shared URL always resolves to display.
- Save or Cancel returns to display mode; a success MessageToast confirms Save.

### Scaffold: cancel-from-edit with quick-confirmation popover

```tsx
'use client';

import { useRef, useState } from 'react';
import { Button, Popover, Text, Bar } from '@ui5/webcomponents-react';
import { Ui5DomRef } from '@ui5/webcomponents-react';

export function CancelWithConfirmation({ onConfirm }: { onConfirm: () => void }) {
  const buttonRef = useRef<Ui5DomRef<HTMLElement> | null>(null);
  const popoverRef = useRef<Ui5DomRef<HTMLElement> | null>(null);
  const [open, setOpen] = useState(false);

  return (
    <>
      <Button
        ref={buttonRef as any}
        design="Transparent"
        onClick={() => setOpen(true)}
      >
        Cancel
      </Button>
      <Popover
        ref={popoverRef as any}
        opener={buttonRef.current?.getDomRef?.() ?? undefined}
        open={open}
        onClose={() => setOpen(false)}
        footer={
          <Bar
            endContent={
              <>
                <Button design="Emphasized" onClick={() => { setOpen(false); onConfirm(); }}>
                  Discard
                </Button>
                <Button design="Transparent" onClick={() => setOpen(false)}>
                  Keep Editing
                </Button>
              </>
            }
          />
        }
      >
        <Text>Discard all changes?</Text>
      </Popover>
    </>
  );
}
```

---

## 4. Message handling

Fiori distinguishes **state messages** (bound to a field, persist until validation passes) from **transient messages** (bound to an action, fire-and-forget).

### Five controls — which to use when

| Control | Purpose | When to use |
|---|---|---|
| `MessageBox` | Interrupting dialog; user must acknowledge / decide | Single transient message; user decision required; non-field messages |
| `MessagePopover` | Multiple field-related messages | In forms / tables during edit; shows state messages grouped by section |
| `MessageView` | Multiple messages in a dialog | Transient messages triggered by action, not tied to fields |
| `MessageToast` | Small auto-dismissing popup | **Standard for success messages**; non-disruptive |
| `MessageStrip` | Inline bar in the content area | Informational context; status of the current object |

### Priority order when multiple messages exist

**Error > Warning > Success > Information.** The message popover button on the footer adopts the color of the highest-priority message.

### Field validation (state messages)

- Highlight the field with a `valueState` (Error / Warning / Success / Information) and a `valueStateMessage`.
- In edit mode, show a **message popover button on the LEFT of the footer toolbar** that lists all field issues with navigation.
- The popover navigation follows a three-tier approach:
  1. If the offending field is visible → focus it.
  2. If the field is on the same page but offscreen → scroll to it and focus.
  3. If the field is on a different tab or subpage → navigate there, scroll, focus, and reopen the popover.

### Transient messages

- **One message, user decision required** → `MessageBox`.
- **Multiple messages** → dialog embedding a `MessageView`.
- **Success (no decision)** → `MessageToast`.

### Quick confirmation popover (Cancel only)

See Object handling above. For **any other confirmation (especially Delete), use a dialog**, not a popover.

### Text patterns

Don't interrupt users unnecessarily. Ask yourself if the message can be avoided by design. When writing messages:

- Be specific: replace "data" / "object" with the actual business noun.
- Be solution-oriented: tell users what to do next.
- Keep titles short; supporting description does the explanation work.

Standard success text templates:
- "[Object name] created."
- "[Object name] saved."
- "[Object name] deleted."
- "Changes discarded."

### MessageToast recipe

```tsx
import { MessageToast } from '@ui5/webcomponents-react';

MessageToast.show('Customer saved.');
// Auto-dismisses after ~3s. No decision, no acknowledgment required.
```

### MessageBox recipe (confirm delete)

```tsx
import { MessageBox, MessageBoxAction, MessageBoxType } from '@ui5/webcomponents-react';
import { createRoot } from 'react-dom/client';
import { useState } from 'react';

export function DeleteConfirmation({
  entityName,
  onDelete,
}: {
  entityName: string;
  onDelete: () => void;
}) {
  const [open, setOpen] = useState(false);
  return (
    <>
      <Button design="Negative" onClick={() => setOpen(true)}>Delete</Button>
      <MessageBox
        open={open}
        type={MessageBoxType.Warning}
        actions={[MessageBoxAction.Delete, MessageBoxAction.Cancel]}
        emphasizedAction={MessageBoxAction.Delete}
        onClose={(e) => {
          setOpen(false);
          if (e.detail.action === MessageBoxAction.Delete) onDelete();
        }}
      >
        Delete {entityName}?
      </MessageBox>
    </>
  );
}
```

---

## 5. Empty states (IllustratedMessage)

Every empty collection, search miss, or error-with-no-data moment is an **empty state**. Empty states must have a message; illustrations and calls-to-action are optional.

### Anatomy

- **Message (mandatory)**: a headline + supporting description.
- **Illustration (optional)**: visual that reinforces the message.
- **Call to action (optional)**: button or link offering a next step.

### Canonical illustration types

- `NoData` — nothing has been added yet.
- `NoEntries` — generic empty list.
- `NoSearchResults` — search returned zero.
- `NoFilterResults` — filters matched nothing.
- `NoTasks` — used for worklists / approval queues with empty state.
- `SuccessScreen` — celebration after finishing all tasks.
- `UnableToLoad` — data fetch failed.

Always **check the Illustrated Message Use Case Library** for the right fit before adding a custom illustration.

### Size rules

- XS, S, M, L.
- Below some threshold the container shows **text only** (no illustration) — the framework drops the illustration automatically.
- Never use an illustration without a message.
- Don't use an illustration in tiles, toasts, strips, or anything smaller than a medium card.

### Text adaptation

Replace generic words with business nouns. Examples:

- Default: "No data found."
- Adapted: "No suppliers found." / "Try adjusting your filter settings."

- Default: "There's no data yet."
- Adapted: "No sales orders yet." / "When there are, you'll find them here."

### Fiori elements standard texts (follow these defaults)

| Context | Title | Description |
|---|---|---|
| List Report, no filters, no data | No results found | Start by providing your search or filter criteria. |
| List Report, filters applied, no data | No results found | Try changing your filter criteria. |
| List Report, multi-view mode, no data | No results found | Try changing the view or filter criteria. |
| Object Page, no subitems | No items available | When there are, you'll find them here. |

### Scaffold

```tsx
import { IllustratedMessage, Button } from '@ui5/webcomponents-react';
import { IllustrationMessageType, IllustrationMessageSize } from '@ui5/webcomponents-fiori';

// Generic list with no data
<IllustratedMessage
  name={IllustrationMessageType.NoData}
  titleText="No suppliers found"
  subtitleText="Try adjusting your filter settings."
  size={IllustrationMessageSize.Auto}
>
  <Button design="Emphasized" onClick={onReset}>Reset filters</Button>
</IllustratedMessage>
```

Note — from v2 the type enum is imported from `@ui5/webcomponents-fiori` via the React wrapper's re-export:

```tsx
import { IllustrationMessageType, IllustrationMessageSize } from '@ui5/webcomponents-react';
```

Both import paths work in v2; the wrapper re-exports the enums.

---

## 6. Loading / busy states

Fiori distinguishes three kinds of "waiting":

1. **Placeholder / skeleton loading** — for full-page or full-card first load when the layout is known. Use when loading exceeds 1 second.
2. **BusyIndicator at control level** — for a specific control (a table, a page section, a button action). Don't block the whole UI if only one area is loading.
3. **BusyDialog (global)** — rare; only for short cross-UI blocks. **Do not use for app or page loading** — it hides the shell bar too, preventing sign-out/search access.

### Rules

- **Never block the whole UI with a BusyDialog during app launch.** Set the busy state at page level, not at window level.
- **Overlapping busy indicators**: the framework shows only the outermost one.
- **If your loading takes longer than 1 second**, prefer skeleton / placeholder loading over a spinner — it gives a better sense of progress.
- **On fetch failure**, switch from busy → IllustratedMessage with `UnableToLoad`.

### v2 API cheat

- `<BusyIndicator active size="Medium" />` — a standalone spinner.
- `<Page loading />` — loading prop on Page blocks page interaction while showing a spinner.
- `<AnalyticalTable loading loadingDelay={0} />` — table-level busy overlay.
- `<Card loading />` — card-level busy overlay.

The v1 `Loader` component is **removed from v2's main package** (moved to compat). Always use `BusyIndicator`.

### Scaffold — page-level loading with graceful empty / error

```tsx
'use client';

import { useEffect, useState } from 'react';
import {
  DynamicPage, DynamicPageTitle, Title, BusyIndicator,
  IllustratedMessage, Button,
} from '@ui5/webcomponents-react';
import { IllustrationMessageType } from '@ui5/webcomponents-react';

export default function CustomersPageClient() {
  const [state, setState] = useState<'loading' | 'empty' | 'error' | 'ready'>('loading');
  const [rows, setRows] = useState<any[]>([]);

  useEffect(() => {
    let cancelled = false;
    fetch('/api/customers')
      .then((r) => (r.ok ? r.json() : Promise.reject(new Error(String(r.status)))))
      .then((data) => {
        if (cancelled) return;
        setRows(data);
        setState(data.length === 0 ? 'empty' : 'ready');
      })
      .catch(() => !cancelled && setState('error'));
    return () => { cancelled = true; };
  }, []);

  return (
    <DynamicPage titleArea={<DynamicPageTitle header={<Title>Customers</Title>} />}>
      {state === 'loading' && (
        <div style={{ padding: 'var(--sapContent_Space_L)', textAlign: 'center' }}>
          <BusyIndicator active size="Medium" />
        </div>
      )}
      {state === 'empty' && (
        <IllustratedMessage
          name={IllustrationMessageType.NoData}
          titleText="No customers yet"
          subtitleText="When there are, you'll find them here."
        />
      )}
      {state === 'error' && (
        <IllustratedMessage
          name={IllustrationMessageType.UnableToLoad}
          titleText="Couldn't load customers"
          subtitleText="Please try again in a moment."
        >
          <Button design="Emphasized" onClick={() => location.reload()}>Retry</Button>
        </IllustratedMessage>
      )}
      {state === 'ready' && (/* render table */ null)}
    </DynamicPage>
  );
}
```

---

## 7. Keyboard & accessibility

Fiori is keyboard-first. All standard controls are keyboard-enabled; your job is to not break that.

### Framework-level shortcuts

| Shortcut | Effect |
|---|---|
| `Tab` / `Shift+Tab` | Move focus forward / backward through controls |
| `F6` / `Shift+F6` | Jump to next / previous **logical group** (shell bar, side nav, filter bar, content) |
| `Enter` | Activate focused control |
| `Space` | Select focused row / item |
| `Esc` | Close dialog / popover / cancel the current modal action |
| `Ctrl+A` | Select all in a list/table |
| `Home` / `End` | Top / bottom in a list or table |
| `Arrow keys` | Navigate within a group (tiles, options, table cells) |

### Conventional app-level shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+S` | Save |
| `Ctrl+Enter` | Save and next (if applicable) |
| `Ctrl+,` | Open table view settings / personalization |
| `Ctrl+Shift+S` | Share |
| `Ctrl+Alt+A` | Select all (alternative) |
| `F2` | Go to detail (where applicable) |

These are **app-defined**; you need to wire them up. Browsers intercept very few shortcuts (mainly `Ctrl+L` for the address bar).

### Logical groups

Every Fiori page has distinct groups: ShellBar, SideNav, FilterBar, Table Toolbar, Table, FooterBar. F6 jumps between them, which saves power users many tabs.

When building custom containers, make them focusable groups by adding `role="region"` + `aria-label`.

### Accessibility checklist

- **Every interactive control has a keyboard handler** (UI5 components handle this automatically).
- **Every form field has a visible label**. UI5 `<FormItem labelContent={<Label>…</Label>}>` is the idiomatic way.
- **Every icon-only button has a tooltip** (`<Button tooltip="Edit">`) — screen readers read it.
- **Every illustrated message has a text message** — illustrations alone are not accessible.
- **Color is never the only signal.** `ObjectStatus`, `valueState`, and message priorities all use icons + colors together.
- **Focus is managed on navigation**: after clicking a line-item to open an Object Page, the focus should land on the page title. After closing a dialog, focus returns to the button that opened it.
- **Do not set `role="application"` at the page root** (v2 removed this). Let the browser's default document role apply.

### Typography and zoom

- Don't hardcode font sizes. Use `var(--sapFontSize)` / `var(--sapFontLargeSize)` / `var(--sapFontSmallSize)`.
- The responsive layout re-flows when users zoom the browser. Don't prevent this.
- Support RTL layouts if your audience needs them. The UI5 theme provides RTL classes automatically.

### Screen reader notes

- UI5 components include proper ARIA attributes by default. Don't override them.
- When you wrap or extend a UI5 control, add an `accessibleName` / `accessibleNameRef` prop if the control exposes one.
- Hide purely decorative icons from screen readers (UI5 handles this for its own icons).

---

## Quick self-review checklist

Before shipping any Fiori page, run through this list. Anything that fails a rule, fix it.

- [ ] Only one `Emphasized` button on the whole page.
- [ ] `Emphasized` and `Positive`/`Negative` not mixed in the same toolbar.
- [ ] Finalizing actions (Save / Post / Submit) are in the `footerArea`.
- [ ] Non-finalizing page actions (Share, Export) are in the title `actionsBar`.
- [ ] Create / Delete-Selected / Export-Selection actions are in the table toolbar, not the page title.
- [ ] Row-level actions are inline (terminal Cell) with `stopPropagation()` on click.
- [ ] Sort / Filter / Group is via the column header popover, not duplicated in the table toolbar.
- [ ] Every page has a clear primary action (or explicitly none).
- [ ] Cancel during Create triggers a "Discard this draft?" quick-confirmation popover.
- [ ] Cancel during Edit triggers a "Discard all changes?" quick-confirmation popover.
- [ ] Delete is confirmed by a dialog, never a popover.
- [ ] Success after a finalize action shows a `MessageToast` with business-specific text.
- [ ] Empty state uses `IllustratedMessage` with a business-specific message (no raw "No data").
- [ ] Loading uses `BusyIndicator` at page/section level, not a global BusyDialog.
- [ ] Error state shows `UnableToLoad` illustrated message with a retry action.
- [ ] `F6` jumps between logical groups; `Tab` walks inside each group; `Esc` closes dialogs.
- [ ] Breadcrumbs represent app-internal hierarchy; don't duplicate ShellBar menu items.
- [ ] Deep-link landing: no back button when history is empty.
- [ ] Display↔Edit toggle does not create a history entry.
- [ ] Every icon-only button has a `tooltip`.
- [ ] Every field has a `<Label>` and the relevant `valueState` on validation.
- [ ] Buttons follow Fiori labels: "Create" (new), "Add" (assign existing), "Save", "Cancel".
