# API Reference

Complete API reference for the 0xio Wallet SDK.

## Table of Contents

- [ZeroXIOWallet Class](#zeroxiowallet-class)
- [Factory Functions](#factory-functions)
- [Utility Functions](#utility-functions)
- [Configuration](#configuration)
- [Network Management](#network-management)

---

## ZeroXIOWallet Class

The main class for interacting with the 0xio Wallet extension.

### Constructor

```typescript
new ZeroXIOWallet(config: SDKConfig)
```

**Parameters:**
- `config`: [SDKConfig](#sdkconfig) - Configuration object

**Example:**
```typescript
import { ZeroXIOWallet } from '@0xgery/0xio-sdk';

const wallet = new ZeroXIOWallet({
  appName: 'My DApp',
  appDescription: 'A decentralized application',
  requiredPermissions: ['read_balance', 'send_transactions']
});
```

---

### Initialization Methods

#### initialize()

Initialize the SDK and establish communication with the wallet extension.

```typescript
wallet.initialize(): Promise<boolean>
```

**Returns:** `Promise<boolean>` - `true` if initialization successful

**Throws:** `ZeroXIOWalletError` if extension not found or initialization fails

**Example:**
```typescript
const initialized = await wallet.initialize();
if (initialized) {
  console.log('SDK initialized successfully');
}
```

---

### Connection Methods

#### connect()

Request connection to the user's wallet.

```typescript
wallet.connect(options?: ConnectOptions): Promise<ConnectEvent>
```

**Parameters:**
- `options` (optional): [ConnectOptions](#connectoptions)

**Returns:** `Promise<ConnectEvent>` containing:
- `address`: string - Connected wallet address
- `balance`: [Balance](#balance) - Current balance
- `networkId`: string - Current network ID
- `permissions`: Permission[] - Granted permissions

**Throws:**
- `CONNECTION_REFUSED` - User denied connection
- `EXTENSION_NOT_FOUND` - Extension not available
- `WALLET_LOCKED` - Wallet is locked

**Example:**
```typescript
try {
  const connection = await wallet.connect();
  console.log('Connected:', connection.address);
  console.log('Balance:', connection.balance.total, 'OCT');
} catch (error) {
  console.error('Connection failed:', error.message);
}
```

#### disconnect()

Disconnect from the wallet.

```typescript
wallet.disconnect(): Promise<void>
```

**Example:**
```typescript
await wallet.disconnect();
console.log('Disconnected');
```

#### isConnected()

Check if currently connected to wallet.

```typescript
wallet.isConnected(): boolean
```

**Returns:** `boolean` - Connection status

**Example:**
```typescript
if (wallet.isConnected()) {
  console.log('Wallet is connected');
}
```

#### getConnectionInfo()

Get current connection information.

```typescript
wallet.getConnectionInfo(): ConnectionInfo
```

**Returns:** [ConnectionInfo](#connectioninfo)

**Example:**
```typescript
const info = wallet.getConnectionInfo();
console.log('Address:', info.address);
console.log('Network:', info.networkId);
console.log('Permissions:', info.permissions);
```

---

### Wallet Information Methods

#### getAddress()

Get the connected wallet address.

```typescript
wallet.getAddress(): string | null
```

**Returns:** `string | null` - Wallet address or null if not connected

**Example:**
```typescript
const address = wallet.getAddress();
if (address) {
  console.log('Address:', address);
}
```

#### getBalance()

Get wallet balance.

```typescript
wallet.getBalance(forceRefresh?: boolean): Promise<Balance>
```

**Parameters:**
- `forceRefresh` (optional): boolean - Force refresh from blockchain (default: false)

**Returns:** `Promise<Balance>` containing:
- `public`: number - Public balance
- `private`: number - Private balance (if permission granted)
- `total`: number - Total balance
- `currency`: 'OCT' - Currency symbol

**Example:**
```typescript
// Get cached balance
const balance = await wallet.getBalance();

// Force refresh from blockchain
const freshBalance = await wallet.getBalance(true);

console.log('Public:', balance.public, 'OCT');
console.log('Private:', balance.private, 'OCT');
console.log('Total:', balance.total, 'OCT');
```

#### getNetworkInfo()

Get current network information.

```typescript
wallet.getNetworkInfo(): Promise<NetworkInfo>
```

**Returns:** `Promise<NetworkInfo>` containing:
- `id`: string - Network ID
- `name`: string - Network name
- `rpcUrl`: string - RPC endpoint URL
- `explorerUrl`: string - Block explorer URL
- `color`: string - Theme color
- `isTestnet`: boolean - Whether it's a testnet

**Example:**
```typescript
const network = await wallet.getNetworkInfo();
console.log('Network:', network.name);
console.log('RPC:', network.rpcUrl);
console.log('Explorer:', network.explorerUrl);
```

---

### Transaction Methods

#### sendTransaction()

Send a public transaction.

```typescript
wallet.sendTransaction(txData: TransactionData): Promise<TransactionResult>
```

**Parameters:**
- `txData`: [TransactionData](#transactiondata) - Transaction details

**Returns:** `Promise<TransactionResult>` containing:
- `txHash`: string - Transaction hash
- `success`: boolean - Whether transaction succeeded
- `message`: string (optional) - Status message
- `explorerUrl`: string (optional) - Block explorer URL

**Throws:**
- `USER_REJECTED` - User rejected transaction
- `INSUFFICIENT_BALANCE` - Not enough balance
- `INVALID_ADDRESS` - Invalid recipient address
- `INVALID_AMOUNT` - Invalid transaction amount
- `TRANSACTION_FAILED` - Transaction processing failed

**Example:**
```typescript
try {
  const result = await wallet.sendTransaction({
    to: 'recipient_address',
    amount: 10.5,
    message: 'Payment for goods',
    feeLevel: 'medium'
  });

  console.log('TX Hash:', result.txHash);
  console.log('Explorer:', result.explorerUrl);
} catch (error) {
  console.error('Transaction failed:', error.message);
}
```

#### getTransactionHistory()

Get transaction history.

```typescript
wallet.getTransactionHistory(page?: number, limit?: number): Promise<TransactionHistory>
```

**Parameters:**
- `page` (optional): number - Page number (default: 1)
- `limit` (optional): number - Items per page (default: 20)

**Returns:** `Promise<TransactionHistory>` containing:
- `transactions`: [Transaction](#transaction)[] - Array of transactions
- `totalCount`: number - Total transaction count
- `page`: number - Current page
- `hasMore`: boolean - Whether more pages exist

**Example:**
```typescript
const history = await wallet.getTransactionHistory(1, 10);

history.transactions.forEach(tx => {
  console.log('Hash:', tx.hash);
  console.log('From:', tx.from);
  console.log('To:', tx.to);
  console.log('Amount:', tx.amount, 'OCT');
  console.log('Status:', tx.status);
});

if (history.hasMore) {
  console.log('More transactions available');
}
```

---

### Private Transaction Methods

#### getPrivateBalanceInfo()

Get private balance information.

```typescript
wallet.getPrivateBalanceInfo(): Promise<PrivateBalanceInfo>
```

**Returns:** `Promise<PrivateBalanceInfo>` containing:
- `encryptedBalance`: number - Encrypted balance amount
- `decryptedBalance`: number - Decrypted balance amount
- `isEncrypted`: boolean - Whether balance is encrypted

**Example:**
```typescript
const privateInfo = await wallet.getPrivateBalanceInfo();
console.log('Encrypted:', privateInfo.encryptedBalance);
console.log('Decrypted:', privateInfo.decryptedBalance);
```

#### encryptBalance()

Encrypt public balance to private balance.

```typescript
wallet.encryptBalance(amount: number): Promise<boolean>
```

**Parameters:**
- `amount`: number - Amount to encrypt

**Returns:** `Promise<boolean>` - Success status

**Example:**
```typescript
const success = await wallet.encryptBalance(50);
if (success) {
  console.log('50 OCT encrypted to private balance');
}
```

#### decryptBalance()

Decrypt private balance to public balance.

```typescript
wallet.decryptBalance(amount: number): Promise<boolean>
```

**Parameters:**
- `amount`: number - Amount to decrypt

**Returns:** `Promise<boolean>` - Success status

**Example:**
```typescript
const success = await wallet.decryptBalance(25);
if (success) {
  console.log('25 OCT decrypted to public balance');
}
```

#### sendPrivateTransfer()

Send a private (encrypted) transaction.

```typescript
wallet.sendPrivateTransfer(data: PrivateTransferData): Promise<TransactionResult>
```

**Parameters:**
- `data`: [PrivateTransferData](#privatetransferdata) - Private transfer details

**Returns:** `Promise<TransactionResult>`

**Example:**
```typescript
const result = await wallet.sendPrivateTransfer({
  to: 'recipient_address',
  amount: 5.0,
  message: 'Private payment'
});

console.log('Private transfer ID:', result.txHash);
```

#### getPendingPrivateTransfers()

Get pending private transfers for the connected wallet.

```typescript
wallet.getPendingPrivateTransfers(): Promise<PendingPrivateTransfer[]>
```

**Returns:** `Promise<PendingPrivateTransfer[]>` - Array of pending transfers

**Example:**
```typescript
const pending = await wallet.getPendingPrivateTransfers();

pending.forEach(transfer => {
  console.log('From:', transfer.sender);
  console.log('Amount:', transfer.amount);
  console.log('Epoch:', transfer.epoch_id);
});
```

#### claimPrivateTransfer()

Claim a pending private transfer.

```typescript
wallet.claimPrivateTransfer(transferId: string): Promise<TransactionResult>
```

**Parameters:**
- `transferId`: string - Transfer ID to claim

**Returns:** `Promise<TransactionResult>`

**Example:**
```typescript
const result = await wallet.claimPrivateTransfer('transfer_id_123');
console.log('Claimed transfer:', result.txHash);
```

---

### Event Methods

#### on()

Add an event listener.

```typescript
wallet.on(event: WalletEventType, listener: (data: any) => void): void
```

**Parameters:**
- `event`: [WalletEventType](#walleteventtype) - Event name
- `listener`: Function - Event handler

**Example:**
```typescript
wallet.on('balanceChanged', (event) => {
  console.log('New balance:', event.newBalance.total);
});
```

#### off()

Remove an event listener.

```typescript
wallet.off(event: WalletEventType, listener: Function): void
```

**Parameters:**
- `event`: [WalletEventType](#walleteventtype) - Event name
- `listener`: Function - Event handler to remove

**Example:**
```typescript
const handler = (event) => console.log(event);
wallet.on('connect', handler);
wallet.off('connect', handler);
```

#### once()

Add a one-time event listener.

```typescript
wallet.once(event: WalletEventType, listener: (data: any) => void): void
```

**Parameters:**
- `event`: [WalletEventType](#walleteventtype) - Event name
- `listener`: Function - Event handler

**Example:**
```typescript
wallet.once('connect', (event) => {
  console.log('Connected once:', event.address);
});
```

#### removeAllListeners()

Remove all listeners for a specific event or all events.

```typescript
wallet.removeAllListeners(event?: WalletEventType): void
```

**Parameters:**
- `event` (optional): [WalletEventType](#walleteventtype) - Event name

**Example:**
```typescript
// Remove all listeners for specific event
wallet.removeAllListeners('balanceChanged');

// Remove all listeners for all events
wallet.removeAllListeners();
```

---

## Factory Functions

### createZeroXIOWallet()

Helper function to create and initialize a wallet instance.

```typescript
createZeroXIOWallet(config: {
  appName: string;
  appDescription?: string;
  debug?: boolean;
  autoConnect?: boolean;
}): Promise<ZeroXIOWallet>
```

**Parameters:**
- `config.appName`: string - Application name (required)
- `config.appDescription`: string - Application description (optional)
- `config.debug`: boolean - Enable debug mode (optional)
- `config.autoConnect`: boolean - Auto-connect on initialization (optional)

**Returns:** `Promise<ZeroXIOWallet>` - Initialized wallet instance

**Example:**
```typescript
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  autoConnect: true,
  debug: false
});
```

### createOctraWallet()

Legacy alias for `createZeroXIOWallet()`. Maintained for backward compatibility.

```typescript
createOctraWallet(config): Promise<ZeroXIOWallet>
```

---

## Utility Functions

### Validation Utilities

#### isValidAddress()

Validate a wallet address.

```typescript
isValidAddress(address: string): boolean
```

**Example:**
```typescript
if (isValidAddress('octxyz123...')) {
  console.log('Valid address');
}
```

#### isValidAmount()

Validate a transaction amount.

```typescript
isValidAmount(amount: number): boolean
```

**Example:**
```typescript
if (isValidAmount(10.5)) {
  console.log('Valid amount');
}
```

#### isValidMessage()

Validate a transaction message.

```typescript
isValidMessage(message: string): boolean
```

#### isValidFeeLevel()

Validate a fee level.

```typescript
isValidFeeLevel(level: string): boolean
```

#### isValidNetworkId()

Validate a network ID.

```typescript
isValidNetworkId(networkId: string): boolean
```

### Formatting Utilities

#### formatOCT()

Format OCT amount for display.

```typescript
formatOCT(amount: number, decimals?: number): string
```

**Parameters:**
- `amount`: number - Amount to format
- `decimals` (optional): number - Decimal places (default: 6)

**Example:**
```typescript
const formatted = formatOCT(123.456789, 2);
console.log(formatted); // "123.46"
```

#### formatAddress()

Format address with ellipsis.

```typescript
formatAddress(address: string, prefixLength?: number, suffixLength?: number): string
```

**Example:**
```typescript
const short = formatAddress('octxyz123...abc', 6, 4);
console.log(short); // "octxyz...abc"
```

#### formatTimestamp()

Format Unix timestamp to readable date.

```typescript
formatTimestamp(timestamp: number): string
```

#### formatTxHash()

Format transaction hash with ellipsis.

```typescript
formatTxHash(hash: string, length?: number): string
```

### Conversion Utilities

#### toMicroOCT()

Convert OCT to micro OCT.

```typescript
toMicroOCT(amount: number): string
```

**Example:**
```typescript
const micro = toMicroOCT(1.5);
console.log(micro); // "1500000"
```

#### fromMicroOCT()

Convert micro OCT to OCT.

```typescript
fromMicroOCT(microAmount: string | number): number
```

**Example:**
```typescript
const oct = fromMicroOCT(1500000);
console.log(oct); // 1.5
```

### Error Utilities

#### createErrorMessage()

Create a formatted error message.

```typescript
createErrorMessage(code: ErrorCode, message: string, details?: any): string
```

#### isErrorType()

Check if error is of specific type.

```typescript
isErrorType(error: any, code: ErrorCode): boolean
```

**Example:**
```typescript
if (isErrorType(error, ErrorCode.INSUFFICIENT_BALANCE)) {
  console.log('Not enough balance');
}
```

### Async Utilities

#### delay()

Create a delayed promise.

```typescript
delay(ms: number): Promise<void>
```

#### retry()

Retry an async function with exponential backoff.

```typescript
retry<T>(fn: () => Promise<T>, attempts?: number, delayMs?: number): Promise<T>
```

#### withTimeout()

Add timeout to a promise.

```typescript
withTimeout<T>(promise: Promise<T>, timeoutMs: number): Promise<T>
```

### Browser Utilities

#### isBrowser()

Check if running in browser environment.

```typescript
isBrowser(): boolean
```

#### checkBrowserSupport()

Check browser compatibility.

```typescript
checkBrowserSupport(): { compatible: boolean; issues: string[] }
```

#### checkSDKCompatibility()

Check SDK compatibility.

```typescript
checkSDKCompatibility(): {
  compatible: boolean;
  issues: string[];
  recommendations: string[];
}
```

---

## Configuration

### getNetworkConfig()

Get configuration for a specific network.

```typescript
getNetworkConfig(networkId?: string): NetworkInfo
```

### getAllNetworks()

Get all available networks.

```typescript
getAllNetworks(): NetworkInfo[]
```

### getDefaultNetwork()

Get the default network configuration.

```typescript
getDefaultNetwork(): NetworkInfo
```

### createDefaultBalance()

Create a default balance object.

```typescript
createDefaultBalance(total?: number): Balance
```

---

## Constants

### SDK_VERSION

Current SDK version.

```typescript
const SDK_VERSION: string
```

### SUPPORTED_EXTENSION_VERSIONS

Supported extension versions.

```typescript
const SUPPORTED_EXTENSION_VERSIONS: string[]
```

### NETWORKS

Available network configurations.

```typescript
const NETWORKS: Record<string, NetworkInfo>
```

### DEFAULT_NETWORK_ID

Default network ID.

```typescript
const DEFAULT_NETWORK_ID: string
```

---

## Type Definitions

For complete type definitions, see the [SDK source code](https://github.com/0xGery/0xio-sdk/blob/main/src/types.ts).
