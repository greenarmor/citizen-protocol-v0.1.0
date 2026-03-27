# CITIZEN-Protocol Consensus Mechanism

[![Protocol](https://img.shields.io/badge/Protocol-CITIZEN-2d6cdf)](./citizen-protocol/README.md)
[![Status](https://img.shields.io/badge/Status-Working%20Draft-orange)](./README.md)
[![Consensus](https://img.shields.io/badge/Consensus-BFT%20%E2%89%A580%25%20quorum-6f42c1)](./README.md)
[![Finality](https://img.shields.io/badge/Finality-Deterministic%20~1s-success)](./README.md)
[![Rust](https://img.shields.io/badge/Rust-Workspace-000000?logo=rust)](https://github.com/greenarmor/citizen.git)
[![License: CPNCL](https://img.shields.io/badge/License-CPNCL%20v1.0-blue)](./LICENSE)

> **Note:** The Rust source code for citizen-protocol v0.1.0 is temporarily kept in a private repository at [https://github.com/greenarmor/citizen.git](https://github.com/greenarmor/citizen.git) until the appropriate time to migrate it into this repository.

## Governance-First Public Ledger with Deterministic Finality, Namespace Safety, and Builder Credit Grants

**Version:** 0.2 (Research-Aligned Draft)
**Status:** Working Whitepaper

---

## Abstract

CITIZEN is a governance-first decentralized public ledger for verifiable records, civic coordination, and institutional accountability. It is explicitly non-speculative: core protocol security does not depend on a transferable token. Instead, system sustainability is based on identity-anchored validator governance, deterministic Byzantine fault tolerant (BFT) finality at an 80% supermajority threshold, and non-transferable builder credit grants for ecosystem growth.

This whitepaper presents a formal system model, consensus proofs, namespace rule semantics, governance mathematics, and a credit-grant mechanism with transparent budget constraints. The result is a protocol that is mathematically analyzable, operationally auditable, and aligned with public-infrastructure use cases.

---

## 1. Introduction

### 1.1 Problem statement

Public digital systems require three properties simultaneously:

1. **Truth integrity:** records must be tamper-evident and final.
2. **Institutional accountability:** system changes must be governed, not unilateral.
3. **Developer usability:** builders need predictable incentives and deterministic APIs.

Many existing ledgers optimize for transferable asset speculation rather than public-record guarantees. CITIZEN instead optimizes for governed truth infrastructure.

### 1.2 Design objectives

Let the protocol objective vector be:

$$
\mathcal{O} = (\text{Safety},\ \text{Liveness},\ \text{Governance Auditability},\ \text{Builder Utility})
$$


CITIZEN maximizes $$\mathcal{O}$$ subject to constraints:

- deterministic finality,
- identity-anchored validator admission,
- namespace-scoped rule enforcement,
- non-speculative budget policy.

---

## 2. System Model

At block height $$h$$ the governance-approved validator allowlist is $$A_h$$ with $$|A_h| = M_h$$. The runtime consensus set is derived deterministically from that allowlist:

- Active validators: $$V_h \subseteq A_h$$ with $$|V_h| = N_h = \left\lceil \frac{M_h}{2} \right\rceil$$
- Standby validators: $$S_h = A_h \setminus V_h$$ with $$|S_h| = M_h - N_h$$
- Quorum threshold: $$Q_h = \left\lceil 0.8N_h \right\rceil$$
- Byzantine bound on the active set: $$f_h \le \lfloor 0.19N_h \rfloor$$

This matches the rollout policy target of a 200% governance allowlist relative to active consensus seats: if the policy target is $$N_h$$ active seats, governance admits approximately $$M_h = 2N_h$$ total validator seats so that roughly half are active and half are standby at any given epoch.

Each active validator $$v_i \in V_h$$ has:

- a unique identity anchor,
- a long-term public key $$pk_i$$,
- governance eligibility status.

### 2.1 Network and synchrony assumptions

CITIZEN assumes partial synchrony:

1. Global stabilization time $$GST$$ exists.
2. After $$GST$$, one-way message delay is bounded by $$\Delta$$.
3. Peer authentication prevents Sybil identity spoofing.

### 2.2 Block definition

A finalized block is:

$$
B_h = (h, H(B_{h-1}), R_h, p_h, t_h, \Pi_h)
$$

where:

- $$R_h$$: Merkle root of valid entries,
- $$p_h$$: deterministic proposer,
- $$\Pi_h$$: aggregate commit proof.

Finalization criterion:

$$
\text{Finalize}(B_h) \iff \text{VerifyAgg}(\Pi_h) \land \text{Signers}(\Pi_h) \subseteq V_h \land |\text{Signers}(\Pi_h)| \ge Q_h
$$

---

## 3. Consensus: 80%-BFT Deterministic Finality

Consensus at height \(h\) proceeds across rounds $$r = 0,1,2,\dots$$:

1. **Propose:** leader $$L_{h,r}$$ proposes candidate block $$B_h^r$$.
2. **Pre-vote:** validators verify structure, signatures, namespace rules, and state transition validity.
3. **Commit:** if pre-votes $$\ge Q_h$$, validators sign commit; aggregate proof finalizes.

If timeout occurs before quorum, round aborts and leader rotates; no state transition is applied.

### 3.1 Deterministic transition function

Let global state at height $$h$$ be $$S_h$$. Then:

$$
S_h = \Gamma(S_{h-1}, B_h)
$$

with $$\Gamma$$ deterministic. Therefore, for any two honest nodes $$i,j$$:

$$
\text{Prefix}_i(h) = \text{Prefix}_j(h) \Rightarrow S_h^{(i)} = S_h^{(j)}
$$

---

## 4. Namespace-First Ledger Semantics

Each entry $$e$$ belongs to exactly one namespace $$\nu(e)$$. For namespace $$\nu$$, let rule predicate be $$\mathcal{R}_{\nu}$$.

Block validity:

$$
\text{Valid}(B_h,S_{h-1}) \iff \forall e \in B_h:\ \text{SigValid}(e) \land \text{Exists}(\nu(e)) \land \mathcal{R}_{\nu(e)}(e,S_{h-1})
$$

If any entry fails, the full block is rejected.

### 4.1 Safety interpretation

Namespace isolation is formalized as a non-interference condition:

$$
\forall e_a \in \nu_a, e_b \in \nu_b,\ \nu_a \ne \nu_b:\ \text{WriteAuth}_{\nu_a}(e_a) \not\Rightarrow \text{WriteAuth}_{\nu_b}(e_b)
$$

Thus no cross-domain privilege escalation is implied by design.

---

## 5. Governance Mathematics

Governance actions are entries in `governance.vote`. Governance approves membership over the full allowlist $$A_h$$, while consensus and commit quorums are computed only over the active subset $$V_h$$.

For proposal $$P$$ at height $$h$$:

$$
\text{Yes}_h(P) = |\{v \in V_h : v\ \text{signs YES on}\ P\}|
$$

Execution condition:

$$
\text{GovExecute}_h(P) \iff \text{Yes}_h(P) \ge Q_h
$$

### 5.1 Validator admission

For candidate $$x$$:

$$
\mathrm{GovExecute}_h(\mathrm{ADD\_VALIDATOR}(x)) \Rightarrow A_{h+1} = A_h \cup \{x\}
$$

The next epoch active set is then recomputed deterministically from $$A_{h+1}$$:

$$
V_{h+1} \subseteq A_{h+1}, \qquad |V_{h+1}| = \left\lceil \frac{|A_{h+1}|}{2} \right\rceil
$$


### 5.2 Validator removal

For validator $$y$$:

$$
\mathrm{GovExecute}_h(\mathrm{REMOVE\_VALIDATOR}(y)) \Rightarrow A_{h+1} = A_h \setminus \{y\}
$$

### 5.3 Emergency governance

Let $$E$$ be an emergency action set (pause, temporary quorum adjustment, suspension, forced upgrade). Then:

$$
\forall e \in E:\ \text{GovExecute}_h(e) \Rightarrow \text{Yes}_h(e) \ge Q_h \land \text{TimeBounded}(e)
$$

This preserves auditability and prevents unilateral control.

---

## 6. Builder Credit Grant Model (Non-Transferable)

CITIZEN introduces **Builder Credits** as governance-issued, non-transferable service credits for protocol usage (e.g., namespace writes, API quotas, deterministic receipt operations). Credits are not monetary assets.

### 6.1 Credit state

For builder $$b$$:

- balance: $$C_b(h) \in \mathbb{R}_{\ge 0}$$
- cumulative grants: $$G_b(h)$$
- cumulative spend: $$U_b(h)$$

State equation:

$$
C_b(h+1) = C_b(h) + g_b(h) - u_b(h),\quad C_b(h+1) \ge 0
$$

where:

- $$g_b(h)$$: governance-approved grant emitted at height $$h$$,
- $$u_b(h)$$: protocol-measured usage burn at height $$h$$.

### 6.2 Grant approval rule

Grant proposal $$P_b = (b, A_b, T_b, M_b)$$ includes amount $$A_b$$, milestone schedule $$T_b$$, and measurable deliverables $$M_b$$.

Grant activation:

$$
\text{GrantActive}(P_b) \iff \text{Execute}(P_b) \land \text{Compliance}(M_b,T_b)
$$

Milestone tranche $$k$$ release:

$$
g_{b,k} = A_b \cdot w_k \cdot \mathbf{1}[\text{Milestone}_k\ \text{verified}],\quad \sum_k w_k = 1
$$

### 6.3 Budget safety constraints

Let epoch grant budget be $$B_e$$, and total allocated in epoch $$e$$:

$$
\mathcal{G}_e = \sum_b \sum_{h \in e} g_b(h)
$$

Safety bound:

$$
\mathcal{G}_e \le B_e
$$

Per-builder concentration cap $$\alpha\in(0,1)$$:

$$
\sum_{h\in e} g_b(h) \le \alpha B_e
$$

This prevents capture and enforces portfolio diversification.

### 6.4 Usage pricing and burn

Let operation classes be $$\Omega$$, each with governance-set weight $$w_\omega$$. If builder $$b$$ performs $$n_{b,\omega}(h)$$ operations at height $$h$$, burn is:

$$
u_b(h)=\sum_{\omega \in \Omega} w_\omega\, n_{b,\omega}(h)
$$

Hence high-impact use consumes proportionally more credits while preserving deterministic accounting.

---

## 7. Formal Safety and Liveness Claims

### 7.1 Safety theorem (no conflicting finalization)

If at least 81% of active validators are honest and non-equivocating, two conflicting blocks at the same height cannot both finalize.

**Proof sketch.** Let $$N = |V_h|$$ be the active validator count for height $$h$$, with $$V_h$$ derived from the full allowlist $$A_h$$ by the deterministic split $$N = \left\lceil M/2 \right\rceil$$ where $$M = |A_h|$$. Suppose conflicting finalized blocks $$B$$ and $$B'$$ have commit signer sets $$C,C' \subseteq V_h$$ with $$|C|,|C'| \ge 0.8N$$. Then:

$$
|C\cap C'| \ge |C|+|C'|-N \ge 0.6N
$$

At least 60% of the active set would have signed both blocks. Since the Byzantine share on the active set is $$\le 19\%$$, at least $$0.6N - 0.19N = 0.41N$$ honest active validators would have equivocated, contradiction. Standby validators do not change this bound because they are excluded from $$V_h$$ and cannot contribute commit signatures until promoted into the next active set.

### 7.2 Liveness theorem

Under partial synchrony, if at least $$Q_h$$ honest validators are online, eventually a correct proposer is scheduled and messages arrive before timeout; therefore one valid block finalizes.

### 7.3 Safe-stall lemma

If online participation $$<Q_h$$, no valid commit proof can be formed, so chain progress halts without violating safety.

---

## 8. Performance Model

For $$N$$ validators under full-gossip voting:

- message complexity per round: $$\Theta(N^2)$$,
- signature verification: $$\Theta(N)$$ per validator for vote checks,
- one aggregate commit verification per finalized block.

Approximate latency model:

$$
T_{final} \approx T_{prop} + T_{pv} + T_{cm} + T_{agg}
$$

Operational target is $$T_{final}\approx1$$ second under healthy 50–100 validator conditions.

---

## 9. Security and Auditability

Security properties:

1. **Deterministic finality:** no probabilistic confirmations.
2. **Sybil resistance:** identity-anchored validator admission with 80% governance gate.
3. **Governance transparency:** all votes signed, public, and on-chain.
4. **Data ownership:** commitments and consent proofs on-chain; sensitive data off-chain and user-controlled.

### 9.1 Off-chain encrypted data commitment model

For a governed off-chain ciphertext object $$C$$ and its canonical manifest metadata $$M$$:

$$
h_C = \mathrm{SHA256}(C)
$$

$$
\pi_M = \mathrm{CanonCommit}(M)
$$

The ledger entry binds off-chain storage to on-chain verification context through
the deterministic commitment tuple:

$$
\Phi = (\pi_M,\ h_C,\ r,\ \tau)
$$

where $$r$$ is a valid consent receipt reference and $$\tau$$ is namespace policy
version metadata. Acceptance condition for off-chain encrypted writes is:

$$
\mathrm{AcceptOffchain}(e) \iff
\mathrm{PolicyOK}_{\nu(e)}(e,\tau) \land
\mathrm{ConsentValid}(r,\pi_M,t_h) \land
\mathrm{HashMatch}(h_C, C) \land
\mathrm{CommitMatch}(\pi_M, M)
$$

This formalizes the implementation path where encrypted bytes remain off-chain
while deterministic commitments and consent-governance checks are enforced by
the API + ledger validation boundary.


Auditability function over history $$\mathcal{H}$$:

$$
\text{Audit}(\mathcal{H}) = \text{VerifySignatures}(\mathcal{H}) \land \text{VerifyQuorums}(\mathcal{H}) \land \text{VerifyStateReplay}(\mathcal{H})
$$

---

CITIZEN defines a mathematically grounded, governance-first public ledger that prioritizes deterministic truth over speculation. Its 80%-BFT consensus provides strong finality, namespaces provide policy-safe domain composition, and builder credit grants create a transparent non-transferable incentive path for ecosystem growth. Together, these elements form a deployable framework for civic and institutional digital infrastructure.
