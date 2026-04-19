# Object Page

The Fiori default for representing a single business object. Most apps end up with more Object Pages than any other floorplan — take the time to get them right.

## When to use

- User needs to **display, create, or edit a single business object** (a customer, an order, a contract, a ticket).
- Object has **multiple logical sections** (header facts, line items, related documents, audit history).
- Structure lets the user **interact with different parts of the object** (scroll or jump between sections, edit one part without disrupting the others).

If the object is trivially small (one section, only a few fields, no related data), a plain `DynamicPage` with a `Form` inside is enough — you don't need the Object Page machinery. But the moment you have two or more sections' worth of content, use Object Page.

> **Also consult** `references/global-patterns.md` for cross-cutting rules (action placement taxonomy, create/edit/delete flows, draft handling, confirmation dialogs, empty states, loading, keyboard). This file covers Object Page specifics; the global patterns cover what every page (including this one) has to obey.

## When NOT to use

- You're listing multiple objects → List Report or Worklist, not Object Page.
- You're creating an object via a long, guided, unfamiliar flow → Wizard.
- You're showing an overview across many apps → Overview Page.

## What you get for free

- **Dynamic page header** with a snapping, collapsing header content area. The title line stays pinned; the rich header collapses when the user scrolls so the content takes focus.
- **Anchor bar or tab bar navigation** between sections (pick one; see below).
- **Sticky section headers** (since v2.19.0).
- **Responsive column layout** for content: 4 cols on XL, 3 on L (or optionally 2), 2 on M, 1 on S.
- **Footer toolbar** that stays anchored for primary actions.
- **Breadcrumb navigation** above the title for parent-child drilldowns.
- **Pin/unpin** the expanded header so the user can keep context while scrolling (can be disabled with `hidePinButton`).
- **Lazy section loading** for long pages (built-in; sections load as they come into view).

## Fiori design rules — apply these in every Object Page you scaffold

### Header area

- **Always use the dynamic page header** (this is what ObjectPage gives you in v2 — don't second-guess it). Do not try to build a custom sticky title bar.
- Put the object's identity (title, key IDs, status) in the **title area**. This stays visible when scrolled.
- Put supplementary context (avatars, KPIs, grouped facts, sparklines, small chart) in the **header area**. This collapses on scroll.
- **Never put editable form fields in the header.** Because the header snaps, any editable input would disappear mid-edit. In display mode, header fields are read-only summaries. In edit mode, a temporary editable "Header" section appears at the top of the content with the editable header fields; the user edits there, not in the snapping header.
- **Breadcrumbs** go above the title, for parent-child navigation (e.g. Customer > Order > Order Item).
- **Subtitle** can summarize collapsed-header context (e.g. "Active · Gold tier · EMEA").
- On small screens, images in the header get smaller; use a responsive `Avatar` rather than a raw `<img>` tag.

### Navigation: anchor bar vs tab bar

Pick one based on content structure.

- **Anchor bar** (default, recommended most of the time): horizontal links; clicking scrolls to the section. User can also scroll freely through sections.
  - Use when sections belong together and users might want to jump or scroll.
  - If there's only one section, no anchor bar is shown.
  - The first section's title is hidden in the content (unless the first section contains a table).
- **Icon tab bar** (`mode="IconTabBar"`): tabs replace anchor links; each section becomes a tab.
  - Use when sections are more like independent views than parts of one story.
  - Useful when one section contains many rows (100–400) — giving that section its own tab means it gets its own scroll context.
  - Text-only tabs (no icons).
- **Never show both** visually. Pick a `mode`.

### Content area: sections and subsections

- **Sections are containers for subsections**. Sections *cannot* directly contain controls. If you're tempted to stick a `Form` or `Table` directly in a section, wrap it in a subsection first.
- **Subsections hold actual content**: forms, tables, charts, lists.
- **`titleText` is required** on both `ObjectPageSection` and `ObjectPageSubSection` in v2.
- If a subsection contains a single `Table`/`AnalyticalTable`/`Chart` with the same title as the subsection, you can hide the control's title to avoid duplication.
- If a section has only one subsection, the subsection title *becomes* the section's label in the anchor bar (no submenu).

### Tables inside an Object Page — the size rule that everyone gets wrong

Fiori has an explicit policy for tables inside an Object Page, keyed to row count. Follow it:

| Row count | Pattern |
|---|---|
| ≤ 50–100 | Embed the full table inline in an anchor-nav section. |
| 100–400 | Switch ObjectPage `mode` to `IconTabBar`. Put the table in its own tab; that tab gets its own scroll context. Also: the table must be the only or last element in the tab, and "scroll to load" (growing) behavior must be enabled. |
| 400+ | Do **not** embed the full table. Show a preview (top 10–20 rows) in a subsection, with a right-aligned **"Show All (N)"** link below the table pointing to a dedicated List Report page for that collection. Apply sensible predefined filters/sorts to the preview. |

### Footer actions

- **All finalizing actions go in `footerArea`**: Save, Cancel, Edit, Delete, Submit, Approve, Reject. Use a `Bar` with `design="FloatingFooter"`.
- **Do not put Save/Cancel/Edit in header buttons.** `actionsBar` in the title is for non-finalizing global actions (Share, Export to PDF, Print).
- If the object is draft-enabled, show "Save" and "Save as Draft" distinctly.

### Edit mode specifics

- Layout stays the same between display and edit modes — content should not relocate.
- Set initial focus on the first empty mandatory field in edit mode. If no mandatory fields, first editable field or first action.
- If the user navigates away from unsaved changes, show a data-loss dialog.
- For partial edit of header content: either a dialog (few fields) or a subpage (many fields).

### Side content

- Optional "dynamic side content" alongside the Object Page is permitted, but only for **contextual** information (chat, timeline, notes, help).
- **Never** put finalizing actions in the side panel (the side panel takes the whole right side; actions there get lost).
- **Never** put object information in the side panel — that belongs in the content area.

## v2 component API cheat sheet

Slot and prop names changed substantially from v1. These are the authoritative v2 names.

**`ObjectPage` props:**
- `titleArea` — takes a `<ObjectPageTitle>` (was `headerTitle` in v1).
- `headerArea` — takes a `<ObjectPageHeader>` (was `headerContent` in v1).
- `footerArea` — takes a `<Bar>` or any JSX element used as footer (was `footer` in v1).
- `image` — string URL or `<Avatar>` element (types updated).
- `mode` — `"Default"` (anchor) or `"IconTabBar"`.
- `headerPinned` — boolean, keep header expanded even on scroll (was `alwaysShowContentHeader` in v1).
- `hidePinButton` — boolean, hide the pin affordance entirely (was `headerContentPinnable` with inverted logic).
- `onToggleHeaderArea` — event when user collapses/expands the header (was `onToggleHeaderContent`).
- `onPinButtonToggle` — event when user pins/unpins (was `onPinnedStateChange`).
- `onSelectedSectionChange` — event when the active section changes.
- `accessibilityAttributes.expandButton.expanded` — customize a11y announcements.

**`ObjectPageTitle` props:**
- `header` — main title content (typically a `<Title>`).
- `subHeader` — subtitle content.
- `snappedHeader` / `snappedSubHeader` — alternative header/subHeader shown when the page is scrolled/collapsed (added in v2.12).
- `breadcrumbs` — takes a `<Breadcrumbs>` element.
- `actionsBar` — takes a `<Toolbar>` with `ToolbarButton` children (was `actions` in v1; was redundant `actionsToolbarProps` removed).
- `navigationBar` — takes a `<Toolbar>` with `ToolbarButton` children (was `navigationActions` in v1).
- `children` — status indicators (`<ObjectStatus>`), tags, etc.

**`ObjectPageHeader` props:** just renders its children in the header facets container. Use flex / grid inside.

**`ObjectPageSection` props:**
- `id` — required for programmatic selection.
- `titleText` — **required** in v2.
- `titleTextUppercase` — boolean, default is no longer `true`.
- `hideTitleText` — boolean, hide title if duplicate.

**`ObjectPageSubSection` props:**
- `id` — required.
- `titleText` — **required** in v2.
- `actions` — subsection-local toolbar.

Removed in v2 (if you see these in user code, they're v1 — migrate):
- `showHideHeaderButton`, `showTitleInHeaderContent` — removed entirely.
- `alwaysShowContentHeader` → `headerPinned`.
- `headerContentPinnable` → `hidePinButton` (logic inverted).
- `actionsToolbarProps`, `navigationActionsToolbarProps` — redundant, pass a `Toolbar` directly.

## Production scaffold — display + edit with footer, breadcrumb, status, sections

```tsx
'use client';

import {
  ObjectPage,
  ObjectPageTitle,
  ObjectPageHeader,
  ObjectPageSection,
  ObjectPageSubSection,
  Breadcrumbs,
  BreadcrumbsItem,
  Title,
  Label,
  Text,
  ObjectStatus,
  Avatar,
  Bar,
  Button,
  Toolbar,
  ToolbarButton,
  ToolbarSpacer,
  Form,
  FormGroup,
  FormItem,
  Input,
  AnalyticalTable,
  MessageStrip,
  BusyIndicator,
} from '@ui5/webcomponents-react';

import editIcon from '@ui5/webcomponents-icons/dist/edit.js';
import deleteIcon from '@ui5/webcomponents-icons/dist/delete.js';
import shareIcon from '@ui5/webcomponents-icons/dist/share.js';

import { useMemo, useState } from 'react';

type Customer = {
  id: string;
  name: string;
  status: 'Active' | 'Blocked' | 'Pending';
  email: string;
  phone: string;
  country: string;
  accountManager: string;
  tier: 'Gold' | 'Silver' | 'Bronze';
  createdOn: string;
};

type Order = {
  id: string;
  createdOn: string;
  total: number;
  status: 'Open' | 'Shipped' | 'Closed';
};

type Props = {
  customer: Customer;
  orders: Order[];              // up to ~50 rows; if >100 switch to IconTabBar and give the tab its own scroll
  ordersTotal: number;          // total backend count; if > orders.length, show "Show All (N)" link
  loading?: boolean;
  onSave: (draft: Partial<Customer>) => Promise<void>;
  onDelete: () => Promise<void>;
  onNavigateToAllOrders: () => void;
};

const statusState = (s: Customer['status']) =>
  s === 'Active' ? 'Positive' : s === 'Blocked' ? 'Negative' : 'Critical';

export default function CustomerObjectPage({
  customer,
  orders,
  ordersTotal,
  loading,
  onSave,
  onDelete,
  onNavigateToAllOrders,
}: Props) {
  const [editing, setEditing] = useState(false);
  const [draft, setDraft] = useState<Partial<Customer>>({});
  const [saving, setSaving] = useState(false);

  const orderColumns = useMemo(
    () => [
      { Header: 'Order', accessor: 'id' },
      { Header: 'Created', accessor: 'createdOn' },
      {
        Header: 'Status',
        accessor: 'status',
        Cell: ({ value }: { value: Order['status'] }) => (
          <ObjectStatus
            state={value === 'Closed' ? 'Positive' : value === 'Open' ? 'Critical' : 'Information'}
          >
            {value}
          </ObjectStatus>
        ),
      },
      { Header: 'Total', accessor: 'total', hAlign: 'End' },
    ],
    [],
  );

  const save = async () => {
    setSaving(true);
    try {
      await onSave(draft);
      setDraft({});
      setEditing(false);
    } finally {
      setSaving(false);
    }
  };

  if (loading) return <BusyIndicator active size="L" style={{ margin: '2rem auto', display: 'block' }} />;

  return (
    <ObjectPage
      image={<Avatar initials={customer.name.slice(0, 2).toUpperCase()} size="L" />}
      titleArea={
        <ObjectPageTitle
          breadcrumbs={
            <Breadcrumbs>
              <BreadcrumbsItem>Customers</BreadcrumbsItem>
              <BreadcrumbsItem>{customer.id}</BreadcrumbsItem>
            </Breadcrumbs>
          }
          header={<Title>{customer.name}</Title>}
          subHeader={
            <Label>
              {customer.id} · {customer.tier} · {customer.country}
            </Label>
          }
          snappedSubHeader={
            <Label>
              {customer.id} · {customer.tier} · {customer.status}
            </Label>
          }
          actionsBar={
            <Toolbar design="Transparent">
              {!editing && (
                <ToolbarButton design="Transparent" icon={shareIcon} text="Share" />
              )}
            </Toolbar>
          }
        >
          <ObjectStatus state={statusState(customer.status)}>{customer.status}</ObjectStatus>
        </ObjectPageTitle>
      }
      headerArea={
        <ObjectPageHeader>
          <div style={{ display: 'grid', gridTemplateColumns: 'repeat(auto-fit, minmax(180px, 1fr))', gap: '1rem', width: '100%' }}>
            <div style={{ display: 'flex', flexDirection: 'column', gap: '0.25rem' }}>
              <Label>Email</Label>
              <Text>{customer.email}</Text>
            </div>
            <div style={{ display: 'flex', flexDirection: 'column', gap: '0.25rem' }}>
              <Label>Phone</Label>
              <Text>{customer.phone}</Text>
            </div>
            <div style={{ display: 'flex', flexDirection: 'column', gap: '0.25rem' }}>
              <Label>Account manager</Label>
              <Text>{customer.accountManager}</Text>
            </div>
            <div style={{ display: 'flex', flexDirection: 'column', gap: '0.25rem' }}>
              <Label>Customer since</Label>
              <Text>{customer.createdOn}</Text>
            </div>
          </div>
        </ObjectPageHeader>
      }
      footerArea={
        <Bar
          design="FloatingFooter"
          endContent={
            editing ? (
              <>
                <Button design="Emphasized" onClick={save} disabled={saving}>
                  {saving ? 'Saving…' : 'Save'}
                </Button>
                <Button
                  design="Transparent"
                  onClick={() => {
                    setDraft({});
                    setEditing(false);
                  }}
                >
                  Cancel
                </Button>
              </>
            ) : (
              <>
                <Button design="Emphasized" icon={editIcon} onClick={() => setEditing(true)}>
                  Edit
                </Button>
                <Button design="Transparent" icon={deleteIcon} onClick={onDelete}>
                  Delete
                </Button>
              </>
            )
          }
        />
      }
    >
      {/* Temporary "Header" section: only visible in edit mode because the header has editable fields.
          Per Fiori guidelines, editable header content moves into a top section during edit. */}
      {editing && (
        <ObjectPageSection id="header-edit" titleText="Header">
          <ObjectPageSubSection id="header-edit-contact" titleText="Contact">
            <Form labelSpan="S12 M4 L4 XL4" layout="S1 M2 L2 XL2">
              <FormItem labelContent={<Label required>Email</Label>}>
                <Input
                  value={draft.email ?? customer.email}
                  onInput={(e) => setDraft((d) => ({ ...d, email: (e.target as HTMLInputElement).value }))}
                />
              </FormItem>
              <FormItem labelContent={<Label>Phone</Label>}>
                <Input
                  value={draft.phone ?? customer.phone}
                  onInput={(e) => setDraft((d) => ({ ...d, phone: (e.target as HTMLInputElement).value }))}
                />
              </FormItem>
            </Form>
          </ObjectPageSubSection>
        </ObjectPageSection>
      )}

      <ObjectPageSection id="general" titleText="General Information">
        <ObjectPageSubSection id="general-contact" titleText="Contact Details">
          <Form labelSpan="S12 M4 L4 XL4" layout="S1 M2 L2 XL2">
            <FormItem labelContent={<Label>Name</Label>}>
              <Text>{customer.name}</Text>
            </FormItem>
            <FormItem labelContent={<Label>Country</Label>}>
              <Text>{customer.country}</Text>
            </FormItem>
            <FormItem labelContent={<Label>Tier</Label>}>
              <Text>{customer.tier}</Text>
            </FormItem>
            <FormItem labelContent={<Label>Status</Label>}>
              <ObjectStatus state={statusState(customer.status)}>{customer.status}</ObjectStatus>
            </FormItem>
          </Form>
        </ObjectPageSubSection>
      </ObjectPageSection>

      <ObjectPageSection id="orders" titleText="Orders">
        <ObjectPageSubSection id="orders-recent" titleText="Recent Orders">
          {orders.length === 0 ? (
            <MessageStrip design="Information" hideCloseButton>
              This customer has no orders yet.
            </MessageStrip>
          ) : (
            <>
              <AnalyticalTable
                data={orders}
                columns={orderColumns}
                visibleRows={Math.min(orders.length, 10)}
                selectionMode="Single"
                onRowClick={(e) => {
                  // TODO: navigate to Order object page:
                  // router.push(`/orders/${e.detail.row.original.id}`)
                }}
              />
              {ordersTotal > orders.length && (
                <div style={{ textAlign: 'end', marginBlockStart: '0.5rem' }}>
                  {/* Fiori "Show All (N)" link: user clicks through to a full List Report */}
                  <Button design="Transparent" onClick={onNavigateToAllOrders}>
                    Show All ({ordersTotal})
                  </Button>
                </div>
              )}
            </>
          )}
        </ObjectPageSubSection>
      </ObjectPageSection>

      <ObjectPageSection id="history" titleText="History">
        <ObjectPageSubSection id="history-audit" titleText="Audit Trail">
          {/* TODO: render audit timeline */}
          <Text>Audit events will appear here.</Text>
        </ObjectPageSubSection>
      </ObjectPageSection>
    </ObjectPage>
  );
}
```

### Notes on the scaffold

- The `image` prop on `ObjectPage` accepts a URL string or an `<Avatar>` JSX element. Always use `Avatar` when the source is an initials/placeholder, not `<img>`.
- The **temporary "Header" section** pattern is Fiori-canonical. In SAP Fiori elements it's automatic; in freestyle (which this is) you implement it yourself by rendering an extra section at the top when `editing === true`.
- The **Show All (N)** link at the bottom of the orders table is the Fiori pattern for "too many rows to show here; click through to the full list report".
- Use `Form` with the responsive `labelSpan` and `layout` strings (`"S12 M4 L4 XL4"` / `"S1 M2 L2 XL2"`) — this is the v2 API. Passing a number like `labelSpan={4}` won't be responsive.
- `onRowClick` on `AnalyticalTable` gives `e.detail.row.original` which is your original row data object.
- Set `selectionMode="Single"` (not `"SingleSelect"` — the v2 enum uses single words).

## Fiori text templates for Object Page

Use these exact patterns. They match Fiori elements defaults and can be adapted with the business noun.

**Confirmation (quick-confirmation popover, triggered by Cancel):**
- Create mode: `Discard this draft?` — button `Discard`.
- Edit mode: `Discard all changes?` — button `Discard`.

**Confirmation dialog (MessageBox) for irreversible actions:**
- Delete: `Delete [customer name]?` — buttons `Delete` (emphasized) / `Cancel`.
- Delete with conflict: `Another user edited this [entity] without saving the changes. Delete anyway?` — buttons `Delete` / `Cancel`.

**Success MessageToast after a finalize action:**
- Create: `[Customer name] created.`
- Save: `[Customer name] saved.`
- Delete: `[Customer name] deleted.`
- Approve / Reject: `[Object] approved.` / `[Object] rejected.`
- Discard: `Changes discarded.`

**Empty-state IllustratedMessage on a subsection with a table:**
- `No items available` / `When there are, you'll find them here.` (Fiori elements default)
- Adapt the noun: `No contacts yet` / `Add a contact to get started.`

**Draft resume dialog (when user returns to an object with an existing draft):**
- Title: `Unsaved changes`
- Body: `We saved a draft of your changes to [customer name] on [date, time]. Do you want to resume editing or discard the changes?`
- Buttons: `Resume` (emphasized) / `Discard` / `Back`.

## Anti-patterns (real ones I've seen in user code)

- Putting `<Button>Save</Button>` in the `actionsBar` of `ObjectPageTitle`. → Move to `footerArea`.
- Using v1 slot names (`headerTitle`, `headerContent`, `footer`). → Rename to `titleArea`, `headerArea`, `footerArea`.
- Passing individual `<Button>` children to `actionsBar`. → Wrap in `<Toolbar>` with `<ToolbarButton>` children.
- Embedding an 800-row table in an anchor-nav section. → Switch to `mode="IconTabBar"` or preview-plus-"Show All".
- Putting editable inputs in `headerArea`. → Move to a temporary edit-mode section; keep the header read-only.
- Omitting `titleText` on sections/subsections. → Required in v2; build will TS-error.
- Using `@ui5/webcomponents-react-compat`'s `Toolbar` in new code. → Use the root-package `Toolbar`.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents`
- `@ui5/webcomponents-fiori` (needed for `ObjectPage` — it's part of the Fiori package)
- `@ui5/webcomponents-icons` (for the edit/delete/share icons)
