# Integration Examples

Framework-specific examples for integrating the 0xio Wallet SDK.

## Table of Contents

- [React](#react)
- [Vue.js](#vuejs)
- [Svelte](#svelte)
- [Vanilla JavaScript](#vanilla-javascript)
- [Next.js](#nextjs)
- [Common Patterns](#common-patterns)

---

## React

### Basic Integration with Hooks

```jsx
import React, { useState, useEffect } from 'react';
import { createZeroXIOWallet, ZeroXIOWallet } from '@0xgery/0xio-sdk';

function WalletConnector() {
  const [wallet, setWallet] = useState<ZeroXIOWallet | null>(null);
  const [connected, setConnected] = useState(false);
  const [address, setAddress] = useState('');
  const [balance, setBalance] = useState(0);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    initializeWallet();
  }, []);

  const initializeWallet = async () => {
    try {
      const walletInstance = await createZeroXIOWallet({
        appName: 'My React DApp',
        appDescription: 'A decentralized application',
        debug: process.env.NODE_ENV === 'development'
      });

      // Setup event listeners
      walletInstance.on('connect', (event) => {
        setConnected(true);
        setAddress(event.address);
        setBalance(event.balance.total);
      });

      walletInstance.on('disconnect', () => {
        setConnected(false);
        setAddress('');
        setBalance(0);
      });

      walletInstance.on('balanceChanged', (event) => {
        setBalance(event.newBalance.total);
      });

      walletInstance.on('accountChanged', (event) => {
        setAddress(event.newAddress);
      });

      setWallet(walletInstance);
    } catch (error) {
      console.error('Failed to initialize wallet:', error);
    }
  };

  const handleConnect = async () => {
    if (!wallet) return;

    try {
      setLoading(true);
      await wallet.connect();
    } catch (error) {
      alert('Connection failed: ' + error.message);
    } finally {
      setLoading(false);
    }
  };

  const handleDisconnect = async () => {
    if (!wallet) return;
    await wallet.disconnect();
  };

  const handleSendTransaction = async () => {
    if (!wallet) return;

    try {
      setLoading(true);
      const result = await wallet.sendTransaction({
        to: 'recipient_address',
        amount: 1,
        message: 'Test transaction'
      });
      alert('Transaction sent: ' + result.txHash);
    } catch (error) {
      alert('Transaction failed: ' + error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="wallet-connector">
      <h2>0xio Wallet</h2>

      {!connected ? (
        <button onClick={handleConnect} disabled={!wallet || loading}>
          {loading ? 'Connecting...' : wallet ? 'Connect Wallet' : 'Loading SDK...'}
        </button>
      ) : (
        <div className="wallet-info">
          <p><strong>Address:</strong> {address}</p>
          <p><strong>Balance:</strong> {balance} OCT</p>

          <div className="actions">
            <button onClick={handleSendTransaction} disabled={loading}>
              {loading ? 'Sending...' : 'Send Transaction'}
            </button>
            <button onClick={handleDisconnect}>Disconnect</button>
          </div>
        </div>
      )}
    </div>
  );
}

export default WalletConnector;
```

### Custom Hook

```tsx
// useWallet.ts
import { useState, useEffect } from 'react';
import { createZeroXIOWallet, ZeroXIOWallet } from '@0xgery/0xio-sdk';

export function useWallet() {
  const [wallet, setWallet] = useState<ZeroXIOWallet | null>(null);
  const [connected, setConnected] = useState(false);
  const [address, setAddress] = useState<string | null>(null);
  const [balance, setBalance] = useState(0);

  useEffect(() => {
    let walletInstance: ZeroXIOWallet;

    async function init() {
      walletInstance = await createZeroXIOWallet({
        appName: 'My DApp',
        debug: process.env.NODE_ENV === 'development'
      });

      walletInstance.on('connect', (event) => {
        setConnected(true);
        setAddress(event.address);
        setBalance(event.balance.total);
      });

      walletInstance.on('disconnect', () => {
        setConnected(false);
        setAddress(null);
        setBalance(0);
      });

      walletInstance.on('balanceChanged', (event) => {
        setBalance(event.newBalance.total);
      });

      setWallet(walletInstance);
    }

    init();

    return () => {
      if (walletInstance) {
        walletInstance.removeAllListeners();
      }
    };
  }, []);

  const connect = async () => {
    if (wallet) {
      await wallet.connect();
    }
  };

  const disconnect = async () => {
    if (wallet) {
      await wallet.disconnect();
    }
  };

  return {
    wallet,
    connected,
    address,
    balance,
    connect,
    disconnect
  };
}

// Usage in component
function MyComponent() {
  const { connected, address, balance, connect, disconnect } = useWallet();

  return (
    <div>
      {connected ? (
        <>
          <p>Address: {address}</p>
          <p>Balance: {balance} OCT</p>
          <button onClick={disconnect}>Disconnect</button>
        </>
      ) : (
        <button onClick={connect}>Connect Wallet</button>
      )}
    </div>
  );
}
```

---

## Vue.js

### Composition API

```vue
<template>
  <div class="wallet-connector">
    <h2>0xio Wallet</h2>

    <div v-if="!connected">
      <button @click="connect" :disabled="!wallet || loading">
        {{ loading ? 'Connecting...' : wallet ? 'Connect Wallet' : 'Loading SDK...' }}
      </button>
    </div>

    <div v-else class="wallet-info">
      <p><strong>Address:</strong> {{ address }}</p>
      <p><strong>Balance:</strong> {{ balance }} OCT</p>

      <div class="actions">
        <button @click="sendTransaction" :disabled="loading">
          {{ loading ? 'Sending...' : 'Send Transaction' }}
        </button>
        <button @click="disconnect">Disconnect</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { createZeroXIOWallet } from '@0xgery/0xio-sdk';

const wallet = ref(null);
const connected = ref(false);
const address = ref('');
const balance = ref(0);
const loading = ref(false);

onMounted(async () => {
  try {
    wallet.value = await createZeroXIOWallet({
      appName: 'My Vue DApp',
      appDescription: 'A decentralized application',
      debug: import.meta.env.DEV
    });

    // Setup event listeners
    wallet.value.on('connect', (event) => {
      connected.value = true;
      address.value = event.address;
      balance.value = event.balance.total;
    });

    wallet.value.on('disconnect', () => {
      connected.value = false;
      address.value = '';
      balance.value = 0;
    });

    wallet.value.on('balanceChanged', (event) => {
      balance.value = event.newBalance.total;
    });
  } catch (error) {
    console.error('Failed to initialize wallet:', error);
  }
});

onUnmounted(() => {
  if (wallet.value) {
    wallet.value.removeAllListeners();
  }
});

const connect = async () => {
  try {
    loading.value = true;
    await wallet.value.connect();
  } catch (error) {
    alert('Connection failed: ' + error.message);
  } finally {
    loading.value = false;
  }
};

const disconnect = async () => {
  await wallet.value.disconnect();
};

const sendTransaction = async () => {
  try {
    loading.value = true;
    const result = await wallet.value.sendTransaction({
      to: 'recipient_address',
      amount: 1,
      message: 'Test transaction'
    });
    alert('Transaction sent: ' + result.txHash);
  } catch (error) {
    alert('Transaction failed: ' + error.message);
  } finally {
    loading.value = false;
  }
};
</script>
```

---

## Svelte

```svelte
<script>
  import { onMount, onDestroy } from 'svelte';
  import { createZeroXIOWallet } from '@0xgery/0xio-sdk';

  let wallet = null;
  let connected = false;
  let address = '';
  let balance = 0;
  let loading = false;

  onMount(async () => {
    try {
      wallet = await createZeroXIOWallet({
        appName: 'My Svelte DApp',
        appDescription: 'A decentralized application',
        debug: import.meta.env.DEV
      });

      wallet.on('connect', (event) => {
        connected = true;
        address = event.address;
        balance = event.balance.total;
      });

      wallet.on('disconnect', () => {
        connected = false;
        address = '';
        balance = 0;
      });

      wallet.on('balanceChanged', (event) => {
        balance = event.newBalance.total;
      });
    } catch (error) {
      console.error('Failed to initialize wallet:', error);
    }
  });

  onDestroy(() => {
    if (wallet) {
      wallet.removeAllListeners();
    }
  });

  async function connect() {
    try {
      loading = true;
      await wallet.connect();
    } catch (error) {
      alert('Connection failed: ' + error.message);
    } finally {
      loading = false;
    }
  }

  async function disconnect() {
    await wallet.disconnect();
  }

  async function sendTransaction() {
    try {
      loading = true;
      const result = await wallet.sendTransaction({
        to: 'recipient_address',
        amount: 1,
        message: 'Test transaction'
      });
      alert('Transaction sent: ' + result.txHash);
    } catch (error) {
      alert('Transaction failed: ' + error.message);
    } finally {
      loading = false;
    }
  }
</script>

<div class="wallet-connector">
  <h2>0xio Wallet</h2>

  {#if !connected}
    <button on:click={connect} disabled={!wallet || loading}>
      {loading ? 'Connecting...' : wallet ? 'Connect Wallet' : 'Loading SDK...'}
    </button>
  {:else}
    <div class="wallet-info">
      <p><strong>Address:</strong> {address}</p>
      <p><strong>Balance:</strong> {balance} OCT</p>

      <div class="actions">
        <button on:click={sendTransaction} disabled={loading}>
          {loading ? 'Sending...' : 'Send Transaction'}
        </button>
        <button on:click={disconnect}>Disconnect</button>
      </div>
    </div>
  {/if}
</div>
```

---

## Vanilla JavaScript

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>0xio Wallet Integration</title>
    <script src="https://unpkg.com/@0xgery/0xio-sdk@latest/dist/index.umd.js"></script>
</head>
<body>
    <div id="app">
        <h2>0xio Wallet Integration</h2>
        <div id="status">Loading...</div>
        <button id="connect-btn" style="display: none;">Connect Wallet</button>
        <div id="wallet-info" style="display: none;">
            <p>Address: <span id="address"></span></p>
            <p>Balance: <span id="balance"></span> OCT</p>
            <button id="send-btn">Send Transaction</button>
            <button id="disconnect-btn">Disconnect</button>
        </div>
    </div>

    <script>
        let wallet;

        async function initWallet() {
            try {
                wallet = await ZeroXIOWalletSDK.createZeroXIOWallet({
                    appName: 'My Vanilla JS DApp',
                    debug: true
                });

                // Setup event listeners
                wallet.on('connect', (event) => {
                    document.getElementById('status').style.display = 'none';
                    document.getElementById('connect-btn').style.display = 'none';
                    document.getElementById('wallet-info').style.display = 'block';
                    document.getElementById('address').textContent = event.address;
                    document.getElementById('balance').textContent = event.balance.total;
                });

                wallet.on('disconnect', () => {
                    document.getElementById('status').style.display = 'block';
                    document.getElementById('status').textContent = 'Disconnected';
                    document.getElementById('connect-btn').style.display = 'block';
                    document.getElementById('wallet-info').style.display = 'none';
                });

                wallet.on('balanceChanged', (event) => {
                    document.getElementById('balance').textContent = event.newBalance.total;
                });

                // Show connect button
                document.getElementById('status').textContent = 'Ready to connect';
                document.getElementById('connect-btn').style.display = 'block';

            } catch (error) {
                document.getElementById('status').textContent = 'Error: ' + error.message;
                console.error('Failed to initialize wallet:', error);
            }
        }

        // Event handlers
        document.getElementById('connect-btn').addEventListener('click', async () => {
            try {
                await wallet.connect();
            } catch (error) {
                alert('Connection failed: ' + error.message);
            }
        });

        document.getElementById('disconnect-btn').addEventListener('click', async () => {
            await wallet.disconnect();
        });

        document.getElementById('send-btn').addEventListener('click', async () => {
            try {
                const result = await wallet.sendTransaction({
                    to: 'recipient_address',
                    amount: 1,
                    message: 'Test transaction'
                });
                alert('Transaction sent: ' + result.txHash);
            } catch (error) {
                alert('Transaction failed: ' + error.message);
            }
        });

        // Initialize on page load
        initWallet();
    </script>
</body>
</html>
```

---

## Next.js

### App Router (Next.js 13+)

```tsx
// app/providers.tsx
'use client';

import { createContext, useContext, useState, useEffect, ReactNode } from 'react';
import { createZeroXIOWallet, ZeroXIOWallet } from '@0xgery/0xio-sdk';

interface WalletContextType {
  wallet: ZeroXIOWallet | null;
  connected: boolean;
  address: string | null;
  balance: number;
  connect: () => Promise<void>;
  disconnect: () => Promise<void>;
}

const WalletContext = createContext<WalletContextType | undefined>(undefined);

export function WalletProvider({ children }: { children: ReactNode }) {
  const [wallet, setWallet] = useState<ZeroXIOWallet | null>(null);
  const [connected, setConnected] = useState(false);
  const [address, setAddress] = useState<string | null>(null);
  const [balance, setBalance] = useState(0);

  useEffect(() => {
    async function init() {
      const walletInstance = await createZeroXIOWallet({
        appName: 'My Next.js DApp',
        debug: process.env.NODE_ENV === 'development'
      });

      walletInstance.on('connect', (event) => {
        setConnected(true);
        setAddress(event.address);
        setBalance(event.balance.total);
      });

      walletInstance.on('disconnect', () => {
        setConnected(false);
        setAddress(null);
        setBalance(0);
      });

      walletInstance.on('balanceChanged', (event) => {
        setBalance(event.newBalance.total);
      });

      setWallet(walletInstance);
    }

    init();
  }, []);

  const connect = async () => {
    if (wallet) await wallet.connect();
  };

  const disconnect = async () => {
    if (wallet) await wallet.disconnect();
  };

  return (
    <WalletContext.Provider value={{ wallet, connected, address, balance, connect, disconnect }}>
      {children}
    </WalletContext.Provider>
  );
}

export function useWallet() {
  const context = useContext(WalletContext);
  if (context === undefined) {
    throw new Error('useWallet must be used within a WalletProvider');
  }
  return context;
}

// app/layout.tsx
import { WalletProvider } from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <WalletProvider>
          {children}
        </WalletProvider>
      </body>
    </html>
  );
}

// app/page.tsx
'use client';

import { useWallet } from './providers';

export default function Home() {
  const { connected, address, balance, connect, disconnect, wallet } = useWallet();

  const handleSend = async () => {
    if (!wallet) return;

    try {
      const result = await wallet.sendTransaction({
        to: 'recipient_address',
        amount: 1
      });
      alert('TX: ' + result.txHash);
    } catch (error) {
      alert('Failed: ' + error.message);
    }
  };

  return (
    <main>
      <h1>0xio Wallet DApp</h1>
      {connected ? (
        <>
          <p>Address: {address}</p>
          <p>Balance: {balance} OCT</p>
          <button onClick={handleSend}>Send</button>
          <button onClick={disconnect}>Disconnect</button>
        </>
      ) : (
        <button onClick={connect}>Connect Wallet</button>
      )}
    </main>
  );
}
```

---

## Common Patterns

### Transaction with Loading State

```typescript
async function sendTransactionWithLoading(wallet: ZeroXIOWallet, txData: TransactionData) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  try {
    setLoading(true);
    setError(null);

    const result = await wallet.sendTransaction(txData);

    console.log('Transaction successful:', result.txHash);
    return result;
  } catch (err) {
    const errorMessage = err instanceof Error ? err.message : 'Transaction failed';
    setError(errorMessage);
    throw err;
  } finally {
    setLoading(false);
  }
}
```

### Form Validation

```typescript
import { isValidAddress, isValidAmount } from '@0xgery/0xio-sdk';

function validateTransactionForm(to: string, amount: string): string | null {
  if (!to) return 'Recipient address is required';
  if (!isValidAddress(to)) return 'Invalid recipient address';

  const numAmount = parseFloat(amount);
  if (!amount || isNaN(numAmount)) return 'Amount is required';
  if (!isValidAmount(numAmount)) return 'Invalid amount';
  if (numAmount <= 0) return 'Amount must be greater than 0';

  return null; // Valid
}
```

### Balance Checking

```typescript
async function sendWithBalanceCheck(wallet: ZeroXIOWallet, amount: number, to: string) {
  const balance = await wallet.getBalance(true); // Force refresh

  if (balance.public < amount) {
    throw new Error(`Insufficient balance. You have ${balance.public} OCT but need ${amount} OCT`);
  }

  return await wallet.sendTransaction({ to, amount });
}
```

---

## More Examples

For more examples and use cases, see:
- [SDK README](https://github.com/0xGery/0xio-sdk/blob/main/README.md)
- [GitHub Examples](https://github.com/0xGery/0xio-sdk/tree/main/examples)
