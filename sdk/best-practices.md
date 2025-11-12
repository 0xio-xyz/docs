# Best Practices

Security and optimization guidelines for integrating the 0xio Wallet SDK.

## Security

### 1. Never Store Private Keys

The SDK never exposes private keys. All signing happens in the wallet extension.

```typescript
// ✅ GOOD - Let wallet handle keys
const result = await wallet.sendTransaction({ to, amount });

// ❌ BAD - Never ask for or store private keys
// Never implement this!
```

### 2. Validate User Input

Always validate addresses and amounts before sending to the SDK.

```typescript
import { isValidAddress, isValidAmount } from '@0xgery/0xio-sdk';

function validateTransaction(to: string, amount: number) {
  if (!isValidAddress(to)) {
    throw new Error('Invalid recipient address');
  }

  if (!isValidAmount(amount) || amount <= 0) {
    throw new Error('Invalid amount');
  }

  return true;
}

// Use before sending
validateTransaction(recipientAddress, sendAmount);
const result = await wallet.sendTransaction({ to: recipientAddress, amount: sendAmount });
```

### 3. Check Permissions

Request only the permissions your app needs.

```typescript
// ✅ GOOD - Request specific permissions
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  requiredPermissions: ['read_balance', 'send_transactions']
});

// ❌ BAD - Don't request unnecessary permissions
const wallet = await createZeroXIOWallet({
  appName: 'Simple Viewer',
  requiredPermissions: ['read_private_balance', 'send_private_transactions'] // Overkill
});
```

### 4. Handle User Rejections Gracefully

Don't show error messages for user-initiated cancellations.

```typescript
import { isErrorType, ErrorCode } from '@0xgery/0xio-sdk';

try {
  await wallet.connect();
} catch (error) {
  if (isErrorType(error, ErrorCode.USER_REJECTED)) {
    // User cancelled - don't show error
    console.log('User declined connection');
    return;
  }

  // Show error for other cases
  alert('Connection failed: ' + error.message);
}
```

### 5. Use HTTPS

Always serve your dApp over HTTPS in production.

```typescript
// ✅ GOOD
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  appUrl: 'https://mydapp.com'
});

// ⚠️ AVOID in production
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  appUrl: 'http://mydapp.com' // Not secure
});
```

## Performance

### 1. Cache Balance Data

Avoid unnecessary balance fetches by caching and using events.

```typescript
let cachedBalance: Balance | null = null;

// Fetch once on connect
wallet.on('connect', async () => {
  cachedBalance = await wallet.getBalance();
});

// Update cache on balance change
wallet.on('balanceChanged', (event) => {
  cachedBalance = event.newBalance;
});

// Use cached value
function displayBalance() {
  if (cachedBalance) {
    console.log('Balance:', cachedBalance.total, 'OCT');
  }
}
```

### 2. Debounce Frequent Requests

Avoid spamming the wallet with rapid requests.

```typescript
import { debounce } from 'lodash';

// Debounce balance refresh
const refreshBalance = debounce(async () => {
  const balance = await wallet.getBalance(true);
  updateUI(balance);
}, 1000);

// Multiple rapid calls will be batched
button.addEventListener('click', refreshBalance);
```

### 3. Use Event Listeners

React to wallet state changes instead of polling.

```typescript
// ✅ GOOD - Event-driven
wallet.on('balanceChanged', (event) => {
  updateBalanceDisplay(event.newBalance);
});

// ❌ BAD - Polling
setInterval(async () => {
  const balance = await wallet.getBalance(true);
  updateBalanceDisplay(balance);
}, 5000); // Wasteful and slow
```

### 4. Lazy Load the SDK

Only load the SDK when needed to reduce initial bundle size.

```typescript
// React example with code splitting
const WalletButton = lazy(() => import('./components/WalletButton'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <WalletButton />
    </Suspense>
  );
}
```

### 5. Batch Related Operations

Group related operations to reduce communication overhead.

```typescript
// ✅ GOOD - Batch operations
async function initializeWallet() {
  await wallet.connect();

  const [balance, address, network] = await Promise.all([
    wallet.getBalance(),
    wallet.getAddress(),
    wallet.getNetwork()
  ]);

  return { balance, address, network };
}

// ❌ BAD - Sequential requests
async function initializeWalletSlow() {
  await wallet.connect();
  const balance = await wallet.getBalance();
  const address = await wallet.getAddress();
  const network = await wallet.getNetwork();
  return { balance, address, network };
}
```

## User Experience

### 1. Show Connection Status

Always display current wallet connection state.

```typescript
wallet.on('connect', () => {
  updateConnectionStatus('Connected');
});

wallet.on('disconnect', () => {
  updateConnectionStatus('Disconnected');
});

wallet.on('extensionLocked', () => {
  updateConnectionStatus('Wallet locked - please unlock');
});
```

### 2. Provide Clear Error Messages

Transform technical errors into user-friendly messages.

```typescript
import { ErrorCode } from '@0xgery/0xio-sdk';

function getUserFriendlyError(error: any): string {
  if (error instanceof ZeroXIOWalletError) {
    switch (error.code) {
      case ErrorCode.EXTENSION_NOT_FOUND:
        return 'Please install 0xio Wallet extension to continue';
      case ErrorCode.INSUFFICIENT_BALANCE:
        return 'You don\'t have enough OCT for this transaction';
      case ErrorCode.NETWORK_ERROR:
        return 'Network error - please check your connection';
      case ErrorCode.WALLET_LOCKED:
        return 'Please unlock your wallet to continue';
      default:
        return 'Something went wrong - please try again';
    }
  }
  return 'An unexpected error occurred';
}
```

### 3. Show Transaction Progress

Keep users informed during long operations.

```typescript
async function sendWithProgress(to: string, amount: number) {
  try {
    updateStatus('Preparing transaction...');

    const result = await wallet.sendTransaction({ to, amount });

    updateStatus('Transaction sent! Waiting for confirmation...');

    // Wait for confirmation
    await new Promise((resolve) => {
      wallet.on('transactionConfirmed', (event) => {
        if (event.txHash === result.txHash) {
          resolve(event);
        }
      });
    });

    updateStatus('Transaction confirmed!');
    showSuccess(result.explorerUrl);
  } catch (error) {
    updateStatus('Transaction failed');
    showError(getUserFriendlyError(error));
  }
}
```

### 4. Handle Extension Not Installed

Guide users to install the wallet if not detected.

```typescript
import { isErrorType, ErrorCode } from '@0xgery/0xio-sdk';

async function connectWallet() {
  try {
    await wallet.initialize();
    await wallet.connect();
  } catch (error) {
    if (isErrorType(error, ErrorCode.EXTENSION_NOT_FOUND)) {
      showInstallPrompt();
      return;
    }
    throw error;
  }
}

function showInstallPrompt() {
  const installUrl = 'https://chromewebstore.google.com/detail/0xio-wallet/anknhjilldkeelailocijnfibefmepcc';

  if (confirm('0xio Wallet extension not found. Install now?')) {
    window.open(installUrl, '_blank');
  }
}
```

### 5. Responsive Loading States

Show loading indicators for async operations.

```typescript
// React example
function WalletBalance() {
  const [balance, setBalance] = useState<Balance | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function loadBalance() {
      setLoading(true);
      try {
        const bal = await wallet.getBalance();
        setBalance(bal);
      } catch (error) {
        console.error('Failed to load balance:', error);
      } finally {
        setLoading(false);
      }
    }

    loadBalance();
  }, []);

  if (loading) return <div>Loading balance...</div>;
  if (!balance) return <div>Failed to load balance</div>;

  return <div>{balance.total} OCT</div>;
}
```

## Testing

### 1. Test Error Scenarios

Test your app's behavior when errors occur.

```typescript
describe('Wallet Integration', () => {
  it('handles extension not found', async () => {
    // Mock wallet not installed
    mockWalletNotInstalled();

    const result = await connectWallet();
    expect(result.success).toBe(false);
    expect(result.error).toBe('EXTENSION_NOT_FOUND');
  });

  it('handles user rejection', async () => {
    mockUserRejects();

    const result = await wallet.connect();
    expect(result.success).toBe(false);
    expect(result.error).toBe('USER_REJECTED');
  });
});
```

### 2. Test with Testnet

Always test on Octra Testnet before deploying.

```typescript
const wallet = await createZeroXIOWallet({
  appName: 'My DApp',
  networkId: 'octra-testnet', // Use testnet for development
  debug: true // Enable detailed logging
});
```

### 3. Mock the SDK for Unit Tests

Create mocks for faster unit testing.

```typescript
// __mocks__/@0xgery/0xio-sdk.ts
export const createZeroXIOWallet = jest.fn().mockResolvedValue({
  connect: jest.fn().mockResolvedValue(true),
  getBalance: jest.fn().mockResolvedValue({
    public: 100,
    total: 100,
    currency: 'OCT'
  }),
  sendTransaction: jest.fn().mockResolvedValue({
    txHash: '0x123',
    success: true
  })
});
```

## Code Organization

### 1. Create a Wallet Service

Encapsulate wallet logic in a dedicated service.

```typescript
// walletService.ts
import { createZeroXIOWallet, ZeroXIOWallet } from '@0xgery/0xio-sdk';

class WalletService {
  private wallet: ZeroXIOWallet | null = null;

  async initialize() {
    this.wallet = await createZeroXIOWallet({
      appName: 'My DApp',
      requiredPermissions: ['read_balance', 'send_transactions']
    });

    await this.wallet.initialize();
    this.setupEventListeners();
  }

  private setupEventListeners() {
    this.wallet?.on('balanceChanged', this.handleBalanceChange);
    this.wallet?.on('disconnect', this.handleDisconnect);
  }

  async connect() {
    return await this.wallet?.connect();
  }

  async getBalance() {
    return await this.wallet?.getBalance();
  }

  // ... other methods
}

export const walletService = new WalletService();
```

### 2. Use TypeScript

Leverage TypeScript for type safety.

```typescript
import { TransactionData, TransactionResult } from '@0xgery/0xio-sdk';

async function sendTypedTransaction(
  to: string,
  amount: number
): Promise<TransactionResult> {
  const txData: TransactionData = {
    to,
    amount,
    feeLevel: 'medium'
  };

  return await wallet.sendTransaction(txData);
}
```

### 3. Separate UI from Logic

Keep wallet logic separate from UI components.

```typescript
// ✅ GOOD - Separated concerns
// hooks/useWallet.ts
export function useWallet() {
  const [balance, setBalance] = useState<Balance | null>(null);

  useEffect(() => {
    wallet.on('balanceChanged', (event) => {
      setBalance(event.newBalance);
    });
  }, []);

  return { balance };
}

// components/Balance.tsx
function Balance() {
  const { balance } = useWallet();
  return <div>{balance?.total} OCT</div>;
}
```

## Deployment

### 1. Environment Configuration

Use environment variables for configuration.

```typescript
const wallet = await createZeroXIOWallet({
  appName: process.env.APP_NAME || 'My DApp',
  appUrl: process.env.APP_URL,
  networkId: process.env.NETWORK_ID || 'octra-testnet',
  debug: process.env.NODE_ENV === 'development'
});
```

### 2. Error Tracking

Integrate error tracking for production.

```typescript
import * as Sentry from '@sentry/browser';

wallet.on('error', (event) => {
  Sentry.captureException(event.error, {
    contexts: {
      wallet: {
        code: event.error.code,
        context: event.context
      }
    }
  });
});
```

### 3. Analytics

Track wallet interactions for insights.

```typescript
wallet.on('connect', () => {
  analytics.track('Wallet Connected');
});

wallet.on('transactionConfirmed', (event) => {
  analytics.track('Transaction Confirmed', {
    txHash: event.txHash,
    status: event.status
  });
});
```

## Summary

**Key Takeaways:**

1. **Security first** - Never handle private keys, validate all inputs
2. **Performance** - Cache data, use events, batch operations
3. **User experience** - Clear status, friendly errors, loading states
4. **Testing** - Test errors, use testnet, mock for unit tests
5. **Organization** - Separate concerns, use TypeScript, encapsulate logic
6. **Production** - Environment config, error tracking, analytics

Following these practices will help you build secure, performant, and user-friendly dApps with the 0xio Wallet SDK.
