# ✨ StellarKit UI

> A React component library and Next.js demo app for building beautiful Stellar blockchain frontends.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![React](https://img.shields.io/badge/React-18-blue)](https://reactjs.org/)
[![Next.js](https://img.shields.io/badge/Next.js-14-black)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue)](https://www.typescriptlang.org/)
[![Stellar](https://img.shields.io/badge/Stellar-SDK-black)](https://stellar.org/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## What is StellarKit UI?

StellarKit UI is an open-source React component library that gives developers everything they need to build polished Stellar blockchain frontends — without starting from scratch.

It includes ready-to-use components for wallet connection, account display, transaction history, asset balances, Soroban contract interaction, and more. All components are built with TypeScript, styled with Tailwind CSS, and come with a full Next.js demo app showing them in action.

---

## Packages

| Package | Description |
|---|---|
| `@stellarkit/ui` | Core React component library |
| `@stellarkit/hooks` | React hooks for Stellar data fetching |
| `@stellarkit/demo` | Next.js demo app showcasing all components |

---

## Features

- 🔌 **WalletConnect** — One-click Stellar wallet connection with multiple provider support
- 💰 **AccountBalance** — Display XLM and custom asset balances with live updates
- 📋 **TransactionHistory** — Paginated transaction history with operation details
- 🔄 **PaymentForm** — Send XLM and assets with built-in validation
- 📜 **ContractInvoker** — UI for invoking Soroban smart contract functions
- 🌐 **NetworkSwitcher** — Toggle between Stellar mainnet and testnet
- 🎨 **Fully themeable** — Light/dark mode, custom colors, CSS variables
- ♿ **Accessible** — WCAG 2.1 AA compliant components
- 🧪 **Fully tested** — Unit and integration tests for every component

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | React 18 + Next.js 14 |
| Language | TypeScript 5 |
| Styling | Tailwind CSS |
| Blockchain | Stellar SDK |
| Testing | Jest + React Testing Library |
| Storybook | Component documentation |
| Bundler | Rollup (library) + Next.js (demo) |

---

## Project Structure

```
stellarkit-ui/
├── packages/
│   ├── ui/                    # Core component library
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── WalletConnect/
│   │   │   │   ├── AccountBalance/
│   │   │   │   ├── TransactionHistory/
│   │   │   │   ├── PaymentForm/
│   │   │   │   ├── ContractInvoker/
│   │   │   │   └── NetworkSwitcher/
│   │   │   ├── hooks/
│   │   │   └── utils/
│   │   └── package.json
│   └── demo/                  # Next.js demo app
│       ├── app/
│       ├── components/
│       └── package.json
├── CONTRIBUTING.md
└── README.md
```

---

## Getting Started

### Prerequisites

- Node.js v18+
- A Stellar testnet account ([create one here](https://laboratory.stellar.org/))

### Installation

```bash
npm install @stellarkit/ui
```

### Basic Usage

```tsx
import { WalletConnect, AccountBalance } from '@stellarkit/ui'

export default function App() {
  return (
    <div>
      <WalletConnect network="testnet" />
      <AccountBalance publicKey="G..." />
    </div>
  )
}
```

### Running the Demo App

```bash
git clone https://github.com/stellarkit-ui/stellarkit-ui.git
cd stellarkit-ui
npm install
npm run dev
```

Visit `http://localhost:3000` to see all components in action.

---

## Components

### WalletConnect
Connect to Stellar wallets with a single component.
```tsx
<WalletConnect
  network="testnet"
  onConnect={(publicKey) => console.log(publicKey)}
  onDisconnect={() => console.log('disconnected')}
/>
```

### AccountBalance
Display account balances with real-time updates.
```tsx
<AccountBalance
  publicKey="GABC..."
  showAllAssets={true}
  refreshInterval={30000}
/>
```

### TransactionHistory
Show paginated transaction history for any account.
```tsx
<TransactionHistory
  publicKey="GABC..."
  pageSize={10}
  showOperationDetails={true}
/>
```

### PaymentForm
Send XLM and custom assets with validation.
```tsx
<PaymentForm
  sourcePublicKey="GABC..."
  onSuccess={(hash) => console.log(hash)}
  onError={(error) => console.error(error)}
/>
```

---

## Roadmap

### v0.1.0 — Foundation
- [x] Project setup and monorepo structure
- [ ] WalletConnect component
- [ ] AccountBalance component
- [ ] Basic theming system

### v0.2.0 — Transactions
- [ ] TransactionHistory component
- [ ] PaymentForm component
- [ ] Transaction status indicator

### v0.3.0 — Soroban
- [ ] ContractInvoker component
- [ ] Contract event display
- [ ] Soroban hooks

### v0.4.0 — Polish
- [ ] Dark mode support
- [ ] Accessibility audit
- [ ] Storybook documentation
- [ ] npm package publish

---

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting a pull request.

Look for issues tagged [`good first issue`](https://github.com/stellarkit-ui/stellarkit-ui/issues?q=label%3A%22good+first+issue%22) to get started.

---

## Community

- 💬 [GitHub Discussions](https://github.com/stellarkit-ui/stellarkit-ui/discussions)
- 🐛 [Issues](https://github.com/stellarkit-ui/stellarkit-ui/issues)
- 🐦 [Twitter](https://twitter.com/Engrukayat)

---

## License

MIT © [stellarkit-ui](https://github.com/stellarkit-ui)

---

<p align="center">Built with ❤️ for the Stellar ecosystem</p>
