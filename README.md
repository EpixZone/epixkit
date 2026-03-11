# EpixKit

Lightweight wallet connection library for Epix Chain dApps. Zero dependencies, single file, works with any EVM wallet.

Handles wallet discovery ([EIP-6963](https://eips.ethereum.org/EIPS/eip-6963)), modal UI, chain switching, and connect/disconnect. Bring your own provider wrapper (ethers.js, viem, web3.js, etc).

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
  excludeWallets: ["phantom"]
})

document.getElementById("connect-btn").onclick = async function() {
  var result = await EpixKit.connect()
  // result.provider = raw EIP-1193 provider
  // result.address  = connected address
  // result.walletName = "MetaMask", "Keplr", etc.
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
| `chainId` | `string` | Hex chain ID (required). `"0x77D"` for testnet, `"0x779"` for mainnet |
| `chainName` | `string` | Human-readable chain name |
| `rpcUrls` | `string[]` | RPC endpoints for `wallet_addEthereumChain` |
| `blockExplorerUrls` | `string[]` | Block explorer URLs |
| `nativeCurrency` | `object` | `{ name, symbol, decimals }` |
| `excludeWallets` | `string[]` | Wallet names or RDNS strings to hide (e.g. `["phantom"]`) |

### `EpixKit.connect()` -> `Promise<{provider, address, walletName, walletIcon}>`

Opens the wallet picker modal (if multiple wallets detected) and connects. Returns the raw EIP-1193 provider, connected address, wallet name, and icon.

- If only one wallet is installed, it auto-selects without showing the modal.
- If no wallets are detected, the promise rejects with an error.
- If the user closes the modal, the promise rejects.

### `EpixKit.disconnect()`

Clears internal connection state.

### `EpixKit.isConnected()` -> `boolean`

### `EpixKit.getAddress()` -> `string | null`

### `EpixKit.getProvider()` -> `EIP1193Provider | null`

Returns the raw EIP-1193 provider from the connected wallet.

### `EpixKit.getWalletName()` -> `string | null`

### `EpixKit.getWallets()` -> `Array<{info, provider}>`

Returns all discovered wallets (after filtering excluded ones) without connecting.

## Chain Presets

| Network | Chain ID | Hex |
|---------|----------|-----|
| Epix Mainnet | 1913 | `0x779` |
| Epix Testnet | 1917 | `0x77D` |

## How It Works

1. On load, EpixKit listens for [EIP-6963](https://eips.ethereum.org/EIPS/eip-6963) wallet announcements
2. When `connect()` is called, it collects all discovered wallets (with `window.ethereum` fallback)
3. Filters out excluded wallets, shows a picker modal if 2+ wallets are available
4. Requests accounts, switches to the configured chain, and returns the provider + address
5. CSS and modal HTML are injected into the DOM on first use (prefixed `epixkit-*` classes)

## License

[MIT](LICENSE)
