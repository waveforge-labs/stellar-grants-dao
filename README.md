# stellar-grants-dao

> A fully on-chain DAO governance and grants system for Stellar Soroban —
> proposal creation, token-weighted voting, trustless treasury disbursement,
> and a real-time governance dashboard.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Built with Soroban](https://img.shields.io/badge/Soroban-Rust-orange)](https://soroban.stellar.org)
[![Drips Wave](https://img.shields.io/badge/Drips-Wave%20Program-blue)](https://drips.network)

---

## Overview

Communities building on Stellar need transparent, trustless governance for
managing grants, protocol upgrades, and treasury allocations.
`stellar-grants-dao` delivers a complete, auditable governance stack:

- 🗳️ **Token-weighted voting** — vote power proportional to governance
  token balance; supports delegation so passive holders still participate
- 📋 **Proposal lifecycle** — Draft → Active → Succeeded/Defeated →
  Queued (timelock) → Executed or Expired
- 🔐 **Trustless treasury** — funds held in `TreasuryContract`; only
  on-chain-approved proposals can trigger disbursements
- ⏱️ **Timelock protection** — approved proposals queue for a configurable
  delay before execution, giving the community time to react
- 📊 **Governance dashboard** — real-time vote counts, quorum progress,
  treasury balance, and delegate leaderboard

---

## Technical Architecture

### Contract Architecture
```
VoteToken (SEP-41 + delegation)
├── delegate(delegatee)
├── voting_power(account) → u128
└── checkpoint_votes(account, ledger) → u128

GovernanceContract
├── create_proposal(title, description, actions, voting_period)
├── cast_vote(proposal_id, support: For|Against|Abstain)
├── queue(proposal_id)        ← after vote succeeds + quorum met
├── execute(proposal_id)      ← after timelock elapses
├── cancel(proposal_id)
└── proposal_state(id) → ProposalState

TreasuryContract
├── disburse(recipient, amount, asset) ← governance-only
├── balance(asset) → i128
└── pending_disbursements() → Vec<Disbursement>
```

### Proposal Lifecycle
```
DRAFT → ACTIVE (voting open)
           ├── quorum met + majority For → SUCCEEDED
           │         └── queue() → QUEUED (timelock)
           │                   └── execute() → EXECUTED ✅
           ├── quorum not met → DEFEATED ❌
           └── majority Against → DEFEATED ❌
```

### Governance Parameters (configurable per deployment)

| Parameter | Default |
|---|---|
| Voting Period | 7 days (in ledgers) |
| Quorum | 4% of total supply |
| Vote Threshold | Simple majority (>50%) |
| Timelock Delay | 2 days |
| Proposal Threshold | 1% of supply to submit |

---

## 🌊 Drips Wave Program

This repository is an **active participant in the
[Drips Wave Program](https://drips.network)** — the final and most
impactful repo in the `stellarforge` ecosystem. Contributors earn
USDC/DAI for clearly scoped GitHub issues.

### How to Participate

1. **Browse Open Issues**
   Visit the [Issues tab](../../issues) and filter by label:
   - 🟢 `drips:trivial` — Fix quorum meter animation, add proposal
     status badges, improve form validation copy (~1–2 hrs)
   - 🟡 `drips:medium` — Implement vote delegation UI, add indexer
     handler for `VoteCast` events, write Rust tests for quorum
     logic, build `<ProposalTimeline />` component (~4–8 hrs)
   - 🔴 `drips:high` — Timelock contract module, on-chain treasury
     multi-asset support, governance simulation CLI tool, full
     e2e governance flow test suite (~1–3 days)

2. **Register on Drips**
   - Visit [drips.network](https://drips.network)
   - Connect your Ethereum-compatible wallet
   - Search for **`stellar-grants-dao`** under "Explore"

3. **Claim an Issue**
   - Comment: `"Claiming via Drips Wave — ETA [X days]"`
   - Maintainer assigns you and locks the bounty on Drips

4. **Submit Your PR**
   - All contract changes must pass `cargo test` and include a
     test case for the new behavior
   - All frontend changes must be TypeScript error-free with a
     screenshot or Loom walkthrough in the PR description
   - Indexer changes must include a migration and handler unit test

5. **Earn Rewards**
   Bounty amounts (USDC/DAI) listed per issue. Auto-released
   to your wallet on PR merge — no admin friction.

---

## Project Structure
```
stellar-grants-dao/
├── contracts/
│   ├── governance/           # Proposal + voting lifecycle contract
│   ├── treasury/             # Trustless fund disbursement contract
│   ├── token/                # SEP-41 governance token + delegation
│   └── tests/                # Rust unit + integration tests
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── proposals/    # Proposal list + create flow
│   │   │   ├── vote/         # Active proposal voting UI
│   │   │   ├── treasury/     # Treasury balance + disbursement history
│   │   │   └── profile/      # Voting history, delegations, power
│   │   ├── components/
│   │   │   ├── ProposalForm/    # Multi-step proposal creation wizard
│   │   │   ├── VotePanel/       # For/Against/Abstain voting UI
│   │   │   ├── TreasuryStats/   # Balance + disbursement chart
│   │   │   ├── ProposalTimeline/# Lifecycle stage progress indicator
│   │   │   ├── DelegateCard/    # Delegate profile + voting power
│   │   │   └── QuorumMeter/     # Live quorum progress bar
│   │   ├── hooks/            # useProposal, useVote, useTreasury
│   │   └── lib/              # Contract clients, vote helpers
│   └── public/
├── indexer/
│   ├── src/
│   │   ├── handlers/         # ProposalCreated, VoteCast, Executed
│   │   ├── models/           # Proposal, Vote, Delegate DB models
│   │   └── utils/            # Event parsing, power snapshots
│   └── migrations/
├── tests/
│   ├── unit/                 # Component + hook + handler tests
│   ├── integration/          # Testnet governance flow tests
│   └── e2e/                  # Full proposal → execution tests
├── scripts/
│   ├── deploy/               # Contract deploy scripts
│   └── simulate/             # Governance scenario dry-runs
├── docs/                     # Governance spec, parameter guide
├── docker-compose.yml
└── .env.example
```

---

## Quick Start
```bash
# 1. Deploy governance stack to Testnet
cd contracts
cargo build --target wasm32-unknown-unknown --release
cd ../scripts/deploy && npx ts-node deploy.ts --network testnet

# 2. Simulate a governance vote locally
cd ../simulate && npx ts-node simulate-vote.ts

# 3. Start indexer
cd ../../indexer && npm install && npm run start

# 4. Start governance dashboard
cd ../frontend && npm install && npm run dev
# Visit http://localhost:3000/proposals
```

---

## License

MIT © stellarforge contributors
