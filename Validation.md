# Agent Validation Guide

**Navigation**: [Registration](Registration.md) | [Reputation](Reputation.md) | **Validation**

ERC-8004 provides a **Validation Registry** that acts as a coordination layer between agents seeking independent verification and the validators who provide it. It is *plumbing, not policy* — the Registry defines how requests and responses are routed and stored, but the incentives, slashing, and trust model are owned by each **validation protocol**.

A validation protocol is a specific set of rules for what a validator checks and how it signs its verdict. This guide covers what the Validation Registry does, who uses it, and how to build a validation protocol — including operator-independence verification as the first documented protocol.

---

## What the Validation Registry does

The Validation Registry is a contract that:

- **Accepts validation requests** from agents (validation requests)
- **Routes them** to registered validators (validators)
- **Stores signed verdicts** on-chain (verdicts)
- **Makes verdicts discoverable** by anyone inspecting a given agent

It does **not** decide which validators to trust, how much to pay them, what happens after a negative verdict, or how disputes are resolved. Those are protocol-level concerns.

---

## The three roles

### Agents (requesters)

Any registered ERC-8004 agent can submit a validation request. Typical reasons:

- **Before deployment**: prove your agent passes a security or policy check
- **During operation**: collect independent verdicts for reputation or compliance
- **On demand**: a counterparty requires a fresh validation before a high-value interaction

### Validators (verifiers)

A validator is an off-chain service or on-chain contract that inspects an agent and returns a signed verdict. Validators register their identity in their ERC-8004 agent registration.

The spec envisions three archetypes (ERC-8004 §6.2):

| Archetype | How it works | Example |
|---|---|---|
| **Judgment** | A human or AI team reviews the agent's code, config, and deployment — then issues a verdict | **Invinoveritas** — neutral pre-action verdicts, signed recomputable proofs, public track record |
| **On-chain computation** | A smart contract re-executes part of the agent's logic and compares the result to a commitment | **Assay Protocol** — on-chain stake-backed accountability, escrow dashboard, 59+ agents indexed on Base |
| **Stake-backed** | An entity posts a bond that is slashed if a false verdict is proven | (slot open for a new protocol) |

A validator MUST have a registered ERC-8004 agent identity so that verdicts can be traced back to a known `clientAddress`.

### The Registry (router)

The Validation Registry contract:

1. Accepts a `validationRequest` containing the agent's manifest, requested checks, and preferred validators
2. Forwards the request to the selected validators (off-chain relay or on-chain call)
3. Accepts signed `ValidationResponse` verdicts and stores them by agent address
4. Exposes a read interface for anyone to query an agent's validation history

---

## Building a validation protocol

A validation protocol is defined by:

1. **What the validator checks** — the scope of inspection
2. **How the agent proves compliance** — what evidence the agent supplies
3. **The verdict format** — the signed response schema
4. **The trust model** — how false verdicts are detected and penalized
5. **Compensation** — how validators are paid (x402, subscription, off-chain)

### Step 1: Choose your check

Example: **operator-independence verification** checks whether the wallet addresses or signing keys listed in an agent's quorum are controlled by distinct entities — not the same entity under multiple aliases.

Why it matters: quorum-based authorization (n-of-m signing) provides zero security if every key is held by the same operator. A validation protocol that verifies operator independence makes the quorum meaningful.

Other candidate checks: endpoint reachability, code hash integrity, declared-vs-actual service availability, compliance with a disclosed policy (budget limits, allowed domains, data retention).

### Step 2: Define the request format

A validation request should include everything the validator needs. Example for operator-independence:

```json
{
  "protocol": "operator-independence-v1",
  "agent": "0x1234...abcd",
  "manifest": {
    "endpoints": [
      {
        "url": "https://signer-1.example.com/rpc",
        "operator_id": "org-alpha",
        "chain_id": 8453
      },
      {
        "url": "https://signer-2.example.com/rpc",
        "operator_id": "org-alpha",
        "chain_id": 8453
      },
      {
        "url": "https://signer-3.example.com/rpc",
        "operator_id": "org-beta",
        "chain_id": 8453
      }
    ]
  },
  "context": {
    "quorum": {
      "min_endpoints": 2,
      "strict_operators": true
    }
  },
  "preferred_validators": [
    "0x8004A169...a432"
  ]
}
```

### Step 3: Define the verdict format

The response should include the check result and enough evidence for independent recomputation:

```json
{
  "protocol": "operator-independence-v1",
  "validator": "0x8004A169...a432",
  "agent": "0x1234...abcd",
  "timestamp": 1765036800,
  "result": "fail",
  "scores": {
    "operator_independence": 33
  },
  "details": {
    "endpoints_queried": 3,
    "endpoints_qualifying": 3,
    "operators_distinct": 2,
    "entries": [
      {
        "url": "https://signer-1.example.com/rpc",
        "operator_id": "org-alpha",
        "status": "qualifying"
      },
      {
        "url": "https://signer-2.example.com/rpc",
        "operator_id": "org-alpha",
        "status": "qualifying"
      },
      {
        "url": "https://signer-3.example.com/rpc",
        "operator_id": "org-beta",
        "status": "qualifying"
      }
    ],
    "reasons": [
      "Operator org-alpha controls 2 of 3 endpoints (67%). With strict_operators=true and min_endpoints=2, at least 2 distinct operators are required."
    ]
  },
  "signature": "0x..."
}
```

A score of 100 means full operator independence; 0 means all endpoints share one operator. A protocol can define its own pass/fail threshold (e.g., `score >= 50` for 2-of-3 with 2 distinct operators).

### Step 4: Choose a trust model

| Model | How it works | Pros | Cons |
|---|---|---|---|
| **Reputation** | Validator's track record is public (ERC-8004 feedback) | Low friction to start | No immediate penalty for false verdicts |
| **Stake / bond** | Validator posts collateral that anyone can challenge | Strong economic guarantees | Requires capital; needs a challenger mechanism |
| **Audit trail** | Verdicts include recomputable evidence; third parties can verify | Transparent, composable | Doesn't prevent a bad verdict, only detects it |

Hybrids are common: a reputation-based validator can add stake as it grows.

### Step 5: Register and deploy

1. **Register as an ERC-8004 agent** — your validator needs an identity
2. **Publish your validation protocol** — document what you check and how
3. **Deploy or integrate with a Validation Registry** — on supported chains
4. **Accept requests** — relay `validationRequest` messages and return signed `ValidationResponse`

---

## Tips for validators

- **Stick to one protocol per registration.** A single validator that checks both operator independence and code integrity under one identity makes it harder for agents to selectively trust one check. Use separate agent identities for separate protocols.
- **Include recomputable evidence in every verdict.** A verdict like "included" with no supporting data is a claim, not a proof. Publish enough detail (operator IDs, endpoint statuses, exclusion reasons) that any third party can re-run the check from the evidence.
- **Publish your own track record.** ERC-8004 feedback lets agents rate validators. Use `tag1 = verdictAccuracy` or similar to build a public record.
- **Set expectations clearly.** If your validator needs 30 minutes to run a full check, say so in your registration description.

## Tips for agents

- **Know your threat model before choosing a validator.** If you need quorum security, operator-independence verification matters more than endpoint reachability. Pick validators whose protocol matches your risk.
- **Use multiple validators.** Diversifying across judgment and on-chain computation validators reduces the blast radius of a single false verdict.
- **Respond to failed validations.** A failed operator-independence check doesn't mean your agent is broken — it means your signing infrastructure is less independent than you claimed. Fix the config (split operators, use distinct wallets per operator) and re-validate.
- **Reference validation results in your agent registration.** Agents that publish their latest validation verdict alongside their service endpoints build trust with prospective users.

---

## Deployment status

The ERC-8004 Identity and Reputation Registries are deployed on **13+ chains** including Ethereum mainnet, Base mainnet, Arbitrum, Avalanche, BSC, Celo, Gnosis, Linea, Mantle, MegaETH, Metis, and more, using deterministic CREATE2 addresses for cross-chain singleton deployments.

The **Validation Registry slot is open on many chains** — including Base mainnet — with no deployed contract yet. The Sepolia VR address exists but has no code deployed on Base.

| Chain | Identity Registry | Reputation Registry | Validation Registry |
|---|---|---|---|
| Ethereum mainnet | ✅ | ✅ | ⬜ |
| Base mainnet | ✅ | ✅ | ⬜ (slot open) |
| Arbitrum | ✅ | ✅ | ⬜ |
| (others) | ✅ | ✅ | ⬜ |

Verification protocols like operator-independence are designed to plug into these open Validation Registry slots, giving agents on every supported chain access to independent verification.

---

## Open questions

The ERC-8004 ecosystem is still forming. Some open questions the community needs to settle:

1. **Where should canonical validation protocol specs live?** In the EIP itself, in this best-practices repo, or in protocol-specific repos?
2. **Who else is building validation protocols?** Several teams (Invinoveritas, Assay Protocol, dinamic.eth) are active, but no formal coordination exists yet.
3. **What compensation model works for validators?** x402 micropayments per verdict? Subscription? Free with reputation upside?
4. **Should the Validation Registry support protocol versioning at the contract level?** Today, protocol negotiation is off-chain; a future Registry version could support it natively.

These questions are open for ecosystem debate. Proposals and comments welcome on the [issue tracker](https://github.com/erc-8004/best-practices/issues).