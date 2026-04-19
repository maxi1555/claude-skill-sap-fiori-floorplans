# Shell and Providers (Next.js App Router integration)

The "outside the page" concerns: app-level chrome (ShellBar, SideNavigation), providers (ThemeProvider), one-time asset imports, Next.js SSR handling, and theming. You do this setup **once per app**; every page then lives inside it.

## Why this file exists

`@ui5/webcomponents-react` is a wrapper around custom elements that mount to the DOM on the client. They don't work in Next.js server components. This file shows the canonical integration, which — if you get it right once — lets every page be a normal `"use client"` page without fighting hydration errors or missing assets.

## The one-time setup

### Install

```bash
npm install @ui5/webcomponents-react @ui5/webcomponents @ui5/webcomponents-fiori @ui5/webcomponents-icons @ui5/webcomponents-react-charts
```

- `@ui5/webcomponents-react` — the React wrapper (main API).
- `@ui5/webcomponents` — the base web components (required peer dep).
- `@ui5/webcomponents-fiori` — ShellBar, FilterBar, VariantManagement, Wizard, ObjectPage, FlexibleColumnLayout, IllustratedMessage. Technically an **optional** peer since v2.14 if your bundler tree-shakes well, but in practice install it — most Fiori pages use at least one Fiori-package component.
- `@ui5/webcomponents-icons` — all SAP icons.
- `@ui5/webcomponents-react-charts` — only if you use charts.

### File layout (Next.js App Router)

```
app/
├── providers.tsx         ← client component with ThemeProvider + Assets + theme setup
├── layout.tsx            ← server component that renders <Providers> around children
├── shell.tsx             ← ShellBar + SideNavigation wrapper (client component)
├── (app)/
│   ├── layout.tsx        ← wraps app pages with <Shell>
│   ├── customers/
│   │   ├── page.tsx           ← dynamic(() => import('./page-client'), { ssr: false })
│   │   └── page-client.tsx    ← the actual 'use client' page using UI5 components
│   └── …
```

### `providers.tsx` — ThemeProvider + one-time Assets import

```tsx
'use client';

import { ReactNode, useEffect } from 'react';
import { ThemeProvider } from '@ui5/webcomponents-react';

// Register assets ONCE for the whole app. This pulls in i18n, CLDR, icon registration,
// and illustration SVGs. Without this, icons render as empty squares and strings are untranslated.
import '@ui5/webcomponents-react/dist/Assets.js';

// Optional: explicit theme selection. sap_horizon is the default Fiori 3+ theme.
import { setTheme } from '@ui5/webcomponents-base/dist/config/Theme.js';

export function Providers({ children }: { children: ReactNode }) {
  useEffect(() => {
    // Runs only on the client after hydration. Safe place to set theme.
    setTheme('sap_horizon');
  }, []);

  return <ThemeProvider>{children}</ThemeProvider>;
}
```

### `app/layout.tsx` — root layout

```tsx
import { ReactNode } from 'react';
import { Providers } from './providers';

export const metadata = {
  title: 'My Fiori App',
};

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en">
      <body style={{ margin: 0 }}>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

Note: `<body>` has `margin: 0` — UI5 components assume a zero-margin body.

### `app/shell.tsx` — ShellBar + SideNavigation

```tsx
'use client';

import { ReactNode, useState } from 'react';
import {
  ShellBar,
  ShellBarItem,
  Avatar,
  SideNavigation,
  SideNavigationItem,
  SideNavigationGroup,
  FlexBox,
  FlexBoxDirection,
} from '@ui5/webcomponents-react';

import homeIcon from '@ui5/webcomponents-icons/dist/home.js';
import customerIcon from '@ui5/webcomponents-icons/dist/customer.js';
import salesOrderIcon from '@ui5/webcomponents-icons/dist/sales-order.js';
import employeeIcon from '@ui5/webcomponents-icons/dist/employee.js';
import actionIcon from '@ui5/webcomponents-icons/dist/action-settings.js';

import { usePathname, useRouter } from 'next/navigation';

export function Shell({ children }: { children: ReactNode }) {
  const router = useRouter();
  const pathname = usePathname();
  const [navCollapsed] = useState(false);

  const navigate = (path: string) => () => router.push(path);

  return (
    <FlexBox direction={FlexBoxDirection.Column} style={{ height: '100vh', width: '100%' }}>
      <ShellBar
        primaryTitle="Customer Central"
        secondaryTitle="Operations"
        showNotifications
        notificationsCount="3"
        showProductSwitch
        logo={<img src="/logo.svg" alt="Company logo" />}
        profile={
          <Avatar>
            <img src="/avatar.png" alt="User avatar" />
          </Avatar>
        }
        onLogoClick={navigate('/')}
        onProfileClick={() => {
          /* open profile menu */
        }}
      >
        <ShellBarItem icon={actionIcon} text="Settings" onClick={navigate('/settings')} />
      </ShellBar>

      <FlexBox style={{ flex: 1, minHeight: 0 }}>
        <SideNavigation
          collapsed={navCollapsed}
          onSelectionChange={(e) => {
            const path = (e.detail.item as HTMLElement).dataset.path;
            if (path) router.push(path);
          }}
        >
          <SideNavigationItem
            text="Home"
            icon={homeIcon}
            data-path="/"
            selected={pathname === '/'}
          />
          <SideNavigationGroup text="Customers">
            <SideNavigationItem
              text="All Customers"
              icon={customerIcon}
              data-path="/customers"
              selected={pathname.startsWith('/customers')}
            />
            <SideNavigationItem
              text="Leads"
              icon={employeeIcon}
              data-path="/leads"
              selected={pathname.startsWith('/leads')}
            />
          </SideNavigationGroup>
          <SideNavigationGroup text="Sales">
            <SideNavigationItem
              text="Orders"
              icon={salesOrderIcon}
              data-path="/orders"
              selected={pathname.startsWith('/orders')}
            />
          </SideNavigationGroup>
        </SideNavigation>

        <main style={{ flex: 1, minWidth: 0, overflow: 'auto' }}>{children}</main>
      </FlexBox>
    </FlexBox>
  );
}
```

### `app/(app)/layout.tsx` — wrap app pages with Shell

```tsx
import { ReactNode } from 'react';
import { Shell } from '../shell';

export default function AppLayout({ children }: { children: ReactNode }) {
  return <Shell>{children}</Shell>;
}
```

### A page: `app/(app)/customers/page.tsx`

```tsx
import dynamic from 'next/dynamic';

// Disable SSR for pages that use UI5 components.
// UI5 is a set of custom elements and cannot render on the server.
const CustomersPageClient = dynamic(() => import('./page-client'), { ssr: false });

export default function Page() {
  return <CustomersPageClient />;
}
```

### The actual page: `app/(app)/customers/page-client.tsx`

```tsx
'use client';

import { DynamicPage, DynamicPageTitle, Title } from '@ui5/webcomponents-react';

export default function CustomersPageClient() {
  return (
    <DynamicPage titleArea={<DynamicPageTitle header={<Title>Customers</Title>} />}>
      {/* ... */}
    </DynamicPage>
  );
}
```

## Why disable SSR per page?

UI5 web components rely on custom elements, which exist only in the browser. Next.js will try to server-render by default. The cleanest pattern:

- **Root and shell layouts**: use `"use client"`. They mount on the client; their children are also client-rendered.
- **Each page that uses UI5 components**: export a thin server component that `dynamic()`-imports the real page-client with `ssr: false`.

This gives you:
- No hydration mismatch errors.
- No "window is not defined" errors from UI5's deep internals.
- Clear separation: server = routing/metadata, client = UI.

If you find the extra file annoying, an alternative is to put `"use client"` directly on a single `page.tsx` and accept that UI5 web components mount in an effect pass. In practice, `dynamic() + ssr: false` is the battle-tested approach.

## Icons — how to use them

UI5 icons are individual ES modules. Import the specific icon, get back a token, pass it as `icon={…}`.

```tsx
import addIcon from '@ui5/webcomponents-icons/dist/add.js';
import editIcon from '@ui5/webcomponents-icons/dist/edit.js';
import deleteIcon from '@ui5/webcomponents-icons/dist/delete.js';
import shareIcon from '@ui5/webcomponents-icons/dist/share.js';
import searchIcon from '@ui5/webcomponents-icons/dist/search.js';

<Button icon={addIcon}>Create</Button>
<Button icon={editIcon} design="Transparent">Edit</Button>
```

Do **not** use string names like `icon="add"` — that works in vanilla UI5 web components but not in the React wrapper without first registering the icon collection. Individual imports are both tree-shakable and self-registering.

### Finding icon names

The full catalog is at the SAP Icon Explorer. Icon file names match the icon IDs: `sap-icon://add` → `@ui5/webcomponents-icons/dist/add.js`.

## Theming

### Available themes

The current default is **`sap_horizon`** (Fiori 3+ light). Other built-in themes:

- `sap_horizon_dark`
- `sap_horizon_hcb` (high-contrast black)
- `sap_horizon_hcw` (high-contrast white)
- `sap_fiori_3` (older Fiori 3 default)
- `sap_fiori_3_dark`
- `sap_horizon_exp` (experimental)

### Setting a theme at runtime

```tsx
import { setTheme } from '@ui5/webcomponents-base/dist/config/Theme.js';

setTheme('sap_horizon_dark');
```

Call this in a `useEffect` in your providers or in a theme-toggle handler. The theme switch is live — all mounted components restyle.

### Persisting theme choice

```tsx
'use client';

import { useEffect, useState } from 'react';
import { setTheme } from '@ui5/webcomponents-base/dist/config/Theme.js';

export function useUserTheme(defaultTheme = 'sap_horizon') {
  const [theme, setLocalTheme] = useState(defaultTheme);

  useEffect(() => {
    const stored = typeof window !== 'undefined' ? localStorage.getItem('ui5-theme') : null;
    const initial = stored ?? defaultTheme;
    setTheme(initial);
    setLocalTheme(initial);
  }, [defaultTheme]);

  const change = (next: string) => {
    setTheme(next);
    setLocalTheme(next);
    localStorage.setItem('ui5-theme', next);
  };

  return { theme, setTheme: change };
}
```

### Fiori CSS variables

Fiori themes expose a large set of CSS variables for spacing, colors, typography. Use them instead of hardcoding.

Commonly useful:
- `--sapContent_Space_S`, `--sapContent_Space_M`, `--sapContent_Space_L`, `--sapContent_Space_XL` — responsive spacing tokens.
- `--sapContent_Margin_Small`, `--sapContent_Margin_Medium`, `--sapContent_Margin_Large` — margins.
- `--sapBrandColor`, `--sapBackgroundColor`, `--sapContent_ForegroundColor` — core colors.
- `--sapList_Background`, `--sapList_Hover_Background` — list backgrounds.
- `--sapNegativeColor`, `--sapPositiveColor`, `--sapCriticalColor`, `--sapInformationColor` — semantic colors.
- `--sapFontSize`, `--sapFontLargeSize`, `--sapFontSmallSize` — typography.

Example:
```tsx
<div style={{ padding: 'var(--sapContent_Space_M)', background: 'var(--sapBackgroundColor)' }}>…</div>
```

Note: the v1 `@ui5/webcomponents-react-base` `spacing` helper object (e.g. `spacing.sapUiContentPadding`) was removed in v2 — those spacing variables no longer exist as CSS vars. Use the `--sapContent_Space_*` variables directly.

## ShellBar — full API cheat sheet

**Props:**
- `primaryTitle` — main app title (hidden on S screens < ~700px).
- `secondaryTitle` — secondary title (hidden on S and M screens).
- `logo` — slot taking `<img>` or `<Avatar>`.
- `profile` — slot taking `<Avatar>`.
- `searchField` — slot for a search input.
- `showSearchField` — boolean; if true, search input slot is rendered (default false).
- `showNotifications` — boolean.
- `notificationsCount` — string displayed on the notifications bell.
- `showProductSwitch` — boolean; shows the product switch icon.
- `showCoPilot` — boolean; shows an SAP CoPilot icon slot (for Joule integration scenarios).
- `startButton` — slot for a back / menu button on the very left.
- `menuItems` — slot for navigation menu items.

**Events:**
- `onLogoClick` — usually navigate home.
- `onProfileClick` — open profile menu/dialog.
- `onNotificationsClick` — open notifications pane.
- `onProductSwitchClick` — open product switch.
- `onSearchButtonClick` — toggle search field visibility.
- `onMenuItemClick` — fired for menu item slot selections.

**Children:** `<ShellBarItem>` for right-side custom actions.

### ShellBarItem

- `icon` — icon module (imported).
- `text` — label shown in overflow menu.
- `tooltip` — hover tooltip.
- `count` — badge count.
- `onClick` — click handler.

## SideNavigation — full API cheat sheet

- `collapsed` — boolean; when true, only icons are shown.
- `fixedItems` — slot for items pinned to the bottom (e.g. Settings, Help, Sign Out).
- `header` — optional header slot.
- `onSelectionChange` — fires on item select; `e.detail.item` is the selected `<SideNavigationItem>`.

### SideNavigationItem

- `text` — label.
- `icon` — icon module.
- `selected` — boolean, controlled selection.
- `expanded` — boolean, for items with child items.
- `children` — nested `<SideNavigationItem>` for submenu.

### SideNavigationGroup

- `text` — group header.
- `children` — group items.

## Anti-patterns

- **Rendering UI5 pages as server components.** You'll get hydration errors and missing icons. Always use `"use client"` or `dynamic(..., { ssr: false })`.
- **Multiple `ThemeProvider` instances.** One at the app root is all you need; nested providers conflict.
- **Forgetting `@ui5/webcomponents-react/dist/Assets.js`.** Icons will render as empty squares; i18n will fall back to English.
- **String icon names in the React wrapper.** `icon="add"` doesn't auto-register in the wrapper. Use module imports.
- **Hard-coded colors / paddings.** Use Fiori CSS variables so dark mode and HCB work.
- **Putting the shell inside individual pages** instead of in a layout. You'll mount/unmount it on every navigation.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents`
- `@ui5/webcomponents-fiori` (for ShellBar, SideNavigation)
- `@ui5/webcomponents-icons`
