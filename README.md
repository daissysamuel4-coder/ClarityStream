
---

## ClarityStream

### 💡 Overview

This smart contract implements **streaming payments** on the Stacks blockchain. It allows a sender to create a **payment stream** that distributes STX tokens to a recipient over a defined time frame. Recipients can **withdraw earned funds** incrementally, while senders can **refuel** the stream or **refund** any remaining balance once the stream ends.

---

### 🚀 Features

* 📤 **Create a Stream:** Start a new payment stream specifying recipient, duration, balance, and payment rate.
* 🔄 **Refuel Stream:** Add more tokens to an active stream.
* 📥 **Withdraw Funds:** Recipients can withdraw their earned portion at any time.
* 💸 **Refund Excess:** Senders can reclaim unearned funds after stream expiration.
* 🔐 **Signature-Based Updates:** Update stream terms (timeframe, payment rate) securely with cryptographic signatures.
* 🧮 **Real-Time Balances:** Get live token balances for both sender and recipient.

---

### 🧱 Data Structures

#### `streams` (map)

| Field               | Type      | Description                    |
| ------------------- | --------- | ------------------------------ |
| `sender`            | principal | Stream creator                 |
| `recipient`         | principal | Stream receiver                |
| `balance`           | uint      | Total locked STX               |
| `withdrawn-balance` | uint      | STX withdrawn by recipient     |
| `payment-per-block` | uint      | Rate of STX per block          |
| `timeframe`         | tuple     | `start-block` and `stop-block` |

#### `latest-stream-id` (data-var)

* Stores the latest used stream ID, incremented with each stream creation.

---

### 📚 Public Functions

#### 1. `stream-to(recipient, initial-balance, timeframe, payment-per-block)`

Creates a new stream.

* Transfers `initial-balance` to the contract.
* Returns the stream ID.

#### 2. `refuel(stream-id, amount)`

Adds more STX to an existing stream.

* Only the stream sender can refuel.

#### 3. `withdraw(stream-id)`

Allows the recipient to withdraw their earned tokens.

* Calculates the balance based on time and payment rate.
* Transfers STX to the recipient.

#### 4. `refund(stream-id)`

Lets the sender reclaim unearned tokens **after** the stream ends.

* Fails if the stream is still active.

#### 5. `update-details(stream-id, payment-per-block, timeframe, signer, signature)`

Updates payment rate and/or timeframe via off-chain signature approval.

* Only allowed if both parties agree via a valid signature.

---

### 📖 Read-Only Functions

#### `balance-of(stream-id, who)`

Returns the balance available for a `who` (either sender or recipient).

#### `calculate-block-delta(timeframe)`

Computes how many blocks have passed since the stream started.

#### `hash-stream(stream-id, new-payment-per-block, new-timeframe)`

Returns a hash of updated stream details to be signed off-chain.

#### `validate-signature(hash, signature, signer)`

Verifies that the given signature matches the hash and signer.

---

### ⚠️ Error Codes

| Code | Meaning             |
| ---- | ------------------- |
| `u0` | Unauthorized        |
| `u1` | Invalid Signature   |
| `u2` | Stream Still Active |
| `u3` | Invalid Stream ID   |

---

### 🛠 Deployment Notes

* Ensure the contract has permission to hold and transfer STX.
* All STX transfers occur using `stx-transfer?` wrapped in `try!` and `as-contract`.

---

### 🔐 Security Notes

* Only stream senders can refuel or refund.
* Only recipients can withdraw earned balances.
* Updates require valid signatures from either party, ensuring mutual consent.

---

### 📦 Example Usage

```clojure
;; Create a stream
(stream-to 'SP...RECIPIENT u10000 {start-block: u200, stop-block: u300} u10)

;; Refuel the stream
(refuel u0 u5000)

;; Recipient withdraws available funds
(withdraw u0)

;; Refund after stream ends
(refund u0)
```

---
