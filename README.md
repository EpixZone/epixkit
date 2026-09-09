# EpixKit

Lightweight wallet connection library for Epix Chain dApps. Zero dependencies, single file, works with any EVM wallet.

Handles wallet discovery ([EIP-6963](https://eips.ethereum.org/EIPS/eip-6963)), modal UI, chain switching, and connect/disconnect. Bring your own provider wrapper (ethers.js, viem, web3.js, etc).

The picker lists only wallets that are actually installed. Inside the EpixNet browser that includes the Epix Wallet, which is listed first. Nothing is shown for wallets the user does not have, and there are no download placeholders.

## Quick Start

```html
<script src="https://cdn.jsdelivr.net/gh/EpixZone/epixkit@main/epixkit.js"></script>
<script>
EpixKit.init({
  chainId: "0x77D",
  chainName: "Epix Testnet",
  nativeCurrency: { name: "EPIX", symbol: "EPIX", decimals: 18 },
  rpcUrls: ["https://evmrpc.testnet.epix.zone"],
  blockExplorerUrls: ["https://testscan.epix.zone"],
  // excludeWallets: ["someOtherWallet"]  // optional; Phantom and Leap are excluded by default
  // installUrl: "https://example.com/wallet", installLabel: "Example Wallet"  // optional, for the empty state
})

document.getElementById("connect-btn").onclick = async function() {
  var result = await EpixKit.connect()
  // result.provider = raw EIP-1193 provider
  // result.address  = connected address
  // result.walletName = "Epix Wallet", "MetaMask", "Rabby", etc.
  // result.walletIcon = wallet icon data URI (or null)
  console.log("Connected:", result.address, "via", result.walletName)
}
</script>
```

### Using with ethers.js v6

```javascript
var result = await EpixKit.connect()
var provider = new ethers.BrowserProvider(result.provider)
var signer = await provider.getSigner()
// Now use signer to send transactions, interact with contracts, etc.
```

## API

### `EpixKit.init(config)`

Configure the library. Call once before `connect()`.

| Option | Type | Description |
|--------|------|-------------|
| `chainId` | `string` | Hex chain ID (required). `"0x77D"` for testnet, `"0x77C"` for mainnet |
| `chainName` | `string` | Human-readable chain name |
| `rpcUrls` | `string[]` | RPC endpoints for `wallet_addEthereumChain` |
| `blockExplorerUrls` | `string[]` | Block explorer URLs |
| `nativeCurrency` | `object` | `{ name, symbol, decimals }` |
| `excludeWallets` | `string[]` | Additional wallet names or RDNS strings to hide. Phantom (Solana only) and Leap (no longer a wallet) are excluded by default. |
| `installUrl` | `string` | Optional download link shown when no wallet is installed. Without it the empty state points at the EpixNet browser, which includes the Epix Wallet. |
| `installLabel` | `string` | Link text for `installUrl` |
| `onReconnect` | `function` | Called with the connect result when the wallet used last time is still authorized on page load |

### `EpixKit.connect()` -> `Promise<{provider, address, walletName, walletIcon}>`

Opens the wallet picker and connects. Returns the raw EIP-1193 provider, connected address, wallet name, and icon.

- The wallet used last time (remembered in `localStorage`) connects again without the picker.
- Otherwise the picker always shows, even with a single installed wallet, so the user sees which wallet is about to connect. The Epix Wallet is listed first.
- If no wallets are detected, the picker shows the empty state and the promise rejects.
- If the user closes the picker, the promise rejects.

### `EpixKit.disconnect()`

Clears internal connection state.

### `EpixKit.isConnected()` -> `boolean`

### `EpixKit.getAddress()` -> `string | null`

### `EpixKit.getProvider()` -> `EIP1193Provider | null`

Returns the raw EIP-1193 provider from the connected wallet.

### `EpixKit.getWalletName()` -> `string | null`

### `EpixKit.getWallets()` -> `Promise<Array<{uuid, name, rdns, icon, provider}>>`

Returns every installed wallet (after filtering excluded ones) without connecting, the Epix Wallet first. Async: it re-requests EIP-6963 announcements and waits for them.

### `EpixKit.EPIX_WALLET_RDNS`

`"zone.epix.wallet"`, the Epix Wallet's EIP-6963 identity, for callers that special-case it.

## Chain Presets

| Network | Chain ID | Hex |
|---------|----------|-----|
| Epix Mainnet | 1916 | `0x77C` |
| Epix Testnet | 1917 | `0x77D` |

## How It Works

1. On load, EpixKit listens for [EIP-6963](https://eips.ethereum.org/EIPS/eip-6963) wallet announcements
2. When `connect()` is called, it re-requests announcements, waits briefly, and collects every announced wallet plus any provider that only injected `window.ethereum` (never listing the same provider twice)
3. Filters out excluded wallets, puts the Epix Wallet first, and shows the picker (or reconnects the remembered wallet silently)
4. Requests accounts, switches to the configured chain, and returns the provider + address
5. CSS and modal HTML are injected into the DOM on first use (prefixed `epixkit-*` classes)

Fallback names for wallets without an EIP-6963 announcement check specific flags (`isRabby`, `isBraveWallet`, `isCoinbaseWallet`, ...) before `isMetaMask`, because most of them also set `isMetaMask` for compatibility.

## License

[MIT](LICENSE)
