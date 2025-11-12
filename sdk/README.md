# 0xio SDK Documentation

Official documentation for the 0xio Wallet SDK - enabling seamless integration between decentralized applications and the 0xio Wallet browser extension on Octra blockchain.

## Table of Contents

1. [Getting Started](./getting-started.md)
2. [API Reference](./api-reference.md)
3. [Examples](./examples.md)
4. [Type Definitions](./types.md)
5. [Error Handling](./error-handling.md)
6. [Best Practices](./best-practices.md)

## Quick Links

- [NPM Package](https://www.npmjs.com/package/@0xgery/0xio-sdk)
- [GitHub Repository](https://github.com/0xGery/0xio-sdk)
- [Chrome Extension](https://chromewebstore.google.com/detail/0xio-wallet/anknhjilldkeelailocijnfibefmepcc)

## Overview

The 0xio SDK is a TypeScript/JavaScript library that provides:

- **Wallet Connection**: Connect to 0xio Wallet extension
- **Transaction Management**: Send public and private transactions
- **Balance Queries**: Read public and private balances
- **Event System**: Real-time updates for wallet state changes
- **Type Safety**: Full TypeScript support with comprehensive types
- **Framework Agnostic**: Works with React, Vue, Svelte, or vanilla JS

## Installation

```bash
npm install @0xgery/0xio-sdk
```

## Quick Start

```typescript
import { createZeroXIOWallet } from '@0xgery/0xio-sdk';

// Create wallet instance
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  appDescription: 'A decentralized application',
  autoConnect: true
});

// Get balance
const balance = await wallet.getBalance();
console.log('Balance:', balance.total, 'OCT');

// Send transaction
const result = await wallet.sendTransaction({
  to: 'recipient_address',
  amount: 10,
  message: 'Payment'
});

console.log('Transaction hash:', result.txHash);
```

## Key Concepts

### Wallet Instance

The main `ZeroXIOWallet` class manages all interactions with the wallet extension:

```typescript
import { ZeroXIOWallet } from '@0xgery/0xio-sdk';

const wallet = new ZeroXIOWallet({
  appName: 'My DApp',
  requiredPermissions: ['read_balance', 'send_transactions']
});

await wallet.initialize();
await wallet.connect();
```

### Event-Driven Architecture

The SDK uses an event system for real-time updates:

```typescript
wallet.on('balanceChanged', (event) => {
  console.log('New balance:', event.newBalance.total);
});

wallet.on('networkChanged', (event) => {
  console.log('Network changed to:', event.network.name);
});

wallet.on('accountChanged', (event) => {
  console.log('Account changed to:', event.newAddress);
});
```

### Transaction Types

The SDK supports multiple transaction types:

- **Public Transactions**: Standard blockchain transactions
- **Private Transactions**: Encrypted transfers with privacy
- **Bulk Transactions**: Multiple recipients in one transaction

### Error Handling

The SDK provides typed errors for better error handling:

```typescript
import { ZeroXIOWalletError, ErrorCode } from '@0xgery/0xio-sdk';

try {
  await wallet.sendTransaction(txData);
} catch (error) {
  if (error instanceof ZeroXIOWalletError) {
    if (error.code === ErrorCode.INSUFFICIENT_BALANCE) {
      console.log('Not enough balance');
    }
  }
}
```

## Browser Compatibility

The SDK requires a Chromium-based browser:

- Chrome 88+
- Edge 88+
- Brave 1.20+
- Opera 74+

## Network Support

- **Octra Testnet**: Default network for development
- **Custom Networks**: Configure custom RPC endpoints

## Next Steps

- [Getting Started Guide](./getting-started.md) - Detailed setup instructions
- [API Reference](./api-reference.md) - Complete API documentation
- [Examples](./examples.md) - Framework-specific integration examples
- [Type Definitions](./types.md) - TypeScript type reference
- [Error Handling](./error-handling.md) - Error codes and handling patterns
- [Best Practices](./best-practices.md) - Security and optimization guidelines

## Support

- **Issues**: [GitHub Issues](https://github.com/0xio-xyz/docs/issues)
- **Telegram**: [@nullXgery](https://t.me/nullXgery)
- **Email**: team@0xio.xyz

## License

MIT License - see the SDK repository for details.
