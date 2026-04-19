# v1 → v2 Migration and API Deltas

Reference for migrating existing ui5-webcomponents-react v1 code to v2, and for catching v1 patterns in new code. All deltas here are confirmed from the official v2 CHANGELOG.

If you're writing new code, this file tells you which v1 patterns to avoid. If you're migrating code, it tells you exactly what to change.

## Big picture changes

### The `Toolbar` was replaced

- The v1 `Toolbar` and related components (`ToolbarSpacer`, `ToolbarSeparator`) were moved to **`@ui5/webcomponents-react-compat`** for backwards compatibility.
- The new `Toolbar` (in the main package `@ui5/webcomponents-react`) is a UI5 web component. It's based on `ui5-toolbar` and takes `ToolbarButton`, `ToolbarSpacer`, `ToolbarSeparator`, `ToolbarSelect`, `ToolbarSelectOption` children.
- The interim name `ToolbarV2` (used briefly during the transition) was **renamed to `Toolbar`**. Same for `ToolbarSpacerV2` → `ToolbarSpacer` and `ToolbarSeparatorV2` → `ToolbarSeparator`.
- **In new code, always import `Toolbar` from `@ui5/webcomponents-react`** (main package). Use the compat package only when you can't migrate yet.

### The `Loader` was removed

- The deprecated `Loader` component has been moved to `@ui5/webcomponents-react-compat`.
- **Use `BusyIndicator`** for all loading states. It's the only loading indicator aligned with current Fiori UX guidelines.

### `Text` was replaced with the UI5 web component

- `Text` is now a wrapper around `ui5-text`.
- `ExpandableText`: removed props — `hyphenated`, `emptyIndicator` (inherited from old Text), `portalContainer`.

### All spacing CSS variables were removed

- The v1 spacing helper object exported from `@ui5/webcomponents-react-base` (`spacing.sapUiContentPadding` and friends) **no longer works** — the underlying CSS vars don't exist in v2 themes.
- **Use the Fiori `--sapContent_Space_*` CSS variables** directly via inline styles or CSS classes. See `references/shell-and-providers.md` for the variable names.

### Role `application` is gone

- v2 does not set `role="application"` on the theme provider / root — matches the UI5 web components' approach. If your app relied on this, set the role yourself at the appropriate level.

## Component-by-component breaking changes

### ObjectPage

| v1 | v2 | Notes |
|---|---|---|
| `headerTitle` (slot) | `titleArea` | Only accepts a `<ObjectPageTitle>`. |
| `headerContent` (slot) | `headerArea` | Only accepts a `<ObjectPageHeader>`. |
| `footer` (slot) | `footerArea` | — |
| `onToggleHeaderContent` | `onToggleHeaderArea` | — |
| `onPinnedStateChange` | `onPinButtonToggle` | — |
| `alwaysShowContentHeader` | `headerPinned` | — |
| `headerContentPinnable` (default `true`) | `hidePinButton` (default `false`) | Logic inverted. |
| `showHideHeaderButton` | removed | — |
| `showTitleInHeaderContent` | removed | — |

### ObjectPageTitle

| v1 | v2 | Notes |
|---|---|---|
| `actions` (multiple children) | `actionsBar` (single Toolbar) | Pass a `<Toolbar>` with `<ToolbarButton>` children. |
| `navigationActions` (multiple children) | `navigationBar` (single Toolbar) | Same pattern as `actionsBar`. |
| `actionsToolbarProps` | removed | Pass props on the Toolbar itself. |
| `navigationActionsToolbarProps` | removed | Same. |
| (no snapped alt content) | `snappedHeader` / `snappedSubHeader` (new) | Added in v2.12 for distinct collapsed-state content. |

### ObjectPageSection / ObjectPageSubSection

| v1 | v2 | Notes |
|---|---|---|
| `titleText` optional | `titleText` **required** | Build will TS-error without it. |
| `titleTextUppercase` default `true` | default removed (natural case) | If you want uppercase explicitly, pass `titleTextUppercase`. |

### FilterBar

| v1 | v2 | Notes |
|---|---|---|
| `onGo` → CustomEvent with `detail.elements`, `detail.filters`, `detail.search`, `detail.nativeDetail` | Now a standard `onClick` from a ToolbarButton; no payload | Read filter values from your React state. |
| `onRestore` → CustomEvent | Plain callback with a payload object | — |
| `onFiltersDialogOpen` target | target is now a ToolbarButton | Update TypeScript types. |
| `portalContainer` | removed | No longer needed (Popover API). |

### FilterGroupItem

| v1 | v2 | Notes |
|---|---|---|
| `visible` (default `true`) | `hidden` (no default) | Logic inverted. |
| `visibleInFilterBar` (default `true`) | `hiddenInFilterBar` (no default) | Logic inverted. |
| `orderId` | `filterKey` | — |

### AnalyticalTable

| v1 | v2 | Notes |
|---|---|---|
| `onRowSelect` → `detail.selectedFlatRows` as array | still `detail.selectedFlatRows` (and `detail.selectedRowIds` object) | v2 additionally provides `selectedRowIds` for perf on huge datasets. |
| Row click/select SPACE fires on `keydown` | Fires on `keyup` | Matches platform convention. |
| `Loader` shown on `loading` | `BusyIndicator` overlay | Per UX guidelines. |
| Legacy `AnalyticalTableColumnDefinition` overloads | Removed in v2.15.1 | Use the current shape. |
| `accessor` typed loosely | Stricter types for `onRowClick` / `accessor` functions | You may need to narrow TS types in your column configs. |

New in v2 AnalyticalTable:
- `onFilter` callback (v2.10+).
- `alwaysShowBusyIndicator` (v2.12+).
- `useF2CellEdit` plugin hook (v2.14+).
- `noDataReason` and `accessibleRole` on `NoDataComponent` (v2.15+).
- `allVisibleRowsSelected` on `onRowSelect` (v2.20+).

### Avatar

| v1 | v2 | Notes |
|---|---|---|
| `img` prop | removed | Pass a child `<img>` element instead: `<Avatar><img src="…" /></Avatar>`. |

### Breadcrumbs

- The v1 custom React `Breadcrumbs` was **replaced with the official UI5 web component**. API shifts slightly — see the UI5 Web Components docs for `ui5-breadcrumbs` / `BreadcrumbsItem`.

### Card

| v1 | v2 | Notes |
|---|---|---|
| `titleText`, `subtitleText`, `status`, `action`, `avatar`, `headerInteractive`, `onHeaderClick` | removed | Use a `<CardHeader>` passed to the new `header` prop. |

Pattern:
```tsx
<Card header={<CardHeader titleText="…" subtitleText="…" status="…" avatar={<Icon name={…} />} interactive onClick={…} />}>
  {content}
</Card>
```

### Carousel

| v1 | v2 | Notes |
|---|---|---|
| `selectedIndex`, `infiniteScrollOffset`, `onLoadMore` | removed | Use `onNavigate` event to track index and load more items. |

### ComboBox

| v1 | v2 | Notes |
|---|---|---|
| `filterValue` | removed | `value` now represents the live value. |

### Dialog

| v1 | v2 | Notes |
|---|---|---|
| `dialogRef.current.open()` | `dialogRef.current.show()` | — |

### DurationPicker

- Removed (it was made private in UI5 web components).

### ObjectStatus

| v1 | v2 | Notes |
|---|---|---|
| `onClick` handler typed to `HTMLDivElement` | type removed | Use correct element types in your handlers. |
| — | `Indication` states added (v2.9+) | More semantic states available. |

## Import and bundling

### Subpath imports

If your bundler doesn't tree-shake well, use subpath imports:

```ts
// Works in every bundler
import { Button } from '@ui5/webcomponents-react/Button';
import { Toolbar } from '@ui5/webcomponents-react/Toolbar';
```

Root imports also work but may bloat dev bundles.

### `@ui5/webcomponents-fiori` is now optional

- Since v2.14, `@ui5/webcomponents-fiori` is an optional peer dependency.
- Still needed if you use: ShellBar, FilterBar, VariantManagement, ObjectPage, DynamicPage, Wizard, FlexibleColumnLayout, IllustratedMessage, SideNavigation.
- In practice, most Fiori apps use at least one of those, so install it.

### Assets

- Import once at app root: `import '@ui5/webcomponents-react/dist/Assets.js';`
- There's also `Assets-fetch.js` (v2.12+) for environments preferring dynamic loading.
- The old `Assets-static.js` was **removed** in v2.10.1.

## Migration steps — practical order

If you're converting an existing v1 app:

1. **Upgrade peer deps** to v2 matching versions (`@ui5/webcomponents` 2.x, `@ui5/webcomponents-react` 2.x, `@ui5/webcomponents-fiori` 2.x).
2. **Run the codemod** (see the README of `@ui5/webcomponents-react-cli`). The codemod handles most prop renames automatically.
3. **Fix `ObjectPage` / `DynamicPage`** slot renames manually — the codemod doesn't always catch them.
4. **Replace `Loader` with `BusyIndicator`**.
5. **Replace `Toolbar` imports** — if from `@ui5/webcomponents-react`, you're fine (new v2 Toolbar). If from `@ui5/webcomponents-react-compat` because you still have a v1 Table, **don't mix v1 and v2 Table in the same app** — it's unsupported. Migrate to v2 Table.
6. **Replace `Text` usages** — most still work; check `ExpandableText` for removed props.
7. **Replace `spacing.sapUi*Padding`** usage with `var(--sapContent_Space_*)` values.
8. **Review `Card` headers** — move all header content into a `<CardHeader>`.
9. **Check `Avatar`** — move `img` prop usage to a child `<img>`.
10. **Search for `visible=` / `visibleInFilterBar=` / `orderId=`** on FilterGroupItem — replace with `hidden` / `hiddenInFilterBar` / `filterKey`.
11. **Fix `ObjectPageSection` / `ObjectPageSubSection`** to have `titleText`.
12. **Test for hydration errors** in Next.js — if anything renders on the server, wrap in `dynamic(..., { ssr: false })`.

## Co-existence caveat for Tables

If you're still using the v1 `Table` from `@ui5/webcomponents-react-compat` in a project that also uses `FilterBar` or `VariantManagement` (v2 components that internally depend on the v2 Table), **the custom element names collide** (`ui5-table`, `ui5-table-row`, `ui5-table-cell`). There are two mitigations:

- **Strongly preferred**: migrate fully to the v2 Table. Use `Table` / `TableRow` / `TableCell` / `TableHeaderRow` / `TableHeaderCell` from `@ui5/webcomponents-react`.
- **Temporary**: run the `@ui5/webcomponents-react-cli patch-compat-table` script, which patches the compat package to use `-v1` suffixed custom element names, avoiding the collision.

Do not leave v1 and v2 Table mixed in new code.

## Quick spot-the-v1 checklist

If you see any of these patterns in user code, it's v1 and needs migration:

- `<ObjectPage headerTitle={…} headerContent={…} footer={…}>` — v1 slot names.
- `<ObjectPageTitle actions={…} navigationActions={…}>` — v1 action slots.
- `<ObjectPageSection>` without a `titleText` — required in v2.
- `<FilterGroupItem visible visibleInFilterBar orderId="x">` — v1 props.
- `import { Loader }` — replace with `BusyIndicator`.
- `avatar.img` on `<Avatar img={…}>` — replace with child `<img>`.
- `<Card titleText="…" subtitleText="…" status="…">` — move to CardHeader.
- `spacing.sapUiContentPadding` / `spacing.sapUiSmallMargin` — replace with CSS vars.
- `dialogRef.current.open()` — replace with `.show()`.
- `role="application"` anywhere — remove.

## Useful tools

- **`@ui5/webcomponents-react-cli`** ships a codemod that handles the bulk of the automatic renames: run `npx @ui5/webcomponents-react-cli codemod` against your project.
- The **MCP server** for ui5-webcomponents-react (added in v2.21) is useful for authoring — gives LLM tools access to the live component docs.
- Storybook for the current version: `https://sap.github.io/ui5-webcomponents-react/v2/` — always verify prop names there if in doubt.
