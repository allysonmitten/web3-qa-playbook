# Web3 Risk Model

A practical framework for identifying and prioritizing quality risks across decentralized applications.

## Purpose

Web3 systems introduce failure modes that differ from traditional applications.

Failures may involve:

* irreversible transactions
* direct financial loss
* incorrect smart contract state
* wallet and signing errors
* RPC instability
* delayed indexer updates
* network-specific behavior
* inconsistent on-chain and off-chain data

A useful Quality Engineering strategy should begin with risk, not test cases.

The purpose of this model is to identify:

1. What can fail?
2. What is the potential impact?
3. How likely is the failure?
4. How detectable is it?
5. Which test layers provide the strongest protection?

---

## Risk Scoring Model

Each risk can be evaluated across three dimensions:

| Dimension     | Description                                                   |
| ------------- | ------------------------------------------------------------- |
| Impact        | How severe is the outcome if the failure occurs?              |
| Likelihood    | How likely is the failure to occur?                           |
| Detectability | How difficult is the problem to detect before reaching users? |

A simple scoring model can use values from 1–5.

### Impact

| Score | Meaning                                                       |
| ----: | ------------------------------------------------------------- |
|     1 | Negligible user impact                                        |
|     2 | Minor degraded experience                                     |
|     3 | Significant functional problem                                |
|     4 | Major business or user impact                                 |
|     5 | Asset loss, security failure, or irreversible critical impact |

### Likelihood

| Score | Meaning  |
| ----: | -------- |
|     1 | Rare     |
|     2 | Unlikely |
|     3 | Possible |
|     4 | Likely   |
|     5 | Frequent |

### Detectability

Higher scores indicate greater difficulty detecting the issue.

| Score | Meaning                                                       |
| ----: | ------------------------------------------------------------- |
|     1 | Immediately obvious                                           |
|     2 | Easily detected through normal testing                        |
|     3 | Requires targeted validation                                  |
|     4 | Difficult to detect                                           |
|     5 | May remain hidden until production or financial impact occurs |

---

## Risk Priority

A simple Risk Priority Number can be calculated as:

**Risk Priority = Impact × Likelihood × Detectability**

Example:

```text
Incorrect contract authorization

Impact:        5
Likelihood:    2
Detectability: 4

Risk Priority = 5 × 2 × 4 = 40
```

The number itself is not the objective.

The purpose of scoring is to create a consistent way to compare risks and guide:

* test investment
* automation priority
* release gates
* observability requirements
* manual review
* security validation

---

# Core Web3 Risk Categories

## 1. Asset & Financial Risk

Failures that may result in loss, locking, duplication, or incorrect movement of assets.

Examples:

* incorrect token transfer amounts
* duplicated transactions
* funds transferred to an incorrect address
* funds permanently locked in a contract
* reward calculations producing incorrect payouts
* precision and rounding errors
* incorrect fee calculations

Typical impact:

**Critical**

Recommended validation:

* smart contract unit tests
* property-based testing
* fuzz testing
* invariant testing
* integration tests
* balance reconciliation
* mainnet-fork testing
* independent security review

---

## 2. Authorization & Access Control Risk

Failures in permissions, ownership, roles, or privileged contract functionality.

Examples:

* unauthorized account executes an admin function
* owner-only restriction missing
* role escalation
* incorrect permission revocation
* compromised upgrade permissions
* incorrect multisig assumptions

Typical impact:

**Critical**

Recommended validation:

* permission matrix testing
* negative contract tests
* fuzz testing
* invariant testing
* privileged action review
* deployment configuration validation

Important questions:

* Who can perform this action?
* Who should never be able to perform it?
* Can privileges change?
* Can privileges be transferred?
* What happens when an account loses permission?

---

## 3. Smart Contract State Risk

Failures involving invalid or unexpected contract state transitions.

Examples:

* balances become inconsistent
* state transition occurs in the wrong order
* impossible state becomes reachable
* accounting totals no longer reconcile
* contract becomes unusable after an edge-case interaction

Recommended validation:

* state transition tests
* unit tests
* property-based testing
* fuzz testing
* invariant testing
* integration tests

Example invariant:

```text
The sum of all user balances must never exceed
the assets controlled by the contract.
```

---

## 4. Transaction Lifecycle Risk

A transaction passes through multiple states:

```text
Created
  ↓
Signed
  ↓
Submitted
  ↓
Pending
  ↓
Included
  ↓
Confirmed
```

At any stage, failure is possible.

Examples:

* wallet rejects transaction
* transaction submission fails
* gas estimation fails
* transaction remains pending
* transaction is replaced
* transaction reverts
* UI reports success before confirmation
* application fails to recognize confirmed transaction

Recommended validation:

* wallet integration tests
* transaction state-machine tests
* RPC-level validation
* UI integration tests
* E2E workflows
* failure injection

---

## 5. Wallet Integration Risk

Wallets introduce another system boundary between the user and application.

Examples:

* wallet fails to connect
* wrong account used
* unsupported chain
* incorrect network switching
* signing request contains unexpected data
* rejected signature is handled incorrectly
* session persists after wallet disconnect
* account switch is not detected

Recommended validation:

* wallet connection tests
* account-switching scenarios
* network-switching scenarios
* signature rejection
* transaction rejection
* disconnected wallet behavior
* multiple wallet providers

---

## 6. RPC Provider Risk

Web3 applications rely heavily on external or self-hosted RPC infrastructure.

Possible failures:

* RPC unavailable
* responses delayed
* rate limits reached
* stale block data returned
* inconsistent results across providers
* unsupported RPC methods
* malformed responses
* provider returns temporary errors

Recommended validation:

* API contract testing
* timeout scenarios
* retry testing
* provider failover testing
* latency testing
* response validation
* multi-provider comparison

Architecture should assume:

**RPC providers can fail.**

Testing should verify what the application does when they do.

---

## 7. Indexer Risk

Many applications do not query blockchain state directly for every request.

Instead:

```text
Blockchain
    ↓
Events
    ↓
Indexer
    ↓
Database
    ↓
API
    ↓
Application
```

This introduces eventual consistency.

Possible failures:

* missed events
* duplicated events
* delayed event processing
* indexing stops
* incorrect block processed
* stale user balances
* chain reorganization not handled
* database state diverges from blockchain state

Recommended validation:

* blockchain-to-indexer reconciliation
* event replay testing
* delayed indexing scenarios
* duplicate event handling
* reorganization scenarios
* eventual consistency checks

---

## 8. On-Chain vs Off-Chain Consistency Risk

A common Web3 quality problem is disagreement between different system layers.

Example:

```text
Smart Contract Balance: 100
Indexer Balance:         100
API Balance:             95
UI Balance:              95
```

The blockchain may be correct while the user experience is wrong.

Testing should therefore validate consistency across:

```text
Contract
   ↓
Blockchain Events
   ↓
Indexer
   ↓
API
   ↓
Frontend
```

Recommended approach:

* treat blockchain state as the source of truth where appropriate
* reconcile indexed data against contract state
* define acceptable consistency windows
* verify eventual convergence

---

## 9. Network & Chain Risk

Applications may behave differently across networks.

Examples:

* incorrect chain ID
* unsupported network
* network-specific contract addresses
* different block times
* different gas behavior
* incorrect deployment configuration
* RPC misconfiguration

Recommended validation:

* environment configuration tests
* chain ID validation
* contract address validation
* multi-network smoke tests
* deployment verification

---

## 10. Chain Reorganization Risk

Blockchain state can occasionally reorganize.

A transaction that appears included may temporarily disappear or move to another block.

Potential effects:

* duplicate processing
* incorrect indexer state
* UI showing transactions as final too early
* event processing inconsistencies

Recommended validation:

* confirmation threshold testing
* indexer reorg handling
* transaction reprocessing tests
* idempotency checks

---

## 11. Upgradeability Risk

Upgradeable contracts introduce additional risk.

Examples:

* storage layout corruption
* implementation mismatch
* initialization errors
* unauthorized upgrades
* upgrade breaks invariants

Recommended validation:

* storage compatibility checks
* pre/post-upgrade state validation
* permission testing
* regression suites
* invariant testing
* upgrade simulation

---

## 12. User Experience Risk

A technically correct blockchain transaction can still produce a poor or misleading user experience.

Examples:

* UI shows "success" while transaction is pending
* wrong balance displayed
* unclear signature request
* transaction failure gives no explanation
* wallet network mismatch is confusing
* user submits the same transaction repeatedly

Recommended validation:

* transaction status UX
* pending-state behavior
* error messaging
* wallet interaction usability
* duplicate action protection
* accessibility testing

---

# Example Risk Matrix

| Risk                                      | Impact | Likelihood | Detectability | Priority |
| ----------------------------------------- | -----: | ---------: | ------------: | -------: |
| Unauthorized contract function            |      5 |          2 |             4 |       40 |
| Incorrect token accounting                |      5 |          3 |             4 |       60 |
| RPC provider outage                       |      3 |          4 |             2 |       24 |
| Stale indexer balance                     |      3 |          4 |             3 |       36 |
| Incorrect network selected                |      3 |          3 |             2 |       18 |
| Transaction shown as successful too early |      4 |          3 |             3 |       36 |
| Wallet account switch not detected        |      3 |          3 |             3 |       27 |
| Cosmetic UI defect                        |      1 |          3 |             1 |        3 |

This makes an important point:

A high-priority risk does not necessarily mean the bug is more likely.

Sometimes the combination of:

**high impact + low detectability**

is enough to justify significant testing investment.

---

# Risk to Test Layer Mapping

Different risks should be validated at different layers.

| Risk                       | Primary Test Layers         |
| -------------------------- | --------------------------- |
| Contract accounting        | Unit, fuzz, invariant       |
| Access control             | Unit, negative, security    |
| Contract state transitions | Unit, property, integration |
| Wallet workflows           | Integration, E2E            |
| Transaction lifecycle      | Integration, E2E, RPC       |
| API correctness            | API, integration            |
| Indexer consistency        | Integration, reconciliation |
| RPC reliability            | API, resilience             |
| User experience            | Component, E2E              |
| Cross-system consistency   | Integration, E2E            |

The goal is not to push every scenario into end-to-end automation.

The goal is to validate each risk at the **lowest effective layer** while maintaining sufficient system-level confidence.

---

# Release Risk

Risk should also influence release decisions.

A release containing changes to:

```text
CSS styling
```

should not necessarily require the same validation strategy as a release modifying:

```text
Smart contract authorization
```

Release validation should consider:

* changed components
* financial exposure
* security impact
* contract state impact
* affected integrations
* blast radius
* reversibility

Higher-risk changes should trigger deeper validation.

For example:

```text
Contract accounting change

→ unit suite
→ fuzz suite
→ invariant suite
→ integration tests
→ regression suite
→ deployment verification
→ security review
```

---

# Guiding Principle

A strong quality strategy should answer:

> What is the most damaging way this system could fail, and what evidence do we have that it will not?

The purpose of risk-based Quality Engineering is to spend testing effort where it creates the greatest reduction in product and engineering risk.
