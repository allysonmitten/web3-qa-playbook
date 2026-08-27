# API & RPC Test Strategy

## Objective

Validate the correctness, reliability, and consistency of APIs and blockchain RPC infrastructure.

## API Validation

Cover:

* response schemas
* status codes
* authentication
* authorization
* input validation
* pagination
* filtering
* error handling
* data correctness

## RPC Validation

Cover:

* successful calls
* malformed responses
* unavailable provider
* timeout
* rate limiting
* stale data
* unsupported methods
* retry behavior
* provider failover

## Blockchain Reconciliation

Where an API exposes blockchain-derived state, compare application data with its authoritative source.

Example:

```text
Contract State
      ↓ compare
Indexer State
      ↓ compare
API Response
```

## Eventual Consistency

Asynchronous systems require explicit expectations.

Define:

* expected indexing delay
* retry interval
* maximum acceptable convergence time
* behavior while data is stale

Tests should distinguish between:

**temporarily inconsistent**

and

**incorrectly inconsistent**.

## Resilience

External infrastructure should be assumed to fail.

Validate application behavior when dependencies are:

* unavailable
* slow
* inconsistent
* rate limited
* returning errors

## Guiding Principle

API correctness in Web3 includes not only response validity, but **consistency with underlying blockchain state**.
