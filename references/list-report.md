# List Report

The Fiori floorplan for **finding and acting on items in a large dataset**. If the user needs to search, filter, sort, group, and then do something with the results, this is what they want. Think "Manage Orders", "All Tickets", "Suppliers".

> **Also consult** `references/global-patterns.md` (especially action placement, empty states, loading states) and `references/action-placement.md` (button-type taxonomy) — the rules there are not repeated here but apply to every List Report.

## When to use

- Dataset is potentially **large** (tens to thousands of rows).
- User does not know the specific items they want — they **find** items via filters and search.
- User takes **actions** on the results (navigate into an Object Page, approve, export, etc.).
- Dataset has **multiple meaningful filter dimensions** (status, date range, owner, category, …).

## When NOT to use

- Dataset is **predefined and scoped to the user** (their open approvals, items assigned to them) → **Worklist** (List Report with no FilterBar is a Worklist, per SAP Fiori elements).
- Primary job is **analysis** (charts, KPIs, drilldown) → **Analytical List Page**.
- Single object → **Object Page** or **Initial Page**.

## What you get for free

- **FilterBar** (mandatory) with adaptable filters, search field, and "Go"/"Adapt Filters".
- **Variant management** — users save named filter-sets and share them (public/private variants).
- **Predefined views** (All, Open, Assigned) via `TabContainer`, `SegmentedButton`, or icon tab bar above the table.
- **Table ↔ chart toggle** (optional).
- **Sticky headers** — table toolbar, column headers, and any icon tab bar stay pinned while scrolling.
- **Line-item navigation** via chevron → Object Page for the selected row.
- **Inline row actions** and **multi-row actions** (via selection + actions in the table toolbar).
- **Export** (to spreadsheet) is a standard entry in the table toolbar.

## Fiori design rules

### Layout

- The List Report is a **full screen** `DynamicPage` floorplan. It can also be placed in the first column of a Flexible Column Layout.
- Structure:
  - `titleArea`: the page title (or `VariantManagement` as title), non-finalizing global actions (Share).
  - `headerArea`: the FilterBar (this is the collapsible header content).
  - **Content area**: one or multiple tables, charts, or a chart+table combination. Can use an `IconTabContainer` above the table to switch between multiple table views (e.g. "All (115)", "Hardware (60)", "Software (55)").
  - Optional `footerArea`: a messaging button and finalizing actions for batch operations. Often empty for pure browsing.

### FilterBar is mandatory

- The FilterBar defines what data the table shows. The FilterBar state and the table state must always be consistent.
- **Implement search separately from the filter bar.** Search is executed against the back end in combination with the set filters. The search reduces the filtered result set to items containing the search string.
- **Deactivate the table's own filter option** when the FilterBar is present. Two filter mechanisms confuse users and can contradict each other.
- **Deactivate the table's own search** if you put a search field in the FilterBar.

### Filter bar initial state

- **Expanded view**: reveals the filters upfront; more discoverable but visually heavier.
- **Collapsed view**: simpler, but users may miss the filter capability.
- Default is expanded. If your app has a strongly curated default variant and most users won't adjust, collapsed is acceptable — but never hide the FilterBar entirely.
- When collapsed, the `DynamicPageTitle` subtitle should show "Filtered By (x): filter1, filter2, …" so users see what's applied. Show up to 5 filters; append "…" if more. If none, show "Not filtered".

### Variant management

- Optional but recommended. When used, the variant replaces the title at the top of the title area.
- Variants save **filter settings, selected tabs, all table states, and all chart states**.
- Users can mark a variant as **default** (loaded when the app opens).
- Users can mark a variant as **public** or **private**.
- "Execute on Select" setting: if on, selecting a variant applies it immediately. If off, the user can review/modify before executing (delayed update). Only expose this option when the FilterBar is in manual update mode.
- If you don't use variant management, show a plain descriptive title of the current view in the title area.

### Predefined views

- For 2–3 view shortcuts (e.g. "All", "Open", "Closed"), use a `SegmentedButton` above the table, left-aligned.
- For 4+ views or when views have their own actions, use a `TabContainer` / icon tab bar.
- Include counters on views: "Open (12)", "Closed (403)".
- If using views, you usually don't also need a title above the table.

### Table specifics inside a List Report

- Pick the table component based on dataset size and needs (see `references/tables.md`):
  - Up to ~40 simple rows: `Table` (responsive, supports pop-in on small screens).
  - Any larger, any need for sorting/grouping/filtering/virtualization: `AnalyticalTable`.
  - Tree data: `AnalyticalTable` with `isTreeTable`.
- **Sticky** the table toolbar and column headers.
- Enable **"scroll to load" / growing** behavior for responsive tables; use `visibleRowCountMode="Fixed"` or `"Auto"` for AnalyticalTable with a reasonable `visibleRows`.
- Line-item navigation via **chevron indicator** on each navigable row.
- Row actions: use inline action cells for frequent operations; put global table actions (Create, Delete Selected, Export) in the table toolbar (`extension` slot of AnalyticalTable or a separate `Toolbar` above the table).

### Action placement in the List Report

- **Title `actionsBar`** (global, non-finalizing): `Share`, `Export`.
- **Table toolbar** (operates on table contents/selection): `Create`, `Delete` (for selected rows), `Copy`, column/sort/group settings, view switch, search.
- **Row inline actions**: `Edit`, status-changing actions. Keep to 1–2 inline; overflow the rest.
- **`footerArea`** (rare for List Report): finalizing actions after batch operation. Usually absent.
- Disable selection-dependent actions when nothing is selected.

### Empty states

- Filter yielded no results → `IllustratedMessage` with `name="NoSearchResults"`, message "No data found. Try adjusting the filter settings."
- Legit no data in the system → `IllustratedMessage` with `name="NoEntries"` and a "Create" action if applicable.

## v2 component API cheat sheet

**`FilterBar` props (v2 — these are the ones that changed):**
- `search` — slot taking the search `<Input>`.
- `hideToggleFiltersButton` — boolean.
- `hideFilterConfiguration` — boolean (hide the "Adapt Filters" button).
- `showGoOnFB` / `showResetButton` / `showClearOnFB` / `showRestoreOnFB` — toggle visible buttons.
- `onGo` — now a standard ToolbarButton `onClick` event. The old `detail.elements` / `detail.filters` / `detail.search` / `detail.nativeDetail` have been removed; read values directly from your state.
- `onRestore` — now a plain callback with a payload object (no longer a CustomEvent).
- `onFiltersDialogOpen` — target is now a ToolbarButton.
- `hideToolbar` — hides the top toolbar.
- `portalContainer` — **removed** in v2.

**`FilterGroupItem` props (v2 — these are the ones that changed):**
- `hidden` (replaces v1 `visible`, logic inverted).
- `hiddenInFilterBar` (replaces v1 `visibleInFilterBar`, logic inverted).
- `filterKey` (replaces v1 `orderId`).
- `label` — visible label for this filter.
- `required` — boolean for filters that must be set.

**`AnalyticalTable` key props:**
- `data` — array of row objects.
- `columns` — array of column definitions `{ Header, accessor, Cell?, hAlign?, disableFilters?, disableSortBy?, width? }`.
- `selectionMode` — `"None"`, `"Single"`, `"Multiple"` (enum: `AnalyticalTableSelectionMode`).
- `selectionBehavior` — `"Row"`, `"RowOnly"`, `"RowSelector"`.
- `visibleRows` — number (default 15).
- `visibleRowCountMode` — `"Fixed"`, `"Auto"`, `"AutoWithEmptyRows"`, `"Interactive"`.
- `minRows` — number (default 5).
- `groupable`, `sortable`, `filterable` — booleans.
- `infiniteScroll`, `infiniteScrollThreshold` (default 20), `onLoadMore` — for backend-paged tables.
- `isTreeTable`, `subRowsKey` (default `'subRows'`) — for tree data.
- `onRowClick` — `e.detail.row.original` for row data.
- `onRowSelect` — selection change.
- `NoDataComponent` — custom empty state (prefer `IllustratedMessage`).
- `loading` — boolean; use with a small delay via `loadingDelay`.

## Production scaffold

```tsx
'use client';

import {
  DynamicPage,
  DynamicPageTitle,
  DynamicPageHeader,
  Title,
  Label,
  Breadcrumbs,
  BreadcrumbsItem,
  FilterBar,
  FilterGroupItem,
  Input,
  DatePicker,
  Select,
  Option,
  AnalyticalTable,
  AnalyticalTableSelectionMode,
  Toolbar,
  ToolbarButton,
  ToolbarSpacer,
  ToolbarSeparator,
  ObjectStatus,
  SegmentedButton,
  SegmentedButtonItem,
  IllustratedMessage,
  IllustratedMessageType,
  BusyIndicator,
} from '@ui5/webcomponents-react';

import addIcon from '@ui5/webcomponents-icons/dist/add.js';
import excelAttachmentIcon from '@ui5/webcomponents-icons/dist/excel-attachment.js';
import shareIcon from '@ui5/webcomponents-icons/dist/share.js';
import deleteIcon from '@ui5/webcomponents-icons/dist/delete.js';

import { useCallback, useMemo, useState } from 'react';

type Order = {
  id: string;
  customer: string;
  status: 'New' | 'InProgress' | 'Shipped' | 'Closed';
  total: number;
  createdOn: string;
  country: string;
};

type View = 'all' | 'open' | 'closed';

type Props = {
  data: Order[];
  loading?: boolean;
  onCreate: () => void;
  onExport: (rows: Order[]) => void;
  onOpenOrder: (id: string) => void;
  onDeleteMany: (ids: string[]) => Promise<void>;
};

const statusState = (s: Order['status']) =>
  s === 'Closed' ? 'Positive' : s === 'Shipped' ? 'Information' : s === 'New' ? 'Critical' : 'Negative';

export default function OrdersListReport({
  data,
  loading,
  onCreate,
  onExport,
  onOpenOrder,
  onDeleteMany,
}: Props) {
  const [search, setSearch] = useState('');
  const [status, setStatus] = useState<string>('');
  const [country, setCountry] = useState<string>('');
  const [from, setFrom] = useState<string>('');
  const [to, setTo] = useState<string>('');
  const [view, setView] = useState<View>('all');
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());

  const filtered = useMemo(() => {
    let rows = data;
    if (view === 'open') rows = rows.filter((r) => r.status === 'New' || r.status === 'InProgress');
    else if (view === 'closed') rows = rows.filter((r) => r.status === 'Closed');

    return rows.filter(
      (r) =>
        (status === '' || r.status === status) &&
        (country === '' || r.country === country) &&
        (from === '' || r.createdOn >= from) &&
        (to === '' || r.createdOn <= to) &&
        (search === '' || r.customer.toLowerCase().includes(search.toLowerCase()) || r.id.toLowerCase().includes(search.toLowerCase())),
    );
  }, [data, status, country, from, to, search, view]);

  const counts = useMemo(() => {
    const all = data.length;
    const open = data.filter((r) => r.status === 'New' || r.status === 'InProgress').length;
    const closed = data.filter((r) => r.status === 'Closed').length;
    return { all, open, closed };
  }, [data]);

  const columns = useMemo(
    () => [
      { Header: 'Order', accessor: 'id', width: 120 },
      { Header: 'Customer', accessor: 'customer' },
      {
        Header: 'Status',
        accessor: 'status',
        Cell: ({ value }: { value: Order['status'] }) => (
          <ObjectStatus state={statusState(value)}>{value}</ObjectStatus>
        ),
        width: 140,
      },
      { Header: 'Country', accessor: 'country', width: 120 },
      { Header: 'Created', accessor: 'createdOn', width: 140 },
      { Header: 'Total', accessor: 'total', hAlign: 'End', width: 120 },
    ],
    [],
  );

  const applyFilters = useCallback(() => {
    // Called when user hits the FilterBar "Go". Since we filter reactively from state,
    // there's no backend round-trip to do here. If you switch to server-side filtering,
    // this is where you'd fire your fetch with the current filter state.
  }, []);

  const resetFilters = useCallback(() => {
    setStatus('');
    setCountry('');
    setFrom('');
    setTo('');
    setSearch('');
  }, []);

  const selectedRows = useMemo(
    () => filtered.filter((r) => selectedIds.has(r.id)),
    [filtered, selectedIds],
  );

  if (loading) return <BusyIndicator active size="L" style={{ margin: '2rem auto', display: 'block' }} />;

  return (
    <DynamicPage
      titleArea={
        <DynamicPageTitle
          breadcrumbs={
            <Breadcrumbs>
              <BreadcrumbsItem>Sales</BreadcrumbsItem>
              <BreadcrumbsItem>Orders</BreadcrumbsItem>
            </Breadcrumbs>
          }
          header={<Title>Orders</Title>}
          subHeader={
            // "Filtered By (x): ..." subtitle shown when header is collapsed
            (() => {
              const parts: string[] = [];
              if (status) parts.push(`Status: ${status}`);
              if (country) parts.push(`Country: ${country}`);
              if (from || to) parts.push(`Date: ${from || '—'} → ${to || '—'}`);
              if (search) parts.push(`Search: "${search}"`);
              return <Label>{parts.length ? `Filtered By (${parts.length}): ${parts.slice(0, 5).join(', ')}${parts.length > 5 ? '…' : ''}` : 'Not filtered'}</Label>;
            })()
          }
          actionsBar={
            <Toolbar design="Transparent">
              <ToolbarButton design="Transparent" icon={shareIcon} text="Share" />
            </Toolbar>
          }
        />
      }
      headerArea={
        <DynamicPageHeader>
          <FilterBar
            search={
              <Input
                placeholder="Search orders"
                value={search}
                onInput={(e) => setSearch((e.target as HTMLInputElement).value)}
              />
            }
            showGoOnFB
            showClearOnFB
            onGo={applyFilters}
            onClear={resetFilters}
          >
            <FilterGroupItem label="Status" filterKey="status">
              <Select
                onChange={(e) =>
                  setStatus((e.detail.selectedOption as HTMLElement).dataset.value ?? '')
                }
              >
                <Option data-value="" selected={status === ''}>All</Option>
                <Option data-value="New" selected={status === 'New'}>New</Option>
                <Option data-value="InProgress" selected={status === 'InProgress'}>In Progress</Option>
                <Option data-value="Shipped" selected={status === 'Shipped'}>Shipped</Option>
                <Option data-value="Closed" selected={status === 'Closed'}>Closed</Option>
              </Select>
            </FilterGroupItem>

            <FilterGroupItem label="Country" filterKey="country">
              <Input
                value={country}
                placeholder="e.g. CH"
                onInput={(e) => setCountry((e.target as HTMLInputElement).value)}
              />
            </FilterGroupItem>

            <FilterGroupItem label="From" filterKey="from">
              <DatePicker
                value={from}
                onChange={(e) => setFrom(e.detail.value)}
              />
            </FilterGroupItem>

            <FilterGroupItem label="To" filterKey="to">
              <DatePicker
                value={to}
                onChange={(e) => setTo(e.detail.value)}
              />
            </FilterGroupItem>
          </FilterBar>
        </DynamicPageHeader>
      }
    >
      {/* Predefined view switcher */}
      <div style={{ padding: 'var(--sapContent_Space_S) 0' }}>
        <SegmentedButton
          onSelectionChange={(e) => {
            const v = (e.detail.selectedItems[0] as HTMLElement).dataset.view as View;
            if (v) setView(v);
          }}
        >
          <SegmentedButtonItem data-view="all" selected={view === 'all'}>
            All ({counts.all})
          </SegmentedButtonItem>
          <SegmentedButtonItem data-view="open" selected={view === 'open'}>
            Open ({counts.open})
          </SegmentedButtonItem>
          <SegmentedButtonItem data-view="closed" selected={view === 'closed'}>
            Closed ({counts.closed})
          </SegmentedButtonItem>
        </SegmentedButton>
      </div>

      {/* Table toolbar — table-level actions go here, NOT in the page titleArea */}
      <Toolbar>
        <Title level="H4">Orders ({filtered.length})</Title>
        <ToolbarSeparator />
        <ToolbarSpacer />
        <ToolbarButton design="Emphasized" icon={addIcon} text="Create" onClick={onCreate} />
        <ToolbarButton
          design="Transparent"
          icon={deleteIcon}
          text={`Delete${selectedRows.length ? ` (${selectedRows.length})` : ''}`}
          disabled={selectedRows.length === 0}
          onClick={() => onDeleteMany(selectedRows.map((r) => r.id))}
        />
        <ToolbarButton
          design="Transparent"
          icon={excelAttachmentIcon}
          text="Export"
          onClick={() => onExport(filtered)}
        />
      </Toolbar>

      {filtered.length === 0 ? (
        <IllustratedMessage
          name={IllustratedMessageType.NoSearchResults}
          titleText="No orders found"
          subtitleText="Try adjusting the filters or clearing the search."
        />
      ) : (
        <AnalyticalTable
          data={filtered}
          columns={columns}
          selectionMode={AnalyticalTableSelectionMode.Multiple}
          selectionBehavior="Row"
          visibleRows={15}
          sortable
          groupable
          onRowClick={(e) => {
            const order = e.detail.row.original as Order;
            onOpenOrder(order.id);
          }}
          onRowSelect={(e) => {
            const nextIds = new Set<string>();
            // e.detail.selectedFlatRows gives all currently-selected rows
            e.detail.selectedFlatRows?.forEach((r: { original: Order }) => nextIds.add(r.original.id));
            setSelectedIds(nextIds);
          }}
        />
      )}
    </DynamicPage>
  );
}
```

### Notes on the scaffold

- **FilterBar Go/Clear:** here they reset local state because filtering is in-memory. If you switch to server-side filtering, fire your fetch in `onGo`. The `onGo` event in v2 is a plain ToolbarButton click — it does not carry filter state in `e.detail`; read state from your React state.
- **Subheader "Filtered By (x):" pattern** is implemented inline per the Fiori rule. When the user collapses the header, they still see what's applied.
- **Predefined views** use `SegmentedButton` with counters in the labels. For more than 3 views, switch to `TabContainer` with `Tab` + `additionalText={count}`.
- **Table toolbar** is separate from the page `titleArea`. Create / Delete / Export live here because they operate on the table, not the page.
- **Empty state** uses `IllustratedMessage` with the canonical `NoSearchResults` illustration.
- **Chevron navigation**: `AnalyticalTable` does not render a chevron by default; to get the Fiori chevron indicator, set `withNavigationHighlight` on the table and return a `withNavigation` marker from the row, or render a custom end-of-row Cell with a navigation icon.

## Fiori text templates for List Report

Use these empty-state text patterns — they match Fiori elements defaults. Adapt the business noun.

**Empty state when no filters applied and no data exists:**
- Title: `No results found`
- Description: `Start by providing your search or filter criteria.`
- Illustration: `NoData` or `NoSearchResults`

**Empty state when filters applied and no data matches:**
- Title: `No [suppliers / orders / tickets] found`
- Description: `Try changing your filter criteria.` or `Try adjusting your filter settings.`
- Illustration: `NoFilterResults`
- Optional action button: `Reset filters` (Emphasized)

**Empty state when multi-view mode (SegmentedButton) is active and no data matches:**
- Title: `No results found`
- Description: `Try changing the view or filter criteria.`
- Illustration: `NoFilterResults`

**Empty state on first use / when data hasn't been added yet:**
- Title: `No [suppliers] yet`
- Description: `When there are, you'll find them here.`
- Illustration: `NoData`
- Optional action button: `Create [supplier]` (Default — the table toolbar already has the page-level primary)

**Success MessageToast after actions from the toolbar:**
- After Create: `[Customer name] created.`
- After Delete Selected (single): `[Customer name] deleted.`
- After Delete Selected (multiple): `3 customers deleted.`
- After Approve Selected: `2 orders approved.` (partial success needs a different pattern — see below)

**Partial-success feedback (some rows succeeded, some failed):**
- Use a `MessageBox` with type `Warning`.
- Title: `2 of 3 [orders] approved`
- Body: `1 [order] could not be approved. See details below.` — embed a `MessageView` listing per-row errors.

**Delete confirmation dialog (MessageBox) for multi-select:**
- `Delete 3 customers?` — buttons `Delete` (emphasized) / `Cancel`.
- Warn if conflicts: `Some of the selected [customers] were edited by other users. Delete anyway?`

## Anti-patterns

- Using a List Report for a user's own task queue (bounded, scoped) → Worklist, not List Report.
- Hiding the FilterBar to "simplify" → if you don't need a FilterBar, it's not a List Report.
- Putting `Create` in the page `actionsBar` → belongs in the table toolbar.
- Putting row actions in `footerArea` → belongs inline or in the table toolbar.
- Not implementing search separately from the filter bar → per Fiori rules, search must be implemented in every List Report.
- Using the `visible`/`visibleInFilterBar` props on `FilterGroupItem` → v1; use `hidden`/`hiddenInFilterBar` in v2.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents`
- `@ui5/webcomponents-fiori` (for FilterBar, VariantManagement)
- `@ui5/webcomponents-icons`
