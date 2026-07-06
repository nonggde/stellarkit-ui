# Stellar Wave Integration Guide

This guide shows how a Stellar Wave app can use StellarKit UI as the wallet-facing layer while keeping network, signing, and Soroban contract logic explicit in the application code.

Use it when your app needs to:

- connect a Stellar wallet before a Wave action can run;
- show the active public key and selected network clearly;
- sign a Stellar transaction from the browser;
- prepare, sign, and submit a Soroban contract invocation.

For the underlying Stellar APIs, see the official JavaScript SDK docs, Soroban transaction invocation guide, and Soroban signing guide:

- https://developers.stellar.org/docs/tools/sdks/client-sdks
- https://developers.stellar.org/docs/build/guides/transactions/invoke-contract-tx-sdk
- https://developers.stellar.org/docs/build/guides/transactions/signing-soroban-invocations

## Recommended app shape

Wave apps usually have three layers:

1. UI state: connect button, account badge, network selector, and transaction status.
2. Wallet adapter: exposes the connected public key and a `signTransaction` function.
3. Stellar service: builds Horizon or Soroban transactions and submits signed XDR.

Keep the UI and wallet state close to StellarKit UI components, but keep transaction construction in a small service module. That makes it easier to test the transaction builder without mocking the whole React tree.

```tsx
import { StellarKitProvider, WalletButton, WalletStatus } from "@stellarkit-ui/react";

const waveNetwork = {
  name: "testnet",
  networkPassphrase: "Test SDF Network ; September 2015",
  horizonUrl: "https://horizon-testnet.stellar.org",
  sorobanRpcUrl: "https://soroban-testnet.stellar.org",
};

export function WaveAppShell({ children }: { children: React.ReactNode }) {
  return (
    <StellarKitProvider network={waveNetwork}>
      <header className="flex items-center justify-between gap-4">
        <WalletStatus />
        <WalletButton />
      </header>
      {children}
    </StellarKitProvider>
  );
}
```

If your project is still wiring a wallet adapter, keep the adapter surface small:

```ts
export type WaveWallet = {
  publicKey: string | null;
  isConnected: boolean;
  connect: () => Promise<void>;
  disconnect: () => Promise<void>;
  signTransaction: (
    transactionXdr: string,
    options: { networkPassphrase: string },
  ) => Promise<string>;
};
```

That shape works for browser wallets that return a signed transaction XDR string and can be wrapped by StellarKit UI later.

## Wallet connection flow

A Wave flow should block action buttons until the wallet is connected and the network is correct. The UI should make three states obvious: disconnected, connected, and wrong network.

```tsx
import { useMemo } from "react";
import { useStellarKit } from "@stellarkit-ui/react";

export function WaveActionGate({ children }: { children: React.ReactNode }) {
  const { wallet, network, connect } = useStellarKit();

  const canRunWaveAction = useMemo(() => {
    return wallet.publicKey && network.name === "testnet";
  }, [wallet.publicKey, network.name]);

  if (!wallet.publicKey) {
    return <button onClick={connect}>Connect wallet</button>;
  }

  if (!canRunWaveAction) {
    return <p>Switch to Stellar testnet before continuing.</p>;
  }

  return <>{children}</>;
}
```

For production Wave apps, also show a shortened public key in the header and put the full address in a copyable account panel. Users should never need to paste a secret key into the browser.

## Transaction signing flow

Use the Stellar SDK to build the unsigned transaction, then ask the connected wallet to sign the XDR. The app submits only the signed XDR.

```ts
import {
  BASE_FEE,
  Networks,
  Operation,
  TransactionBuilder,
  Horizon,
} from "@stellar/stellar-sdk";

export async function signAndSubmitPayment(params: {
  sourcePublicKey: string;
  destinationPublicKey: string;
  amount: string;
  signTransaction: (xdr: string, options: { networkPassphrase: string }) => Promise<string>;
}) {
  const server = new Horizon.Server("https://horizon-testnet.stellar.org");
  const sourceAccount = await server.loadAccount(params.sourcePublicKey);

  const transaction = new TransactionBuilder(sourceAccount, {
    fee: BASE_FEE,
    networkPassphrase: Networks.TESTNET,
  })
    .addOperation(
      Operation.payment({
        destination: params.destinationPublicKey,
        asset: Operation.nativeAsset(),
        amount: params.amount,
      }),
    )
    .setTimeout(30)
    .build();

  const signedXdr = await params.signTransaction(transaction.toXDR(), {
    networkPassphrase: Networks.TESTNET,
  });

  return server.submitTransaction(
    TransactionBuilder.fromXDR(signedXdr, Networks.TESTNET),
  );
}
```

Use this pattern for Wave actions such as joining a campaign, sending a contribution, or confirming a payout. The UI should show `building`, `waiting for wallet`, `submitting`, `confirmed`, and `failed` states so the user understands where the action stopped.

## Soroban contract interaction pattern

Soroban contract calls need an extra preparation step. Build the contract invocation, simulate or prepare it with Stellar RPC, ask the wallet to sign the prepared XDR, then submit the signed transaction.

```ts
import {
  BASE_FEE,
  Contract,
  Networks,
  TransactionBuilder,
  nativeToScVal,
  rpc,
} from "@stellar/stellar-sdk";

export async function invokeWaveContract(params: {
  sourcePublicKey: string;
  contractId: string;
  campaignId: string;
  signTransaction: (xdr: string, options: { networkPassphrase: string }) => Promise<string>;
}) {
  const server = new rpc.Server("https://soroban-testnet.stellar.org");
  const sourceAccount = await server.getAccount(params.sourcePublicKey);
  const contract = new Contract(params.contractId);

  const transaction = new TransactionBuilder(sourceAccount, {
    fee: BASE_FEE,
    networkPassphrase: Networks.TESTNET,
  })
    .addOperation(
      contract.call("join_campaign", nativeToScVal(params.campaignId)),
    )
    .setTimeout(30)
    .build();

  const prepared = await server.prepareTransaction(transaction);
  const signedXdr = await params.signTransaction(prepared.toXDR(), {
    networkPassphrase: Networks.TESTNET,
  });

  return server.sendTransaction(
    TransactionBuilder.fromXDR(signedXdr, Networks.TESTNET),
  );
}
```

When a contract requires auth entries or sponsored fees, keep that logic in the service layer and expose one UI action per transaction. Avoid batching unrelated Soroban operations together in the same user action unless the contract explicitly expects it.

## Wave integration checklist

Before shipping a Wave app, verify these items:

- The connect button is visible before any gated action.
- The active public key and network are visible after connection.
- The app never asks for a secret key.
- The transaction preview explains asset, amount, destination, contract ID, and function name.
- The wallet signing request uses the same network passphrase shown in the UI.
- Soroban calls are prepared through Stellar RPC before signing.
- Success and failure states include transaction hash or error details.
- Testnet and mainnet configuration are separated by environment variables.

## Testing examples

For docs-only examples, test the logic around small service functions instead of rendering the whole app. A good first test is to mock the wallet signer and assert that the builder passes the expected network passphrase.

```ts
import { strict as assert } from "node:assert";

const signedXdr = "AAAA...";
const signer = async (_xdr: string, options: { networkPassphrase: string }) => {
  assert.equal(options.networkPassphrase, "Test SDF Network ; September 2015");
  return signedXdr;
};

await signer("unsigned-xdr", {
  networkPassphrase: "Test SDF Network ; September 2015",
});
```

This keeps wallet behavior deterministic while still checking the contract between StellarKit UI, the wallet adapter, and the Stellar service module.
