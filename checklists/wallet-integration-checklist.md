# Wallet Integration Checklist

## Connection

* [ ] Supported wallet connects successfully
* [ ] Connection rejection handled
* [ ] Disconnection handled
* [ ] Session restoration validated

## Accounts

* [ ] Active account detected
* [ ] Account switching detected
* [ ] UI state updates after account switch

## Networks

* [ ] Correct chain detected
* [ ] Unsupported chain handled
* [ ] Network switching works
* [ ] Rejected network switch handled

## Signing

* [ ] Expected message presented for signing
* [ ] Successful signature handled
* [ ] Rejected signature handled

## Transactions

* [ ] Correct contract targeted
* [ ] Correct function called
* [ ] Correct arguments supplied
* [ ] Transaction rejection handled
* [ ] Pending state represented correctly
* [ ] Revert handled correctly
* [ ] Confirmation detected
* [ ] Duplicate submission prevented

## State

* [ ] UI updates after confirmed transaction
* [ ] Account state remains synchronized
* [ ] Network state remains synchronized
