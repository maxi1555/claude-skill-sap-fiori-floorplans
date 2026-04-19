# Action Placement — cross-cutting cheat sheet

The single biggest source of "this doesn't feel Fiori" is **actions in the wrong place, or styled wrong**. Fiori has a clear hierarchy of action slots AND a strict styling taxonomy. This file is the cheat sheet.

For the full cross-cutting Fiori patterns (navigation, object handling, messaging, empty states, keyboard), see `references/global-patterns.md`. This file focuses specifically on where actions live and how they're styled.

## Part 1 — The four action types

Every action on a Fiori page belongs to exactly one of these types:

| Type | Purpose | Example |
|---|---|---|
| **Primary** | The single most important action on the page | Save (on an edit page), Post (on an approval), Create (on a new-object page) |
| **Secondary** | Supporting actions that don't dominate | Edit (next to Save in display mode is rare), Download, Duplicate |
| **Semantic** | Positive / negative workflow decisions | Accept, Reject, Approve, Decline |
| **Negative path** | Leave the workflow without changes | Cancel, Close, Discard |

## Part 2 — Button styling per action type

This is the **official Fiori mapping**. Get it right and pages look right.

| Action type | Header / footer toolbar | Content toolbar |
|---|---|---|
| Primary | `Emphasized` | `Emphasized` (only if the page has no other primary); else `Default` |
| Secondary | `Default` | `Transparent` |
| Semantic (Accept) | `Positive` | `Positive` |
| Semantic (Reject) | `Negative` | `Negative` |
| Negative path | `Transparent` | `Transparent` |

### Styling rules — non-negotiable

1. **Maximum one `Emphasized` button per toolbar.** Emphasized is a primary marker; two defeat the purpose.
2. **Never mix `Emphasized` with `Positive`/`Negative` in the same toolbar.** If a workflow is semantic (Accept / Reject), those two are your primary-like buttons — no Emphasized Save beside them.
3. **Ideally, one `Emphasized` per entire page**, across all toolbars. If you need a content-toolbar primary and the page already has one, demote the content-toolbar primary to `Default`.
4. **Icon OR text, never both** — except inline row actions in tables, where combining makes sense.
5. **Always text** for primary, secondary, semantic, and negative-path actions. Icon-only is allowed only when the icon's meaning is globally unambiguous (add, delete, share).
6. **Split buttons and menu buttons**: always `Transparent` in content toolbars; `Default` (ghost) in header/footer toolbars. Never apply semantic styling (Positive/Negative) to a split or menu button.
7. **Toggle buttons**: for secondary actions only; use `Default` styling in header/footer and `Transparent` in content toolbars. Label must communicate toggleability.

### Action priority (overflow behavior)

Fiori defines four priority values used when a toolbar overflows:

| Priority | Behavior |
|---|---|
| `Low` | First to overflow when space runs out |
| `High` (default) | Overflow only after all Low-priority actions have overflowed |
| `AlwaysOverflow` | Always in the overflow menu, even when space is available |
| `NeverOverflow` | Never moved to overflow; label truncates instead |

**Rule**: `Emphasized` and primary-action buttons must **never** be moved to overflow. If you implement custom overflow handling, mark them `NeverOverflow`.

## Part 3 — Where actions live (the hierarchy)

From outermost (app-wide) to innermost (row-level):

1. **ShellBar** — app-level, across all pages.
2. **Page title `actionsBar`** — non-finalizing page-global actions.
3. **`navigationBar`** (ObjectPageTitle only) — secondary nav within a parent collection.
4. **Page `footerArea`** — finalizing page-global actions.
5. **Subsection-local toolbar** (ObjectPageSubSection.actions) — subsection-scoped actions.
6. **Table toolbar** — collection-level actions.
7. **Row inline actions** — row-scoped actions.
8. **Column header popover** — column settings.

### Rule 1 — finalizing actions go in the footer

Finalizing = commits a transaction. Save, Cancel, Delete, Submit, Approve, Reject, Post, Accept.

✅ `footerArea` via `<Bar design="FloatingFooter">`.
❌ `actionsBar` of DynamicPageTitle / ObjectPageTitle.
❌ ShellBar.

**Exception**: apps with only 1–2 actions total and no other header actions may place workflow actions in the header. Rare.

### Rule 2 — non-finalizing global actions go in the title actionsBar

Non-finalizing = doesn't commit; supports the user around the task. Share, Export, Print, Settings, Copy Link.

✅ `actionsBar` in the title as a `<Toolbar design="Transparent">` with `<ToolbarButton>` children.
❌ Footer (reserved for finalizing).

### Rule 3 — app-global actions go in the ShellBar

App-global = not scoped to this specific page. Help, Settings, Notifications, Product Switch, Profile, Sign Out.

✅ ShellBar slots (`profile`, `showNotifications`, `showProductSwitch`) or `ShellBarItem` children.
❌ Page title (mixes app and page scope).
❌ Footer.

### Rule 4 — collection actions go in the table toolbar

Collection actions operate on the whole table or its selection. Create, Delete Selected, Export, Assign to…

✅ A `<Toolbar>` rendered above the table (or in AnalyticalTable's `extension` slot).
❌ Page title.
❌ Footer.

When a collection action depends on selection (Delete Selected), **disable it when nothing is selected**, and append the count when something is: `Delete (3)`.

### Rule 5 — row-scoped actions go inline or in a row menu

Row actions operate on a single row. Edit this, Approve this one, Reject this one.

✅ Inline action buttons in a terminal Cell column — up to 2 visible, rest in a Menu.
❌ In the table toolbar (that's selection-scoped, not row-scoped).
❌ In a footer.

Always `e.stopPropagation()` on the inline button container so row-click navigation doesn't also fire.

### Rule 6 — column settings go in the column header popover

Column settings: sort by this column, filter by this column, group by this column, pin this column, resize this column.

✅ The automatic column header popover (built into AnalyticalTable).
❌ Separate "Sort", "Group by", "Filter" buttons in the table toolbar — the popover is the Fiori way.

Exception: very rarely, if you want explicit "table layout variant" (column visibility / order / width), you may add a VariantManagement at the table toolbar level. Only do this with a strong use case.

### Rule 7 — subsection-local actions live in the subsection

If a specific subsection has its own scoped actions (e.g. "Contacts" subsection with "Add Contact"), put them in the subsection's `actions` slot — not on the page level.

## Part 4 — Ordering within a toolbar

### Header toolbar (right-aligned; primary on the LEFT of its group)

Groups from left to right:
1. **Business actions** (Edit, Delete)
2. **Manage content** (Filter, Search)
3. **Manage layout** (Full screen, Close column)
4. **Generic** (Share, Export, Print)

The primary action, if it lives in the header, goes leftmost — even if it logically belongs to a group further right.

### Footer toolbar (right-aligned; primary on the LEFT of its group)

Groups from left to right:
1. **Forward path** (Post, Accept, Save). Can be semantic positive (Accept / Approve).
2. **Alternative path** — secondary (Reject, Send to Manager).
3. **Negative path** — leaves without changes (Cancel, Close).

### Table toolbar

No strict Fiori group order within a table toolbar — convention is:
- Title / count on the left.
- Spacer in the middle.
- Primary collection action on the right (e.g. Create).
- Selection-dependent actions to its right (Delete, Copy).
- Generic (Export, Settings) last.

## Part 5 — The map: which action goes where, per floorplan

### ShellBar (every app)

| Action | Goes in |
|---|---|
| Help | ShellBarItem or menuItems |
| Settings | ShellBarItem |
| Notifications | `showNotifications` + `onNotificationsClick` |
| Profile / Sign Out | `profile` + `onProfileClick` |
| Product switcher | `showProductSwitch` |
| Search | `searchField` + `showSearchField` |
| Back | `startButton` |
| Home (logo click) | `onLogoClick` |

### Object Page

| Action | Goes in | Style |
|---|---|---|
| Edit | `footerArea` (or `actionsBar` as a state-toggle) | Emphasized if primary in display mode |
| Save | `footerArea` | Emphasized |
| Cancel | `footerArea` (with quick-confirm popover) | Transparent |
| Delete | `footerArea` (with confirmation dialog) | Transparent, or Negative if it IS the primary |
| Submit / Approve | `footerArea` | Positive (semantic) |
| Reject | `footerArea` | Negative (semantic) |
| Share | `actionsBar` | Transparent |
| Export to PDF | `actionsBar` | Transparent |
| Print | `actionsBar` | Transparent |
| Add subobject | Subsection `actions` or table toolbar | Default |
| Reorder items | Subsection `actions` | Transparent |
| Forward / Backward in parent list | `navigationBar` | Transparent |
| Full-screen (FCL) | `actionsBar` | Transparent |
| Close column (FCL) | `actionsBar` | Transparent |

### List Report

| Action | Goes in | Style |
|---|---|---|
| Share | title `actionsBar` | Transparent |
| Export (full result set) | title `actionsBar` or table toolbar | Transparent |
| Create | Table toolbar | Default (page has no footer primary usually) or Emphasized |
| Delete Selected | Table toolbar (disabled with no selection) | Transparent |
| Copy Selected | Table toolbar | Transparent |
| Assign to… | Table toolbar | Transparent |
| Edit (one row) | Row inline action | Transparent icon |
| Approve / Reject (one row) | Row inline action | Positive / Negative |
| Adjust Filters | FilterBar toolbar | Transparent |
| Manage Variants | `VariantManagement` replacing the title | — |

### Worklist

| Action | Goes in | Style |
|---|---|---|
| Share | title `actionsBar` | Transparent |
| Approve / Reject (one) | Row inline action | Positive / Negative |
| Approve / Reject Selected | Table toolbar (disabled when empty) | Positive / Negative |
| Assign Selected | Table toolbar | Transparent |
| Switch view (All / Pending / Overdue) | TabContainer or SegmentedButton | — |

### Analytical List Page

| Action | Goes in | Style |
|---|---|---|
| Share | title `actionsBar` | Transparent |
| Export | title `actionsBar` | Transparent |
| Manage Variants | `VariantManagement` as title | — |
| Toggle Chart / Table / Hybrid | SegmentedButton | — |
| Drill-down (on chart) | Chart click | — |
| Clear drill-down | Content area affordance | Transparent |
| Adjust Filters | FilterBar toolbar | Transparent |

### Overview Page

| Action | Goes in | Style |
|---|---|---|
| Share | title `actionsBar` | Transparent |
| Manage Cards | title `actionsBar` | Transparent |
| Filter all cards | FilterBar in `headerArea` | — |
| Card action (quick-view only) | Card's own `<Bar design="Footer">` | — |
| Navigate to parent app | `CardHeader.onClick` | — |
| Navigate to one item inside a card | List item / table row click | — |

### Wizard

| Action | Goes in | Style |
|---|---|---|
| Next (Step N) | Inside current step content | Emphasized |
| Previous | `footerArea` (optional) | Transparent |
| Review | Inside the last-step-before-review | Emphasized, labeled "Review" |
| Create / Save / Submit | `footerArea` (on Review step only) | Emphasized |
| Save as Draft | `footerArea` | Transparent |
| Cancel | `footerArea` (with "Discard this <object>?" popover) | Transparent |
| Navigate to previous step | Progress bar (built-in) | — |

### Initial Page

| Action | Goes in | Style |
|---|---|---|
| Open (typed ID) | Button right of the input, or Enter key | Emphasized |
| Value help (F4) | Icon inside the input | — |

### Flexible Column Layout

| Action | Goes in | Style |
|---|---|---|
| Navigate into an item | Row / list-item click | — |
| Close a column | Icon button in column's title bar | Transparent |
| Toggle full-screen | Icon button in column's title bar | Transparent |
| Resize columns | Drag separator | — |
| Back | Browser back or Shell back button | — |

## Part 6 — Visual patterns (verified v2)

### Title `actionsBar` — transparent Toolbar with ToolbarButtons

```tsx
<ObjectPageTitle
  actionsBar={
    <Toolbar design="Transparent">
      <ToolbarButton design="Transparent" icon={shareIcon} text="Share" />
      <ToolbarButton design="Transparent" icon={pdfIcon} text="Export PDF" />
    </Toolbar>
  }
/>
```

### FooterArea — Floating-footer Bar (save + cancel)

```tsx
<ObjectPage
  footerArea={
    <Bar
      design="FloatingFooter"
      endContent={
        <>
          <Button design="Emphasized">Save</Button>       {/* primary */}
          <Button design="Transparent">Cancel</Button>    {/* negative path */}
        </>
      }
    />
  }
>
  …
</ObjectPage>
```

### FooterArea — semantic approval (Accept / Reject)

```tsx
<ObjectPage
  footerArea={
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
  }
>
  …
</ObjectPage>
```

**Note**: do not add `Emphasized` here. Semantic styling owns the primary slot.

### Table toolbar with selection-dependent action

```tsx
<Toolbar>
  <Title level="H4">Orders ({rows.length})</Title>
  <ToolbarSeparator />
  <ToolbarSpacer />
  <ToolbarButton design="Default" icon={addIcon} text="Create" />
  <ToolbarButton
    design="Transparent"
    icon={deleteIcon}
    text={selected.length > 0 ? `Delete (${selected.length})` : 'Delete'}
    disabled={selected.length === 0}
  />
  <ToolbarButton design="Transparent" icon={excelIcon} text="Export" />
</Toolbar>
<AnalyticalTable … />
```

Note: Create uses `Default` (ghost) if the page already has a footer primary. If the page has no footer, Create can be `Emphasized`.

### Inline row actions — terminal Cell

```tsx
{
  Header: 'Actions',
  accessor: '.',
  disableFilters: true,
  disableSortBy: true,
  disableGroupBy: true,
  disableResizing: true,
  width: 200,
  Cell: ({ row }) => (
    <div style={{ display: 'flex', gap: '0.25rem' }} onClick={(e) => e.stopPropagation()}>
      <Button design="Positive" icon={acceptIcon} onClick={() => approve(row.original.id)}>
        Approve
      </Button>
      <Button design="Negative" icon={declineIcon} onClick={() => reject(row.original.id)}>
        Reject
      </Button>
    </div>
  ),
}
```

## Part 7 — Quick "is this in the right place?" test

Ask in this order:

1. **Does the action commit / change state for the whole page?** → `footerArea`.
2. **Does it support the whole page but not commit?** → title `actionsBar`.
3. **Is it app-wide, not page-specific?** → ShellBar.
4. **Does it operate on the entire table or on selected rows?** → table toolbar.
5. **Is it a single-row operation?** → row inline action.
6. **Is it a column concern (sort/filter/group/resize)?** → column header popover.
7. **Is it scoped to one subsection?** → subsection `actions` slot.

If the action fits in two places, pick the more specific one. If it doesn't fit any, you probably don't need that action on this page.

## Part 8 — Anti-patterns (the most common mistakes)

- **Two Emphasized buttons on one page.** Demote one to Default or Transparent.
- **Emphasized Save next to Positive Accept.** Pick one: semantic OR emphasized for this workflow.
- **Save / Cancel in the title actionsBar.** Move to footer.
- **Create in the page title instead of the table toolbar.** Move to table toolbar.
- **Page-level Edit AND inline row-level Edit.** Pick one per workflow.
- **Settings in both ShellBar and page title.** ShellBar wins (app scope).
- **"Sort" button in the table toolbar** when the column popover already handles sort. Remove the button.
- **Footer actions that vary by tab.** Footer is page-global; tab-scoped actions go in the tab's content area.
- **Icon-only button without a tooltip.** Screen readers and new users can't parse it.
- **Emphasized button inside a menu / split button.** Menu/split buttons should always be Transparent or Default.
- **Using string names for icons** (`icon="add"`). Must be module imports (`import addIcon from '@ui5/webcomponents-icons/dist/add.js'; icon={addIcon}`).
- **Letting the primary action go into overflow.** Mark it `NeverOverflow`.
