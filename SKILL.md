---
name: sap-fiori-floorplans
description: Designs and scaffolds pages built with @ui5/webcomponents-react v2 following SAP Fiori floorplan guidelines and Fiori's global design patterns (navigation, action placement, object handling, messaging, empty states, accessibility). Use whenever the user is building, refactoring, or reviewing any SAP Fiori / UI5 / ui5-webcomponents-react page — including mentions of "object page", "list report", "overview page", "worklist", "wizard", "initial page", "analytical list page", "ALP", "FCL", "flexible column layout", "dynamic page", "shellbar", "filter bar", or requests like "build a page to show X", "create a screen for Y", "make a dashboard for Z" when the project is SAP-adjacent. Trigger even when no floorplan is named — picking the right one is part of the job. Also use when reviewing Fiori UI for compliance (action placement, toolbar styling, OVP/FCL rules, Fiori action-type taxonomy, data-loss patterns, empty-state messaging) or migrating from v1 to v2.
---

# SAP Fiori Floorplans — @ui5/webcomponents-react v2

Your job when this skill triggers: **select the correct Fiori floorplan for the user's use case, explain the UX reasoning briefly, and scaffold a production-quality page using `@ui5/webcomponents-react` v2 (current: 2.21.x) in a Next.js (App Router) project.** All generated code must respect the Fiori design guidelines — both the floorplan-specific rules and the global patterns (action placement, object handling, messaging, etc.).

This skill's target stack is **Next.js App Router + `@ui5/webcomponents-react` v2**. Every scaffold is a client component (`"use client"`). Read `references/shell-and-providers.md` once for the app-level setup, `references/global-patterns.md` for the cross-cutting UX rules, and the matching floorplan reference file for the specific page.

---

## The golden rules

These apply regardless of floorplan. Generate code that obeys all of them. Call them out to the user only if the user's draft/intent violates one.

### Technical / framework rules

1. **Every top-level page starts with `'use client';`.** UI5 web components touch the DOM; they cannot render on the server.
2. **`ThemeProvider` wraps the whole app tree**, once, in a client providers file (see `references/shell-and-providers.md`).
3. **Import `@ui5/webcomponents-react/dist/Assets.js` once** (in that providers file) — this registers icon assets, i18n bundles, and CLDR data.
4. **Icons are individual imports**, e.g. `import activateIcon from '@ui5/webcomponents-icons/dist/activate.js'` → `icon={activateIcon}`. Do not use string names like `icon="activate"`.
5. **v2 slot names differ from v1.** For ObjectPage: `titleArea` (was `headerTitle`), `headerArea` (was `headerContent`), `footerArea` (was `footer`), `actionsBar` (was `actions`), `navigationBar` (was `navigationActions`). For `actionsBar` / `navigationBar`, pass a `<Toolbar>` with `ToolbarButton` children — not individual buttons.
6. **`titleText` is required on `ObjectPageSection` and `ObjectPageSubSection` in v2.** The `titleTextUppercase` default changed from `true` to natural case.
7. **`FilterGroupItem` uses `hidden` / `hiddenInFilterBar` in v2.** The v1 props `visible` / `visibleInFilterBar` were removed. `filterKey` replaced `orderId`.
8. **`Loader` is deprecated.** Use `BusyIndicator` for loading states. If you see `Loader` in existing user code, migrate it.
9. **Use `Toolbar` and `ToolbarButton` / `ToolbarSpacer` / `ToolbarSeparator` from the root package.** The v1 `Toolbar` now lives in `@ui5/webcomponents-react-compat`; do not use it in new code.
10. **In Next.js App Router, disable SSR for pages using UI5 components.** Use `dynamic(() => import('./page-client'), { ssr: false })` from `next/dynamic` for each page.

### Fiori UX rules (from the global-patterns guidelines)

11. **Exactly one `Emphasized` primary action per page**, across all toolbars combined. Never two. Ideally not mixed with semantic `Positive`/`Negative` buttons.
12. **Finalizing actions (Save, Post, Submit, Approve, Reject) always go in the `footerArea`.** Never in the title `actionsBar`, never in the ShellBar. Non-finalizing actions (Share, Export, Print) go in the title `actionsBar`.
13. **Apply the Fiori action-type styling**: Primary → `Emphasized`; Secondary → `Default`; Semantic → `Positive`/`Negative`; Negative path (Cancel/Close) → `Transparent`. Never mix `Emphasized` with semantic buttons in the same toolbar. See `references/action-placement.md` for the full mapping.
14. **Create vs Add**: button labeled "**Create**" makes a brand-new object; "**Add**" assigns an existing object. Don't use "New" / "Assign" / "Link" for these.
15. **Cancel during Create → quick-confirmation popover "Discard this draft?" / button "Discard".** Cancel during Edit → popover "Discard all changes?" / button "Discard". Never use a full dialog for Cancel confirmation.
16. **Delete is always confirmed by a dialog (MessageBox), never a popover.** After successful delete, show a `MessageToast` with a business-specific success message.
17. **Success feedback uses `MessageToast`** (auto-dismiss). Use `MessageBox` only when the user must acknowledge or decide. Use `MessagePopover` for form-field validation grouped in edit mode.
18. **Empty states use `IllustratedMessage`** with a business-specific headline and description. Never "No data" — use "No suppliers found", "No sales orders yet", etc. Illustrations are optional; the message is mandatory.
19. **Loading states use `BusyIndicator` at page or control level.** Never use a global BusyDialog that covers the ShellBar — users lose access to sign-out and search.
20. **Edit ↔ Display toggle is not navigation.** No new browser-history entry; a shared URL always resolves to display mode.
21. **Draft handling: drafts auto-save every 20 seconds, but users still need a Save button to commit.** When drafts exist and the user returns via bookmark, ask "Resume editing or discard?"
22. **F6 jumps between logical groups** (ShellBar, SideNav, FilterBar, Content, Footer). Don't break this by over-nesting containers. Every field has a `<Label>`; every icon-only button has a `tooltip`.

---

## Workflow

1. **Read the relevant references.** Always `references/global-patterns.md` for cross-cutting rules, plus the matching floorplan file.
2. **Classify the use case** (Step 1 below).
3. **Communicate the decision briefly** (Step 2).
4. **Scaffold the code** using verified v2 APIs (Step 3).
5. **Cross-check against the Fiori UX guardrails** (Step 4).
6. **Hand off** with file location, peer deps, and follow-up suggestions (Step 5).

---

## Step 1 — Classify the use case

Work through these questions in order. Stop at the first match.

### Q1 — Does the user need to reach one specific object via its identifier?

(e.g. "open purchase order PO-1234", "track shipment by tracking number")

→ **Initial Page**. See `references/initial-page.md`.

*Runner-up: if there's any browsing or filtering need, use List Report instead.*

### Q2 — Is this a guided, multi-step creation or editing task that the user does infrequently?

→ **Wizard**. See `references/wizard.md`.

Qualifiers for Wizard (all must hold):
- **3 to 8 steps** (strict: fewer → Form on Object Page; more → rethink the task).
- User is **not an expert** at this task (or task is new/unfamiliar).
- Task benefits from a **summary/review screen** before commit.

*Runner-up: if the user is an expert and the task is daily, use a plain Form inside an Object Page instead.*

### Q3 — Does the user need an at-a-glance dashboard aggregating data from multiple apps/sources?

→ **Overview Page (OVP)**. See `references/overview-page.md`.

Qualifiers (all must hold):
- Aggregates **at least two** different business objects/apps.
- Content is **read-and-navigate**, never edit-in-place.
- **Cannot** be placed inside a Flexible Column Layout — OVP is always a standalone app.

*Runner-up: Analytical List Page (single dataset with drilldown) or List Report (one object type).*

### Q4 — Does the user need to work with a list of objects?

Sub-classify:

- **Large, potentially unknown dataset where the user needs to filter/search to find items, then act on them** → **List Report** (`references/list-report.md`).
  - FilterBar is **mandatory**.
  - Search field goes in the header, not in the table.
  - Line-item navigation to an Object Page via chevron.

- **Bounded, predefined queue of items the user must process** ("my approvals", "unassigned tickets") → **Worklist** (`references/worklist.md`).
  - No full FilterBar; tabs/SegmentedButton for predefined views with counters.
  - In SAP Fiori elements v4, Worklist is "List Report with FilterBar disabled".

- **Analytical use case: chart + table, KPI header, visual filters, drilldown for root-cause analysis** → **Analytical List Page** (`references/analytical-list-page.md`).
  - If they just need to see data with filters, it's a List Report, not an ALP.

### Q5 — Does the user need to display, edit, or create a single business object with multiple logical sections?

→ **Object Page** (`references/object-page.md`).

This is the **default recommended floorplan** for object representation in modern Fiori. Key sizing rules for in-page tables:
- ≤ 50–100 rows: anchor navigation with a full in-section table is fine.
- 100–400 rows: switch from anchor to tab navigation so each section becomes its own tab.
- 400+ rows: do not embed the full table. Show a preview of the top 10–20 rows with a "Show All (x)" link to a dedicated List Report.

### Q6 — Does the user need list ↔ detail ↔ sub-detail navigation side by side?

→ **Flexible Column Layout (FCL)** (`references/flexible-column-layout.md`).

Hard restrictions:
- FCL **cannot contain an Overview Page** in any column.
- FCL **cannot contain the launchpad**.
- For list-detail-detail hierarchical navigation only — not for workbenches, dashboards, or two unrelated apps side by side.
- Columns pass **HTML elements as slot props**; each column component must spread `slot={props.slot}` on its **root HTML element** — this is the single most common integration bug.

### If nothing matches

If the user's need is a single simple form, a settings page, or something else outside the floorplan set, scaffold a plain `DynamicPage` (`references/dynamic-page.md`) with an appropriate `Form` — no floorplan is needed.

---

## Step 2 — Communicate the decision

Before (or alongside) generating code, give the user a short, structured rationale (~5 sentences):

1. **Floorplan name** and a one-sentence justification keyed to the classification question.
2. **One or two key features the floorplan gives them for free** (snapping header, anchor navigation, variant management, etc.).
3. **Named runner-up** with a one-sentence "switch to this if X" pivot.
4. **Any Fiori-specific constraint that changes their design** (e.g. "OVP must be standalone, so we won't nest it in your FCL").

Skip the rationale only if the user says "just give me the code" — and even then, open with a one-sentence naming of the floorplan so the user can catch a mis-pick.

---

## Step 3 — Scaffold the code

### Floorplan ↔ component mapping in webcomponents-react v2

| Fiori floorplan | Primary components | Notes |
|---|---|---|
| Initial Page | `DynamicPage` + centered `Input` (with `showSuggestions` or value help) | No dedicated component. |
| Wizard | `Wizard` + `WizardStep` + custom next/previous `Button`s **inside each step** | Wizard does not provide built-in next buttons. |
| Overview Page | `DynamicPage` + `FilterBar` (global) + responsive grid of `Card` + `CardHeader` | Use CSS Grid for card layout. `CardHeader.interactive` + `onClick` for navigation. |
| List Report | `DynamicPage` + `DynamicPageTitle` (with `actionsBar`) + `FilterBar` in `headerArea` + `AnalyticalTable` | FilterBar mandatory. `VariantManagement` can replace title for saved views. |
| Worklist | `DynamicPage` + `TabContainer` + `Tab` (with counter via `additionalText`) + `AnalyticalTable` / `Table` + `IllustratedMessage` | No FilterBar. |
| Analytical List Page | `DynamicPage` + `FilterBar` + `SegmentedButton` + chart from `@ui5/webcomponents-react-charts` + `AnalyticalTable` | KPI in title. Chart and table synced. |
| Object Page | `ObjectPage` + `ObjectPageTitle` + `ObjectPageHeader` + `ObjectPageSection` (with required `titleText`) + `ObjectPageSubSection` | Slots: `titleArea`, `headerArea`, `footerArea`. |
| Flexible Column Layout | `FlexibleColumnLayout` with `startColumn` / `midColumn` / `endColumn` + `FCLLayout` enum | Each column component must forward `slot={props.slot}` to a real HTML element at its root. |

Supporting app chrome (almost always used):

- `ShellBar` + `ShellBarItem` — top-level app header (see `references/shell-and-providers.md`).
- `SideNavigation` + items — left nav when the app has multiple pages.
- `ThemeProvider` — wraps the app once.
- `BusyIndicator` — loading states.
- `MessageToast`, `MessageBox`, `MessagePopover`, `MessageStrip`, `MessageView` — see `references/global-patterns.md` §4.
- `IllustratedMessage` — empty / error / success states. Always built-in illustrations; never roll your own.

---

## Step 4 — Apply the Fiori UX guardrails

After generating code, cross-check it against these high-leverage rules. Each floorplan's reference file has the full list; these are the cross-cutting ones. The full cross-cutting ruleset lives in `references/global-patterns.md`; this is the quick-check summary.

### Action placement
1. **Finalizing actions in footer, non-finalizing in title actionsBar.** Page-level primary goes in the footer.
2. **One Emphasized button per page.** Demote any second primary to Default or Transparent.
3. **Semantic (Positive/Negative) and Emphasized don't mix** in one toolbar.
4. **Create in the table toolbar** (List Report / Worklist), not in the page title.
5. **Row actions inline** (terminal Cell) with `stopPropagation()`. Never in the page footer.
6. **Column settings via the column header popover**, not via Sort/Filter/Group buttons in the table toolbar.

### Object handling
7. **Never put editable fields in the ObjectPage header.** The header collapses on scroll.
8. **Create button labels: "Create" for new, "Add" for existing.** Finalizing button labels: "Create" (on create page), "Save" (on edit page).
9. **Cancel → quick confirmation popover** ("Discard this draft?" or "Discard all changes?"). Delete → confirmation dialog.
10. **Pick one of partial / local / global flow** for objects with subpages; never mix.

### Messaging
11. **Success → MessageToast.** Decision required → MessageBox. Multiple field issues in edit mode → MessagePopover button on the left of the footer toolbar. Priority order: Error > Warning > Success > Information.
12. **Empty state text uses business nouns**, not "data" or "object". "No suppliers found / Try adjusting your filter settings", not "No data available".

### Structure & navigation
13. **Never hide the FilterBar on a List Report** — it's the defining UI. If the user wants "a table without filters", they want a Worklist.
14. **Never combine a FilterBar with the table's own filter/search.** They fight each other.
15. **Sticky headers on tables.** Sticky behavior on table toolbar, column headers, and (if present) the TabContainer.
16. **Don't nest OVP inside FCL.** OVP must be standalone.
17. **One floorplan per FCL column.**
18. **Sections cannot contain controls directly** — only subsections can. Wrap forms/tables in an `ObjectPageSubSection`.
19. **Display↔Edit toggle does not create a history entry.** No browser-back to a previous edit state.

### Accessibility
20. **Every icon-only button has a `tooltip`.** Every form field has a `<Label>`. Every illustrated message has a text message (illustration optional).
21. **F6 jumps between logical groups.** Don't over-nest; keep shell / nav / content / footer as distinct focusable regions.

---

## Step 5 — Handoff

Tell the user briefly:

- Which file to place the scaffold in (e.g. `app/customers/[id]/page.tsx` for an Object Page, plus `app/customers/[id]/page-client.tsx` for the actual content).
- What peer deps they need (`@ui5/webcomponents-react`, `@ui5/webcomponents`, `@ui5/webcomponents-fiori` if using ShellBar / FilterBar / VariantManagement; `@ui5/webcomponents-icons` for icons).
- Where the `TODO:` hooks are (data fetching, navigation, action handlers).
- One or two Fiori-specific follow-ups ("consider adding `VariantManagement` when users need saved filter sets", "add `BusyIndicator` at page level while fetching", "wire up Ctrl+S to Save").
- The self-review checklist in `references/global-patterns.md` (§7 end) — tell the user to run through it once before shipping.

---

## Reference files

Read the file(s) that match the chosen floorplan before you write code. Each file contains: when-to-use criteria, full Fiori do's and don'ts specific to that floorplan, v2 API reference for the components involved, and a complete working scaffold.

**Cross-cutting (start here for UX rules):**
- `references/global-patterns.md` — Navigation, action placement, object handling, messaging, empty states, loading, keyboard & accessibility. Always relevant.
- `references/action-placement.md` — Where actions go + Fiori action-type taxonomy + styling mapping. Always relevant.

**Floorplans:**
- `references/object-page.md` — Object Page (the default for objects)
- `references/list-report.md` — List Report (find & act on large lists)
- `references/worklist.md` — Worklist (bounded processing queue)
- `references/analytical-list-page.md` — Analytical List Page (BI-flavored list with KPI, chart, drilldown)
- `references/overview-page.md` — Overview Page (role-based dashboard of cards)
- `references/wizard.md` — Wizard (guided multi-step creation/editing)
- `references/initial-page.md` — Initial Page (navigate to one object by ID)
- `references/flexible-column-layout.md` — FCL (list-detail-detail side-by-side)
- `references/dynamic-page.md` — Dynamic Page primitives (the foundation of most floorplans)

**Supporting:**
- `references/shell-and-providers.md` — ShellBar, SideNavigation, ThemeProvider, Next.js integration, icons, theming. Read once per project.
- `references/tables.md` — Choosing between AnalyticalTable, Table, and List; Fiori-compliant patterns.
- `references/v2-migration-and-api-deltas.md` — v1 → v2 breaking changes (useful for migrations).

If the user wants a floorplan not covered — "create page", "fact sheet", "object view" (the old flat view), "split-screen" — map it to the closest covered floorplan and tell them. Create page ≈ single-section Object Page or Wizard depending on complexity. Object view (flat) → Object Page. Split-screen → Flexible Column Layout (it's the replacement).

---

## Short-circuit: the user already knows what they want

If the user says "build me an object page for Customer with fields X, Y, Z", you don't need the full classification discussion. Open with one sentence naming the floorplan, read `references/global-patterns.md` for the cross-cutting rules and `references/object-page.md` for the floorplan, then generate the code. Save the longer rationale for cases where the choice is ambiguous or the user pushed back.

## Short-circuit: the user is reviewing existing code

If the user shares code and asks "is this Fiori-compliant?", classify what floorplan it attempts, read the relevant floorplan file plus `references/global-patterns.md` and `references/action-placement.md`, and go through the do's/don'ts list checking each against the code. Point to specific violations with Fiori rationale, not just "this is wrong." Suggest concrete fixes, showing small code diffs. Focus areas (in order) for review:

1. Action placement and button styling (most common issue).
2. Slot names and v2 API (if migrating from v1).
3. Empty and loading states.
4. Data-loss and confirmation patterns.
5. Accessibility (tooltips, labels, keyboard support).
