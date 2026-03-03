# OpenMontage Governance Framework

**Version 1.0 — How Decisions Get Made**

---

## 1. Governance Philosophy

OpenMontage governance is built on four foundational principles:

### Meritocracy

Influence in the OpenMontage ecosystem is earned through contributions, not purchased through connections. A first-time creator whose segment wins a community vote has the same economic rights as a veteran with dozens of merged clips. Quality is the only currency that matters for film-level decisions. Platform-level governance introduces token-weighted voting, but caps and reputation multipliers ensure that deep engagement always outweighs raw capital.

### Transparency

Every vote is recorded on-chain. Every revenue distribution is auditable. Every governance proposal, discussion, and outcome is public. There are no backroom deals — the blockchain is the single source of truth for all platform decisions. Monthly treasury reports, quarterly audits, and real-time dashboards ensure that every participant can independently verify that the system operates as documented.

### Progressive Decentralization

OpenMontage does not launch as a fully decentralized DAO on day one. That would be irresponsible — early-stage protocols need the ability to respond quickly to bugs, exploits, and market feedback. Instead, governance decentralizes in deliberate phases: from a founding team multi-sig, to token-weighted advisory votes, to binding on-chain governance, and ultimately to a constitution-based DAO with dispute courts. Each phase hands off more authority to the community as the protocol matures and the tooling can support it.

### Separation of Concerns

Film-level governance (which segment gets merged into Shot 042) and platform-level governance (should we raise the platform fee from 3% to 4%) are fundamentally different decisions requiring different participants, different quorums, and different timelines. OpenMontage enforces this separation structurally — a $MONTAGE whale cannot override a film community's segment vote, and individual film contributors cannot unilaterally change platform economics.

---

## 2. Governance Layers

### 2.1 Platform Governance (Meta-Level)

Platform governance controls the parameters and evolution of the OpenMontage protocol itself. $MONTAGE token holders vote on:

- **Platform fee changes** (currently 2-5% of film revenue)
- **New feature prioritization** (roadmap direction, tooling investment)
- **Treasury allocation** (grants, partnerships, ecosystem development)
- **Smart contract upgrades** (proxy upgrades, migration proposals)

#### Proposal Process

All platform governance proposals follow a structured pipeline:

1. **Discussion Phase** (minimum 7 days): Author posts proposal in Discord `#governance-proposals` channel. Community discusses, asks questions, and suggests amendments. Proposals must include: motivation, specification, risk analysis, and implementation plan.

2. **Signal Vote** (48-72 hours): Off-chain vote on Snapshot.org to gauge community sentiment. This is non-binding but serves as a filter — proposals that fail the signal vote are sent back for revision rather than proceeding to on-chain execution.

3. **On-Chain Execution**: If the signal vote passes, the proposal is submitted to the Governor contract (OpenZeppelin Governor with timelock). Token holders vote on-chain with their staked $MONTAGE balance.

4. **Quorum and Threshold**: 4% of circulating $MONTAGE supply must participate (quorum), and 50%+1 of votes cast must be in favor (simple majority). Contract upgrades and treasury allocations above $100,000 require a supermajority of 67%.

5. **Timelock**: Approved proposals enter a 48-hour timelock before execution, allowing the community to review and flag any last-minute concerns.

#### Emergency Actions (Phase 1)

Before full DAO governance is active, emergency actions (contract pause, critical bug fixes, security responses) are controlled by a **3-of-5 Gnosis Safe multi-sig** composed of founding team members. Emergency actions are retroactively ratified by the community within 7 days.

---

### 2.2 Film-Level Governance

Film-level governance determines which segments get merged into a film's Main Branch. This is the creative heart of OpenMontage.

#### Shot Voting Mechanics

When contributors submit video segments for a Shot, the community decides which version becomes the canonical one:

- **Voting opens**: Automatically triggered when a Shot reaches a minimum of **3 submissions**
- **Voting period**: **72 hours** from opening
- **Minimum votes required**: Configurable by the film's director (default: **10 votes**)
- **Voting weight**: Combined score of $MONTAGE stake and curator reputation score
- **Result**: Highest weighted-vote-total submission is merged into Main Branch

#### Insufficient Submissions

If a Shot does not reach the 3-submission threshold within **30 days** (configurable by the director), the following fallback rules apply:

- **2 submissions**: The director may trigger an early vote on the available submissions, using the standard 72-hour voting period
- **1 submission**: The director may auto-merge the single submission after the timeout period, bypassing the voting process
- **0 submissions**: The shot remains open; the director may extend the deadline, adjust the shot's requirements, or break it into smaller shots to attract contributors

Directors can also extend the submission deadline at any time before the timeout to allow more time for contributors.

Voters evaluate submissions on three criteria:
1. Standard Library adherence (visual/audio consistency)
2. Technical quality (lighting, motion, composition, resolution)
3. Creative interpretation within the film's artistic guidelines

#### Replacement Voting

Once a segment is merged, it can still be replaced if the community believes a superior version exists. Replacement voting has a deliberately higher bar to protect existing creators from frivolous challenges:

- **Trigger**: A new submission for an already-merged Shot, plus a **20% improvement signal** (at least 20% of eligible voters must signal they want to re-evaluate)
- **Approval threshold**: **60% supermajority** (vs. 50%+1 for initial merge)
- **Challenge window**: The original creator has **7 days** to respond — they may submit an improved version of their own, which is included in the replacement vote
- **Cooldown**: After a successful replacement, the same Shot cannot be challenged again for 14 days

#### Dispute Resolution

If a participant believes a vote outcome was unfair (due to manipulation, technical glitch, or rule violation):

1. **Appeal Window**: 48 hours after the vote result is finalized
2. **Appeal Stake**: Appellant must stake **10 $MONTAGE** (slashed if the appeal fails, returned if it succeeds)
3. **Jury Selection**: 7 jurors randomly selected from a pool of high-reputation curators (reputation score >= 75th percentile, no participation in the original vote)
4. **Jury Deliberation**: 72 hours to review submissions, vote records, and appellant's argument
5. **Jury Verdict**: Simple majority (4/7) to overturn; if overturned, the vote is re-run with the disputed submission(s) flagged
6. **Final Escalation**: If either party contests the jury verdict, the dispute escalates to a platform-level DAO vote (binding, but rare)

---

### 2.3 Standard Library Governance

The Standard Library is the artistic backbone of each film. Changes to it affect every contributor, so updates are governed carefully:

#### Minor Updates (Director Authority)

The film's director can unilaterally push **minor version** changes (PATCH and MINOR under SemVer):
- Bug fixes in LoRA models or prompt templates
- Adding new assets (e.g., a new supporting character model)
- Style guide clarifications

These updates increment the version (e.g., 1.2.0 -> 1.2.1 or 1.3.0) and do not invalidate existing work-in-progress submissions.

#### Major Updates (Community Approval Required)

**Breaking changes** that would invalidate existing submissions require community consensus:
- Re-training core character LoRAs with different base weights
- Fundamental art style changes (e.g., switching from photorealistic to stylized)
- Removing or replacing ControlNet rigs

Process for major updates:
1. Director publishes a **migration proposal** detailing what changes and why
2. **70% approval** from active contributors (anyone with a submission in the last 30 days)
3. **2-week migration period**: Existing work-in-progress submissions can be completed and voted on under the old version; new submissions must use the new version after the migration period ends
4. Version increments to next MAJOR (e.g., 1.x.x -> 2.0.0)

#### Versioning

Standard Libraries follow **Semantic Versioning (SemVer)**:
- **MAJOR**: Breaking changes requiring re-generation of existing work
- **MINOR**: New assets or non-breaking enhancements
- **PATCH**: Bug fixes and documentation corrections

---

## 3. Roles and Voting Power

Voting power in OpenMontage is designed to reward active, high-quality participation while preventing plutocratic capture.

| Role | Definition | Base Voting Power | Notes |
|------|-----------|-------------------|-------|
| **Creator** | Any wallet that has submitted >=1 clip | 1x | Minimum participation threshold |
| **Verified Creator** | >=3 merged clips across any films | 1.5x | Proven track record of quality |
| **Curator** | Staking >=100 $MONTAGE for curation | Stake-weighted | 1 $MONTAGE staked = 1 additional vote unit |
| **Director** | Film initiator who built the Standard Library | Special role | See restrictions below |
| **Platform Team** | OpenMontage founding/engineering team | No voting power | Cannot vote on film-level decisions |

### Director Restrictions

Directors hold a privileged position as film initiators but face governance constraints to prevent conflicts of interest:
- Directors **cannot vote** on segment selection for their own films
- Directors **can** set film parameters (minimum votes, voting period length, quality thresholds)
- Directors **can** veto a single merge decision per month (emergency override) — but the veto is publicly logged and can be overridden by a 75% community supermajority

### Voting Power Caps

To prevent whale dominance in any single decision:
- **Maximum voting power cap**: No single address can control more than **5% of total vote weight** in any individual decision
- **Quadratic scaling** (optional, per-film setting): Voting cost increases quadratically (`cost = votes^2`), making it progressively more expensive to concentrate influence

---

## 4. Anti-Corruption Measures

Governance integrity requires structural safeguards beyond code — it requires rules that make corruption economically irrational.

### Conflict of Interest Rules
- **Directors cannot vote** on their own film's segment selection
- **Creators cannot vote** on their own submissions
- **Voters must disclose** if they have a financial relationship with a submission's creator (enforced via on-chain social graph analysis where possible)

### Vote Integrity
- **All vote records are public and on-chain** — anyone can audit any vote at any time
- **Blind voting**: Submissions are anonymized during the voting period; creator identity is revealed only after votes are finalized
- **Vote locking**: Once cast, votes cannot be changed (prevents last-minute strategic switching)

### Economic Safeguards
- **Maximum voting power cap**: 5% per address per decision (see Section 3)
- **Stake slashing**: Voters who consistently support submissions that are later flagged for Standard Library non-compliance lose a portion of their staked $MONTAGE
- **Appeal bond**: 10 $MONTAGE stake required to appeal (prevents frivolous appeals; slashed on loss)

### Delegation
- Vote delegation is allowed — token holders can delegate their voting power to trusted curators
- Delegation is revocable at any time
- Delegates cannot re-delegate (no delegation chains)

---

## 5. Progressive Decentralization Timeline

OpenMontage follows a deliberate path from centralized control to full community sovereignty:

### Phase 1: Guided Launch (Q2-Q3 2026)
- **Control**: 3-of-5 founding team multi-sig
- **Community role**: Advisory votes via Discord polls (non-binding)
- **Rationale**: Rapid iteration, bug fixes, and security responses require centralized authority. The protocol is unproven and needs the ability to pivot quickly.
- **Commitments**: All multi-sig actions logged publicly; monthly governance transparency reports

### Phase 2: Hybrid Governance (Q4 2026)
- **Control**: Multi-sig for platform operations; token-weighted governance for film-level decisions
- **Community role**: Binding votes on segment merges and film parameters; advisory votes on platform changes
- **Rationale**: Film-level decisions are lower-risk and benefit from decentralization first. Platform-level decisions still need team oversight during mainnet stabilization.

### Phase 3: Full DAO (2027)
- **Control**: On-chain Governor contract for all platform governance; multi-sig retained only for emergency pause
- **Community role**: Binding votes on all decisions — platform fees, treasury allocation, contract upgrades, and film-level governance
- **Rationale**: Protocol is stable, audited, and battle-tested. Community has developed sufficient governance expertise.
- **Transition**: Multi-sig signers publicly commit to executing all DAO-approved proposals; emergency pause authority transfers to DAO with a fast-track voting mechanism (24h quorum, 67% threshold)

### Phase 4: Constitutional Governance (2028+)
- **Control**: Ratified constitution defining immutable rights and governance procedures
- **Community role**: Constitutional amendments require 75% supermajority + 30-day deliberation
- **Features**: On-chain dispute courts (Kleros-style), elected governance council for day-to-day operations, formal separation of executive/legislative/judicial functions
- **Rationale**: Mature protocols benefit from stable, predictable governance that cannot be easily captured by transient token majorities

---

## 6. Transparency Requirements

Trust in governance requires radical transparency. OpenMontage commits to the following:

### Financial Transparency
- **Monthly treasury reports**: Published on-chain and summarized in Discord `#treasury-reports`; includes all inflows, outflows, token burns, and grant disbursements
- **Real-time revenue tracking**: Public dashboard showing per-film revenue, creator earnings, and treasury balance — updated every block
- **Quarterly protocol audits**: Independent financial review of treasury operations and fee collection accuracy

### Governance Transparency
- **Public dashboard**: All governance proposals, votes, outcomes, and execution status displayed in real-time
- **Vote explorer**: Any user can look up any address's voting history across all films and platform proposals
- **Multi-sig transparency**: All multi-sig transactions published with signer identities and rationale within 24 hours of execution

### Technical Transparency
- **Open-source contracts**: All smart contract code is verified on-chain and published on GitHub
- **Audit reports**: All security audit reports published in full (no redactions except for active vulnerabilities under remediation)
- **Incident reports**: Post-mortems for any security incident, governance failure, or unplanned downtime published within 7 days

---

## Conclusion

OpenMontage governance is designed to be **fair, transparent, and progressively decentralized**. By separating film-level creative decisions from platform-level economic decisions, capping voting power, requiring on-chain transparency, and following a deliberate decentralization timeline, the protocol ensures that governance serves the community rather than any single stakeholder.

The ultimate goal: a self-sustaining ecosystem where films improve through merit-based competition, creators earn proportionally to their contributions, and governance decisions reflect the collective wisdom of active participants — not the capital concentration of passive holders.

---

**Version**: 1.0
**Last Updated**: 2026-03-03
**Authors**: OpenMontage Core Team
