# aptos-wallet-adapter

React `WalletProvider` supporting loads of aptos wallets.

Supports:

- [Pontem Wallet](https://pontem.network/pontem-wallet)
- [Aptos official wallet](https://github.com/aptos-labs/aptos-core/releases/tag/wallet-v0.1.1)
- [Martian wallet](https://martianwallet.xyz/)
- [Fewcha wallet](https://fewcha.app/)
- [Hippo wallet](https://github.com/hippospace/hippo-wallet)
- [Hippo web wallet](https://hippo-wallet-test.web.app/)
- [Spika wallet](https://spika.app)
- [Rise Wallet](https://rise)
- [Fletch wallet](http://fletchwallet.io/)

Working on (PR welcome):

- [Nightly wallet](https://chrome.google.com/webstore/detail/nightly/injggoambcadlfkkjcgdfbejanmgfgfm/related?hl=en&authuser=0)

# Installation

with `yarn`

```
yarn add @manahippo/aptos-wallet-adapter
```

with `npm`

```
npm install @manahippo/aptos-wallet-adapter
```

# Examples

## **Frontend Integration**

Here's an example of how we integrate the adapter into [hippo's frontend](https://github.com/hippospace/hippo-frontend/blob/main/src/Providers.tsx):

### **Wallet integration**

Wallets source code [here](https://github.com/hippospace/aptos-wallet-adapter/tree/main/src/WalletAdatpers).

# Use React Provider

```typescript
import React from 'react';
import {
  WalletProvider,
  PontemWalletAdapter,
  HippoWalletAdapter,
  AptosWalletAdapter,
  HippoExtensionWalletAdapter,
  MartianWalletAdapter,
  FewchaWalletAdapter,
  SpikaWalletAdapter,
  RiseWalletAdapter,
  FletchWalletAdapter
} from '@manahippo/aptos-wallet-adapter';

const wallets = [
  new PontemWalletAdapter(),
  new HippoWalletAdapter(),
  new MartianWalletAdapter(),
  new AptosWalletAdapter(),
  new FewchaWalletAdapter(),
  new HippoExtensionWalletAdapter(),
  new SpikaWalletAdapter(),
  new RiseWalletAdapter(),
  new FletchWalletAdapter()
];

const App: React.FC = () => {
  return (
    <WalletProvider
      wallets={wallets}
      autoConnect={true | false} /** allow auto wallet connection or not **/
      onError={(error: Error) => {
        console.log('Handle Error Message', error);
      }}>
      {/* your website */}
    </WalletProvider>
  );
};

export default App;
```

# Web3 Hook

```typescript
import { useWallet } from '@manahippo/aptos-wallet-adapter';

const { connected, account, ...rest } = useWallet();

/*
  ** Properties available: **

  wallets: Wallet[]; - Array of wallets
  wallet: Wallet | null; - Selected wallet
  account(): AccountKeys | null; - Wallet info: address, publicKey, authKey
  connected: boolean; - check the website is connected yet
  connect(walletName: string): Promise<void>; - trigger connect popup
  disconnect(): Promise<void>; - trigger disconnect action
  signAndSubmitTransaction(
    transaction: TransactionPayload
  ): Promise<PendingTransaction>; - function to sign and submit the transaction to chain
*/
```

# Connect & Disconnect

```typescript
import { AptosWalletName, useWallet } from "@manahippo/aptos-wallet-adapter"

...

const { connect, disconnect, connected, select } = useWallet();

/** If auto-connect is not enabled, you will require to do the connect() manually **/
useEffect(() => {
  if (!autoConnect && currentWallet?.adapter) {
    connect();
  }
}, [autoConnect, currentWallet, connect]);
/** this is only required if you do not want auto connect wallet **/

if (!connected) {
  return (
    <button
      onClick={() => {
        select(); // E.g. connecting to the Aptos official wallet (Breaking Change)
      }}
    >
      Connect
    </button>
  );
} else {
  return (
    <button
      onClick={() => {
        disconnect();
      }}
    >
      Disconnect
    </button>
  );
}
```

# Hippo Wallet Client

```typescript
import { HippoSwapClient, HippoWalletClient } from '@manahippo/hippo-sdk';
import { getParserRepo } from '@manahippo/hippo-sdk';

export const hippoWalletClient = async (account: ActiveAptosWallet) => {
  if (!account) return undefined;
  const { netConf } = readConfig();
  const repo = getParserRepo();
  const walletClient = await HippoWalletClient.createInTwoCalls(
    netConf,
    aptosClient,
    repo,
    account
  );

  return walletClient;
};
```

# Hippo Swap Client

```typescript
import { HippoSwapClient, HippoWalletClient } from '@manahippo/hippo-sdk';
import { getParserRepo } from '@manahippo/hippo-sdk/';

export const hippoSwapClient = async () => {
  const { netConf } = readConfig();
  const repo = getParserRepo();
  const swapClient = await HippoSwapClient.createInOneCall(netConf, aptosClient, repo);

  return swapClient;
};
```

# Submit and sign transaction

**Request faucet**

```typescript
const { signAndSubmitTransaction } = useWallet();

const payload = await hippoWallet?.makeFaucetMintToPayload(uiAmtUsed, symbol);
if (payload) {
  const result = await signAndSubmitTransaction(payload);
  if (result) {
    message.success('Transaction Success');
    await hippoWallet?.refreshStores();
  }
}
```

**Swap Token**

```typescript
const bestQuote = await hippoSwap.getBestQuoteBySymbols(fromSymbol, toSymbol, uiAmtIn, 3);
if (!bestQuote) {
  throw new Error(`No route exists from ${fromSymbol} to ${toSymbol}`);
}
const payload = await bestQuote.bestRoute.makeSwapPayload(uiAmtIn, uiAmtOutMin);
const result = await signAndSubmitTransaction(payload);
if (result) {
  message.success('Transaction Success');
  setRefresh(true);
}
```

**Deposit Transaction**

```typescript
const pool = hippoSwap.getDirectPoolsBySymbolsAndPoolType(lhsSymbol, rhsSymbol, poolType);
if (pool.length === 0) {
  throw new Error('Desired pool does not exist');
}
const payload = await pool[0].makeAddLiquidityPayload(lhsUiAmt, rhsUiAmt);
const result = await signAndSubmitTransaction(payload);
if (result) {
  message.success('Transaction Success');
  setRefresh(true);
}
```