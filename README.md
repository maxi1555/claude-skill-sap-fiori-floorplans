# SAP Fiori Floorplans — Claude Skill

A Claude skill that designs and scaffolds SAP Fiori pages using
`@ui5/webcomponents-react` v2 on Next.js (App Router). It picks the
right floorplan for the use case, applies the Fiori global patterns
(action placement, object handling, messaging, empty states,
accessibility), and generates production-grade TSX scaffolds verified
against the v2 API.

## Install

1. Download `sap-fiori-floorplans.skill` from the latest [Release](../../releases/latest).
2. In Claude, go to Settings → Capabilities → Skills → Upload skill.
3. Trigger it by asking Claude to build, review, or migrate any Fiori page.

## What's inside

- Classification flow for picking the right floorplan
- Production scaffolds for Object Page, List Report, Worklist, ALP,
  OVP, Wizard, Initial Page, Flexible Column Layout, Dynamic Page
- Cross-cutting references: global patterns (navigation, action
  placement, object handling, messaging, empty states, loading,
  accessibility), action placement taxonomy, tables, shell & providers,
  v1 → v2 migration

## Stack

- `@ui5/webcomponents-react` v2 (the React wrapper, not vanilla UI5
  Web Components)
- Next.js 14+ App Router

## License

MIT
