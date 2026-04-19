# Analytical List Page (ALP)

The Fiori floorplan for **extracting insight from transactional data**. List Report + analytical layer on top. Users see the aggregated picture (chart + KPIs), filter visually, drill down into segments, and act on the underlying rows.

> **Also consult** `references/global-patterns.md` (empty states, loading states, action placement). The ALP's KPI tiles and drilldown have specific empty-and-loading considerations that the global file covers.

It's the most complex floorplan in Fiori. Don't reach for it unless analysis is genuinely the user's primary job.

## When to use

- Primary job is **analysis**: "why are returns up this quarter?", "which suppliers missed SLA?", "where are we bleeding margin?"
- User needs to **drill down** into aggregates (slice and dice) and eventually **act** on specific items.
- You have a **clear global KPI** whose value is affected by the filters applied.
- You want **visual filters** (each filter is itself a mini chart) or at minimum a KPI-led page header.

## When NOT to use

- User just needs to list items with some filters → **List Report** (even if it has a chart toggle).
- User needs a high-level role dashboard across multiple business objects → **Overview Page**.
- User's job is to process items one by one → **Worklist**.
- Dataset has no meaningful aggregate / KPI angle → Any analytical framing is forced.

## What you get for free

- Everything a List Report has (VariantManagement, filters, table, row actions).
- A **KPI header** area reflecting the current filter impact.
- **Filter view switch** between a regular FilterBar and a **visual filter bar** (each filter rendered as a chart the user clicks into).
- **Content view switch**: chart-only, table-only, or hybrid (both, with the table showing drill-down context of the chart).
- **Drill-down on chart** → table filters to the clicked segment. Chart selection and table selection stay in sync.

## Fiori design rules

### Layout

- `DynamicPage` with:
  - `titleArea` holding `VariantManagement` (as page variant), KPI tags and/or KPI cards, optional global actions.
  - `headerArea` holding the FilterBar (visual or compact) and an "Adapt Filters (x)" link.
  - Content holding the chart, the table, or both (hybrid view).

### The KPI

- One primary KPI belongs in the title area. Its value updates as filters change. The KPI shows trend (delta from previous period) and semantic color (Positive/Critical/Negative).
- Multiple small KPI tags are OK. Multiple full KPI cards tips the page toward "Overview Page" — don't overdo it.

### Filter modes

- **Compact** (default): classic FilterBar, row of dropdowns and inputs.
- **Visual**: each filter is a small chart (e.g. a micro bar chart of "sales by region"). The user filters by clicking a chart segment. Takes significantly more space and more visual energy.
- Provide a toggle between compact and visual modes. Default compact unless your users are explicitly comfortable with visual analytics.

### View switch

- **Chart + table** (hybrid): chart on top, table below, kept in sync.
- **Chart only**: for when the user wants maximum chart real estate.
- **Table only**: for users who've found the item and just want the rows.
- Toggle with `SegmentedButton` above the content, or with the Fiori chart-container toggle if you're using one.

### Syncing chart and table

- When the user clicks a chart segment, filter the table to the rows behind that segment.
- When the user selects rows, consider highlighting the corresponding chart segment.
- Always keep aggregation consistent: if the chart is aggregated by month, the table should have a `Month` column the user understands, not a row per transaction with a confusing chart breakdown.

### Page variant

- `VariantManagement` is the page variant — it captures:
  - Current filter view switch (visual vs compact).
  - Current filter values.
  - Current content view (chart/table/hybrid).
  - Chart/table configurations (measures, dimensions, sort, group).
  - Chart drill-down state.
  - Selected rows indicator.

### Desktop-centric table caveat

If the content table is a grid/analytical/tree table (which is typical on ALP), the dynamic page header **does not snap on scroll** because the table owns all vertical space. Instead, provide an explicit **"Hide Filters" / "Show Filters"** button in the content toolbar to collapse/expand the filter area. Same rule applies whenever the page content is a chart container or a desktop-centric table.

## v2 component API cheat sheet

- `FilterBar` — same API as in List Report; use `collapsed` initially for ALP with dense charts.
- Charts are in `@ui5/webcomponents-react-charts`:
  - `BarChart`, `ColumnChart`, `LineChart`, `ComposedChart`, `PieChart`, `DonutChart`, `ScatterChart`, `BulletChart`, `RadarChart`, `RadialChart`, `ColumnChartWithTrend`.
  - API: `dataset`, `dimensions=[{accessor}]`, `measures=[{accessor, label}]`, plus interactions like `onClick`, `onLegendClick`, `onDataPointClick`.
- `AnalyticalTable` — use `groupable`, `sortable`, `filterable`. For drilldown, wire the chart `onClick` to set filter state that the table consumes.
- `SegmentedButton` + `SegmentedButtonItem` — for the content view switch.
- `ObjectStatus` (large) or `NumericSideIndicator` — for KPI display in the title area.

## Production scaffold

```tsx
'use client';

import {
  DynamicPage,
  DynamicPageTitle,
  DynamicPageHeader,
  Title,
  Label,
  ObjectStatus,
  ValueState,
  FilterBar,
  FilterGroupItem,
  Select,
  Option,
  DatePicker,
  SegmentedButton,
  SegmentedButtonItem,
  AnalyticalTable,
  AnalyticalTableSelectionMode,
  Toolbar,
  ToolbarButton,
  ToolbarSpacer,
  IllustratedMessage,
  IllustratedMessageType,
  VariantManagement,
  VariantItem,
} from '@ui5/webcomponents-react';
import { ColumnChart } from '@ui5/webcomponents-react-charts';

import excelAttachmentIcon from '@ui5/webcomponents-icons/dist/excel-attachment.js';
import barChartIcon from '@ui5/webcomponents-icons/dist/horizontal-bar-chart.js';
import tableViewIcon from '@ui5/webcomponents-icons/dist/table-view.js';
import combineIcon from '@ui5/webcomponents-icons/dist/combine.js';

import { useMemo, useState } from 'react';

type Sale = {
  id: string;
  region: string;
  product: string;
  month: string;        // "YYYY-MM"
  revenue: number;
  cost: number;
};

type ContentView = 'hybrid' | 'chart' | 'table';

type Props = {
  data: Sale[];
  loading?: boolean;
  target: number;       // target revenue, used for KPI delta
  onExport: (rows: Sale[]) => void;
  onOpenSale: (id: string) => void;
};

export default function SalesAnalyticalListPage({ data, target, onExport, onOpenSale }: Props) {
  const [region, setRegion] = useState('');
  const [product, setProduct] = useState('');
  const [from, setFrom] = useState('');
  const [to, setTo] = useState('');
  const [view, setView] = useState<ContentView>('hybrid');
  // When the user clicks a chart column, we filter the table to that month
  const [drillMonth, setDrillMonth] = useState<string | null>(null);

  const filtered = useMemo(
    () =>
      data.filter(
        (s) =>
          (region === '' || s.region === region) &&
          (product === '' || s.product === product) &&
          (from === '' || s.month >= from) &&
          (to === '' || s.month <= to),
      ),
    [data, region, product, from, to],
  );

  const tableRows = useMemo(
    () => (drillMonth ? filtered.filter((s) => s.month === drillMonth) : filtered),
    [filtered, drillMonth],
  );

  const revenueTotal = useMemo(
    () => filtered.reduce((sum, s) => sum + s.revenue, 0),
    [filtered],
  );
  const marginPct = useMemo(() => {
    const rev = filtered.reduce((s, x) => s + x.revenue, 0);
    const cost = filtered.reduce((s, x) => s + x.cost, 0);
    return rev === 0 ? 0 : Math.round(((rev - cost) / rev) * 100);
  }, [filtered]);

  const vsTarget = revenueTotal - target;
  const kpiState: ValueState =
    revenueTotal >= target ? ValueState.Positive : revenueTotal >= target * 0.9 ? ValueState.Critical : ValueState.Negative;

  const chartData = useMemo(() => {
    const byMonth = new Map<string, number>();
    filtered.forEach((s) => byMonth.set(s.month, (byMonth.get(s.month) ?? 0) + s.revenue));
    return [...byMonth.entries()]
      .sort(([a], [b]) => a.localeCompare(b))
      .map(([month, revenue]) => ({ month, revenue }));
  }, [filtered]);

  const columns = useMemo(
    () => [
      { Header: 'ID', accessor: 'id', width: 110 },
      { Header: 'Region', accessor: 'region', width: 120 },
      { Header: 'Product', accessor: 'product' },
      { Header: 'Month', accessor: 'month', width: 120 },
      { Header: 'Revenue', accessor: 'revenue', hAlign: 'End', width: 130 },
      { Header: 'Cost', accessor: 'cost', hAlign: 'End', width: 130 },
    ],
    [],
  );

  return (
    <DynamicPage
      titleArea={
        <DynamicPageTitle
          // Page-variant as title: users save filter+view combos as named variants.
          header={
            <VariantManagement>
              <VariantItem selected>Standard (Default)</VariantItem>
              <VariantItem>EMEA only</VariantItem>
              <VariantItem>Last 90 days</VariantItem>
            </VariantManagement>
          }
          subHeader={<Label>Revenue performance by region and product</Label>}
          actionsBar={
            <Toolbar design="Transparent">
              <ToolbarButton
                design="Transparent"
                icon={excelAttachmentIcon}
                text="Export"
                onClick={() => onExport(tableRows)}
              />
            </Toolbar>
          }
        >
          {/* KPI tags — lightweight, update with filters */}
          <ObjectStatus state={kpiState} large>
            €{revenueTotal.toLocaleString()}
          </ObjectStatus>
          <ObjectStatus state={kpiState} large>
            {vsTarget >= 0 ? '▲' : '▼'} €{Math.abs(vsTarget).toLocaleString()} vs target
          </ObjectStatus>
          <ObjectStatus
            state={marginPct >= 30 ? ValueState.Positive : marginPct >= 20 ? ValueState.Critical : ValueState.Negative}
            large
          >
            {marginPct}% margin
          </ObjectStatus>
        </DynamicPageTitle>
      }
      headerArea={
        <DynamicPageHeader>
          <FilterBar
            showGoOnFB
            showClearOnFB
            onClear={() => {
              setRegion('');
              setProduct('');
              setFrom('');
              setTo('');
              setDrillMonth(null);
            }}
          >
            <FilterGroupItem label="Region" filterKey="region">
              <Select
                onChange={(e) =>
                  setRegion((e.detail.selectedOption as HTMLElement).dataset.value ?? '')
                }
              >
                <Option data-value="" selected={region === ''}>All</Option>
                <Option data-value="EMEA" selected={region === 'EMEA'}>EMEA</Option>
                <Option data-value="AMER" selected={region === 'AMER'}>AMER</Option>
                <Option data-value="APAC" selected={region === 'APAC'}>APAC</Option>
              </Select>
            </FilterGroupItem>

            <FilterGroupItem label="Product" filterKey="product">
              <Select
                onChange={(e) =>
                  setProduct((e.detail.selectedOption as HTMLElement).dataset.value ?? '')
                }
              >
                <Option data-value="" selected={product === ''}>All</Option>
                <Option data-value="Basic" selected={product === 'Basic'}>Basic</Option>
                <Option data-value="Pro" selected={product === 'Pro'}>Pro</Option>
                <Option data-value="Enterprise" selected={product === 'Enterprise'}>Enterprise</Option>
              </Select>
            </FilterGroupItem>

            <FilterGroupItem label="From" filterKey="from">
              <DatePicker value={from} onChange={(e) => setFrom(e.detail.value)} />
            </FilterGroupItem>

            <FilterGroupItem label="To" filterKey="to">
              <DatePicker value={to} onChange={(e) => setTo(e.detail.value)} />
            </FilterGroupItem>
          </FilterBar>
        </DynamicPageHeader>
      }
    >
      {/* Content view switch — table / chart / hybrid */}
      <Toolbar>
        <SegmentedButton
          onSelectionChange={(e) => {
            const v = (e.detail.selectedItems[0] as HTMLElement).dataset.view as ContentView;
            if (v) setView(v);
          }}
        >
          <SegmentedButtonItem data-view="hybrid" icon={combineIcon} selected={view === 'hybrid'}>
            Chart & Table
          </SegmentedButtonItem>
          <SegmentedButtonItem data-view="chart" icon={barChartIcon} selected={view === 'chart'}>
            Chart
          </SegmentedButtonItem>
          <SegmentedButtonItem data-view="table" icon={tableViewIcon} selected={view === 'table'}>
            Table
          </SegmentedButtonItem>
        </SegmentedButton>
        <ToolbarSpacer />
        {drillMonth && (
          <ToolbarButton
            design="Transparent"
            text={`Clear drill-down: ${drillMonth}`}
            onClick={() => setDrillMonth(null)}
          />
        )}
      </Toolbar>

      {(view === 'hybrid' || view === 'chart') && (
        <div style={{ height: view === 'chart' ? '480px' : '320px', margin: '1rem 0' }}>
          {chartData.length === 0 ? (
            <IllustratedMessage
              name={IllustratedMessageType.NoData}
              titleText="No data for the current filters"
            />
          ) : (
            <ColumnChart
              dataset={chartData}
              dimensions={[{ accessor: 'month' }]}
              measures={[{ accessor: 'revenue', label: 'Revenue' }]}
              onClick={(e) => {
                // drill-down: clicking a column filters the table to that month
                const row = (e as any)?.payload as { month: string } | undefined;
                if (row?.month) setDrillMonth(row.month);
              }}
            />
          )}
        </div>
      )}

      {(view === 'hybrid' || view === 'table') && (
        tableRows.length === 0 ? (
          <IllustratedMessage
            name={IllustratedMessageType.NoSearchResults}
            titleText="No rows for the current selection"
            subtitleText={
              drillMonth
                ? 'Try clearing the chart drill-down.'
                : 'Try adjusting the filters.'
            }
          />
        ) : (
          <AnalyticalTable
            data={tableRows}
            columns={columns}
            visibleRows={view === 'table' ? 20 : 10}
            selectionMode={AnalyticalTableSelectionMode.Single}
            sortable
            groupable
            onRowClick={(e) => onOpenSale((e.detail.row.original as Sale).id)}
          />
        )
      )}
    </DynamicPage>
  );
}
```

### Notes on the scaffold

- **KPIs are plain `ObjectStatus` with `large`** — it's the simplest way to display a big colored value in the title area. You can also use `NumericSideIndicator` or compose a custom KPI card; which you pick depends on how many KPIs and how much visual weight you want.
- **Drill-down on the chart**: clicking a column sets `drillMonth`, which filters the table. There's an explicit "Clear drill-down" button in the toolbar so the user doesn't get stuck.
- **Visual filter bar** is not shown here — it's significantly more code and `@ui5/webcomponents-react` doesn't ship a packaged "VisualFilterBar" equivalent to SAPUI5's ALP template. For a true visual FB, embed small charts inside `FilterGroupItem` children and wire their clicks back to the FilterBar state.
- **`VariantManagement`** is shown in the title. If you don't want saved views, replace with a plain `<Title>`.
- **Export** is a title-bar action because it operates on the whole filtered set, not on a selection.

## Anti-patterns

- Using an ALP "because it looks impressive". → If the user doesn't have a clear analytical workflow, the extra complexity of ALP hurts.
- Multiple big KPI cards in the title. → Use small `ObjectStatus` tags; if you want big KPI cards, you're building an Overview Page.
- Chart and table disagreeing (e.g., chart aggregated, table unfiltered). → They must be synced.
- Leaving the user stuck with a drill-down they can't clear. → Always expose a "clear drill-down" affordance when drilled in.
- Using dynamic header snap with a full-height `AnalyticalTable`. → Provide an explicit Hide/Show Filters toggle; the header won't snap-on-scroll here.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents-react-charts`
- `@ui5/webcomponents`
- `@ui5/webcomponents-fiori`
- `@ui5/webcomponents-icons`
