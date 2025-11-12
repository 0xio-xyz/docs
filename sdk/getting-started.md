# Getting Started with 0xio SDK

This guide will walk you through setting up and integrating the 0xio Wallet SDK into your decentralized application.

## Prerequisites

Before you begin, ensure you have:

- Node.js 16 or higher
- A Chromium-based browser (Chrome, Edge, Brave, Opera)
- 0xio Wallet extension installed ([Chrome Web Store](https://chromewebstore.google.com/detail/0xio-wallet/anknhjilldkeelailocijnfibefmepcc))
- Basic knowledge of JavaScript/TypeScript
- Understanding of async/await patterns

## Installation

### NPM

```bash
npm install @0xgery/0xio-sdk
```

### Yarn

```bash
yarn add @0xgery/0xio-sdk
```

### CDN (for testing)

```html
<script src="https://unpkg.com/@0xgery/0xio-sdk@latest/dist/index.umd.js"></script>
```

## Basic Setup

### 1. Import the SDK

```typescript
import { createZeroXIOWallet } from '@0xgery/0xio-sdk';
```

### 2. Create Wallet Instance

```typescript
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  appDescription: 'A decentralized application on Octra',
  autoConnect: false,  // Set to true for automatic connection
  debug: false         // Enable in development for detailed logs
});
```

### 3. Connect to Wallet

```typescript
try {
  const connection = await wallet.connect();
  console.log('Connected to:', connection.address);
  console.log('Balance:', connection.balance.total, 'OCT');
} catch (error) {
  console.error('Connection failed:', error.message);
}
```

### 4. Check Connection Status

```typescript
if (wallet.isConnected()) {
  const address = wallet.getAddress();
  console.log('Wallet address:', address);
}
```

## Configuration Options

### SDKConfig Interface

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

### Example with All Options

```typescript
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  appDescription: 'Decentralized trading platform',
  appVersion: '1.0.0',
  appUrl: 'https://mydapp.com',
  appIcon: 'https://mydapp.com/icon.png',
  requiredPermissions: ['read_balance', 'send_transactions', 'read_private_balance'],
  networkId: 'octra-testnet',
  debug: true
});
```

### Permissions

Available permissions:

- `read_balance` - Read public balance
- `read_private_balance` - Read private balance
- `send_transactions` - Send public transactions
- `send_private_transactions` - Send private transactions
- `sign_messages` - Sign arbitrary messages

## Event Handling

### Listen to Wallet Events

```typescript
// Balance changed
wallet.on('balanceChanged', (event) => {
  console.log('Previous:', event.previousBalance?.total);
  console.log('New:', event.newBalance.total);
});

// Network changed
wallet.on('networkChanged', (event) => {
  console.log('Network:', event.network.name);
  console.log('Chain ID:', event.network.id);
});

// Account changed
wallet.on('accountChanged', (event) => {
  console.log('Previous:', event.previousAddress);
  console.log('New:', event.newAddress);
});

// Disconnected
wallet.on('disconnect', () => {
  console.log('Wallet disconnected');
});

// Transaction confirmed
wallet.on('transactionConfirmed', (event) => {
  console.log('TX Hash:', event.txHash);
  console.log('Status:', event.status);
});

// Extension locked/unlocked
wallet.on('extensionLocked', () => {
  console.log('Extension is locked');
});

wallet.on('extensionUnlocked', () => {
  console.log('Extension is unlocked');
});

// Errors
wallet.on('error', (event) => {
  console.error('Error:', event.error.message);
  console.error('Code:', event.error.code);
});
```

### Remove Event Listeners

```typescript
const handler = (event) => {
  console.log('Balance:', event.newBalance.total);
};

// Add listener
wallet.on('balanceChanged', handler);

// Remove specific listener
wallet.off('balanceChanged', handler);

// Remove all listeners for an event
wallet.removeAllListeners('balanceChanged');
```

## Reading Wallet Data

### Get Balance

```typescript
// Get current balance (cached)
const balance = await wallet.getBalance();

// Force refresh from blockchain
const freshBalance = await wallet.getBalance(true);

console.log('Public:', balance.public, 'OCT');
console.log('Private:', balance.private, 'OCT');
console.log('Total:', balance.total, 'OCT');
```

### Get Address

```typescript
const address = wallet.getAddress();
console.log('Address:', address);
```

### Get Network Information

```typescript
const network = await wallet.getNetworkInfo();
console.log('Network:', network.name);
console.log('RPC URL:', network.rpcUrl);
console.log('Explorer:', network.explorerUrl);
console.log('Is Testnet:', network.isTestnet);
```

### Get Connection Info

```typescript
const info = wallet.getConnectionInfo();
console.log('Connected:', info.isConnected);
console.log('Address:', info.address);
console.log('Network:', info.networkId);
console.log('Permissions:', info.permissions);
```

## Sending Transactions

### Public Transaction

```typescript
try {
  const result = await wallet.sendTransaction({
    to: 'recipient_address',
    amount: 10.5,
    message: 'Payment for services',
    feeLevel: 'medium'  // 'low', 'medium', 'high'
  });

  console.log('Transaction hash:', result.txHash);
  console.log('Explorer:', result.explorerUrl);

  if (result.success) {
    console.log('Transaction sent successfully!');
  }
} catch (error) {
  console.error('Transaction failed:', error.message);
}
```

### Private Transaction

```typescript
const result = await wallet.sendPrivateTransfer({
  to: 'recipient_address',
  amount: 5.0,
  message: 'Private payment'
});

console.log('Private transfer ID:', result.txHash);
```

### Transaction with Validation

```typescript
import { isValidAddress, isValidAmount } from '@0xgery/0xio-sdk';

const recipient = 'user_input_address';
const amount = parseFloat('user_input_amount');

// Validate inputs
if (!isValidAddress(recipient)) {
  alert('Invalid recipient address');
  return;
}

if (!isValidAmount(amount)) {
  alert('Invalid transaction amount');
  return;
}

// Check balance
const balance = await wallet.getBalance();
if (balance.public < amount) {
  alert('Insufficient balance');
  return;
}

// Send transaction
const result = await wallet.sendTransaction({
  to: recipient,
  amount: amount
});
```

## Disconnecting

```typescript
await wallet.disconnect();
console.log('Disconnected from wallet');
```

## Error Handling

### Using Error Codes

```typescript
import { ZeroXIOWalletError, ErrorCode, isErrorType } from '@0xgery/0xio-sdk';

try {
  await wallet.sendTransaction(txData);
} catch (error) {
  if (isErrorType(error, ErrorCode.EXTENSION_NOT_FOUND)) {
    alert('Please install 0xio Wallet extension');
  } else if (isErrorType(error, ErrorCode.USER_REJECTED)) {
    console.log('User cancelled transaction');
  } else if (isErrorType(error, ErrorCode.INSUFFICIENT_BALANCE)) {
    alert('Not enough balance for this transaction');
  } else if (isErrorType(error, ErrorCode.WALLET_LOCKED)) {
    alert('Please unlock your wallet');
  } else {
    alert('Transaction failed: ' + error.message);
  }
}
```

### All Error Codes

- `EXTENSION_NOT_FOUND` - Wallet extension not installed
- `CONNECTION_REFUSED` - User denied connection
- `USER_REJECTED` - User rejected transaction
- `INSUFFICIENT_BALANCE` - Not enough balance
- `INVALID_ADDRESS` - Invalid wallet address
- `INVALID_AMOUNT` - Invalid transaction amount
- `NETWORK_ERROR` - Network communication failure
- `TRANSACTION_FAILED` - Transaction processing failed
- `PERMISSION_DENIED` - Missing required permissions
- `WALLET_LOCKED` - Wallet needs to be unlocked
- `RATE_LIMIT_EXCEEDED` - Too many requests

## Development Tips

### Enable Debug Mode

```typescript
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  debug: true  // Enables detailed logging
});
```

### Check Browser Support

```typescript
import { checkSDKCompatibility } from '@0xgery/0xio-sdk';

const check = checkSDKCompatibility();
if (!check.compatible) {
  console.error('Browser not compatible:', check.issues);
  console.log('Recommendations:', check.recommendations);
}
```

### Development Utilities

In development mode, access debugging utilities from the browser console:

```javascript
// Enable debug mode
window.__ZEROXIO_SDK_UTILS__.enableDebugMode();

// Get SDK info
window.__ZEROXIO_SDK_UTILS__.getSDKInfo();

// Simulate events (for testing)
window.__ZEROXIO_SDK_UTILS__.simulateExtensionEvent('balanceChanged', {
  newBalance: { total: 100, public: 100, private: 0, currency: 'OCT' }
});
```

## Next Steps

- [API Reference](./api-reference.md) - Complete API documentation
- [SDK Package README](https://github.com/0xGery/0xio-sdk/blob/main/README.md) - Framework-specific examples and integration guides
- [NPM Package](https://www.npmjs.com/package/@0xgery/0xio-sdk) - Latest version and changelog

## Common Issues

### Extension Not Detected

Make sure:
1. 0xio Wallet extension is installed
2. Extension is enabled in browser
3. Page is loaded over HTTPS (except localhost)
4. Using a Chromium-based browser

### Connection Timeout

```typescript
// Increase timeout (default: 30 seconds)
const wallet = new ZeroXIOWallet({
  appName: 'My DApp',
  timeout: 60000  // 60 seconds
});
```

### Permission Denied

Ensure you request the required permissions:

```typescript
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  requiredPermissions: ['read_balance', 'send_transactions']
});
```

## Support

If you encounter issues:

1. Review the Common Issues section above
2. Search [GitHub Issues](https://github.com/0xGery/0xio-sdk/issues)
3. Ask on [Telegram](https://t.me/nullXgery)
4. Email: 0xgery@proton.me
