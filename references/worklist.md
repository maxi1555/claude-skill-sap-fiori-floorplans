# Worklist

The Fiori floorplan for a **bounded, scoped queue of items the user must process**. Think inbox. Think "my open approvals", "unassigned tickets", "items due today", "rejected invoices awaiting my review".

> **Also consult** `references/global-patterns.md` (empty states, loading, action placement) and `references/action-placement.md` (semantic Accept/Reject styling, inline row-action rules). Worklists live or die by good empty states and correct inline-action styling.

The defining difference from a List Report:

- **List Report**: "find items in a large dataset and act on them." → Wide filter options, search required.
- **Worklist**: "process items already scoped to me." → Narrow scope, predefined views instead of free-form filters.

In SAP Fiori elements v4 the distinction is encoded directly: a Worklist **is** a List Report with the FilterBar disabled. That's also how you think about it in a freestyle app.

## When to use

- Items are **scoped ahead of time** (by user, role, or context). The user doesn't need to search a universe.
- User's job is to **work through the list**, eventually emptying it (or keeping it low).
- Common dispositions (Approve/Reject, Assign/Defer) should be **one click from the list**, not buried behind a detail page.
- Item count is **bounded** — not thousands of rows. If it is thousands, you probably want a List Report.

## When NOT to use

- Large dataset with open-ended filtering → **List Report**.
- Need for chart+KPI analysis → **Analytical List Page**.
- Cross-app aggregation → **Overview Page**.

## What you get for free

- **Predefined views** (tabs or segmented button) with **item counters**: "All (12)", "Overdue (3)".
- **Empty states** via `IllustratedMessage` — an important worklist surface because users *want* it empty.
- **Inline row actions** for quick dispositions.
- **Sortable, groupable, growable table** via `AnalyticalTable` or a simple responsive `Table`.
- Optional simple search; **no full FilterBar needed**.

## Fiori design rules

- **No FilterBar.** A FilterBar signals "this is a List Report". If the user asks for one, they probably want a List Report.
- **Predefined views** cover the minor filtering need. Counters on every view are strongly recommended: "Pending (7)", "Overdue (2)", "Due today (5)".
- **Sort by urgency or due date** by default.
- **Inline row actions** for the 1–2 most common dispositions (Approve, Reject). Rarer actions go in an overflow `<Menu>` or "More Actions" action sheet per row, or in the table toolbar after selecting rows.
- **Celebrate emptiness.** If there are zero pending items, show an `IllustratedMessage` with `name="SuccessScreen"` or `"NoTasks"` and a warm message, not a blank table.
- **Do not require a click-through to detail for the primary action.** The worklist's whole point is that Approve/Reject should be one click.
- **Do not paginate across many pages.** A worklist is bounded; if it has thousands of rows, you've modelled the wrong thing.
- **Sticky** the table toolbar and column headers, same as List Report.
- Title/variant management: optional. Most worklists have a fixed title ("My Open Approvals"). Variant management only if users really want saved sort/column arrangements.

## Action placement in a Worklist

- **Title `actionsBar`** (page-global): usually empty or just `Share`.
- **Table toolbar**: actions on selected rows — `Approve Selected`, `Reject Selected`, `Assign to…`, `Export`.
- **Row inline actions**: the one or two most common per-row dispositions.
- **`footerArea`** (rare): only if a finalizing batch action needs confirmation.

## v2 component API cheat sheet

**`TabContainer` + `Tab`:**
- `TabContainer.onTabSelect` — event with `e.detail.tab` being the clicked `Tab` element.
- `Tab.text` — display label.
- `Tab.additionalText` — the counter pill shown next to the text.
- `Tab.selected` — controlled selection.
- `Tab.design` — `"Default"`, `"Positive"`, `"Critical"`, `"Negative"`, `"Neutral"` (for semantic tinting).

**`IllustratedMessage`:**
- `name` — use the `IllustratedMessageType` enum: `NoData`, `NoEntries`, `NoSearchResults`, `NoTasks`, `SuccessScreen`, `UnableToLoad`, `NoFilterResults`, etc.
- `titleText`, `subtitleText` — texts.
- `children` — optional action buttons.
- `size` — `"Auto"`, `"Base"`, `"Dot"`, `"Spot"`, `"Dialog"`, `"Scene"`.

## Production scaffold

```tsx
'use client';

import {
  DynamicPage,
  DynamicPageTitle,
  Title,
  Label,
  TabContainer,
  Tab,
  AnalyticalTable,
  AnalyticalTableSelectionMode,
  Toolbar,
  ToolbarButton,
  ToolbarSpacer,
  ToolbarSeparator,
  Button,
  ButtonDesign,
  ObjectStatus,
  IllustratedMessage,
  IllustratedMessageType,
  BusyIndicator,
} from '@ui5/webcomponents-react';

import acceptIcon from '@ui5/webcomponents-icons/dist/accept.js';
import declineIcon from '@ui5/webcomponents-icons/dist/decline.js';
import assignIcon from '@ui5/webcomponents-icons/dist/employee.js';
import shareIcon from '@ui5/webcomponents-icons/dist/share.js';

import { useMemo, useState } from 'react';

type Approval = {
  id: string;
  title: string;
  requester: string;
  amount: number;
  currency: string;
  dueDate: string;            // ISO date
  status: 'Pending' | 'Overdue';
  priority: 'Low' | 'Medium' | 'High';
};

type View = 'all' | 'pending' | 'overdue' | 'due-today';

type Props = {
  data: Approval[];
  loading?: boolean;
  onApprove: (ids: string[]) => Promise<void>;
  onReject: (ids: string[]) => Promise<void>;
  onAssign: (ids: string[]) => void;
  onOpen: (id: string) => void;
};

const isDueToday = (isoDate: string) => {
  const today = new Date().toISOString().slice(0, 10);
  return isoDate.slice(0, 10) === today;
};

const urgencyState = (a: Approval) =>
  a.status === 'Overdue' ? 'Negative' : a.priority === 'High' ? 'Critical' : 'Information';

export default function ApprovalsWorklist({
  data,
  loading,
  onApprove,
  onReject,
  onAssign,
  onOpen,
}: Props) {
  const [view, setView] = useState<View>('all');
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());

  const counts = useMemo(
    () => ({
      all: data.length,
      pending: data.filter((a) => a.status === 'Pending').length,
      overdue: data.filter((a) => a.status === 'Overdue').length,
      dueToday: data.filter((a) => isDueToday(a.dueDate)).length,
    }),
    [data],
  );

  const rows = useMemo(() => {
    const base =
      view === 'pending'
        ? data.filter((a) => a.status === 'Pending')
        : view === 'overdue'
          ? data.filter((a) => a.status === 'Overdue')
          : view === 'due-today'
            ? data.filter((a) => isDueToday(a.dueDate))
            : data;
    // Fiori worklist default sort: by urgency then due date
    return [...base].sort((a, b) => {
      if (a.status !== b.status) return a.status === 'Overdue' ? -1 : 1;
      return a.dueDate.localeCompare(b.dueDate);
    });
  }, [data, view]);

  const selectedRows = useMemo(
    () => rows.filter((r) => selectedIds.has(r.id)),
    [rows, selectedIds],
  );

  const columns = useMemo(
    () => [
      { Header: 'ID', accessor: 'id', width: 110 },
      { Header: 'Title', accessor: 'title' },
      { Header: 'Requester', accessor: 'requester', width: 150 },
      {
        Header: 'Amount',
        accessor: 'amount',
        hAlign: 'End',
        width: 130,
        Cell: ({ row }: { row: { original: Approval } }) =>
          `${row.original.amount.toLocaleString()} ${row.original.currency}`,
      },
      { Header: 'Due', accessor: 'dueDate', width: 130 },
      {
        Header: 'Status',
        accessor: 'status',
        width: 130,
        Cell: ({ row }: { row: { original: Approval } }) => (
          <ObjectStatus state={urgencyState(row.original)}>{row.original.status}</ObjectStatus>
        ),
      },
      {
        Header: 'Actions',
        accessor: '.',
        disableFilters: true,
        disableSortBy: true,
        disableGroupBy: true,
        width: 220,
        Cell: ({ row }: { row: { original: Approval } }) => (
          <div style={{ display: 'flex', gap: '0.25rem' }} onClick={(e) => e.stopPropagation()}>
            <Button
              design={ButtonDesign.Positive}
              icon={acceptIcon}
              onClick={() => onApprove([row.original.id])}
            >
              Approve
            </Button>
            <Button
              design={ButtonDesign.Negative}
              icon={declineIcon}
              onClick={() => onReject([row.original.id])}
            >
              Reject
            </Button>
          </div>
        ),
      },
    ],
    [onApprove, onReject],
  );

  if (loading) {
    return <BusyIndicator active size="L" style={{ margin: '2rem auto', display: 'block' }} />;
  }

  return (
    <DynamicPage
      titleArea={
        <DynamicPageTitle
          header={<Title>My Approvals</Title>}
          subHeader={<Label>{counts.pending} pending · {counts.overdue} overdue</Label>}
          actionsBar={
            <Toolbar design="Transparent">
              <ToolbarButton design="Transparent" icon={shareIcon} text="Share" />
            </Toolbar>
          }
        />
      }
    >
      <TabContainer
        onTabSelect={(e) => {
          const v = (e.detail.tab as HTMLElement).dataset.view as View;
          if (v) setView(v);
        }}
      >
        <Tab
          data-view="all"
          text="All"
          additionalText={String(counts.all)}
          selected={view === 'all'}
        />
        <Tab
          data-view="pending"
          text="Pending"
          additionalText={String(counts.pending)}
          selected={view === 'pending'}
        />
        <Tab
          data-view="overdue"
          text="Overdue"
          additionalText={String(counts.overdue)}
          selected={view === 'overdue'}
          design="Negative"
        />
        <Tab
          data-view="due-today"
          text="Due today"
          additionalText={String(counts.dueToday)}
          selected={view === 'due-today'}
          design="Critical"
        />
      </TabContainer>

      <Toolbar>
        <Title level="H4">
          {view === 'all'
            ? 'All approvals'
            : view === 'pending'
              ? 'Pending approvals'
              : view === 'overdue'
                ? 'Overdue approvals'
                : 'Due today'}{' '}
          ({rows.length})
        </Title>
        <ToolbarSeparator />
        <ToolbarSpacer />
        <ToolbarButton
          design="Positive"
          icon={acceptIcon}
          text={`Approve${selectedRows.length ? ` (${selectedRows.length})` : ''}`}
          disabled={selectedRows.length === 0}
          onClick={() => onApprove(selectedRows.map((r) => r.id))}
        />
        <ToolbarButton
          design="Negative"
          icon={declineIcon}
          text={`Reject${selectedRows.length ? ` (${selectedRows.length})` : ''}`}
          disabled={selectedRows.length === 0}
          onClick={() => onReject(selectedRows.map((r) => r.id))}
        />
        <ToolbarButton
          design="Transparent"
          icon={assignIcon}
          text="Assign to…"
          disabled={selectedRows.length === 0}
          onClick={() => onAssign(selectedRows.map((r) => r.id))}
        />
      </Toolbar>

      {rows.length === 0 ? (
        <IllustratedMessage
          name={
            view === 'all' && counts.all === 0
              ? IllustratedMessageType.SuccessScreen
              : IllustratedMessageType.NoTasks
          }
          titleText={
            view === 'all' && counts.all === 0
              ? 'All caught up!'
              : 'Nothing in this view'
          }
          subtitleText={
            view === 'all' && counts.all === 0
              ? 'You have no approvals waiting for you.'
              : 'Switch to another view to see more.'
          }
        />
      ) : (
        <AnalyticalTable
          data={rows}
          columns={columns}
          visibleRows={Math.min(rows.length, 15)}
          selectionMode={AnalyticalTableSelectionMode.Multiple}
          selectionBehavior="RowSelector"
          onRowClick={(e) => onOpen((e.detail.row.original as Approval).id)}
          onRowSelect={(e) => {
            const next = new Set<string>();
            e.detail.selectedFlatRows?.forEach((r: { original: Approval }) =>
              next.add(r.original.id),
            );
            setSelectedIds(next);
          }}
        />
      )}
    </DynamicPage>
  );
}
```

### Notes on the scaffold

- **Default sort by urgency then due date** is baked in. Don't leave users to figure out what to do first.
- **Row-level actions are inline**. The `stopPropagation` inside the Actions cell prevents a click-on-button from bubbling to `onRowClick` (which would open the detail page).
- **Selection + toolbar batch actions** for users who want to process multiple at once.
- **Empty state** switches between `SuccessScreen` ("All caught up!" when the entire dataset is empty) and `NoTasks` ("Nothing in this view" when a specific view is empty).
- **Tab `design` on semantic views** — `"Negative"` for Overdue, `"Critical"` for Due today — gives a subtle color cue without being loud.

## Fiori text templates for Worklist

Worklist empty states are particularly important — an empty queue is often a "win" state (all caught up) rather than a sad state.

**Empty state — queue is empty because the user cleared it (the "win"):**
- Title: `All caught up!` or `No open approvals.`
- Description: `Nice work. Check back later for new [approvals / requests].`
- Illustration: `SuccessScreen` or `NoTasks`

**Empty state — filter/tab currently shows nothing but other tabs have items:**
- Title: `No [high-priority items] in this view`
- Description: `Try switching to another view.`
- Illustration: `NoFilterResults`

**Success MessageToast after inline actions:**
- After single Approve: `[Order 12345] approved.`
- After single Reject: `[Order 12345] rejected.`
- After bulk Approve Selected: `3 orders approved.`
- After bulk Reject Selected: `3 orders rejected.`

**Partial-success (some succeeded, some failed):**
- `MessageBox` with type `Warning`.
- Title: `2 of 3 orders approved`
- Body: `1 order could not be approved. See details.` — embed a `MessageView`.

**Tab counter labels:**
- `All (48)` / `Open (12)` / `Overdue (3)` — use `additionalText` on the `Tab` for the count.
- Update counters in real time after actions.

## Anti-patterns

- Adding a FilterBar for "just one filter". → If there's a need for filtering beyond predefined views, use a List Report.
- Making the user click into detail to approve. → One-click inline.
- Empty-table blank state. → `IllustratedMessage` always.
- Pagination across dozens of pages. → Bound the scope at the query level.
- Row counter in parentheses but with a dash: "All — 12". → Fiori uses parentheses: "All (12)".

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents`
- `@ui5/webcomponents-fiori` (optional — pulls in IllustratedMessage assets)
- `@ui5/webcomponents-icons`
