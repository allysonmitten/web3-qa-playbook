# DApp Test Strategy

## Objective

Validate that users can safely and predictably interact with decentralized functionality across the frontend, wallet, blockchain, and supporting services.

## Critical Boundaries

```text
Frontend ↔ Wallet
Wallet ↔ RPC
RPC ↔ Blockchain
Blockchain ↔ Indexer
Indexer ↔ API
API ↔ Frontend
```

## Core Scenarios

### Wallet

Validate:

* connection
* disconnection
* account switching
* unsupported wallet
* rejected connection

### Network

Validate:

* correct network
* incorrect network
* network switching
* unsupported network
* chain configuration

### Signing

Validate:

* signature approval
* signature rejection
* unexpected cancellation
* correct signing payload

### Transactions

Validate:

* submission
* rejection
* pending state
* confirmation
* revert
* replacement
* timeout
* duplicate submission protection

### State Synchronization

Validate that confirmed blockchain activity eventually appears correctly through:

```text
Blockchain → Indexer → API → UI
```

## E2E Philosophy

E2E tests should focus on critical journeys.

Example:

```text
Connect Wallet
      ↓
Approve Token
      ↓
Deposit
      ↓
Confirm Transaction
      ↓
Verify Contract State
      ↓
Verify Indexed State
      ↓
Verify UI
```

Protocol rules should remain primarily covered at lower test layers.

## Guiding Principle

A successful DApp test validates more than what the browser displays; it validates that the **user action produced the intended system outcome**.
