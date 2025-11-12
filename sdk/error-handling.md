# Error Handling Guide

Comprehensive guide to handling errors in the 0xio Wallet SDK.

## Error Types

### ZeroXIOWalletError

All SDK errors extend `ZeroXIOWalletError`:

```typescript
import { ZeroXIOWalletError, ErrorCode } from '@0xgery/0xio-sdk';

try {
  await wallet.connect();
} catch (error) {
  if (error instanceof ZeroXIOWalletError) {
    console.log('Error code:', error.code);
    console.log('Message:', error.message);
    console.log('Details:', error.details);
  }
}
```

## Error Codes

| Code | Description | User Action |
|------|-------------|-------------|
| `EXTENSION_NOT_FOUND` | Wallet extension not installed | Install extension |
| `CONNECTION_REFUSED` | User denied connection | Try again |
| `USER_REJECTED` | User rejected transaction | Try again |
| `INSUFFICIENT_BALANCE` | Not enough balance | Add funds |
| `INVALID_ADDRESS` | Invalid wallet address | Check address |
| `INVALID_AMOUNT` | Invalid transaction amount | Check amount |
| `NETWORK_ERROR` | Network communication failure | Retry |
| `TRANSACTION_FAILED` | Transaction processing failed | Check details |
| `PERMISSION_DENIED` | Missing required permissions | Grant permissions |
| `WALLET_LOCKED` | Wallet needs to be unlocked | Unlock wallet |
| `RATE_LIMIT_EXCEEDED` | Too many requests | Wait and retry |

## Handling Specific Errors

### Extension Not Found

```typescript
import { isErrorType, ErrorCode } from '@0xgery/0xio-sdk';

try {
  await wallet.initialize();
} catch (error) {
  if (isErrorType(error, ErrorCode.EXTENSION_NOT_FOUND)) {
    alert('Please install 0xio Wallet extension');
    window.open('https://chromewebstore.google.com/detail/0xio-wallet/anknhjilldkeelailocijnfibefmepcc');
  }
}
```

### User Rejection

```typescript
try {
  await wallet.connect();
} catch (error) {
  if (isErrorType(error, ErrorCode.USER_REJECTED)) {
    console.log('User cancelled connection');
    // Don't show error - user intentionally cancelled
    return;
  }
  // Show error for other cases
  alert('Connection failed: ' + error.message);
}
```

### Insufficient Balance

```typescript
try {
  await wallet.sendTransaction({ to, amount });
} catch (error) {
  if (isErrorType(error, ErrorCode.INSUFFICIENT_BALANCE)) {
    const balance = await wallet.getBalance();
    alert(`Insufficient balance. You have ${balance.public} OCT but need ${amount} OCT`);
  }
}
```

## Retry Logic

### Automatic Retry

```typescript
import { retry } from '@0xgery/0xio-sdk';

// Retry up to 3 times with 1 second delay
const balance = await retry(
  () => wallet.getBalance(true),
  3,
  1000
);
```

### Custom Retry with Exponential Backoff

```typescript
async function sendWithRetry(txData, maxAttempts = 3) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await wallet.sendTransaction(txData);
    } catch (error) {
      if (attempt === maxAttempts) throw error;

      if (isErrorType(error, ErrorCode.NETWORK_ERROR)) {
        const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }

      throw error; // Don't retry non-network errors
    }
  }
}
```

## Error Boundaries (React)

```tsx
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

class WalletErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Wallet Error</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Try Again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Global Error Handling

```typescript
wallet.on('error', (event) => {
  console.error('Wallet error:', event.error);

  // Log to error tracking service
  logErrorToService({
    code: event.error.code,
    message: event.error.message,
    context: event.context,
    timestamp: new Date().toISOString()
  });

  // Show user-friendly message
  showErrorNotification(getFriendlyErrorMessage(event.error.code));
});

function getFriendlyErrorMessage(code: ErrorCode): string {
  const messages = {
    [ErrorCode.EXTENSION_NOT_FOUND]: 'Please install 0xio Wallet',
    [ErrorCode.WALLET_LOCKED]: 'Please unlock your wallet',
    [ErrorCode.NETWORK_ERROR]: 'Network error. Please try again.',
    [ErrorCode.INSUFFICIENT_BALANCE]: 'Insufficient balance',
  };

  return messages[code] || 'An error occurred';
}
```

## Validation Before Operations

```typescript
import { isValidAddress, isValidAmount } from '@0xgery/0xio-sdk';

async function sendSafely(to: string, amount: number) {
  // Validate inputs
  if (!isValidAddress(to)) {
    throw new Error('Invalid recipient address');
  }

  if (!isValidAmount(amount)) {
    throw new Error('Invalid amount');
  }

  // Check balance
  const balance = await wallet.getBalance();
  if (balance.public < amount) {
    throw new Error('Insufficient balance');
  }

  // Send transaction
  return await wallet.sendTransaction({ to, amount });
}
```

## Best Practices

1. **Always check error types** before showing messages to users
2. **Don't retry user rejections** - respect user decisions
3. **Provide context** in error messages
4. **Log errors** for debugging
5. **Validate inputs** before sending to SDK
6. **Handle network errors gracefully** with retries
7. **Use error boundaries** in React applications
8. **Test error scenarios** during development

## Debug Mode

Enable debug mode for detailed error information:

```typescript
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  debug: true // Enable detailed logging
});
```

## Support

For help with specific errors:
- [GitHub Issues](https://github.com/0xGery/0xio-sdk/issues)
- [Telegram](https://t.me/nullXgery)
