# Web3 Risk-Based Test Matrix

A practical framework for translating Web3 product and engineering risks into test coverage, automation priorities, and release controls.

## Purpose

A risk model identifies what can go wrong.

A risk-based test matrix answers the next question:

> Given the risk, where and how should we validate it?

The objective is not to automate every possible scenario.

The objective is to invest testing effort where failure would create the greatest impact while selecting the test layer that provides the fastest and most reliable feedback.

---

## Risk Classification

Risks are evaluated using three dimensions:

| Dimension     | Scale | Question                                                        |
| ------------- | ----: | --------------------------------------------------------------- |
| Impact        |   1–5 | How damaging would the failure be?                              |
| Likelihood    |   1–5 | How likely is the failure to occur?                             |
| Detectability |   1–5 | How difficult would the failure be to detect before production? |

Risk Priority is calculated as:

`Impact × Likelihood × Detectability`

The resulting score helps guide testing investment.

---

## Priority Levels

|  Score | Priority | Expected Response                                |
| -----: | -------- | ------------------------------------------------ |
|   1–15 | Low      | Normal functional coverage                       |
|  16–30 | Medium   | Targeted automated coverage                      |
|  31–60 | High     | Multiple layers of automated validation          |
| 61–125 | Critical | Defense-in-depth validation and release controls |

These thresholds are guidelines rather than absolute release rules.

Context matters.

A lower-scoring security or financial risk may still require critical-level controls because of its potential blast radius.

---

# Risk-Based Test Matrix

| Risk                                        |  I |  L |  D | Score | Primary Validation         | Additional Protection     |
| ------------------------------------------- | -: | -: | -: | ----: | -------------------------- | ------------------------- |
| Unauthorized contract access                |  5 |  2 |  4 |    40 | Contract negative tests    | Fuzzing, security review  |
| Incorrect token accounting                  |  5 |  3 |  4 |    60 | Unit + invariant tests     | Fuzzing, reconciliation   |
| Funds permanently locked                    |  5 |  2 |  5 |    50 | Contract/invariant tests   | Security review           |
| Incorrect reward calculation                |  5 |  3 |  3 |    45 | Unit/property tests        | Integration tests         |
| Contract upgrade corrupts state             |  5 |  2 |  5 |    50 | Upgrade simulation         | Storage validation        |
| Wrong contract address configured           |  5 |  2 |  2 |    20 | Deployment validation      | Environment smoke test    |
| Transaction incorrectly reports success     |  4 |  3 |  3 |    36 | Integration tests          | E2E transaction test      |
| Duplicate transaction submission            |  4 |  3 |  3 |    36 | Integration tests          | UI protection             |
| Wallet account change not detected          |  3 |  3 |  3 |    27 | Wallet integration test    | E2E                       |
| Incorrect network selected                  |  3 |  3 |  2 |    18 | Integration test           | E2E                       |
| RPC provider unavailable                    |  3 |  4 |  2 |    24 | Resilience testing         | Monitoring/failover       |
| RPC returns stale data                      |  4 |  3 |  4 |    48 | RPC validation             | Multi-provider comparison |
| Indexer misses blockchain event             |  4 |  3 |  4 |    48 | Reconciliation test        | Monitoring                |
| Indexer processes event twice               |  4 |  2 |  4 |    32 | Idempotency test           | Data reconciliation       |
| API returns stale balance                   |  3 |  4 |  3 |    36 | API/integration test       | On-chain reconciliation   |
| Chain reorganization corrupts indexed state |  5 |  2 |  5 |    50 | Reorg simulation           | Reconciliation            |
| Transaction remains pending                 |  3 |  3 |  2 |    18 | Transaction lifecycle test | UX validation             |
| Wallet signature rejected                   |  2 |  4 |  1 |     8 | Integration test           | E2E                       |
| Cosmetic UI defect                          |  1 |  3 |  1 |     3 | Component/E2E              | Visual testing            |

---

# Validation by Test Layer

A strong test strategy distributes coverage across multiple layers.

## Smart Contract Layer

Best suited for:

* authorization
* accounting
* state transitions
* mathematical correctness
* invariants
* revert conditions
* protocol rules

Typical techniques:

* unit tests
* fuzz testing
* invariant testing
* property-based testing
* negative testing

These tests should provide the fastest feedback for contract-level behavior.

---

## API & RPC Layer

Best suited for:

* API contracts
* RPC responses
* blockchain queries
* backend validation
* error handling
* provider resilience
* timeout behavior

These tests avoid unnecessary browser dependencies while validating service-level behavior.

---

## Integration Layer

Best suited for boundaries between systems.

Examples:

`Contract → Events → Indexer`

`Indexer → Database → API`

`Wallet → Transaction → RPC`

`API → Blockchain`

Integration tests are particularly important in Web3 because many failures occur between components rather than within individual components.

---

## End-to-End Layer

Best suited for critical user journeys.

Examples:

`Connect Wallet → Sign → Submit → Confirm → Update UI`

or:

`Stake Tokens → Confirm Transaction → Verify Position`

E2E tests should focus on high-value workflows rather than attempting to reproduce all lower-level coverage through the browser.

---

## Production Validation

Some risks cannot be eliminated entirely before deployment.

Production controls may include:

* transaction monitoring
* contract event monitoring
* RPC health monitoring
* indexer lag monitoring
* balance reconciliation
* anomaly detection
* error-rate monitoring

Testing and observability should therefore be treated as complementary quality controls.

---

# Example: Testing a Token Deposit

Consider a user depositing tokens into a protocol.

The visible workflow may appear simple:

`Connect → Approve → Deposit → Confirm`

But the underlying risks span several layers.

| Risk                                  | Best Validation      |
| ------------------------------------- | -------------------- |
| User cannot deposit zero              | Contract unit test   |
| User cannot deposit more than balance | Contract test        |
| Contract accounting remains correct   | Invariant test       |
| Approval transaction fails            | Integration test     |
| Deposit transaction reverts           | Integration test     |
| UI handles rejected signature         | E2E test             |
| UI tracks pending transaction         | E2E test             |
| Indexer receives deposit event        | Integration test     |
| API reflects new deposit              | API/integration test |
| UI eventually displays new position   | E2E test             |

This prevents a common automation mistake:

> Recreating every possible validation through the UI.

The browser should validate the user journey.

The contract should validate protocol rules.

The API layer should validate service behavior.

Integration tests should validate system boundaries.

---

# Automation Priority

Risk can also determine when tests execute.

### Pull Request

Fast tests providing immediate developer feedback:

* static analysis
* unit tests
* contract tests
* critical API tests
* selected integration tests

### Merge / Main Branch

Broader validation:

* complete contract suite
* integration tests
* API regression
* critical E2E workflows

### Scheduled / Nightly

Expensive or long-running validation:

* large fuzz campaigns
* extended invariant testing
* cross-browser suites
* multi-network validation
* resilience testing
* reconciliation testing

### Pre-Release

Release-specific validation:

* deployment configuration
* contract address validation
* environment smoke tests
* critical transaction workflows
* upgrade simulation where applicable
* release risk review

---

# Release Decision Model

Testing should contribute evidence to a release decision rather than simply produce a pass/fail test count.

A release assessment should consider:

**Change Risk**

What changed?

**Coverage**

Which risks were validated?

**Results**

What failed or remains unknown?

**Residual Risk**

Which risks still exist after testing?

**Observability**

Can those risks be detected quickly in production?

**Rollback / Recovery**

Can the system recover if the risk materializes?

For immutable or financially sensitive blockchain operations, recovery may be difficult or impossible.

That should increase the required level of pre-release confidence.

---

# Guiding Principle

Risk-based testing asks a different question from traditional test planning.

Instead of:

> How many test cases do we need?

Ask:

> What evidence do we need to demonstrate that the most important risks are adequately controlled?

That shift turns testing from a test-execution activity into a Quality Engineering discipline.
