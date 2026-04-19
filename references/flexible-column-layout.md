# Flexible Column Layout (FCL)

The Fiori layout control for **list ↔ detail ↔ sub-detail navigation side by side**. Users drill through hierarchy without losing context of higher levels. Classic example: mail app (folder → thread → message).

> **Also consult** `references/global-patterns.md` (navigation — back button rules differ in FCL vs full-screen) and `references/action-placement.md` (close-column and full-screen icon placement in the column's title `actionsBar`).

FCL is a **layout**, not a floorplan — it hosts floorplans inside each column. Each column's content should itself be a valid floorplan (List Report, Object Page, etc.).

## When to use

- You have a **hierarchical** navigation pattern: list → detail, or list → detail → sub-detail.
- Keeping higher-level context visible while drilling down is genuinely useful (user needs to cross-reference).
- The levels are **closely related** — same domain, same user flow.

## When NOT to use (hard rules)

- **Overview Page cannot be placed inside FCL.** OVP must always be standalone.
- **Launchpad cannot be embedded in a column.**
- You want a **workbench** / tools layout with persistent side panels → use dynamic side content, not FCL.
- You want to **show two unrelated apps side by side** → use separate pages or a dashboard, not FCL.
- You want a **main + left + right panels** layout → FCL does list/detail/sub-detail horizontally, not main+side. Use dynamic side content.
- **Multiple instances** of the same object type open at once → use the multi-instance handling floorplan, not FCL.
- You want to **split a single object into multiple columns** → use the Object Page's responsive layout, not FCL.

## What you get for free

- Three auto-responsive columns that collapse to 2 or 1 as the screen narrows:
  - ≥ 1024px: up to 3 columns.
  - 599–1023px: up to 2 columns.
  - < 599px: 1 column (full screen for whichever is active).
- **Minimum column width 248px** when more than one column is showing.
- Several preset layouts via the `FCLLayout` enum:
  - `OneColumn`
  - `TwoColumnsStartExpanded` / `TwoColumnsMidExpanded`
  - `ThreeColumnsStartExpanded` / `ThreeColumnsMidExpanded` / `ThreeColumnsEndExpanded`
  - `MidColumnFullScreen`
  - `EndColumnFullScreen`
- **Drag-resizable column separators** (the user can resize columns freely within the 248px min constraint).
- **Layout arrows** next to dividers to expand one column at a time.
- **Full-screen** toggle for the rightmost column.
- Built-in **keyboard fast-nav** (F6 / Shift+F6) between columns — you opt in by importing `@ui5/webcomponents-base/dist/features/F6Navigation.js`.

## The slot gotcha — the single most common FCL bug

This is the mistake developers make over and over. The `FlexibleColumnLayout` passes a `slot` prop into each column's content. The content component **must** forward that slot to its **root HTML element** — not to a React wrapper component, not as a plain prop.

Wrong — component reference:
```tsx
<FlexibleColumnLayout startColumn={StartColumn} />
```

Wrong — component not forwarding slot:
```tsx
const StartColumn = () => <div>Start</div>;
<FlexibleColumnLayout startColumn={<StartColumn />} />
```

Wrong — slot passed as a React prop on a wrapper component:
```tsx
const StartColumn = ({ slot }: { slot?: string }) => <Bar slot={slot}>Start</Bar>;
// Bar is a React wrapper; the `slot` ends up on the wrapper's outer element, which may not be what FCL needs.
```

Correct:
```tsx
const StartColumn = (props: { slot?: string }) => (
  <div slot={props.slot}>Start</div>
);
<FlexibleColumnLayout startColumn={<StartColumn />} />
```

The root must be a real HTML element (`<div>`, `<section>`, `<main>`) carrying `slot={props.slot}`. You can put any React content (including ObjectPage, DynamicPage, List, etc.) inside that root element.

## Fiori design rules

- **One floorplan per column.** `startColumn` = List Report. `midColumn` = Object Page. `endColumn` = another Object Page or a sub-section view. Don't try to cram multiple floorplans into one column.
- Keep the floorplan in each column **responsive down to phone size** (this is a hard FCL requirement).
- **URL-sync the layout state.** The user's chosen layout, selected items, and drill path should be reflected in the URL so back/forward and deep-links work. Use Next.js route params / search params.
- **Close and Full-Screen actions are manual** in freestyle FCL. Add explicit close buttons (e.g. icon `decline`) and a full-screen toggle where appropriate. In Fiori elements these are built-in.
- **Back navigation closes columns.** If the user clicks browser-back or a back button, go from 3-column to 2-column to 1-column. Back does *not* restore layout changes (resizing); it only unwinds the drill-path.
- Each column has **its own scrollbar**. There's no "page" scroll in FCL.
- **Dialogs opened from any column center over the whole screen**, not over the column.
- **Tables**: in a draft-enabled List Report placed inside FCL, you cannot use tree table or analytical table — use responsive `Table`. (This is a Fiori-elements-level rule; observe it in freestyle too.)

## Layout transitions — the expected pattern

- Start: `OneColumn` showing a List Report.
- User picks a list item → switch to `TwoColumnsMidExpanded`, show the Object Page in `midColumn`.
- User drills into an item-of-an-item in the Object Page → switch to `ThreeColumnsMidExpanded`, show the sub-detail in `endColumn`.
- User closes the `endColumn` → back to `TwoColumnsMidExpanded`.
- User closes the `midColumn` → back to `OneColumn`.

## v2 component API cheat sheet

**`FlexibleColumnLayout`:**
- `startColumn`, `midColumn`, `endColumn` — JSX element slots (see slot gotcha above).
- `layout` — `FCLLayout` enum member (string): `OneColumn`, `TwoColumnsStartExpanded`, `TwoColumnsMidExpanded`, `ThreeColumnsStartExpanded`, `ThreeColumnsMidExpanded`, `ThreeColumnsEndExpanded`, `MidColumnFullScreen`, `EndColumnFullScreen`.
- `onLayoutChange` — fires when the user resizes / uses layout arrows / toggles full-screen. `e.detail.layout` is the new layout; `e.detail.columnLayout` is an array of column widths.
- `onColumnResize` — fires during drag-resize.
- `hideArrows` — hide the expand/collapse arrows if you want manual control.
- `disableResizing` — disable drag-to-resize.
- `accessibilityAttributes` — slot-labeling texts for screen readers.

**`FCLLayout`** — import the enum alongside `FlexibleColumnLayout`:
```tsx
import { FlexibleColumnLayout, FCLLayout } from '@ui5/webcomponents-react';
```

## Production scaffold — Orders → Order → Order Item

```tsx
'use client';

import {
  FlexibleColumnLayout,
  FCLLayout,
  List,
  ListItemStandard,
  ObjectPage,
  ObjectPageTitle,
  ObjectPageHeader,
  ObjectPageSection,
  ObjectPageSubSection,
  Title,
  Label,
  Text,
  ObjectStatus,
  Bar,
  Button,
  Toolbar,
  ToolbarButton,
  ToolbarSpacer,
  IllustratedMessage,
  IllustratedMessageType,
} from '@ui5/webcomponents-react';

import fullScreenIcon from '@ui5/webcomponents-icons/dist/full-screen.js';
import exitFullScreenIcon from '@ui5/webcomponents-icons/dist/exit-full-screen.js';
import declineIcon from '@ui5/webcomponents-icons/dist/decline.js';

import { useState } from 'react';

type Order = { id: string; customer: string; status: 'Open' | 'Closed'; total: number };
type Item = { id: string; product: string; qty: number; unitPrice: number };

// ---------- Column components ----------
// Each column component receives a `slot` prop from FCL and MUST forward it
// to its root HTML element (a real DOM node, not a React wrapper component).

const OrdersColumn = (props: {
  slot?: string;
  orders: Order[];
  selectedId: string | null;
  onSelect: (id: string) => void;
}) => (
  <div slot={props.slot} style={{ height: '100%', overflow: 'auto' }}>
    <Bar startContent={<Title level="H3">Orders</Title>} />
    <List
      selectionMode="Single"
      onSelectionChange={(e) => {
        const id = (e.detail.selectedItems[0] as HTMLElement).dataset.id;
        if (id) props.onSelect(id);
      }}
    >
      {props.orders.map((o) => (
        <ListItemStandard
          key={o.id}
          data-id={o.id}
          description={`${o.customer} · €${o.total.toLocaleString()}`}
          selected={props.selectedId === o.id}
          additionalText={o.status}
        >
          {o.id}
        </ListItemStandard>
      ))}
    </List>
  </div>
);

const OrderDetailColumn = (props: {
  slot?: string;
  order: Order | null;
  items: Item[];
  selectedItemId: string | null;
  onSelectItem: (id: string) => void;
  onClose: () => void;
  onToggleFullScreen: () => void;
  isFullScreen: boolean;
}) => (
  <div slot={props.slot} style={{ height: '100%', overflow: 'auto' }}>
    {props.order ? (
      <ObjectPage
        titleArea={
          <ObjectPageTitle
            header={<Title>{props.order.id}</Title>}
            subHeader={<Label>{props.order.customer}</Label>}
            actionsBar={
              <Toolbar design="Transparent">
                <ToolbarButton
                  design="Transparent"
                  icon={props.isFullScreen ? exitFullScreenIcon : fullScreenIcon}
                  text={props.isFullScreen ? 'Exit Full Screen' : 'Full Screen'}
                  onClick={props.onToggleFullScreen}
                />
                <ToolbarButton
                  design="Transparent"
                  icon={declineIcon}
                  text="Close"
                  onClick={props.onClose}
                />
              </Toolbar>
            }
          >
            <ObjectStatus state={props.order.status === 'Closed' ? 'Positive' : 'Critical'}>
              {props.order.status}
            </ObjectStatus>
          </ObjectPageTitle>
        }
        headerArea={
          <ObjectPageHeader>
            <div style={{ display: 'flex', gap: '2rem' }}>
              <div><Label>Total</Label><Text>€{props.order.total.toLocaleString()}</Text></div>
              <div><Label>Line items</Label><Text>{props.items.length}</Text></div>
            </div>
          </ObjectPageHeader>
        }
      >
        <ObjectPageSection id="items" titleText="Line Items">
          <ObjectPageSubSection id="items-list" titleText="Items">
            <List
              selectionMode="Single"
              onSelectionChange={(e) => {
                const id = (e.detail.selectedItems[0] as HTMLElement).dataset.id;
                if (id) props.onSelectItem(id);
              }}
            >
              {props.items.map((i) => (
                <ListItemStandard
                  key={i.id}
                  data-id={i.id}
                  description={`Qty ${i.qty} · €${i.unitPrice.toFixed(2)}`}
                  selected={props.selectedItemId === i.id}
                >
                  {i.product}
                </ListItemStandard>
              ))}
            </List>
          </ObjectPageSubSection>
        </ObjectPageSection>
      </ObjectPage>
    ) : (
      <IllustratedMessage
        name={IllustratedMessageType.NoEntries}
        titleText="Select an order to see details"
      />
    )}
  </div>
);

const ItemDetailColumn = (props: {
  slot?: string;
  item: Item | null;
  onClose: () => void;
  onToggleFullScreen: () => void;
  isFullScreen: boolean;
}) => (
  <div slot={props.slot} style={{ height: '100%', overflow: 'auto' }}>
    <Bar
      startContent={<Title level="H3">{props.item?.product ?? 'Item'}</Title>}
      endContent={
        <>
          <Button
            design="Transparent"
            icon={props.isFullScreen ? exitFullScreenIcon : fullScreenIcon}
            onClick={props.onToggleFullScreen}
            accessibleName={props.isFullScreen ? 'Exit Full Screen' : 'Full Screen'}
          />
          <Button design="Transparent" icon={declineIcon} onClick={props.onClose} accessibleName="Close" />
        </>
      }
    />
    {props.item ? (
      <div style={{ padding: 'var(--sapContent_Space_M)', display: 'grid', gap: '0.5rem' }}>
        <Text><strong>Item ID:</strong> {props.item.id}</Text>
        <Text><strong>Product:</strong> {props.item.product}</Text>
        <Text><strong>Quantity:</strong> {props.item.qty}</Text>
        <Text><strong>Unit price:</strong> €{props.item.unitPrice.toFixed(2)}</Text>
        <Text><strong>Line total:</strong> €{(props.item.qty * props.item.unitPrice).toFixed(2)}</Text>
        {/* TODO: richer item details here — could itself be a nested ObjectPage */}
      </div>
    ) : (
      <IllustratedMessage
        name={IllustratedMessageType.NoEntries}
        titleText="Select an item to see details"
      />
    )}
  </div>
);

// ---------- Main component ----------

type Props = {
  orders: Order[];
  itemsByOrder: Record<string, Item[]>;
};

export default function OrdersFCL({ orders, itemsByOrder }: Props) {
  const [layout, setLayout] = useState<FCLLayout>(FCLLayout.OneColumn);
  const [selectedOrderId, setSelectedOrderId] = useState<string | null>(null);
  const [selectedItemId, setSelectedItemId] = useState<string | null>(null);

  const order = orders.find((o) => o.id === selectedOrderId) ?? null;
  const items = selectedOrderId ? (itemsByOrder[selectedOrderId] ?? []) : [];
  const item = items.find((i) => i.id === selectedItemId) ?? null;

  // Handlers — note how each transition reflects user intent in both
  // the layout state and the selected-id state.
  const selectOrder = (id: string) => {
    setSelectedOrderId(id);
    setSelectedItemId(null);
    setLayout(FCLLayout.TwoColumnsMidExpanded);
  };
  const selectItem = (id: string) => {
    setSelectedItemId(id);
    setLayout(FCLLayout.ThreeColumnsMidExpanded);
  };
  const closeOrder = () => {
    setSelectedOrderId(null);
    setSelectedItemId(null);
    setLayout(FCLLayout.OneColumn);
  };
  const closeItem = () => {
    setSelectedItemId(null);
    setLayout(FCLLayout.TwoColumnsMidExpanded);
  };
  const toggleFsMid = () =>
    setLayout((l) => (l === FCLLayout.MidColumnFullScreen ? FCLLayout.TwoColumnsMidExpanded : FCLLayout.MidColumnFullScreen));
  const toggleFsEnd = () =>
    setLayout((l) => (l === FCLLayout.EndColumnFullScreen ? FCLLayout.ThreeColumnsMidExpanded : FCLLayout.EndColumnFullScreen));

  return (
    <FlexibleColumnLayout
      layout={layout}
      onLayoutChange={(e) => {
        // User may change layout via drag or arrow — keep state in sync
        setLayout(e.detail.layout as FCLLayout);
      }}
      startColumn={
        <OrdersColumn
          orders={orders}
          selectedId={selectedOrderId}
          onSelect={selectOrder}
        />
      }
      midColumn={
        <OrderDetailColumn
          order={order}
          items={items}
          selectedItemId={selectedItemId}
          onSelectItem={selectItem}
          onClose={closeOrder}
          onToggleFullScreen={toggleFsMid}
          isFullScreen={layout === FCLLayout.MidColumnFullScreen}
        />
      }
      endColumn={
        <ItemDetailColumn
          item={item}
          onClose={closeItem}
          onToggleFullScreen={toggleFsEnd}
          isFullScreen={layout === FCLLayout.EndColumnFullScreen}
        />
      }
    />
  );
}
```

### Notes on the scaffold

- Each column component forwards `slot={props.slot}` to a real HTML `<div>` at its root. **This is the FCL slot rule.**
- Each column handles **its own scroll** (`overflow: 'auto'`).
- **Close buttons** on midColumn and endColumn transition the layout back. **Full-screen toggles** switch between the natural multi-column layout and the full-screen variant. These have to be wired manually — FCL doesn't ship them automatically for freestyle apps.
- **`onLayoutChange`** is crucial: the user can drag separators or click expand arrows and the layout will change without your code initiating it. Always sync React state from `e.detail.layout`.
- **URL sync** is omitted here for brevity. In Next.js App Router, use `useSearchParams` + `router.replace()` with `?orderId=…&itemId=…&layout=…` for shareable / back-button-friendly deep links.

## Anti-patterns

- Passing a component reference instead of a JSX element: `startColumn={StartColumn}`. → Pass `<StartColumn />`.
- Forgetting `slot={props.slot}` on the column's root. → Content won't render correctly.
- Putting `slot` on a React wrapper component (like a Bar or Card) instead of a raw `<div>`. → Still wrong; UI5WC doesn't know where to insert it.
- Putting an Overview Page in any column. → OVP is prohibited in FCL.
- Using `hideArrows` with no alternative close/expand controls. → Users will be stuck.
- Reacting to `onLayoutChange` only for some layouts. → React state will desync from UI.
- Not giving each column its own scroll. → Content breaks out of the column.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents`
- `@ui5/webcomponents-fiori` (for FlexibleColumnLayout)
- `@ui5/webcomponents-icons`
