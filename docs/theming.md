# Theming StellarKit UI

StellarKit UI is designed to fit into product interfaces instead of forcing a single visual style. This guide explains the theme layers used by the project today and the recommended customization points for applications built with Tailwind CSS and Next.js.

## Theme Layers

Use the layers in this order:

1. Global CSS variables for app-wide foreground, background, and fonts.
2. Tailwind theme tokens for reusable utility classes.
3. `StellarKitProvider` theme values for StellarKit UI component defaults.
4. Per-component `className` props for local overrides.

Keeping these layers separate makes it easier to ship a branded app without patching component internals.

## Current CSS Variables

The current app defines these variables in `app/globals.css`.

| Variable | Default | Dark-mode value | Purpose |
| --- | --- | --- | --- |
| `--background` | `#ffffff` | `#0a0a0a` | Page background color used by `body` and mapped into Tailwind. |
| `--foreground` | `#171717` | `#ededed` | Primary text color used by `body` and mapped into Tailwind. |
| `--color-background` | `var(--background)` | `var(--background)` | Tailwind v4 inline color token for `bg-background`. |
| `--color-foreground` | `var(--foreground)` | `var(--foreground)` | Tailwind v4 inline color token for `text-foreground`. |
| `--font-sans` | `var(--font-geist-sans)` | `var(--font-geist-sans)` | Tailwind font token for sans-serif UI text. |
| `--font-mono` | `var(--font-geist-mono)` | `var(--font-geist-mono)` | Tailwind font token for code, keys, and hashes. |

## Add StellarKit UI Component Tokens

Applications can extend the base variables with component-specific tokens. Prefixing them with `--sk-` keeps them distinct from app tokens.

```css
/* app/globals.css */
:root {
  --background: #ffffff;
  --foreground: #171717;

  --sk-surface: #ffffff;
  --sk-surface-muted: #f8fafc;
  --sk-border: #dbe4ef;
  --sk-primary: #2563eb;
  --sk-primary-foreground: #ffffff;
  --sk-success: #16a34a;
  --sk-warning: #d97706;
  --sk-danger: #dc2626;
  --sk-radius: 0.75rem;
}

@media (prefers-color-scheme: dark) {
  :root {
    --background: #0a0a0a;
    --foreground: #ededed;

    --sk-surface: #111827;
    --sk-surface-muted: #1f2937;
    --sk-border: #334155;
    --sk-primary: #60a5fa;
    --sk-primary-foreground: #0f172a;
    --sk-success: #4ade80;
    --sk-warning: #fbbf24;
    --sk-danger: #f87171;
  }
}
```

## Tailwind v4 Customization

The project uses Tailwind v4 through `@import "tailwindcss"` and `@theme inline`. Map your CSS variables into Tailwind tokens so the same theme values work in components and ordinary page markup.

```css
@import "tailwindcss";

:root {
  --background: #ffffff;
  --foreground: #171717;
  --sk-surface: #ffffff;
  --sk-border: #dbe4ef;
  --sk-primary: #2563eb;
}

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-sk-surface: var(--sk-surface);
  --color-sk-border: var(--sk-border);
  --color-sk-primary: var(--sk-primary);
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);
}
```

After mapping the tokens, use them with Tailwind utilities:

```tsx
export function WalletPanel({ children }: { children: React.ReactNode }) {
  return (
    <section className="rounded-xl border border-sk-border bg-sk-surface p-4 text-foreground">
      {children}
    </section>
  );
}
```

## Tailwind v3 Config Variant

If your consuming app still uses Tailwind v3, map the same variables through `tailwind.config.ts`.

```ts
// tailwind.config.ts
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./app/**/*.{ts,tsx}",
    "./components/**/*.{ts,tsx}",
    "./node_modules/@stellarkit/ui/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        background: "var(--background)",
        foreground: "var(--foreground)",
        sk: {
          surface: "var(--sk-surface)",
          border: "var(--sk-border)",
          primary: "var(--sk-primary)",
          danger: "var(--sk-danger)",
        },
      },
      borderRadius: {
        sk: "var(--sk-radius)",
      },
    },
  },
};

export default config;
```

## StellarKitProvider Theme Prop

Use the provider for library-wide defaults that should be shared by wallet controls, account displays, payment forms, and transaction components.

```tsx
"use client";

import { StellarKitProvider } from "@stellarkit/ui";

const theme = {
  colors: {
    surface: "var(--sk-surface)",
    surfaceMuted: "var(--sk-surface-muted)",
    border: "var(--sk-border)",
    primary: "var(--sk-primary)",
    primaryForeground: "var(--sk-primary-foreground)",
    success: "var(--sk-success)",
    warning: "var(--sk-warning)",
    danger: "var(--sk-danger)",
  },
  radius: {
    md: "var(--sk-radius)",
  },
};

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <StellarKitProvider network="testnet" theme={theme}>
      {children}
    </StellarKitProvider>
  );
}
```

Keep provider values as CSS variables when possible. That allows dark mode and brand changes to update without remounting React components.

## Per-Component Overrides

Use component-level classes for one-off layout or emphasis changes.

```tsx
import { PaymentForm } from "@stellarkit/ui";

export function CheckoutPayment() {
  return (
    <PaymentForm
      className="rounded-2xl border border-sk-border bg-sk-surface p-5 shadow-sm"
      submitButtonClassName="bg-sk-primary text-white hover:opacity-90"
    />
  );
}
```

Prefer per-component classes for spacing and layout. Prefer provider theme values for shared colors, borders, and status states.

## Dark Mode Example

For system-driven dark mode, keep the existing `prefers-color-scheme` media query and override both app variables and StellarKit UI variables.

```css
@media (prefers-color-scheme: dark) {
  :root {
    --background: #0a0a0a;
    --foreground: #ededed;
    --sk-surface: #111827;
    --sk-surface-muted: #1f2937;
    --sk-border: #334155;
    --sk-primary: #60a5fa;
    --sk-primary-foreground: #0f172a;
  }
}
```

For a manual toggle, set a class or data attribute on the document root.

```tsx
"use client";

import { useEffect, useState } from "react";

export function ThemeToggle() {
  const [dark, setDark] = useState(false);

  useEffect(() => {
    document.documentElement.dataset.theme = dark ? "dark" : "light";
  }, [dark]);

  return (
    <button type="button" onClick={() => setDark((value) => !value)}>
      {dark ? "Use light theme" : "Use dark theme"}
    </button>
  );
}
```

```css
:root[data-theme="dark"] {
  --background: #0a0a0a;
  --foreground: #ededed;
  --sk-surface: #111827;
  --sk-border: #334155;
  --sk-primary: #60a5fa;
}
```

## Accessibility Notes

- Keep foreground and background contrast at or above WCAG AA for normal text.
- Do not rely on color alone for transaction status; pair color with labels such as `Pending`, `Confirmed`, or `Failed`.
- Preserve visible focus rings when overriding button, input, and link styles.
- Test both light and dark themes with wallet connection, payment submission, and error states.

## Recommended Token Checklist

Before shipping a custom theme, define and review these values:

- Page colors: `--background`, `--foreground`
- Surfaces: `--sk-surface`, `--sk-surface-muted`, `--sk-border`
- Brand action: `--sk-primary`, `--sk-primary-foreground`
- Status states: `--sk-success`, `--sk-warning`, `--sk-danger`
- Shape: `--sk-radius`
- Fonts: `--font-sans`, `--font-mono`
