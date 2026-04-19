# Overview Page (OVP)

The Fiori floorplan for a **role-based dashboard**. One page, many cards, each card pulling from a different business object or app, filter-bar at the top affects all cards. Designed for decision-making at a glance, not for detailed work.

> **Also consult** `references/global-patterns.md` (navigation — OVP card click patterns) and `references/action-placement.md` (card footer actions). Card empty states and card-level loading follow the rules in global-patterns §5 and §6.

This is the floorplan people most often reach for and most often misuse. Before scaffolding, confirm the user really needs an OVP — not a List Report with a chart, not an ALP, not a plain page of cards.

## When to use

- User plays a **specific role** (buyer, plant manager, controller) and wants a single landing page for that role.
- Page aggregates data from **at least two** different business objects or apps.
- Content is **read-and-navigate**. Every card either shows data or links into the app where action can be taken.
- Users can filter globally (e.g. "this quarter", "EMEA") and all cards respond.

## When NOT to use

- **Single-object focus** → Object Page.
- **Cross-analytical drilldown on a single dataset** → Analytical List Page.
- **Processing a queue of items** → Worklist.
- **Finding items in a single dataset** → List Report.
- **You want OVP-like cards inside a list report column** → You want a set of Cards, not an OVP. OVP must be standalone.

## Hard rules (Fiori policy)

- **OVP must always be a standalone application.** Cannot be embedded in a Flexible Column Layout. Cannot be placed inside another floorplan.
- **No editable fields.** Cards are read-and-navigate only.
- At least **two business objects/apps** are aggregated. Otherwise the OVP is just "a fancy List Report".
- Cards are **clickable and navigate to the owning app's detail/list page** (List Report or Object Page).

## What you get for free

- Consistent card framework with hover state, interactive header, header-area + content-area + (quick-view) footer-area.
- Automatic responsive card grid (flows to 1–5 columns based on screen size).
- Drag-and-drop reordering of cards (at the framework level in Fiori elements; in freestyle apps you implement it yourself).
- **Manage Cards** dialog (in Fiori elements) to hide/show cards — also a freestyle feature.
- Global FilterBar at the top; cards subscribe.

## Card type taxonomy

OVP cards aren't freeform — Fiori has a specific typology. Know the types so you can pick the right one for each metric.

### Single-object cards

Card about **one** object or data point.
- **Quick view card** — small profile of a single object (e.g., "My Top Supplier: Acme Corp"). The only card type that may have **footer actions**.
- **Analytical card** — one chart visualizing a metric (KPI header + chart in content area). No footer actions; interaction areas inside the chart.

### Object group cards

Card showing a **list of items grouped by a common criterion**.
- **List card** — vertical list of items (uses `List` + `ListItemStandard`). Header navigates to full list in the parent app. Row click navigates to the item.
- **Table card** — 3-column table of items. Same navigation pattern as list card.
- **Bar chart list card** — list of items with a mini bar chart per row (compare values visually).

### Link list cards

Navigation shortcuts: collection of links to targets (apps, objects, external URLs). Like a "quick actions" card.

### Stack cards

Aggregate a set of single-object cards sharing a topic. Click opens up to 20 cards in an "object stream" — a carousel-ish view.

### Picking a type

- "Show me a KPI" → analytical card.
- "Show me top N items" → list card or table card.
- "Show me top N with comparative values" → bar chart list card.
- "Show me navigation shortcuts" → link list card.
- "Show me one specific object (my biggest customer)" → quick view card.
- "Show me a set of related things I can flip through" → stack card.

A typical OVP mixes 4–8 cards across 2–4 types. Keep **card sizes visually consistent** (Fiori has 1×1 and 2×1 grid slots — don't mix five tiny and one giant).

## Fiori design rules

- **Each card is clickable.** Header area navigates to the owning app (usually a List Report for "see all").
- **Do not edit in cards.** If you feel a card needs an action, it's almost always just a navigation — clicking navigates to the app where editing happens. The only cards allowed to have footer actions are quick view cards.
- **Line items are selectable** and navigate to detail within the parent app.
- **Consistent text** formatting across all cards (UI text guidelines apply).
- **Do not load thousands of rows into a card**. Show top N (usually 3–6 for list cards; 3 for table cards); the "Show All" is the card header click.
- **Aggregate list items sparingly.** Rows should usually be real individual items, not pre-aggregated summaries (unless aggregation is the explicit purpose).
- **Global filter bar at the top** filters all cards. Cards must react within a reasonable time.
- **Share / export** via a menu above the filter bar (Share as Tile, Email, etc.) — optional.

## Layout

- Responsive grid: up to 5 columns on XL, fewer on smaller sizes.
- Use CSS Grid: `grid-template-columns: repeat(auto-fill, minmax(320px, 1fr))` gives Fiori-appropriate rhythm without any layout library.
- Card `width` varies by card type. Analytical and quick view cards are typically 1×1; table/list cards can be 2×1 for readability.

## v2 component API cheat sheet

- `Card` — wraps card content.
  - `header` — takes a `<CardHeader>` (replaces v1 `titleText`, `subtitleText`, `status`, etc.).
- `CardHeader` — the clickable header area.
  - `titleText`, `subtitleText` — required title, optional subtitle.
  - `status` — small pill text (e.g. "12 open").
  - `avatar` — `<Avatar>` or `<Icon>`.
  - `interactive` — make the header clickable.
  - `onClick` — navigation handler.
  - `action` — slot for a small action in the header (rare).
- For chart cards, use `AnalyticalCardHeader` from `@ui5/webcomponents-react` which is a KPI-oriented header.
- Charts from `@ui5/webcomponents-react-charts`.
- For quick view cards, use `Card` + `CardHeader` + `Bar` with actions at the bottom.

## Production scaffold

```tsx
'use client';

import {
  DynamicPage,
  DynamicPageTitle,
  DynamicPageHeader,
  Title,
  Label,
  FilterBar,
  FilterGroupItem,
  Select,
  Option,
  Card,
  CardHeader,
  AnalyticalCardHeader,
  List,
  ListItemStandard,
  Bar,
  Button,
  Icon,
  ValueState,
  Toolbar,
  ToolbarButton,
} from '@ui5/webcomponents-react';
import { LineChart, ColumnChart } from '@ui5/webcomponents-react-charts';

import shareIcon from '@ui5/webcomponents-icons/dist/share.js';
import alertIcon from '@ui5/webcomponents-icons/dist/alert.js';
import employeeIcon from '@ui5/webcomponents-icons/dist/employee.js';
import actionIcon from '@ui5/webcomponents-icons/dist/action.js';

import { useMemo, useState } from 'react';

type Props = {
  onOpenOpenPOs: () => void;
  onOpenInvoiceQueue: () => void;
  onOpenSpendAnalysis: () => void;
  onOpenSuppliers: () => void;
  onOpenSupplier: (id: string) => void;
  onOpenPurchaseOrderApp: () => void;
};

export default function PurchasingOverviewPage({
  onOpenOpenPOs,
  onOpenInvoiceQueue,
  onOpenSpendAnalysis,
  onOpenSuppliers,
  onOpenSupplier,
  onOpenPurchaseOrderApp,
}: Props) {
  const [period, setPeriod] = useState<'Q1' | 'Q2' | 'Q3' | 'Q4'>('Q1');
  const [region, setRegion] = useState('All');

  // --- mock data (replace with your fetches, keyed to filter state) ---
  const spendTrend = useMemo(
    () =>
      [
        { month: 'Jan', spend: 120_000 },
        { month: 'Feb', spend: 145_000 },
        { month: 'Mar', spend: 132_000 },
      ],
    [],
  );
  const spendByCategory = useMemo(
    () =>
      [
        { category: 'Raw materials', amount: 480_000 },
        { category: 'Services', amount: 220_000 },
        { category: 'Travel', amount: 45_000 },
        { category: 'IT', amount: 95_000 },
      ],
    [],
  );
  const topSuppliers = useMemo(
    () =>
      [
        { id: 'S-1001', name: 'Acme Corp', amount: '€212,500' },
        { id: 'S-1002', name: 'Globex Ltd', amount: '€184,200' },
        { id: 'S-1003', name: 'Stark Industries', amount: '€141,800' },
        { id: 'S-1004', name: 'Wayne Enterprises', amount: '€96,400' },
      ],
    [],
  );
  const openPOs = { count: 12, items: [
    { id: 'PO-1001', meta: 'Vendor A · €12,500' },
    { id: 'PO-1002', meta: 'Vendor B · €4,200' },
    { id: 'PO-1003', meta: 'Vendor C · €9,800' },
  ] };
  const overdueInvoices = 3;

  return (
    <DynamicPage
      titleArea={
        <DynamicPageTitle
          header={<Title>Purchasing Overview</Title>}
          subHeader={<Label>Role: Buyer · Period: {period} · Region: {region}</Label>}
          actionsBar={
            <Toolbar design="Transparent">
              <ToolbarButton design="Transparent" icon={shareIcon} text="Share" />
            </Toolbar>
          }
        />
      }
      headerArea={
        <DynamicPageHeader>
          <FilterBar>
            <FilterGroupItem label="Period" filterKey="period">
              <Select
                onChange={(e) =>
                  setPeriod(((e.detail.selectedOption as HTMLElement).dataset.value as typeof period) ?? 'Q1')
                }
              >
                <Option data-value="Q1" selected={period === 'Q1'}>Q1</Option>
                <Option data-value="Q2" selected={period === 'Q2'}>Q2</Option>
                <Option data-value="Q3" selected={period === 'Q3'}>Q3</Option>
                <Option data-value="Q4" selected={period === 'Q4'}>Q4</Option>
              </Select>
            </FilterGroupItem>
            <FilterGroupItem label="Region" filterKey="region">
              <Select
                onChange={(e) =>
                  setRegion((e.detail.selectedOption as HTMLElement).dataset.value ?? 'All')
                }
              >
                <Option data-value="All" selected={region === 'All'}>All</Option>
                <Option data-value="EMEA" selected={region === 'EMEA'}>EMEA</Option>
                <Option data-value="AMER" selected={region === 'AMER'}>AMER</Option>
                <Option data-value="APAC" selected={region === 'APAC'}>APAC</Option>
              </Select>
            </FilterGroupItem>
          </FilterBar>
        </DynamicPageHeader>
      }
    >
      <div
        style={{
          display: 'grid',
          gridTemplateColumns: 'repeat(auto-fill, minmax(320px, 1fr))',
          gap: 'var(--sapContent_Space_M)',
          padding: 'var(--sapContent_Space_M)',
        }}
      >
        {/* List card — Top suppliers. Header navigates to the Suppliers list report. */}
        <Card
          header={
            <CardHeader
              titleText="Top Suppliers"
              subtitleText={`Period: ${period}`}
              status={`${topSuppliers.length} of 214`}
              avatar={<Icon name={employeeIcon} />}
              interactive
              onClick={onOpenSuppliers}
            />
          }
        >
          <List
            onItemClick={(e) => {
              const id = ((e.detail.item as HTMLElement).dataset as { id?: string }).id;
              if (id) onOpenSupplier(id);
            }}
          >
            {topSuppliers.map((s) => (
              <ListItemStandard key={s.id} data-id={s.id} description={s.amount}>
                {s.name}
              </ListItemStandard>
            ))}
          </List>
        </Card>

        {/* Analytical card — spend trend. Uses AnalyticalCardHeader for the KPI up top. */}
        <Card
          header={
            <AnalyticalCardHeader
              titleText="Monthly Spend"
              subtitleText={`Period: ${period}`}
              value="€397K"
              valueState={ValueState.Positive}
              unit="YTD"
              trendState="Good"
              trend="Up"
              indicatorState={ValueState.Positive}
              state="Good"
              description="vs. target"
              onClick={onOpenSpendAnalysis}
            />
          }
        >
          <div style={{ height: '180px', padding: '0 0.5rem 0.5rem' }}>
            <LineChart
              dataset={spendTrend}
              dimensions={[{ accessor: 'month' }]}
              measures={[{ accessor: 'spend', label: 'Spend (€)' }]}
            />
          </div>
        </Card>

        {/* Analytical card — spend by category. */}
        <Card
          header={
            <CardHeader
              titleText="Spend by Category"
              subtitleText={`Period: ${period}`}
              interactive
              onClick={onOpenSpendAnalysis}
            />
          }
        >
          <div style={{ height: '220px', padding: '0.5rem' }}>
            <ColumnChart
              dataset={spendByCategory}
              dimensions={[{ accessor: 'category' }]}
              measures={[{ accessor: 'amount', label: 'Amount (€)' }]}
            />
          </div>
        </Card>

        {/* List card — open purchase orders, navigates to PO List Report */}
        <Card
          header={
            <CardHeader
              titleText="Open Purchase Orders"
              subtitleText="Awaiting approval"
              status={String(openPOs.count)}
              interactive
              onClick={onOpenOpenPOs}
            />
          }
        >
          <List
            onItemClick={(e) => {
              const id = ((e.detail.item as HTMLElement).dataset as { id?: string }).id;
              // TODO: onOpenPO(id)
            }}
          >
            {openPOs.items.map((i) => (
              <ListItemStandard key={i.id} data-id={i.id} description={i.meta}>
                {i.id}
              </ListItemStandard>
            ))}
          </List>
        </Card>

        {/* Simple status card — one prominent number pulling user toward action */}
        <Card
          header={
            <CardHeader
              titleText="Overdue Invoices"
              subtitleText="Needs your attention"
              status={String(overdueInvoices)}
              avatar={<Icon name={alertIcon} />}
              interactive
              onClick={onOpenInvoiceQueue}
            />
          }
        />

        {/* Quick view card — the one card type where footer actions are allowed.
            Use sparingly. Here: a highlighted "today's action" kind of card. */}
        <Card
          header={
            <CardHeader
              titleText="Approve Monthly Plan"
              subtitleText="Due in 2 days"
              avatar={<Icon name={actionIcon} />}
            />
          }
        >
          <div style={{ padding: '1rem' }}>
            <Label>
              The monthly purchasing plan for {period} is ready for your review. Expected spend is
              within 4% of target.
            </Label>
          </div>
          <Bar
            design="Footer"
            endContent={
              <>
                <Button design="Emphasized" onClick={onOpenPurchaseOrderApp}>
                  Open plan
                </Button>
                <Button design="Transparent">Snooze</Button>
              </>
            }
          />
        </Card>
      </div>
    </DynamicPage>
  );
}
```

### Notes on the scaffold

- **Grid layout** uses CSS Grid directly — no layout library. `auto-fill minmax(320px, 1fr)` gives the Fiori card rhythm.
- **Analytical card** uses `AnalyticalCardHeader` with a KPI-style header. For a simpler feel, you can use a plain `CardHeader` with a `status` pill.
- **Quick view card** is the only card type with footer actions (`Bar design="Footer"`). Use it sparingly — mostly OVP is read-and-navigate.
- **Card sizes** are controlled by the grid cell width; don't over-customize individual card sizes. Consistency is more important than perfect fit.
- **Navigation** from a card header is the default; row clicks inside list/table cards navigate to specific items in the owning app.

## Anti-patterns

- Embedding an OVP inside an FCL. → Prohibited. OVP is standalone.
- Editable fields in cards. → Not allowed; cards navigate.
- Scrolling list inside a card with dozens of rows. → That's a List Report, not a card. Show top 3–6 and link out.
- Mixing huge and tiny cards randomly. → Keep the grid rhythm.
- A "list of 20" card with no "show all" navigation. → The card header must navigate.
- OVP with only one data source. → That's a List Report with a summary area.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents-react-charts` (for analytical cards)
- `@ui5/webcomponents`
- `@ui5/webcomponents-fiori`
- `@ui5/webcomponents-icons`
