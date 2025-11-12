# Type Definitions

Complete TypeScript type reference for the 0xio Wallet SDK.

## Core Types

### SDKConfig

Configuration object for initializing the SDK.

```typescript
interface SDKConfig {
  // Required
  appName: string;

  // Optional
  appDescription?: string;
  appVersion?: string;
  appUrl?: string;
  appIcon?: string;
  requiredPermissions?: Permission[];
  networkId?: string;
  debug?: boolean;
}
```

**Fields:**
- `appName` (required): Your application name
- `appDescription`: Description shown to user
- `appVersion`: Application version (default: '1.0.0')
- `appUrl`: Application URL (auto-detected if not provided)
- `appIcon`: URL to app icon
- `requiredPermissions`: Array of required permissions
- `networkId`: Target network ID (default: 'octra-testnet')
- `debug`: Enable debug logging

---

### Permission

Available permission types.

```typescript
type Permission =
  | 'read_balance'
  | 'read_private_balance'
  | 'send_transactions'
  | 'send_private_transactions'
  | 'sign_messages';
```

---

### Balance

Wallet balance information.

```typescript
interface Balance {
  readonly public: number;
  readonly private?: number;
  readonly total: number;
  readonly currency: 'OCT';
}
```

**Fields:**
- `public`: Public balance amount
- `private`: Private balance amount (if permission granted)
- `total`: Total balance (public + private)
- `currency`: Currency symbol ('OCT')

---

### WalletAddress

Wallet address information.

```typescript
interface WalletAddress {
  readonly address: string;
  readonly publicKey?: string;
}
```

---

### ConnectionInfo

Current connection information.

```typescript
interface ConnectionInfo {
  readonly isConnected: boolean;
  readonly address?: string;
  readonly networkId?: string;
  readonly permissions?: Permission[];
}
```

---

### ConnectOptions

Options for connecting to wallet.

```typescript
interface ConnectOptions {
  requestedPermissions?: Permission[];
  networkId?: string;
}
```

---

## Transaction Types

### TransactionData

Data for sending a transaction.

```typescript
interface TransactionData {
  to: string;
  amount: number;
  message?: string;
  feeLevel?: 'low' | 'medium' | 'high';
}
```

**Fields:**
- `to` (required): Recipient address
- `amount` (required): Amount to send
- `message`: Optional transaction message
- `feeLevel`: Fee priority level (default: 'medium')

---

### TransactionResult

Result of a transaction.

```typescript
interface TransactionResult {
  readonly txHash: string;
  readonly success: boolean;
  readonly message?: string;
  readonly explorerUrl?: string;
}
```

**Fields:**
- `txHash`: Transaction hash
- `success`: Whether transaction succeeded
- `message`: Status message
- `explorerUrl`: Block explorer URL for this transaction

---

### Transaction

Transaction details.

```typescript
interface Transaction {
  readonly hash: string;
  readonly from: string;
  readonly to: string;
  readonly amount: number;
  readonly fee: number;
  readonly timestamp: number;
  readonly status: 'pending' | 'confirmed' | 'failed';
  readonly message?: string;
  readonly blockHeight?: number;
}
```

---

### TransactionHistory

Paginated transaction history.

```typescript
interface TransactionHistory {
  readonly transactions: Transaction[];
  readonly totalCount: number;
  readonly page: number;
  readonly hasMore: boolean;
}
```

---

### SignedTransaction

Signed transaction data.

```typescript
interface SignedTransaction {
  readonly raw: string;
  readonly signature: string;
  readonly txHash: string;
}
```

---

## Network Types

### NetworkInfo

Network configuration information.

```typescript
interface NetworkInfo {
  readonly id: string;
  readonly name: string;
  readonly rpcUrl: string;
  readonly explorerUrl: string;
  readonly color: string;
  readonly isTestnet: boolean;
}
```

**Fields:**
- `id`: Network identifier
- `name`: Display name
- `rpcUrl`: RPC endpoint URL
- `explorerUrl`: Block explorer URL
- `color`: Theme color (hex)
- `isTestnet`: Whether this is a testnet

---

## Event Types

### WalletEventType

Available event types.

```typescript
type WalletEventType =
  | 'connect'
  | 'disconnect'
  | 'accountChanged'
  | 'balanceChanged'
  | 'networkChanged'
  | 'transactionConfirmed'
  | 'error'
  | 'extensionLocked'
  | 'extensionUnlocked';
```

---

### ConnectEvent

Emitted when wallet connects.

```typescript
interface ConnectEvent {
  readonly address: string;
  readonly balance: Balance;
  readonly networkId: string;
  readonly permissions: Permission[];
}
```

---

### DisconnectEvent

Emitted when wallet disconnects.

```typescript
interface DisconnectEvent {
  readonly reason?: string;
}
```

---

### AccountChangedEvent

Emitted when account changes.

```typescript
interface AccountChangedEvent {
  readonly previousAddress?: string;
  readonly newAddress: string;
}
```

---

### BalanceChangedEvent

Emitted when balance changes.

```typescript
interface BalanceChangedEvent {
  readonly address: string;
  readonly previousBalance: Balance | undefined;
  readonly newBalance: Balance;
}
```

---

### NetworkChangedEvent

Emitted when network changes.

```typescript
interface NetworkChangedEvent {
  readonly previousNetwork?: NetworkInfo;
  readonly network: NetworkInfo;
}
```

---

### TransactionConfirmedEvent

Emitted when transaction is confirmed.

```typescript
interface TransactionConfirmedEvent {
  readonly txHash: string;
  readonly status: 'confirmed' | 'failed';
  readonly blockHeight?: number;
}
```

---

### ErrorEvent

Emitted when an error occurs.

```typescript
interface ErrorEvent {
  readonly error: ZeroXIOWalletError;
  readonly context?: string;
}
```

---

## Private Transaction Types

### PrivateBalanceInfo

Private balance information.

```typescript
interface PrivateBalanceInfo {
  readonly encryptedBalance: number;
  readonly decryptedBalance: number;
  readonly isEncrypted: boolean;
}
```

---

### PrivateTransferData

Data for sending a private transaction.

```typescript
interface PrivateTransferData {
  to: string;
  amount: number;
  message?: string;
}
```

---

### PendingPrivateTransfer

Pending private transfer information.

```typescript
interface PendingPrivateTransfer {
  readonly id: number;
  readonly sender: string;
  readonly encrypted_data: string;
  readonly ephemeral_key: string;
  readonly epoch_id: number;
}
```

---

## Error Types

### ErrorCode

Available error codes.

```typescript
enum ErrorCode {
  EXTENSION_NOT_FOUND = 'EXTENSION_NOT_FOUND',
  CONNECTION_REFUSED = 'CONNECTION_REFUSED',
  USER_REJECTED = 'USER_REJECTED',
  INSUFFICIENT_BALANCE = 'INSUFFICIENT_BALANCE',
  INVALID_ADDRESS = 'INVALID_ADDRESS',
  INVALID_AMOUNT = 'INVALID_AMOUNT',
  NETWORK_ERROR = 'NETWORK_ERROR',
  TRANSACTION_FAILED = 'TRANSACTION_FAILED',
  PERMISSION_DENIED = 'PERMISSION_DENIED',
  WALLET_LOCKED = 'WALLET_LOCKED',
  RATE_LIMIT_EXCEEDED = 'RATE_LIMIT_EXCEEDED'
}
```

---

### ZeroXIOWalletError

SDK error class.

```typescript
class ZeroXIOWalletError extends Error {
  readonly code: ErrorCode;
  readonly details?: any;

  constructor(code: ErrorCode, message: string, details?: any);
}
```

**Properties:**
- `code`: Error code
- `message`: Error message
- `details`: Additional error details

**Example:**
```typescript
try {
  await wallet.sendTransaction(txData);
} catch (error) {
  if (error instanceof ZeroXIOWalletError) {
    console.log('Error code:', error.code);
    console.log('Message:', error.message);
    console.log('Details:', error.details);
  }
}
```

---

## Communication Types

### ExtensionRequest

Internal request type (for advanced usage).

```typescript
interface ExtensionRequest {
  readonly id: string;
  readonly method: string;
  readonly params?: any;
  readonly timestamp: number;
}
```

---

### ExtensionResponse

Internal response type (for advanced usage).

```typescript
interface ExtensionResponse {
  readonly id: string;
  readonly success: boolean;
  readonly data?: any;
  readonly error?: {
    code: string;
    message: string;
  };
}
```

---

## Utility Types

### Type Guards

```typescript
// Check if value is a valid Balance
function isBalance(value: any): value is Balance;

// Check if error is specific error type
function isErrorType(error: any, code: ErrorCode): boolean;

// Check if value is a valid address
function isValidAddress(address: string): boolean;

// Check if value is a valid amount
function isValidAmount(amount: number): boolean;
```

---

## Usage Examples

### Type-Safe Transaction

```typescript
import { TransactionData, TransactionResult, ZeroXIOWallet } from '@0xgery/0xio-sdk';

async function sendTypedTransaction(
  wallet: ZeroXIOWallet,
  recipient: string,
  amount: number
): Promise<TransactionResult> {
  const txData: TransactionData = {
    to: recipient,
    amount: amount,
    message: 'Payment',
    feeLevel: 'medium'
  };

  const result: TransactionResult = await wallet.sendTransaction(txData);
  return result;
}
```

### Type-Safe Event Handling

```typescript
import { BalanceChangedEvent, ZeroXIOWallet } from '@0xgery/0xio-sdk';

function setupBalanceListener(wallet: ZeroXIOWallet) {
  wallet.on('balanceChanged', (event: BalanceChangedEvent) => {
    console.log('Previous:', event.previousBalance?.total);
    console.log('New:', event.newBalance.total);
    console.log('Currency:', event.newBalance.currency); // Type-safe: 'OCT'
  });
}
```

### Custom Type Narrowing

```typescript
import { ZeroXIOWalletError, ErrorCode } from '@0xgery/0xio-sdk';

function handleTransactionError(error: unknown) {
  if (error instanceof ZeroXIOWalletError) {
    switch (error.code) {
      case ErrorCode.INSUFFICIENT_BALANCE:
        return 'Not enough balance';
      case ErrorCode.USER_REJECTED:
        return 'Transaction cancelled';
      case ErrorCode.INVALID_ADDRESS:
        return 'Invalid recipient address';
      default:
        return `Transaction failed: ${error.message}`;
    }
  }
  return 'Unknown error occurred';
}
```

---

## Source Code

For the complete, up-to-date type definitions, see:
- [types.ts on GitHub](https://github.com/0xGery/0xio-sdk/blob/main/src/types.ts)
