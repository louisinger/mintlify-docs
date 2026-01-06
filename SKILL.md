---
name: "arkade-builder"
description: "Assists developers building Bitcoin applications on Arkade - a programmable execution layer. Handles wallet integration, payment processing, VTXO lifecycle management, Lightning Network integration, smart contracts, and troubleshooting. Use when developers are working with @arkade-os/sdk, implementing payments, managing virtual UTXOs, integrating Lightning swaps, or building Arkade applications."
license: "MIT"
compatibility: "Node.js v22+, TypeScript/JavaScript, React Native, Web browsers. Works with @arkade-os/sdk and related packages."
---

# Arkade Builder Skill

This skill trains Claude Code agents to effectively assist developers building applications on Arkade - a programmable execution layer for Bitcoin that enables fast, self-custodial financial applications without requiring Bitcoin consensus changes.

## When to Use This Skill

Activate this skill when developers are:

- Setting up Arkade wallets in web, mobile, or server environments
- Implementing payment processing (Arkade-to-Arkade, Lightning, Bitcoin)
- Managing VTXO lifecycle (renewal, recovery, expiration)
- Integrating Lightning Network via submarine swaps
- Building smart contracts with Arkade Script
- Troubleshooting Arkade SDK integration issues
- Implementing custom storage adapters
- Handling settlement strategies and VTXO states

## Core Concepts

### What is Arkade?

Arkade extends the Ark protocol with:

- **VTXOs (Virtual Transaction Outputs)**: Offchain Bitcoin outputs with unilateral exit rights
- **Virtual Mempool**: DAG-structured offchain execution with parallel processing
- **Batch Outputs**: Compression of multiple VTXOs into single onchain outputs
- **Smart Contracts**: Stateful contracts via Arkade Script
- **Dynamic Settlement**: User-controlled choice between instant preconfirmation and Bitcoin finality

### Architecture

```text
Bitcoin Layer (Settlement)
    ↑
Batch Outputs (Compression)
    ↑
Virtual Mempool (Execution)
    ↑
VTXOs (State)
    ↑
Wallets & Applications (Client Layer)
```

### Critical Concepts for Developers

**VTXOs** have two spending paths:

1. Collaborative (user + operator): Instant offchain execution
2. Unilateral (user alone + CSV delay): Exit without operator cooperation

**VTXO States**:

- `preconfirmed`: Offchain, operator-cosigned, requires operator trust
- `unconfirmed`: In commitment transaction, awaiting Bitcoin confirmation
- `settled`: Onchain, full Bitcoin security
- `recoverable`: Valid offchain but cannot be exited unilaterally (expired, swept, sub-dust)
- `spent`: Used as input to another transaction

**Liveness Requirement**: VTXOs must be renewed before batch expiry or operator reclaims funds. This is the primary operational consideration for Arkade applications.

**Settlement Strategies**:

- **Preconfirmation**: Instant execution, operator trust required
- **Bitcoin Finality**: Full Bitcoin security via onchain batch swap

**Unilateral Exit**: Users can always exit by broadcasting presigned transactions, but deep transaction chains increase exit costs (1 Bitcoin transaction per chain level).

## Quick Start Patterns

### Basic Wallet Setup

```typescript
import { SingleKey, Wallet } from '@arkade-os/sdk'

const identity = SingleKey.fromHex('private-key-hex')

const wallet = await Wallet.create({
  identity,
  arkServerUrl: 'https://arkade.computer', // or mutinynet, signet, localhost
})

const address = await wallet.getAddress()
// Returns: ark1... (mainnet) or tark1... (testnet)
```

### Operator URLs

- **Bitcoin mainnet**: `https://arkade.computer`
- **Mutinynet testnet**: `https://mutinynet.arkade.sh`
- **Signet testnet**: `https://signet.arkade.sh`
- **Local regtest**: `http://localhost:7070` (via `nigiri start --ark`)

### Storage Adapters

Choose based on environment:

```typescript
// Browser
import { LocalStorageAdapter } from '@arkade-os/sdk/adapters/localStorage'
import { IndexedDBAdapter } from '@arkade-os/sdk/adapters/indexedDB'

// React Native
import { AsyncStorageAdapter } from '@arkade-os/sdk/adapters/asyncStorage'

// Node.js
import { FileSystemAdapter } from '@arkade-os/sdk/adapters/fileSystem'

const wallet = await Wallet.create({
  identity,
  arkServerUrl,
  storage: new LocalStorageAdapter() // or other adapter
})
```

### Balance Management

```typescript
const balance = await wallet.getBalance()
// {
//   available: BigInt,     // Spendable balance
//   total: BigInt,         // All VTXOs
//   preconfirmed: BigInt,  // Offchain VTXOs
//   settled: BigInt,       // Onchain VTXOs
//   recoverable: BigInt,   // Expired/swept VTXOs
//   boarding: BigInt       // Pending deposits
// }

const vtxos = await wallet.getVtxos()
```

### Sending Payments

```typescript
// Arkade-to-Arkade (instant, near-zero fees)
await wallet.send({
  to: 'ark1...',
  amount: 50000n, // BigInt required
})

// Bitcoin onchain (via swap)
await wallet.send({
  to: 'bc1...',
  amount: 100000n,
})
```

### Receiving Payments

```typescript
const offchainAddress = await wallet.getAddress()

// Monitor incoming funds
const stopMonitoring = wallet.notifyIncomingFunds((notification) => {
  if (notification.type === 'vtxo') {
    for (const vtxo of notification.vtxos) {
      console.log(`Received ${vtxo.amount} sats`)
      console.log(`State: ${vtxo.virtualStatus.state}`)
    }
  }
})
```

### VTXO Lifecycle Management

```typescript
import { VtxoManager } from '@arkade-os/sdk'

const manager = new VtxoManager(wallet, {
  enabled: true,
  thresholdPercentage: 10 // Renew when 10% of lifetime remains
})

// Check and renew expiring VTXOs
const expiringVtxos = await manager.getExpiringVtxos()
if (expiringVtxos.length > 0) {
  await manager.renewVtxos()
}

// Recover swept VTXOs
const balance = await manager.getRecoverableBalance()
if (balance.recoverable > 0n) {
  await manager.recoverVtxos()
}
```

## Lightning Network Integration

Install: `npm install @arkade-os/boltz-swap`

```typescript
import { ArkadeLightning, BoltzSwapProvider } from '@arkade-os/boltz-swap'

const swapProvider = new BoltzSwapProvider({
  apiUrl: 'https://api.ark.boltz.exchange',
  network: 'bitcoin',
})

const arkadeLightning = new ArkadeLightning({
  wallet,
  swapProvider,
})

// Receive Lightning payment
const result = await arkadeLightning.createLightningInvoice({
  amount: 50000,
  description: 'Payment to Arkade wallet',
})
const receivalResult = await arkadeLightning.waitAndClaim(result.pendingSwap)

// Send to Lightning invoice
const payment = await arkadeLightning.sendLightningPayment({
  invoice: 'lnbc...',
  maxFeeSats: 1000,
})
```

## Common Development Patterns

### Pattern: Automatic VTXO Renewal

```typescript
const manager = new VtxoManager(wallet, {
  enabled: true,
  thresholdPercentage: 10
})

// Daily renewal check
setInterval(async () => {
  const expiring = await manager.getExpiringVtxos()
  if (expiring.length > 0) {
    await manager.renewVtxos()
  }
}, 24 * 60 * 60 * 1000)
```

### Pattern: Universal Payment Handler

```typescript
async function handlePayment(destination: string, amount: number) {
  if (destination.startsWith('lnbc') || destination.startsWith('lntb')) {
    // Lightning payment
    return await arkadeLightning.sendLightningPayment({
      invoice: destination,
      maxFeeSats: Math.floor(amount * 0.01) // 1% max fee
    })
  } else if (destination.startsWith('ark1') || destination.startsWith('tark1')) {
    // Arkade payment
    return await wallet.send({ to: destination, amount: BigInt(amount) })
  } else {
    // Bitcoin onchain
    return await wallet.send({ to: destination, amount: BigInt(amount) })
  }
}
```

### Pattern: React Native Mobile Wallet

```typescript
import { AsyncStorageAdapter } from '@arkade-os/sdk/adapters/asyncStorage'
import BackgroundFetch from 'react-native-background-fetch'

const storage = new AsyncStorageAdapter()
const wallet = await Wallet.create({
  identity: SingleKey.fromHex(privateKey),
  arkServerUrl: 'https://mutinynet.arkade.sh',
  storage
})

// Background VTXO renewal
BackgroundFetch.configure({
  minimumFetchInterval: 15
}, async (taskId) => {
  await checkAndRenewVtxos(wallet)
  BackgroundFetch.finish(taskId)
})
```

## Troubleshooting Guide

### Issue: Wallet creation fails

**Symptoms**: `Wallet.create()` throws error or hangs

**Diagnosis**:

- Check operator URL is correct and accessible
- Verify network connectivity
- Ensure operator is running and healthy

**Solution**:

```typescript
// Test operator connectivity
const response = await fetch('https://arkade.computer/v1/info')
const info = await response.json()
console.log('Operator info:', info)

// Validate private key format
if (!/^[0-9a-fA-F]{64}$/.test(privateKeyHex)) {
  throw new Error('Invalid private key format - must be 64 hex characters')
}
```

### Issue: VTXOs expired and funds locked

**Symptoms**: Balance shows `recoverable` but not `available`

**Solution**:

```typescript
const manager = new VtxoManager(wallet)
const balance = await manager.getRecoverableBalance()

if (balance.recoverable > 0n) {
  const txid = await manager.recoverVtxos()
}

// Implement automatic renewal to prevent future expiry
```

### Issue: Insufficient funds error despite showing balance

**Diagnosis**:

- VTXOs in `preconfirmed` state being spent rapidly
- Recoverable balance not included in available balance
- Boarding transaction not yet confirmed

**Solution**:

```typescript
const balance = await wallet.getBalance()
console.log('Available:', balance.available)
console.log('Preconfirmed:', balance.preconfirmed)
console.log('Recoverable:', balance.recoverable)
console.log('Boarding:', balance.boarding)

// Recover if needed
if (balance.recoverable > 0n) {
  await new VtxoManager(wallet).recoverVtxos()
}
```

### Issue: Lightning swap fails

**Common causes**: Invoice expired, insufficient funds, network routing issues

**Solution**:

```typescript
import {
  InvoiceExpiredError,
  InsufficientFundsError,
  NetworkError
} from '@arkade-os/boltz-swap'

try {
  await arkadeLightning.sendLightningPayment({
    invoice: 'lnbc...',
    maxFeeSats: 1000
  })
} catch (error) {
  if (error instanceof InvoiceExpiredError) {
    // Request new invoice
  } else if (error instanceof InsufficientFundsError) {
    // Show required amount
  } else if (error.isRefundable && error.pendingSwap) {
    // Claim refund
    await arkadeLightning.refundVHTLC(error.pendingSwap)
  }
}
```

### Issue: Deep transaction chains causing high exit costs

**Problem**: Extended offchain activity without settlement increases unilateral exit costs

**Solution**: Periodically trigger batch swaps for Bitcoin finality to reset chain depth

## Best Practices

### Security

- Never hardcode private keys
- Validate addresses before sending
- Ensure users can broadcast presigned transactions if operator disappears
- Monitor VTXO expiry with automated renewal
- Backup presigned transactions securely

### Performance

- Use preconfirmation for small/frequent transactions
- Use Bitcoin finality for large/final settlements
- Implement caching for balance queries
- Use background workers for VTXO management

### User Experience

- Show clear balance breakdown (available, preconfirmed, settled, recoverable)
- Explain settlement options (instant vs final)
- Alert users before VTXO expiry
- Show transaction state clearly (pending, preconfirmed, settled)

### Development

- Use testnet (Mutinynet/Signet) before mainnet
- Implement comprehensive error handling
- Test storage adapters across app restarts
- Monitor operator health (check `/v1/info` endpoint)

## Key Terminology

- **Ark Protocol**: Foundation (VTXOs, batch outputs, batch swaps)
- **Arkade**: Execution layer (virtual mempool, smart contracts, programmability)
- **VTXO**: Virtual Transaction Output with unilateral exit rights
- **Batch Output**: Onchain Bitcoin output compressing multiple VTXOs
- **Virtual Mempool**: DAG-structured offchain execution environment
- **Batch Swap**: Atomic onchain settlement exchanging VTXOs
- **Preconfirmation**: Instant offchain execution with operator trust
- **Bitcoin Finality**: Full Bitcoin security after onchain commitment
- **Unilateral Exit**: User withdrawal without operator cooperation
- **Liveness Requirement**: VTXOs must be renewed before expiry

## Agent Guidelines

When assisting Arkade developers:

1. **Clarify concepts**: Always ask before making assumptions about VTXOs, settlement, or exit mechanisms
2. **Show working code**: Provide runnable examples before explanations
3. **Reference docs**: Link to relevant sections at [docs.arkade.sh](https://docs.arkade.sh)
4. **Distinguish Ark vs Arkade**: Ark = protocol, Arkade = execution layer
5. **Address liveness**: Always mention VTXO expiry and renewal when relevant
6. **Explain trust model**: Be clear about preconfirmed vs settled trade-offs
7. **Warn about exit costs**: Deep transaction chains increase unilateral exit costs
8. **Use correct terminology**: VTXOs, batch outputs, virtual mempool, batch swaps
9. **Recommend testnet first**: Always suggest Mutinynet or Signet before mainnet
10. **Implement error handling**: Show comprehensive error handling for production

### Response Template

**"How do I setup an Arkade wallet?"**
→ Show Wallet.create() with operator URL and appropriate storage adapter

**"How do I send/receive payments?"**
→ Show wallet.send() and notifyIncomingFunds() with preconfirmation explanation

**"What happens if VTXOs expire?"**
→ Explain liveness requirement and show VtxoManager usage

**"How do I integrate Lightning?"**
→ Show @arkade-os/boltz-swap setup with submarine swap examples

**"What's preconfirmed vs settled?"**
→ Explain: preconfirmed = instant + operator trust, settled = Bitcoin finality

**"How does unilateral exit work?"**
→ Explain presigned transactions, exit path requirements, cost considerations

## Resources

- TypeScript SDK: <https://github.com/arkade-os/ts-sdk>
- Documentation: <https://docs.arkade.sh>
- API Reference: <https://buf.build/arkade-os/arkd>
- Community: <https://t.me/arkade_os>

For comprehensive reference material including advanced smart contracts, detailed API documentation, and platform-specific deployment guides, see [references/REFERENCE.md](references/REFERENCE.md).
