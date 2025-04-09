# BitGovern DAO - On-Chain Governance Protocol Documentation

## Overview

BitGovern DAO is a sophisticated governance protocol designed for Bitcoin-aligned decentralized organizations operating on Stacks Layer 2. This implementation enables secure, transparent decision-making and treasury management through a combination of Bitcoin-secured smart contracts and STX-based economic incentives.

### Key Features

- **Bitcoin-Aligned Governance**: Leverages Stacks L2 for Bitcoin finality and proof-of-transfer consensus
- **Enterprise-Grade Treasury Management**: Time-locked deposits with minimum 1M microSTX (1 STX) requirement
- **Dynamic Voting System**: Configurable proposal durations (1-14 days) with token-weighted voting
- **Anti-Sybil Mechanics**: 1:1 STX-backed governance token minting system
- **Compliance-Ready Architecture**: Principal-based accounting with microSTX denomination tracking
- **Execution Enforcement**: Automated proposal execution upon successful quorum approval

## Technical Specifications

| Category                | Detail                        |
| ----------------------- | ----------------------------- |
| Blockchain              | Stacks Layer 2                |
| Smart Contract Language | Clarity                       |
| Token Standard          | Native STX-backed governance  |
| Minimum Deposit         | 1,000,000 microSTX (1 STX)    |
| Base Lock Period        | 1,440 blocks (~10 days)       |
| Voting Range            | 144-20,160 blocks (1-14 days) |
| Proposal Storage        | On-chain permanent record     |

## Core Functionality

### 1. Deposit & Token System

#### `deposit (amount: uint)`

- Converts STX to governance tokens at 1:1 ratio
- Requirements:
  - Minimum 1 STX deposit
  - Funds locked for 1,440 blocks (~10 days)
- Effects:
  - Mints governance tokens to caller's balance
  - Records time-locked deposit position

### 2. Proposal Lifecycle

#### `create-proposal (description, amount, target, duration)`

- Requirements:
  - Minimum 1 governance token balance
  - Valid duration (144-20,160 blocks)
  - Non-zero STX amount
  - External target address
- Creates proposal with:
  - Human-readable description (256 char max)
  - Requested STX amount
  - Beneficiary principal
  - Custom voting period

### 3. Voting Mechanism

#### `vote (proposal-id: uint, vote-for: bool)`

- Token-weighted voting system:
  - 1 microSTX deposited = 1 voting power
- Requirements:
  - Active proposal status
  - No previous vote cast
  - Valid governance token balance
- Records vote immutably on-chain

### 4. Proposal Execution

#### `execute-proposal (proposal-id: uint)`

- Automatic execution conditions:
  - Voting period concluded
  - Majority approval (yes > no)
  - Sufficient treasury balance
  - Single-use execution
- Transfers funds to designated target

### 5. Withdrawal System

#### `withdraw (amount: uint)`

- Requirements:
  - Completed lock period
  - Sufficient token balance
- Process:
  - Burns governance tokens
  - Releases proportional STX
  - Atomic STX transfer

## Governance Parameters

```clarity
;; Time Parameters (10-minute blocks)
(define-constant minimum-duration u144)    ;; 1 day
(define-constant maximum-duration u20160)  ;; 14 days

;; Economic Parameters
(define-data-var minimum-deposit u1000000) ;; 1 STX
(define-data-var lock-period u1440)        ;; 10 days
```

## Error Reference

| Error Code | Description                        |
| ---------- | ---------------------------------- |
| u100       | Owner-only function call           |
| u101       | Contract not initialized           |
| u102       | Already initialized                |
| u103       | Insufficient balance               |
| u104       | Invalid amount input               |
| ...        | ... (Full table in technical docs) |

## Security Model

1. **Bitcoin Finality**: All operations inherit Bitcoin's security through Stacks L2 anchors
2. **Time-Lock Economics**: 10-day withdrawal lock prevents governance attacks
3. **STX-Backed Tokens**: 1:1 asset collateralization prevents synthetic inflation
4. **Execution Guards**:
   - Proposal timeouts enforced at block height
   - Re-entrancy protection through state locks
   - Balance checks before fund transfers

## Compliance Features

- **Principal-Based Accounting**: All transactions mapped to Stacks wallet addresses
- **microSTX Tracking**: Native support for financial reporting requirements
- **Transparent History**: Immutable record of all governance actions
- **KYC Readiness**: Address-based system compatible with identity attestations

## Usage Examples

### Creating a Proposal

```clarity
(contract-call? .bitgovern-dao create-proposal
    "Fund developer grant program"
    u5000000
    'SP3ABC...
    u10080)  ;; 7 day voting
```

### Voting on Proposal #42

```clarity
(contract-call? .bitgovern-dao vote u42 true)
```

### Checking Governance Power

```clarity
(contract-call? .bitgovern-dao get-balance 'SP3XYZ...)
```

## Audit Considerations

1. **STX Escrow Verification**: Confirm contract balance matches total deposits
2. **Time-Lock Enforcement**: Validate block height comparisons
3. **Vote Isolation**: Ensure per-address single voting enforcement
4. **Proposal State Machine**: Verify executed proposals cannot be modified
