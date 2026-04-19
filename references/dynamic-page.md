# Dynamic Page

The foundation layout for most Fiori floorplans. If a floorplan doesn't fit, a plain `DynamicPage` is almost always the correct fallback. List Report, Worklist, Analytical List Page, Overview Page, and Initial Page are all built on top of `DynamicPage`. Object Page is a close cousin with its own richer component.

> **Also consult** `references/global-patterns.md` (action placement, empty/loading states) and `references/action-placement.md` (title `actionsBar` vs `footerArea` styling) for anything beyond raw page chrome.

Think of `DynamicPage` as the Fiori equivalent of "a page with a header, content area, and footer toolbar, with the right spacing, snapping behavior, and responsive breakpoints built in".

## When to use DynamicPage directly

- The user's use case **doesn't map cleanly to a named floorplan** (not a list, not an object, not a wizard, not a dashboard).
- You need the **dynamic page chrome** (title, collapsible header content, footer actions) without the specific constraints of a named floorplan.
- You're composing something unusual: a settings page, a config editor, a bespoke workflow screen.

## When NOT to use DynamicPage directly

- You're showing a single object → **Object Page** (own component, richer behavior).
- You match any named floorplan → use the floorplan's scaffold, which uses `DynamicPage` under the hood.

## What you get for free

- **Title area** (always visible).
- **Header area** (expandable / collapsible; collapses on scroll by default).
- **Pin button** to keep the header expanded even while scrolling.
- **Footer area** for finalizing actions (stays pinned at the bottom).
- **Responsive padding** via Fiori's spacing system.
- **Keyboard focus handling** — header expands automatically when keyboard focus enters it; collapses when it leaves.

## Anatomy

```
┌─────────────────────────────────────────────────┐
│  ShellBar (app-level, outside DynamicPage)      │
├─────────────────────────────────────────────────┤
│  DynamicPageTitle                               │   ← always visible
│    Breadcrumbs                                  │
│    Title / VariantManagement                    │
│    Subtitle (or "Filtered By (x): …")           │
│    Status chips, ObjectStatus                   │
│    actionsBar (global page actions, Toolbar)    │
├─────────────────────────────────────────────────┤
│  DynamicPageHeader                              │   ← collapsible
│    FilterBar / KPI tiles / contextual facets    │
│    — collapses on scroll                        │
├─────────────────────────────────────────────────┤
│  Content                                        │
│    Tables, forms, charts, sections              │
│    (scrolls underneath the sticky title)        │
├─────────────────────────────────────────────────┤
│  footerArea (Bar, design="FloatingFooter")      │   ← pinned at bottom
│    Finalizing actions (Save, Cancel, Submit)    │
└─────────────────────────────────────────────────┘
```

## Fiori design rules

### Title

- **Breadcrumbs** go above the title, optional, used for parent-child drilldowns.
- **Title** is mandatory. A plain `<Title>`, or `<VariantManagement>` when the page supports saved views.
- **Subtitle** is optional. Commonly used to show collapsed-header context like `"Filtered By (x): status, country"`. When used as a summary, the subtitle should itself be clickable to expand/collapse the header.
- **actionsBar** (right side of the title) holds **non-finalizing** page-global actions (Share, Export, Settings). Not Save. Not Create. Not Delete.

### Header content

- Header content is optional but typical. Contains the FilterBar (for List Report / ALP), KPI tiles (for ALP / OVP), or contextual facets (for Object Page).
- **Never put editable fields in the header.** The header collapses on scroll, and editable content would disappear mid-edit.
- When the header is collapsed, the title subtitle can summarize its state ("Filtered By (3): …").

### The desktop-centric-table exception — important

The dynamic page header is designed to snap (collapse on scroll). But if the content area is a **grid table, analytical table, or tree table** (what Fiori calls "desktop-centric tables"), the table takes up all available space and there's no page-level scroll to trigger the snap. Same story for full chart containers.

In that case:

- The snap-on-scroll behavior is inactive.
- The **pin/unpin button is obsolete** and should be removed (`hidePinButton`).
- Provide an **explicit Hide Filters / Show Filters button** in the content toolbar to collapse/expand the header manually.

The scaffold below shows both cases.

### Footer

- `footerArea` takes a `<Bar>` with `design="FloatingFooter"` — the Fiori floating footer look.
- Contents: finalizing actions only. Save, Cancel, Submit, Approve, Delete. Aligned to the end.
- If no finalizing actions apply to the whole page, omit the footer.

## v2 component API cheat sheet

**`DynamicPage`:**
- `titleArea` — takes a `<DynamicPageTitle>`.
- `headerArea` — takes a `<DynamicPageHeader>`.
- `footerArea` — takes a `<Bar>` (typically `design="FloatingFooter"`).
- `headerPinned` — boolean, keep header expanded on scroll (was `alwaysShowContentHeader` in v1).
- `hidePinButton` — boolean (was `headerContentPinnable` in v1, logic inverted).
- `onToggleHeaderArea` — event when user collapses/expands (was `onToggleHeaderContent` in v1).
- `onPinButtonToggle` — event when user pins/unpins (was `onPinnedStateChange` in v1).
- `showFooter` — boolean, animate the footer in.

**`DynamicPageTitle`:**
- `header` — slot for main title content (usually `<Title>` or `<VariantManagement>`).
- `subHeader` — slot for subtitle (`<Label>` or `<Text>`).
- `snappedHeader` / `snappedSubHeader` — alternate content shown when the header is collapsed (new in v2.12).
- `breadcrumbs` — slot for `<Breadcrumbs>`.
- `actionsBar` — slot for a `<Toolbar>`.
- `navigationBar` — slot for a `<Toolbar>` of navigation actions (rarely used outside Object Page).
- `children` — status chips, tags, `ObjectStatus`.

**`DynamicPageHeader`:**
- Children are freeform (FilterBar, KPI tiles, a grid of facets).

## Production scaffolds

### Scaffold 1 — regular DynamicPage with snap-on-scroll

Appropriate when the content area is a form, a list of cards, or a responsive table that produces page-level scroll.

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
  Toolbar,
  ToolbarButton,
  Bar,
  Button,
  Form,
  FormGroup,
  FormItem,
  Input,
  Select,
  Option,
} from '@ui5/webcomponents-react';

import shareIcon from '@ui5/webcomponents-icons/dist/share.js';

export default function SettingsPage() {
  return (
    <DynamicPage
      titleArea={
        <DynamicPageTitle
          breadcrumbs={
            <Breadcrumbs>
              <BreadcrumbsItem>Admin</BreadcrumbsItem>
              <BreadcrumbsItem>Settings</BreadcrumbsItem>
            </Breadcrumbs>
          }
          header={<Title>Application Settings</Title>}
          subHeader={<Label>Global configuration for all users</Label>}
          actionsBar={
            <Toolbar design="Transparent">
              <ToolbarButton design="Transparent" icon={shareIcon} text="Share" />
            </Toolbar>
          }
        />
      }
      headerArea={
        <DynamicPageHeader>
          <Label>
            Changes affect all users of the application. Consider previewing on a staging tenant before saving.
          </Label>
        </DynamicPageHeader>
      }
      footerArea={
        <Bar
          design="FloatingFooter"
          endContent={
            <>
              <Button design="Emphasized">Save</Button>
              <Button design="Transparent">Cancel</Button>
            </>
          }
        />
      }
    >
      <Form labelSpan="S12 M4 L4 XL4" layout="S1 M2 L2 XL2">
        <FormGroup headerText="Identity">
          <FormItem labelContent={<Label>Tenant name</Label>}>
            <Input />
          </FormItem>
          <FormItem labelContent={<Label>Default locale</Label>}>
            <Select>
              <Option>en-US</Option>
              <Option>de-DE</Option>
              <Option>fr-FR</Option>
            </Select>
          </FormItem>
        </FormGroup>
        <FormGroup headerText="Security">
          <FormItem labelContent={<Label>Session timeout (min)</Label>}>
            <Input type="Number" />
          </FormItem>
        </FormGroup>
      </Form>
    </DynamicPage>
  );
}
```

### Scaffold 2 — DynamicPage with a desktop-centric table (manual Hide/Show Filters)

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
  Input,
  AnalyticalTable,
  Toolbar,
  ToolbarButton,
} from '@ui5/webcomponents-react';

import showFilterIcon from '@ui5/webcomponents-icons/dist/filter.js';
import hideFilterIcon from '@ui5/webcomponents-icons/dist/filter-fields.js';

import { useMemo, useState } from 'react';

export default function DesktopTablePage() {
  const [filtersVisible, setFiltersVisible] = useState(true);
  const [name, setName] = useState('');
  const data = useMemo(
    () => Array.from({ length: 500 }, (_, i) => ({ id: `R${i}`, name: `Row ${i}` })),
    [],
  );
  const columns = useMemo(
    () => [
      { Header: 'ID', accessor: 'id', width: 100 },
      { Header: 'Name', accessor: 'name' },
    ],
    [],
  );

  return (
    <DynamicPage
      // The pin button is obsolete here because the table owns vertical space; hide it.
      hidePinButton
      titleArea={
        <DynamicPageTitle
          header={<Title>All Rows</Title>}
          subHeader={<Label>500 rows · desktop-centric table</Label>}
          actionsBar={
            <Toolbar design="Transparent">
              <ToolbarButton
                design="Transparent"
                icon={filtersVisible ? hideFilterIcon : showFilterIcon}
                text={filtersVisible ? 'Hide Filters' : 'Show Filters'}
                onClick={() => setFiltersVisible((v) => !v)}
              />
            </Toolbar>
          }
        />
      }
      headerArea={
        // Conditionally render the header area — this is the Fiori "Hide Filters" pattern
        filtersVisible ? (
          <DynamicPageHeader>
            <FilterBar>
              <FilterGroupItem label="Name" filterKey="name">
                <Input
                  value={name}
                  onInput={(e) => setName((e.target as HTMLInputElement).value)}
                />
              </FilterGroupItem>
            </FilterBar>
          </DynamicPageHeader>
        ) : undefined
      }
    >
      <AnalyticalTable
        data={data.filter((r) => !name || r.name.toLowerCase().includes(name.toLowerCase()))}
        columns={columns}
        visibleRows={20}
      />
    </DynamicPage>
  );
}
```

## Anti-patterns

- Using v1 slot names (`headerTitle`, `headerContent`). → v2 uses `titleArea`, `headerArea`.
- Putting editable inputs in `headerArea`. → Always in the content or in a temporary edit section for Object Page.
- Snap-on-scroll with an AnalyticalTable filling the viewport. → The table owns scroll; snap won't fire. Use manual Hide/Show Filters instead, and set `hidePinButton`.
- `design="FloatingFooter"` without a `<Bar>` wrapper. → `footerArea` expects a Bar or similar; the floating look comes from the Bar `design` prop.
- Duplicate breadcrumb entries that also repeat in the ShellBar nav menu. → Keep breadcrumbs app-internal hierarchy only.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents`
- `@ui5/webcomponents-fiori` (for FilterBar if used)
- `@ui5/webcomponents-icons`
