# Initial Page

The Fiori floorplan for **navigating to a single known object by its identifier**. One input, type (or pick from suggestions), done — the user lands on the Object Page for that item.

> **Also consult** `references/global-patterns.md` (navigation — deep-link short-circuit rule, error states when the ID doesn't exist). The `UnableToLoad` empty state is the right response to a missing ID.

Classic example: "Track Purchase Order". The user knows (or can look up) the PO number and wants to pull up its Object Page. They are not browsing.

## When to use

- User has (or can find) the **identifier** of one specific object.
- You want to get them into the Object Page as fast as possible.
- There's **one primary input**, not a form.

## When NOT to use

- User is browsing / filtering / comparing → **List Report**.
- User is searching across many object types / roles → use the SAP Fiori launchpad enterprise search, not an app-scoped Initial Page.
- You want to "show the user a few options on the landing page" → that's a small Overview Page, not an Initial Page.

## What you get for free

- A clean, centered focal input with generous whitespace.
- Value-help dialog (F4) support via `Input` + `SuggestionItem` or a dedicated `ValueHelpDialog`.
- Search-as-you-type via `Input` with `showSuggestions`.
- Deep-link support (when the URL carries the identifier, skip the Initial Page entirely and go straight to the Object Page).

## Fiori design rules

- **Exactly one primary input.** The whole point is "one field → one object".
- **Value help or live suggestions required.** Users won't always remember the exact identifier.
- **Enter key submits.**
- **Pre-fill from URL query param** when deep-linked. If the identifier is valid, skip the Initial Page and redirect to the Object Page.
- **Keep the page otherwise empty.** Maybe a short helper sentence above the input, nothing else. Resist the urge to add "recent items", "popular items", etc — that starts turning into an Overview Page.
- **Header is a plain `DynamicPageTitle`** with just the page title (e.g. "Track Purchase Order"). No subtitle needed since the rest of the page is clearly an input.

## v2 component API cheat sheet

- `Input` — the focal input. Props: `value`, `placeholder`, `showSuggestions`, `onInput`, `onKeyDown`, `onSuggestionItemSelect`.
- `SuggestionItem` — `text`, `additionalText`, `description`, `image`.
- `ValueHelpDialog` — for a full F4 dialog. Lives in `@ui5/webcomponents-react-compat` in v2 — use it only if suggestions aren't rich enough.
- `FlexBox` — for centering the input vertically and horizontally.
- `FlexBoxDirection`, `FlexBoxAlignItems`, `FlexBoxJustifyContent` — enum helpers for layout.

## Production scaffold

```tsx
'use client';

import {
  DynamicPage,
  DynamicPageTitle,
  Title,
  Label,
  Input,
  SuggestionItem,
  Button,
  FlexBox,
  FlexBoxDirection,
  FlexBoxAlignItems,
  FlexBoxJustifyContent,
  MessageStrip,
} from '@ui5/webcomponents-react';

import searchIcon from '@ui5/webcomponents-icons/dist/search.js';

import { useEffect, useMemo, useState } from 'react';

type Suggestion = {
  id: string;
  description: string;
};

type Props = {
  /** Known POs for suggestions. In production, debounce-fetch from the backend instead. */
  suggestions: Suggestion[];
  /** Called when the user picks / submits a valid ID. */
  onOpen: (id: string) => void;
  /** Optional pre-filled ID (from URL query param, e.g. /track?id=PO-1001). */
  initialId?: string;
};

export default function TrackOrderInitialPage({ suggestions, onOpen, initialId }: Props) {
  const [value, setValue] = useState(initialId ?? '');
  const [error, setError] = useState<string | null>(null);

  // Deep-link: if initialId matches a valid item, navigate immediately.
  useEffect(() => {
    if (initialId && suggestions.some((s) => s.id === initialId)) {
      onOpen(initialId);
    }
  }, [initialId, suggestions, onOpen]);

  const matches = useMemo(() => {
    if (!value) return suggestions.slice(0, 10);
    const needle = value.toLowerCase();
    return suggestions
      .filter((s) => s.id.toLowerCase().includes(needle) || s.description.toLowerCase().includes(needle))
      .slice(0, 10);
  }, [value, suggestions]);

  const submit = () => {
    const picked = suggestions.find((s) => s.id === value);
    if (!picked) {
      setError(`No order found with ID "${value}". Check the ID or use a suggestion.`);
      return;
    }
    setError(null);
    onOpen(picked.id);
  };

  return (
    <DynamicPage
      titleArea={<DynamicPageTitle header={<Title>Track Purchase Order</Title>} />}
    >
      <FlexBox
        direction={FlexBoxDirection.Column}
        alignItems={FlexBoxAlignItems.Center}
        justifyContent={FlexBoxJustifyContent.Center}
        style={{ minHeight: '60vh', gap: 'var(--sapContent_Space_M)' }}
      >
        <Label>Enter a purchase order number to view its details.</Label>

        <Input
          icon={<Button icon={searchIcon} design="Transparent" onClick={submit} accessibleName="Open" />}
          placeholder="e.g. PO-1001"
          showSuggestions
          value={value}
          onInput={(e) => {
            setValue((e.target as HTMLInputElement).value);
            setError(null);
          }}
          onKeyDown={(e) => {
            if (e.key === 'Enter' && value.trim().length > 0) {
              submit();
            }
          }}
          onSuggestionItemSelect={(e) => {
            const picked = (e.detail.item as HTMLElement).dataset.id ?? '';
            if (picked) {
              setValue(picked);
              onOpen(picked);
            }
          }}
          style={{ inlineSize: '360px' }}
        >
          {matches.map((s) => (
            <SuggestionItem key={s.id} data-id={s.id} text={s.id} additionalText={s.description} />
          ))}
        </Input>

        <Button design="Emphasized" disabled={!value.trim()} onClick={submit}>
          Open
        </Button>

        {error && (
          <MessageStrip design="Negative" hideCloseButton style={{ maxInlineSize: '360px' }}>
            {error}
          </MessageStrip>
        )}
      </FlexBox>
    </DynamicPage>
  );
}
```

### Notes on the scaffold

- **Deep-link short-circuit**: if the URL has `?id=PO-1001` and that ID is valid, the `useEffect` skips the Initial Page entirely and navigates to the Object Page.
- **Suggestions are slice-of-backend** for illustration. In production, debounce the input and fetch suggestions from the backend.
- **Error MessageStrip** appears when the user submits an unknown ID. Fiori-compliant because the error doesn't interrupt the flow.
- **No value help dialog shown here**: the suggestions list handles typical cases. If you need a richer F4-style dialog (column selection, filtering, pagination), use `ValueHelpDialog` from `@ui5/webcomponents-react-compat` — note it's in the compat package in v2.

## Anti-patterns

- Adding a second input "because what if they don't know the ID". → Put that behind a value-help dialog instead, not as a second primary input.
- Adding "recent items" and "popular items" on the Initial Page. → That's an Overview Page.
- Not supporting Enter key submission. → Fiori apps are keyboard-friendly.
- Not supporting deep-link. → URL-based navigation should pre-fill and skip the page.
- Making the input full-width. → Fiori convention is a compact, centered input.

## Peer dependencies

- `@ui5/webcomponents-react`
- `@ui5/webcomponents`
- `@ui5/webcomponents-icons`
