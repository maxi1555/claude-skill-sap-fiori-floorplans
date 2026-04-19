# Tables — choosing between AnalyticalTable, Table, and List

Tables are where most Fiori apps spend their visual budget. Picking the right component matters: the wrong pick makes your List Report feel wrong, your Object Page section sluggish, or your responsive behavior broken.

## TL;DR decision tree

1. **Chart + grid scenario, or need for sorting / grouping / filtering / virtualization / tree rows / lots of rows (>100)** → `AnalyticalTable`.
2. **Simple tabular data, responsive pop-in on small screens, up to a few dozen rows, need for selection + line-item navigation** → `Table` (the v2 `ui5-table`).
3. **Vertical list of items, each a one-liner or small card, typical "list of emails", "list of contacts"** → `List` with `ListItemStandard` / `ListItemCustom`.

Edge cases:

- **Tree data** → `AnalyticalTable` with `isTreeTable`.
- **Inside an FCL column + draft-enabled List Report** → `Table` (AnalyticalTable and tree tables not supported by Fiori elements here).
- **Inside an OVP card** → `List` (for list cards) or a mini `Table` (for table cards, max 3 columns).

## AnalyticalTable

The flagship desktop-centric data grid. Virtualized rows, rich column headers, grouping, sorting, filtering, selection, tree-mode, infinite scroll, sub-rows. It's what a Fiori List Report uses for large data.

### Key props

- `data` — row objects.
- `columns` — column definitions (see below).
- `selectionMode` — `"None"` | `"Single"` | `"Multiple"` (enum `AnalyticalTableSelectionMode`).
- `selectionBehavior` — `"Row"` | `"RowOnly"` | `"RowSelector"` (enum `AnalyticalTableSelectionBehavior`). Controls whether clicking a row selects it or just triggers click.
- `visibleRows` — number of rows to display (default 15).
- `visibleRowCountMode` — `"Fixed"` | `"Auto"` | `"AutoWithEmptyRows"` | `"Interactive"` (enum `AnalyticalTableVisibleRowCountMode`).
- `minRows` — minimum row slots even with less data (default 5).
- `rowHeight` — fixed row height in px.
- `groupable`, `sortable`, `filterable` — booleans.
- `isTreeTable` — enable tree rendering.
- `subRowsKey` — key in data used to find children (default `'subRows'`).
- `scaleWidthMode` — `"Default"` | `"Smart"` | `"Grow"` (enum `AnalyticalTableScaleWidthMode`). How columns distribute available width.
- `retainColumnWidth` — keep user-resized columns stable across prop changes.
- `infiniteScroll`, `infiniteScrollThreshold`, `onLoadMore` — for backend-paged datasets.
- `loading`, `loadingDelay` — show a busy overlay.
- `NoDataComponent` — custom empty state (prefer `IllustratedMessage`).
- `noDataText` — plain-text empty fallback.
- `showOverlay` — blocks interaction.
- `withRowHighlight`, `withNavigationHighlight` — visual cues for status / navigability.
- `markNavigatedRow` — function returning a boolean; the marked row gets a distinct left border.
- `alwaysShowSubComponent`, `renderRowSubComponent`, `subComponentsBehavior` — expandable sub-components per row.
- `tableInstance` — ref to the internal react-table instance.
- `reactTableOptions` — escape hatch for react-table config.
- `tableHooks` — custom plugin hooks.

### Column definition

Each column in `columns` is an object with at minimum:

```ts
{
  Header: 'Customer',    // string or ReactNode
  accessor: 'customer',  // key in the row object, or an accessor function
}
```

Optional:

- `id` — stable column id (required if `accessor` is a function).
- `Cell` — render function: `({ cell, row, value }) => ReactNode`.
- `width` — fixed width in px.
- `minWidth`, `maxWidth` — responsive bounds.
- `hAlign` — `"Start"` | `"Center"` | `"End"`.
- `vAlign` — `"Top"` | `"Middle"` | `"Bottom"`.
- `disableFilters`, `disableSortBy`, `disableGroupBy`, `disableResizing`, `disableDragAndDrop` — per-column opt-outs.
- `autoResizable` — double-click header separator auto-fits width.
- `canReorder` — allow drag-reordering.
- `filter` — custom filter function.
- `Filter` — custom filter UI (rendered in column popover).

### Events

- `onRowClick` — `e.detail.row.original` is your row data.
- `onRowSelect` — `e.detail.selectedFlatRows` is an array of row instances; `e.detail.row.original` is the last-clicked row.
- `onRowExpandChange` — tree/sub-component expansion.
- `onSort` — sorting changed.
- `onGroup` — grouping changed.
- `onFilter` — filter applied (in v2.10+).
- `onColumnsReorder` — reordering via drag.
- `onLoadMore` — fired when the scroll approaches `infiniteScrollThreshold`.
- `onAutoResize` — column auto-resized.
- `onTableScroll` — raw scroll event.

### Fiori rules for AnalyticalTable

- **Sticky header** is built-in.
- **Selection**: if the table supports row navigation *and* multi-select, use `selectionBehavior="RowSelector"` — the checkbox selects; clicking the row navigates. Otherwise `"Row"` — clicking the row toggles selection.
- **Line-item navigation**: enable via `withNavigationHighlight`, and provide a chevron visual via a terminal Cell column or via styling. Navigation happens on row click (when selectionBehavior is `"RowSelector"`).
- **Empty / no-data state**: pass an `<IllustratedMessage>` via `NoDataComponent`. Never leave a blank white box.
- **Column settings**: accessed via the per-column header popover. Users can resize, sort, filter, group, pin.
- **Variant management for table layout**: Fiori allows a separate table-level variant (column visibility, order, widths). Rare — only offer this if users have a *strong* need to switch between table layouts independently of filter variants. Otherwise use the page-level `VariantManagement`.

## Table (ui5-table v2)

Simpler, responsive table. Supports column-to-row "pop-in" on narrow screens — each row becomes a mini card. Use when you don't need grouping / virtualization.

### Key props

- `headerRow` — slot taking a `<TableHeaderRow>` with `<TableHeaderCell>` children.
- `rowActionCount` — number of row-action slots on each row.
- `features` — slot for advanced features (`<TableSelection>`, `<TableGrowing>`).
- `overflowMode` — `"Scroll"` | `"Popin"`. Popin is the Fiori-native responsive behavior.
- `noDataText` — string fallback; prefer a Cell-level IllustratedMessage via a `noData` slot if your version supports it.

### TableRow / TableCell / TableHeaderCell

- `TableHeaderCell.width` — fixed width.
- `TableHeaderCell.minWidth` — responsive minimum.
- `TableHeaderCell.popinText` — label shown when the column pops into the row.
- `TableCell` children are your rendered content.

### When to use Table vs AnalyticalTable

Choose `Table` when:
- Small dataset (up to ~40 rows).
- Responsive pop-in behavior is important.
- You don't need column resize / grouping / tree / virtualization.
- You're inside an FCL column where tree/analytical tables aren't supported.

Choose `AnalyticalTable` when:
- Medium to large dataset.
- Users expect grouping, sorting, tree, drill-down, column settings.
- You need virtualization for performance.

## List

Vertical list of items, optimized for "read one, tap one". The best choice for:

- Email-inbox-like UIs.
- List cards on an Overview Page.
- Simple selection lists.
- Master columns in pre-FCL split-screen patterns (though FCL has replaced most of these).

### Key props

- `selectionMode` — `"None"` | `"Single"` | `"Multiple"` | `"Delete"`.
- `growing` — `"None"` | `"Button"` | `"Scroll"` — how more items appear.
- `mode` — legacy; use `selectionMode` in v2.
- `headerText` — optional list header.
- `header` — slot for a custom list header.
- `noDataText` — plain-text fallback.
- `onItemClick`, `onSelectionChange`, `onLoadMore`.

### Item types

- `ListItemStandard` — title + optional description + optional additionalText + icon/avatar.
- `ListItemCustom` — arbitrary children for complex layouts.
- `ListItemGroup` — group header.

## Fiori action patterns inside tables

- **Header-row actions** (affect the whole collection): Create, Delete Selected, Export → table toolbar (a `<Toolbar>` above the table, or the `extension` slot of AnalyticalTable).
- **Row-selector-based actions** (affect selected subset): put them in the same table toolbar; disable when nothing is selected; append a count.
- **Inline row actions** (affect one row): render in a terminal Cell column; use up to 2 buttons; overflow rest into a menu. For AnalyticalTable, wrap the inline-actions container with `onClick={(e) => e.stopPropagation()}` so it doesn't trigger `onRowClick`.
- **Column settings** (sort, filter, group by this column): accessed from the column header popover. Don't add a separate Sort/Group/Filter button in the toolbar; use the column popover.
- **Line-item navigation** (go to Object Page for this row): clicking the row. Provide a chevron visual cue.

## Common recipes

### AnalyticalTable with inline actions and navigation

```tsx
import { AnalyticalTable, AnalyticalTableSelectionMode, Button, ButtonDesign } from '@ui5/webcomponents-react';
import editIcon from '@ui5/webcomponents-icons/dist/edit.js';
import deleteIcon from '@ui5/webcomponents-icons/dist/delete.js';

<AnalyticalTable
  data={rows}
  columns={[
    { Header: 'ID', accessor: 'id', width: 120 },
    { Header: 'Name', accessor: 'name' },
    {
      Header: 'Actions',
      accessor: '.',
      disableFilters: true,
      disableSortBy: true,
      disableGroupBy: true,
      disableResizing: true,
      width: 140,
      Cell: ({ row }) => (
        <div style={{ display: 'flex', gap: '0.25rem' }} onClick={(e) => e.stopPropagation()}>
          <Button icon={editIcon} design={ButtonDesign.Transparent} onClick={() => onEdit(row.original.id)} />
          <Button icon={deleteIcon} design={ButtonDesign.Transparent} onClick={() => onDelete(row.original.id)} />
        </div>
      ),
    },
  ]}
  selectionMode={AnalyticalTableSelectionMode.None}
  onRowClick={(e) => onOpen(e.detail.row.original.id)}
  visibleRows={15}
/>
```

### AnalyticalTable with infinite scroll

```tsx
<AnalyticalTable
  data={rows}
  columns={columns}
  infiniteScroll
  infiniteScrollThreshold={20}
  loading={isLoadingMore}
  onLoadMore={() => fetchNextPage()}
/>
```

### AnalyticalTable as tree

```tsx
<AnalyticalTable
  data={rowsWithSubRows}       // each row may have `subRows: [...]`
  columns={columns}
  isTreeTable
  subRowsKey="subRows"
  reactTableOptions={{ initialState: { expanded: {} } }}
/>
```

### Simple responsive Table

```tsx
import {
  Table, TableHeaderRow, TableHeaderCell, TableRow, TableCell,
} from '@ui5/webcomponents-react';

<Table
  overflowMode="Popin"
  headerRow={
    <TableHeaderRow>
      <TableHeaderCell width="140px">Order</TableHeaderCell>
      <TableHeaderCell>Customer</TableHeaderCell>
      <TableHeaderCell popinText="Status">Status</TableHeaderCell>
    </TableHeaderRow>
  }
>
  {rows.map((r) => (
    <TableRow key={r.id}>
      <TableCell>{r.id}</TableCell>
      <TableCell>{r.customer}</TableCell>
      <TableCell>{r.status}</TableCell>
    </TableRow>
  ))}
</Table>
```

### List with growing

```tsx
import { List, ListItemStandard } from '@ui5/webcomponents-react';

<List
  headerText="Recent activity"
  growing="Scroll"
  onLoadMore={() => fetchNextPage()}
  onItemClick={(e) => open((e.detail.item as HTMLElement).dataset.id!)}
>
  {items.map((i) => (
    <ListItemStandard
      key={i.id}
      data-id={i.id}
      description={i.summary}
      additionalText={i.date}
    >
      {i.title}
    </ListItemStandard>
  ))}
</List>
```

## Multi-selection and mass-edit patterns

Multi-selection is everywhere in Fiori — from List Reports through Worklists to Object Page item tables. Two related patterns need to be done right.

### Multi-selection UX

- **Use `selectionMode="Multiple"`** on AnalyticalTable, or `TableSelection` with `mode="Multiple"` on the v2 Table.
- **Use `selectionBehavior="RowSelector"`** when rows are also clickable for navigation — the checkbox selects; clicking the row opens. Otherwise `"Row"` — clicking the row toggles selection.
- **Show selection count in the table toolbar** — update the title to `Orders (12 of 48 selected)` or similar.
- **Disable selection-dependent actions when nothing is selected.** Enable them as soon as ≥1 row is selected. Append the count to destructive labels: `Delete` → `Delete (3)`.
- **Select-all** is built in via the header checkbox. Clicking it selects only the currently loaded rows; if you have server-side pagination, provide an explicit "Select all N rows" link after selection, and handle server-side bulk ops accordingly.
- **Range-select** via `Shift+Click` on the header checkboxes works out of the box.
- **Keyboard**: `Space` toggles the focused row's selection; `Ctrl+A` selects all visible rows.

### Partial-success pattern

When a bulk action runs server-side and some rows succeed while others fail, the Fiori rule is:

- **Never fail silently.** Don't assume the whole bulk succeeded just because the request returned 200.
- **Show a summary MessageBox** with type `Warning`.
  - Title: `2 of 3 orders approved`
  - Body: `1 order could not be approved. See details.`
  - Embed a `MessageView` listing per-row errors and optionally a "Retry failed" action.
- **After dismissal**, keep the failed rows selected in the table so the user can fix them and retry.
- **Do NOT use a single MessageToast** — a toast will auto-dismiss and the user loses the per-row failure reasons.

### Mass-edit pattern (editing a field across many rows at once)

The Fiori mass-edit pattern is specifically for **editing the same fields on many selected rows via a single dialog** — not for replacing multi-row inline edit.

How to build it:

1. User selects N rows in the table.
2. Table toolbar shows a `Mass Edit` action (enabled only with ≥1 selection).
3. Clicking opens a **`Dialog`** titled `Edit 3 customers` (plural/singular based on count).
4. The dialog contains a `Form` with the mass-editable fields. Each field has **three states**:
   - **Leave unchanged** (default): the field's label shows "<Keep existing values>" or similar; no change applied.
   - **Clear value**: explicitly null out the field (use a checkbox "Clear value").
   - **Set new value**: user enters a new value, which applies to all selected rows.
5. Footer has `Save` (emphasized) + `Cancel` (transparent).
6. On save, run the bulk update. Show a MessageToast for full success or a MessageBox with MessageView for partial success.

### Mass-edit scaffold

```tsx
'use client';

import {
  Dialog, Bar, Button, Form, FormGroup, FormItem,
  Label, Input, Select, Option, CheckBox,
} from '@ui5/webcomponents-react';
import { useState } from 'react';

type MassEditValues = {
  status?: { mode: 'keep' | 'clear' | 'set'; value?: string };
  owner?: { mode: 'keep' | 'clear' | 'set'; value?: string };
};

export function MassEditDialog({
  open,
  selectedCount,
  onCancel,
  onSave,
}: {
  open: boolean;
  selectedCount: number;
  onCancel: () => void;
  onSave: (values: MassEditValues) => void;
}) {
  const [values, setValues] = useState<MassEditValues>({});
  const updateField = (key: keyof MassEditValues, next: MassEditValues[keyof MassEditValues]) =>
    setValues((v) => ({ ...v, [key]: next }));

  return (
    <Dialog
      open={open}
      headerText={`Edit ${selectedCount} ${selectedCount === 1 ? 'customer' : 'customers'}`}
      footer={
        <Bar
          endContent={
            <>
              <Button design="Emphasized" onClick={() => onSave(values)}>Save</Button>
              <Button design="Transparent" onClick={onCancel}>Cancel</Button>
            </>
          }
        />
      }
      onClose={onCancel}
    >
      <Form labelSpan="S12 M4 L4 XL4" layout="S1 M1 L1 XL1" style={{ minWidth: '28rem' }}>
        <FormGroup headerText="Status">
          <FormItem labelContent={<Label>Status</Label>}>
            <Select
              value={values.status?.mode ?? 'keep'}
              onChange={(e) =>
                updateField('status', {
                  mode: (e.detail.selectedOption as HTMLElement).dataset.mode as any,
                })
              }
            >
              <Option data-mode="keep">&lt;Keep existing values&gt;</Option>
              <Option data-mode="clear">&lt;Clear&gt;</Option>
              <Option data-mode="set">Active</Option>
              <Option data-mode="set">Inactive</Option>
            </Select>
          </FormItem>
          <FormItem labelContent={<Label>Owner</Label>}>
            <div style={{ display: 'flex', flexDirection: 'column', gap: '0.25rem', width: '100%' }}>
              <CheckBox
                text="Keep existing values"
                checked={(values.owner?.mode ?? 'keep') === 'keep'}
                onChange={() => updateField('owner', { mode: 'keep' })}
              />
              <CheckBox
                text="Clear value"
                checked={values.owner?.mode === 'clear'}
                onChange={() => updateField('owner', { mode: 'clear' })}
              />
              <Input
                placeholder="Or enter new owner"
                value={values.owner?.mode === 'set' ? (values.owner?.value ?? '') : ''}
                onInput={(e) =>
                  updateField('owner', {
                    mode: 'set',
                    value: (e.target as HTMLInputElement).value,
                  })
                }
              />
            </div>
          </FormItem>
        </FormGroup>
      </Form>
    </Dialog>
  );
}
```

### When NOT to use mass-edit

- If the field varies meaningfully per row (e.g. "set the delivery date per row" — that's inline edit, not mass edit).
- If only one row is selected — just open the Object Page in edit mode for that one row.
- If the mass edit would cascade complex side effects per row — build a proper bulk workflow with preview, confirmation, and a result page.

## Anti-patterns

- Using `AnalyticalTable` for a 5-row key-value display. → Use a `Form` or a plain `Table`.
- Using `Table` for 10,000 rows. → It won't virtualize; switch to `AnalyticalTable`.
- Using `List` for tabular data that has meaningful columns. → A table communicates structure; a list flattens it.
- Putting sort/group/filter buttons in the table toolbar when the column header already has them. → Duplicate UI, users get confused.
- Omitting `stopPropagation` on inline action buttons. → Row click fires alongside the action.
- `AnalyticalTable` with `selectionMode="Multiple"` but no `selectionBehavior` setting. → Clicking a cell may accidentally toggle selection; set `"RowSelector"` if that's the UX you want.
- Not providing a `NoDataComponent`. → Users see a blank box.
- Using the compat `Table` (v1) in new code. → Migrate to v2 `ui5-table` — the compat version has known accessibility gaps and won't receive features.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents` (for `Table`, `List`)
- `@ui5/webcomponents-fiori` (for `AnalyticalTable`, `IllustratedMessage`)
- `@ui5/webcomponents-icons`
