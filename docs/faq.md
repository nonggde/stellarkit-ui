# StellarKit UI FAQ

This FAQ collects the most common integration questions for developers adding StellarKit UI to a React or Next.js application. The examples assume the public package entry point is `@stellarkit/ui`; adjust the import path if your project uses a workspace alias.

## 1. How do I use StellarKit UI with the Next.js App Router?

Wrap the application in a client-side provider from `app/providers.tsx`, then render that provider from `app/layout.tsx`.

```tsx
// app/providers.tsx
"use client";

import { StellarKitProvider } from "@stellarkit/ui";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <StellarKitProvider network="testnet">
      {children}
    </StellarKitProvider>
  );
}
```

```tsx
// app/layout.tsx
import { Providers } from "./providers";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

## 2. How do I use StellarKit UI with the Pages Router?

Wrap `Component` in `_app.tsx` so every page receives the wallet, network, and Horizon context.

```tsx
// pages/_app.tsx
import type { AppProps } from "next/app";
import { StellarKitProvider } from "@stellarkit/ui";

export default function App({ Component, pageProps }: AppProps) {
  return (
    <StellarKitProvider network="testnet">
      <Component {...pageProps} />
    </StellarKitProvider>
  );
}
```

## 3. How do I connect a specific wallet such as Freighter or Albedo?

Limit the wallet list or set a preferred wallet in your connect control. Keep a fallback wallet available if your audience may not have the preferred extension installed.

```tsx
import { ConnectWalletButton } from "@stellarkit/ui";

export function WalletConnect() {
  return (
    <ConnectWalletButton
      preferredWallet="freighter"
      wallets={["freighter", "albedo"]}
    />
  );
}
```

## 4. How do I switch between testnet and mainnet?

Pass the target network to the provider and keep the Horizon URL aligned with that network. A common pattern is to choose values from environment variables.

```tsx
import { StellarKitProvider } from "@stellarkit/ui";

const network = process.env.NEXT_PUBLIC_STELLAR_NETWORK === "mainnet"
  ? "mainnet"
  : "testnet";

const horizonUrl = network === "mainnet"
  ? "https://horizon.stellar.org"
  : "https://horizon-testnet.stellar.org";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <StellarKitProvider network={network} horizonUrl={horizonUrl}>
      {children}
    </StellarKitProvider>
  );
}
```

## 5. How do I customize component styles?

Prefer component `className` props for local changes and CSS variables or Tailwind theme tokens for app-wide theming. This keeps StellarKit UI defaults intact while letting your product match its own brand.

```tsx
import { PaymentForm } from "@stellarkit/ui";

export function BrandedPaymentForm() {
  return (
    <PaymentForm
      className="rounded-xl border border-slate-200 bg-white p-4 shadow-sm"
      submitButtonClassName="bg-indigo-600 text-white hover:bg-indigo-700"
    />
  );
}
```

## 6. How do I handle transaction errors?

Treat wallet rejection, simulation failure, Horizon submission failure, and timeout as separate UI states. Show the user a concise message and keep the raw error available for logs.

```tsx
import { PaymentForm, type StellarKitError } from "@stellarkit/ui";

function getErrorMessage(error: StellarKitError) {
  if (error.code === "USER_REJECTED") return "The transaction was cancelled.";
  if (error.code === "NETWORK_ERROR") return "The Stellar network is unavailable.";
  return "The transaction could not be submitted.";
}

export function PaymentPanel() {
  return (
    <PaymentForm
      onError={(error) => {
        console.error(error);
        alert(getErrorMessage(error));
      }}
    />
  );
}
```

## 7. How do I avoid exposing secret keys?

Never put Stellar secret keys in client-side code, environment variables prefixed with `NEXT_PUBLIC_`, localStorage, or screenshots. Use wallet signing on the client and keep server-side secrets in a backend secret store.

```tsx
import { useWallet } from "@stellarkit/ui";

export function SignButton({ xdr }: { xdr: string }) {
  const { signTransaction } = useWallet();

  return (
    <button onClick={() => signTransaction(xdr)}>
      Sign with connected wallet
    </button>
  );
}
```

## 8. How do I display balances after a wallet connects?

Use the connected account public key to load balances from Horizon. Render a loading state until the account data is available and an empty state for unfunded testnet accounts.

```tsx
import { AccountBalance, useWallet } from "@stellarkit/ui";

export function BalanceSummary() {
  const { publicKey } = useWallet();

  if (!publicKey) return <p>Connect a wallet to view balances.</p>;

  return <AccountBalance publicKey={publicKey} assets={["XLM", "USDC"]} />;
}
```

## 9. How do I create a payment flow with validation?

Validate destination, asset, and amount before building or submitting a transaction. Disable submission while a transaction is pending to avoid duplicate payments.

```tsx
import { PaymentForm } from "@stellarkit/ui";

export function Checkout() {
  return (
    <PaymentForm
      assetCode="XLM"
      minAmount="1"
      onSuccess={(result) => {
        console.log("Transaction hash:", result.hash);
      }}
    />
  );
}
```

## 10. How do I prevent hydration errors in Next.js?

Render wallet-dependent UI only inside client components. If a widget reads browser APIs such as `window`, `localStorage`, or wallet extensions, keep it behind `"use client"`.

```tsx
// components/client-wallet-panel.tsx
"use client";

import { ConnectWalletButton } from "@stellarkit/ui";

export function ClientWalletPanel() {
  return <ConnectWalletButton />;
}
```

## 11. How do I test components that depend on StellarKit UI?

Wrap the component under test in `StellarKitProvider` and mock wallet methods when the test needs a connected account. Keep network calls mocked so tests are deterministic.

```tsx
import { render, screen } from "@testing-library/react";
import { StellarKitProvider } from "@stellarkit/ui";
import { BalanceSummary } from "./balance-summary";

test("prompts the user to connect a wallet", () => {
  render(
    <StellarKitProvider network="testnet">
      <BalanceSummary />
    </StellarKitProvider>
  );

  expect(screen.getByText(/connect a wallet/i)).toBeInTheDocument();
});
```

## 12. Where should I configure Horizon endpoints?

Configure Horizon once at the provider boundary. This avoids hard-coded URLs inside individual components and makes testnet, mainnet, and custom infrastructure easier to switch.

```tsx
<StellarKitProvider
  network="testnet"
  horizonUrl="https://horizon-testnet.stellar.org"
  requestTimeoutMs={15_000}
>
  {children}
</StellarKitProvider>
```

## 13. What should I show while wallet or network data is loading?

Use a skeleton or compact status message. Avoid rendering stale balances while a new wallet or network is being loaded.

```tsx
import { LoadingSkeleton, useAccount } from "@stellarkit/ui";

export function AccountCard({ publicKey }: { publicKey: string }) {
  const { account, isLoading } = useAccount(publicKey);

  if (isLoading) return <LoadingSkeleton lines={3} />;
  if (!account) return <p>No account data found.</p>;

  return <pre>{JSON.stringify(account.balances, null, 2)}</pre>;
}
```

## 14. How do I report integration issues?

Open a GitHub issue with the StellarKit UI version, framework version, network, wallet used, and the smallest reproduction you can share. Include the transaction hash only if it is already public and relevant to debugging.
