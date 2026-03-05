# OpenMontage Economics & Governance Framework

**Version 3.0 | Economic Model & Governance Design**

---

## Executive Summary

OpenMontage replaces opinion-based token voting with **Shot Market Menu Auctions** — per-shot sealed auctions where backers submit valuation menus expressing preferences across ALL competing clips. The clip with the highest aggregate quadratic-weighted community valuation wins. Adapted from the Bernheim-Whinston (1986) menu auction framework, this mechanism produces efficient outcomes under the quadratic-weighted objective (the quadratic transformation trades strict Pareto efficiency for whale resistance), eliminates the "backing a loser" penalty, and encourages full preference revelation. Winners earn streaming revenue; creators earn from competition reward pools and ongoing revenue proportional to screen time.

| Metric | Value |
|--------|-------|
| **Total $MONTAGE Supply** | 1,000,000,000 (1 billion tokens) |
| **Initial Circulating at TGE** | ~8% (80,000,000 tokens) |
| **TGE Date** | Q1 2027 |
| **Platform Fee** | 2-5% of film revenue (used for $MONTAGE buyback & burn) |
| **Standard Revenue Split** | 75% creators / 10% Film NFT holder / 5% Standard Library contributors / 10% clip backers |
| **With Film Ticket NFTs** | 65% creators / 10% Film NFT holder / 5% Standard Library contributors / 10% clip backers / 10% Ticket NFT holders |
| **Winner Selection Mechanism** | Menu Auction — highest aggregate quadratic-weighted valuation wins |
| **Film Creation Bond** | Minimum 500 USDC; recommended 50 USDC x planned shot count |
| **Creator Bond (Submission Stake)** | 50 USDC per clip (fixed; DAO-adjustable) |
| **Challenge Bond** | 100 USDC (2x Creator Bond) |
| **Losing Creator Refund Rate** | 75% of Creator Bond returned (25% loss) |
| **Competition Tax** | 5% of losing Creator Bonds -> Film Treasury |
| **Reward Pool** | 20% of losing Creator Bonds -> split winning creator (50%) / winning backers (50%) |
| **Primary Network** | Base (Coinbase L2); Arbitrum One as fallback |
| **Commit Phase** | 5 days |
| **Reveal Phase** | 2 days |
| **Default Submission Phase** | 14 days |
| **Minimum Backer Deposit** | 10 USDC per backer per shot |
| **Maximum Backer Deposit** | 5,000 USDC per backer per shot |
| **Challenge Cooldown** | 21 days |

---

## 1. Design Philosophy

### 1.1 Replace Opinion with Priced Preferences

Traditional creative platforms use token-weighted voting or reputation systems that are easily gamed: wealth buys influence, social pressure drives herding, and the cost of a wrong vote is zero. OpenMontage's Menu Auction mechanism requires backers to express *priced valuations* across all competing clips. Each backer's valuation menu is a binding economic commitment: they pay their stated valuation for whichever clip the community selects. When preferences carry real financial weight, they become honest.

Unlike single-clip backing systems where backing the wrong clip results in a punitive loss, the menu auction eliminates the "backing a loser" problem entirely. Backers pay only their stated valuation for the winning clip — if they valued it at zero, they pay nothing and receive a full refund. This encourages participation and full preference revelation rather than risk-averse abstention.

### 1.2 Earn Proportionally to Contribution

Every stakeholder in an OpenMontage film earns proportionally to their actual contribution:
- Creators earn based on how many seconds of the final film they produced
- Backers earn based on how much they valued the winning clips (their stated valuations become their streaming revenue stake)
- Directors earn based on film revenue and the governance quality of their project
- Standard Library contributors earn based on the films that depend on their assets

No stakeholder earns passively by holding tokens. All income flows from real creative or economic work.

### 1.3 Films Are Living Documents

A merged clip is not permanent — it can be replaced by a superior submission through the Challenge mechanism. This means films improve over time as AI video generation tools advance. A film produced in 2026 may incorporate dramatically better clips by 2027 as Sora v4, Runway Gen-4, and their successors make higher-quality generation accessible. The economic model incentivizes continuous improvement: superior clips earn streaming revenue, creating perpetual pressure on incumbents.

### 1.4 Prevent Capital Concentration via Quadratic Mechanics

Quadratic weighting on backer valuations ensures that the breadth of community support outweighs any single backer's depth of capital. One whale valuing a clip at 10,000 USDC produces a weight of sqrt(10,000) = 100. One hundred people each valuing the same clip at 100 USDC produce a combined weight of 100 x sqrt(100) = 1,000 — ten times greater with the same total capital. This structural property prevents wealthy actors from unilaterally determining film outcomes and rewards genuine community consensus.

---

## 2. NFT Asset System

### 2.1 Clip NFT (Segment Ownership)

Every video segment submitted by a creator is minted as an independent ERC-721 NFT. Clip NFTs are the atomic unit of creative contribution on OpenMontage.

**Metadata Structure**:

```json
{
  "clip_id": "space-odyssey-2077-shot042-clipA",
  "film_id": "space-odyssey-2077",
  "shot_id": "042",
  "creator": "0x456abc...",
  "created_at": "2026-04-15T09:22:00Z",
  "standard_library_version": "1.2.0",
  "ai_model": "Runway Gen-4",
  "generation_params": { "resolution": "1920x1080", "fps": 24, "duration_seconds": 8.5 },
  "video_ipfs": "ipfs://QmXkZ9...",
  "status": "merged",
  "auction_result": {
    "shot_market_id": "sm-042-round1",
    "selection_weight": 196.81,
    "total_backer_valuations": 1250,
    "resolved_at": "2026-04-28T18:00:00Z"
  },
  "backer_pool": [
    { "backer": "0xaaa...", "valuation_usdc": 40, "sqrt_weight": 6.32 },
    { "backer": "0xbbb...", "valuation_usdc": 35, "sqrt_weight": 5.92 }
  ]
}
```

**Backer Pool Registry**: Each Clip NFT maintains an immutable on-chain registry of all backers whose valuation menus included a positive valuation for this clip when it won. This registry is frozen at resolution — backers of a winning clip retain their revenue share for as long as the clip remains merged.

**Status Machine**:

| Status | Meaning |
|--------|----------|
| `Submitted` | Clip is entered in an active Shot Market; awaiting auction outcome |
| `Merged` | Clip won the auction and is currently part of the film's Main Branch |
| `Archived` | Clip was previously merged but replaced by a successful challenge |

Transitions: `Submitted -> Merged` (win auction), `Submitted -> Archived` (lose auction), `Merged -> Archived` (lose challenge).

**Secondary Sale Royalty**: Creators receive a 10% perpetual royalty on all secondary sales of their Clip NFTs, implemented via ERC-2981.

### 2.2 Film NFT (Film Copyright)

A Film NFT is minted when a director creates a film, representing the composite work and its copyright.

**Film Creation Bond**:

| Property | Value |
|----------|-------|
| Minimum Bond | 500 USDC |
| Recommended Bond | 50 USDC x planned shot count |
| Treasury Seed | 50% of bond -> Film Treasury immediately |
| Completion Escrow | 50% of bond -> locked until >=80% shots merged |
| Refund Condition | mergedShots >= totalShots x 80% |
| Abandonment | 90 days of inactivity -> escrow distributed pro-rata to clip submitters |

The Film NFT is a full ERC-721 token. Transfer of the Film NFT transfers all director rights and the 10% revenue share.

### 2.3 Film Ticket NFTs (Crowdfunding)

Film Ticket NFTs enable audience crowdfunding of film production. Each Ticket NFT represents a fractional revenue share taken from the creator pool:

| Property | Value |
|----------|-------|
| Revenue Source | 10% redirected from creator pool (75% -> 65%) |
| Other Shares | Unchanged (director 10%, stdlib 5%, backers 10%) |
| Ticket Supply | Director-configurable per film |
| Pricing | Director-set; funds flow to Film Treasury |

---

## 3. Shot Market Menu Auction — Core Innovation

The Shot Market Menu Auction is the central mechanism replacing both opinion-based voting and single-clip backing. Each shot in a film becomes a **sealed menu auction** where backers express valuations across all competing clips. The clip with the broadest and deepest community support, measured via aggregate quadratic-weighted valuations, wins.

### 3.1 Mechanism Overview

```
+----------------------+     +----------------------+     +----------------------+
|   SUBMISSION PHASE   |---->|   AUCTION PHASE       |---->|   RESOLUTION         |
|   (14 days default)  |     |   (7 days total)      |     |                      |
|                      |     |                       |     |  Winner = highest    |
|  Creators submit     |     |  Commit: 5 days       |     |  aggregate quadratic |
|  clips + pay 50 USDC |     |  (sealed menus)       |     |  SelectionWeight     |
|  Creator Bond        |     |  Reveal: 2 days       |     |                      |
|                      |     |  (menus opened)       |     |  Payouts             |
|  Clip NFTs minted    |     |                       |     |  distributed         |
+----------------------+     +----------------------+     +----------------------+
```

All Shot Market transactions are denominated in **USDC** (or platform-approved stablecoin). Backers pay only their stated valuation for the winning clip; excess deposits are refunded in full.

**Key Difference from Single-Clip Backing**: In single-clip backing, choosing the wrong clip costs 25% of the deposit. In the menu auction, backers express valuations for ALL clips and pay only their valuation for whichever clip wins. There is no punitive loss for "wrong" preferences — the mechanism rewards honest, complete preference revelation.

**Academic Foundation**: The mechanism is adapted from Bernheim & Whinston (1986), "Menu Auctions, Resource Allocation, and Economic Influence" (*Quarterly Journal of Economics* 101(1): 1-31). In truthful equilibrium, each backer's menu reflects their genuine marginal valuation for each clip, producing efficient outcomes under the quadratic-weighted objective — the clip with the highest aggregate quadratic-weighted community value wins. (Note: the quadratic transformation modifies the standard B-W framework to resist whale dominance, trading strict aggregate surplus maximization for equitable influence.)

### 3.2 Submission Phase (14 Days)

**Duration**: 14 days by default; director-configurable between 7 and 30 days.

**Creator Submission Flow**:
1. Creator downloads the film's Standard Library (LoRA models, style guides, prompt templates, ControlNets)
2. Creator generates a video segment using the Standard Library assets for visual consistency
3. Creator calls `ShotMarket.submitClip(filmId, shotId, videoHash)` and deposits the **Creator Bond** (50 USDC)
4. Contract mints a Clip NFT to the creator's wallet and enters the clip into the shot competition
5. Multiple creators may submit clips for the same shot — no submission cap

**Creator Bond Details**:

| Property | Value |
|----------|-------|
| Amount | 50 USDC (fixed across all films) |
| Adjustability | DAO governance vote required to change |
| Winner outcome | Full bond returned + reward pool share |
| Loser outcome | 75% refunded (25% penalty) |
| Purpose | Prevents spam, signals creator confidence |

**Minimum Submissions Rule**: The auction phase only opens if a shot has **>=2 submissions** at the deadline.
- If only 1 clip submitted: If a shot receives only 1 clip by the submission deadline and no additional submissions arrive within 30 days, the director may auto-merge the single submission (full Creator Bond refunded; no competition occurs).
- If 0 clips submitted: shot remains open; director may extend the deadline or re-describe requirements

### 3.3 Auction Phase — Sealed Menu Commit-Reveal (7 Days)

The 7-day auction phase is divided into two mandatory sub-phases: Commit (5 days) and Reveal (2 days). Backers submit **valuation menus** — a set of valuations, one per clip — expressing how much each clip is worth to them.

#### Step 1: Commit Phase (Days 1-5)

1. Backer evaluates all submitted clips for the shot
2. Backer constructs a **valuation menu**: a set of (clipId, valuation) pairs covering some or all clips
   - Each valuation represents the backer's willingness-to-pay for that clip to win
   - Valuations can range from 0 (no interest) to the deposit amount
   - Unvalued clips are treated as 0
3. Backer creates a cryptographic commitment:
   ```
   commitment = keccak256(abi.encodePacked(clipId_1, v_1, clipId_2, v_2, ..., clipId_N, v_N, salt))
   ```
4. Backer calls `ShotMarket.commitBacking(shotId, commitment)` and deposits USDC equal to **max(v_i)** — the highest valuation in their menu
5. Contract records the commitment hash and locks the deposit
6. During the commit phase, commitment **counts** per shot are publicly visible on-chain — backers can see how many others have committed. However, the **valuations, clip preferences, and backer identities** remain cryptographically hidden until the reveal phase.

**Why deposit = max(v_i)**: The deposit covers the backer's maximum possible payment. If the clip they valued most wins, they pay their full deposit. If a clip they valued less wins, the excess is refunded. This ensures the contract can always settle without additional transfers.

#### Step 2: Reveal Phase (Days 6-7)

1. Backer calls `ShotMarket.revealMenuBacking(shotId, clipIds[], valuations[], salt)`
2. Contract verifies:
   - `keccak256(abi.encodePacked(clipIds[0], valuations[0], ..., clipIds[N], valuations[N], salt)) == storedCommitment`
   - `max(valuations[]) <= deposit`
   - All valuations >= 0
3. Valuations are recorded for each specified clip

**Non-Reveal Penalty**: If a backer fails to reveal during the 2-day reveal window, their entire deposit is **forfeited** and added to the Reward Pool. This penalty eliminates the strategic incentive to commit but withhold a reveal.

**Auction Parameters**:

| Parameter | Value | Rationale |
|-----------|-------|----------|
| Minimum deposit | 10 USDC per backer per shot | Makes Sybil wallet attacks expensive |
| Maximum deposit | 5,000 USDC per backer per shot | Caps single-backer influence beyond quadratic dampening |
| Multiple shot participation | Allowed — back as many shots as desired | Encourages broad curation |
| Backers per shot | No limit | Allows full community participation |

### 3.4 Simplified UX — Rank-Based Menus

To reduce cognitive load, backers can submit a **rank order** instead of explicit valuations. The platform auto-generates valuations using geometric decay:

**Discord Command (Simple)**:
```
/back-shot film:space-odyssey-2077 shot:042 deposit:100 rank:clipA,clipB,clipC
```

**Auto-Generated Valuations**:

| Rank | Valuation | Formula |
|------|-----------|--------|
| 1st choice | 100% of deposit | deposit |
| 2nd choice | 50% of deposit | deposit / 2 |
| 3rd choice | 25% of deposit | deposit / 4 |
| kth choice | deposit / 2^(k-1) | Geometric decay |
| Unranked | 0 | No valuation |

**Example**: Backer deposits 100 USDC and ranks `[clipA, clipB, clipC]`:
- clipA valuation: 100 USDC
- clipB valuation: 50 USDC
- clipC valuation: 25 USDC
- Deposit locked: max(100, 50, 25) = 100 USDC

**Discord Command (Advanced)**:
```
/back-shot-menu film:space-odyssey-2077 shot:042 deposit:200 values:clipA:200,clipB:80,clipC:30
```

Advanced mode allows arbitrary valuations. Both modes produce the same commitment hash and on-chain behavior.

### 3.5 Winner Determination — Aggregate Quadratic SelectionWeight

The winner is determined by aggregate quadratic-weighted valuations across all backers. Quadratic weighting ensures breadth of support matters more than depth of capital.

**Formula**:

```
SelectionWeight(clip_c) = sqrt(CreatorBond_c) + SUM_backers sqrt(v_i_c)

    for all backers i who valued clip c at v_i_c > 0
```

The creator's own bond counts toward the clip's weight, representing the creator's implicit confidence in their own work. Each backer's valuation for clip c contributes sqrt(v_i_c) to that clip's weight.

**Why Crowds Beat Whales — Concrete Comparison**:

| Scenario | Total Valuation for Clip | Quadratic SelectionWeight |
|----------|--------------------------|---------------------------|
| 1 whale values clip at 10,000 USDC | 10,000 USDC | sqrt(10,000) = **100** |
| 25 people each value clip at 400 USDC | 10,000 USDC | 25 x sqrt(400) = **500** |
| 100 people each value clip at 100 USDC | 10,000 USDC | 100 x sqrt(100) = **1,000** |

All three scenarios represent identical total valuation. The clip backed by 100 people outweighs the whale 10-to-1. Crowd wisdom structurally dominates capital concentration.

**Key Difference from Single-Clip Backing**: In the menu auction, a backer's valuation contributes to EVERY clip they valued. A backer who values clipA at 100 and clipB at 50 contributes sqrt(100) = 10 to clipA's weight AND sqrt(50) = 7.07 to clipB's weight. This allows the mechanism to aggregate richer preference information.

**Tie-breaking**: Ties (rare in practice) are broken by earliest submission timestamp.

### 3.6 Complete Settlement Calculation

#### Symbol Definitions

| Symbol | Meaning |
|--------|----------|
| `V_w` | Total backer valuations paid for the winning clip: SUM(v_i_winner) |
| `CB_l` | Total Creator Bonds deposited by losing creators |
| `tau` | Competition Tax rate = **5%** (applied to losing Creator Bonds) |
| `rho` | Loser Creator Refund rate = **75%** |
| `alpha` | Winning Creator Reward Share = **50%** of Reward Pool |
| `beta` | Winning Backer Reward Share = **50%** of Reward Pool |

#### Backer Settlements

```
For each backer i:
  Payment = v_i_winner     (their stated valuation for the winning clip)
  Refund  = deposit_i - v_i_winner   (excess deposit returned immediately)

Backer's streaming revenue stake = v_i_winner
  (recorded in Treasury for ongoing 10% backer pool distribution)
```

Backers who valued the winning clip at 0 pay nothing and receive their full deposit back. They earn no streaming revenue from this shot.

#### Creator Bond Settlements

```
Reward Pool = (1 - tau - rho) x CB_l + F = 20% x CB_l + F
  where F = SUM(forfeited deposits from non-reveals), see §3.3

Winning Creator Payout = CreatorBond + alpha x RewardPool
                       = 50 USDC + 50% x RewardPool

Losing Creator j: refund_j = rho x CreatorBond_j = 75% x 50 = 37.50 USDC
```

#### Competition Tax and Reward Pool

```
Competition Tax = tau x CB_l = 5% x CB_l           -> flows to Film Treasury
Loser Refunds   = rho x CB_l = 75% x CB_l          -> returned to losing creators
Reward Pool     = (1 - tau - rho) x CB_l = 20% x CB_l
    -> 50% x RewardPool to Winning Creator
    -> 50% x RewardPool to Winning Backers
       (proportional to sqrt(v_i_winner) / SUM sqrt(v_j_winner))
```

#### Funds Conservation Proof

Every USDC deposited is fully accounted for — the system conserves 100% of funds:

```
Let F = SUM(forfeited deposits from non-reveals)

Total In  = SUM(CreatorBonds) + SUM(BackerDeposits)
          = SUM(CreatorBonds) + SUM(RevealedDeposits) + F

Total Out = WinningCreatorBond                      (returned in full)
          + rho x CB_l                              (75% refund to losing creators)
          + tau x CB_l                              (5% competition tax -> Film Treasury)
          + (1 - tau - rho) x CB_l + F              (20% reward pool + forfeitures -> distributed to winners)
          + SUM_revealed(v_i_winner)                (backer payments recorded as stakes)
          + SUM_revealed(deposit_i - v_i_winner)    (excess deposits refunded to revealed backers)

Creator Bonds: 50 + 0.75*CB_l + 0.05*CB_l + 0.20*CB_l = 50 + CB_l  checkmark
Backer Deposits: SUM_revealed(v_i_winner) + SUM_revealed(deposit_i - v_i_winner) + F
               = SUM(RevealedDeposits) + F = SUM(BackerDeposits)  checkmark
```

Non-revealed backer deposits (F) are forfeited to the Reward Pool per §3.3, increasing winner payouts. No USDC is destroyed and no USDC is created. The system is a pure redistribution mechanism.

#### Large Example — 3 Clips, 10 Backers

| Clip | Creator Bond | Backers | Aggregate Valuations for This Clip | SelectionWeight |
|------|-------------|---------|-----------------------------------|------------------|
| **Clip A** | 50 USDC | 8 backers valued it | 1,200 USDC total | sqrt(50) + SUM sqrt(v_i_A) = **196.81** |
| **Clip B** | 50 USDC | 6 backers valued it | 450 USDC total | sqrt(50) + SUM sqrt(v_i_B) = **89.23** |
| **Clip C** | 50 USDC | 4 backers valued it | 200 USDC total | sqrt(50) + SUM sqrt(v_i_C) = **47.07** |

Note: Backers can appear in multiple clips' columns since menus span all clips. A backer who valued clipA at 100 and clipB at 50 contributes to both.

**Winner**: Clip A (SelectionWeight = 196.81 — highest)

**Backer Settlements**:
- Each backer pays their valuation for Clip A (v_i_A)
- Excess deposits refunded: deposit_i - v_i_A
- Total backer payments (V_w) = 1,200 USDC (recorded as streaming revenue stakes)

**Creator Bond Settlements**:
- CB_l = 50 + 50 = 100 USDC (Clips B and C creator bonds)
- Competition Tax = 5% x 100 = **5.00 USDC** -> Film Treasury
- Loser Refunds = 75% x 100 = **75.00 USDC** -> Clips B and C creators (37.50 each)
- Reward Pool = 20% x 100 = **20.00 USDC**
  - Winning Creator A: 50% x 20 = **10.00 USDC**
  - Winning Backers of A: 50% x 20 = **10.00 USDC** (split proportionally by sqrt(v_i_A))

**Creator A total from market**: 50 (bond back) + 10 (reward) = **60.00 USDC**
**Backers of A**: 1,200 USDC recorded as streaming revenue stakes + 10.00 USDC reward pool

#### Menu Auction Settlement Example — Individual Backer

**Backer Alice** deposits 200 USDC and submits this valuation menu:
- Clip A: 200 USDC (her top choice)
- Clip B: 80 USDC
- Clip C: 0 USDC

Deposit = max(200, 80, 0) = 200 USDC (locked at commit time)

**If Clip A wins**: Alice pays 200 USDC (her valuation). Refund = 0. Her streaming revenue stake = 200.
**If Clip B wins**: Alice pays 80 USDC. Refund = 120 USDC. Her streaming revenue stake = 80.
**If Clip C wins**: Alice pays 0 USDC. Refund = 200 USDC. No streaming revenue stake.

In all cases, Alice pays exactly what she said the winning clip was worth to her. No punitive loss.

### 3.7 Full Settlement Example — Shot 042

**Setup**: Shot 042 in "Space Odyssey 2077" receives 3 clip submissions and 10 backers participate.

**Backer Valuation Menus** (revealed):

| Backer | Deposit | Clip A Valuation | Clip B Valuation | Clip C Valuation |
|--------|---------|-----------------|-----------------|------------------|
| Backer 1 | 200 | 200 | 50 | 0 |
| Backer 2 | 150 | 150 | 100 | 30 |
| Backer 3 | 100 | 100 | 100 | 50 |
| Backer 4 | 80 | 80 | 40 | 20 |
| Backer 5 | 100 | 100 | 0 | 0 |
| Backer 6 | 120 | 120 | 60 | 0 |
| Backer 7 | 50 | 0 | 50 | 50 |
| Backer 8 | 80 | 80 | 80 | 0 |
| Backer 9 | 60 | 60 | 30 | 30 |
| Backer 10 | 100 | 100 | 50 | 25 |

**SelectionWeight Computation**:

```
Clip A: sqrt(50) + sqrt(200) + sqrt(150) + sqrt(100) + sqrt(80) + sqrt(100) + sqrt(120) + sqrt(80) + sqrt(60) + sqrt(100)
      = 7.07 + 14.14 + 12.25 + 10.00 + 8.94 + 10.00 + 10.95 + 8.94 + 7.75 + 10.00
      = 100.04

Clip B: sqrt(50) + sqrt(50) + sqrt(100) + sqrt(100) + sqrt(40) + sqrt(60) + sqrt(50) + sqrt(80) + sqrt(30) + sqrt(50)
      = 7.07 + 7.07 + 10.00 + 10.00 + 6.32 + 7.75 + 7.07 + 8.94 + 5.48 + 7.07
      = 76.77

Clip C: sqrt(50) + sqrt(30) + sqrt(50) + sqrt(20) + sqrt(50) + sqrt(30) + sqrt(25)
      = 7.07 + 5.48 + 7.07 + 4.47 + 7.07 + 5.48 + 5.00
      = 41.64
```

**Winner**: Clip A (SelectionWeight = 100.04)

**Backer Settlements**:

| Backer | Deposit | Pays (v_i_A) | Refund | Streaming Stake |
|--------|---------|-------------|--------|------------------|
| Backer 1 | 200 | 200 | 0 | 200 |
| Backer 2 | 150 | 150 | 0 | 150 |
| Backer 3 | 100 | 100 | 0 | 100 |
| Backer 4 | 80 | 80 | 0 | 80 |
| Backer 5 | 100 | 100 | 0 | 100 |
| Backer 6 | 120 | 120 | 0 | 120 |
| Backer 7 | 50 | 0 | 50 | 0 |
| Backer 8 | 80 | 80 | 0 | 80 |
| Backer 9 | 60 | 60 | 0 | 60 |
| Backer 10 | 100 | 100 | 0 | 100 |
| **Total** | **1,040** | **990** | **50** | **990** |

Note: Backer 7 valued Clip A at 0, so pays nothing and gets full refund. All others happened to set Clip A as their max valuation in this example.

**Creator Bond Settlements**:
- CB_l = 100 USDC (Clips B and C creator bonds)
- Competition Tax = 5% x 100 = **5.00 USDC**
- Loser Refunds = 75% x 100 = **75.00 USDC** (37.50 each)
- Reward Pool = 20% x 100 = **20.00 USDC**
  - Creator A reward: 50% x 20 = **10.00 USDC**
  - Backers of A reward: 50% x 20 = **10.00 USDC**

---

## 4. Replacement Challenge Mechanism

Once a clip is merged into the Main Branch, it can be replaced if a genuinely superior version emerges. The mechanism uses deliberate friction to protect incumbent creators from frivolous harassment.

### 4.1 Challenge Process (6 Steps)

1. **Challenger submits** a new clip for an already-merged shot and calls `ShotMarket.initChallenge(filmId, shotId)`
2. **Challenge Bond paid**: Challenger deposits **100 USDC** (2x the standard Creator Bond). The higher barrier filters out speculative or harassing challenges
3. **Incumbent notification**: The incumbent creator is notified and has **7 days to respond** by submitting an improved version of their own clip at no additional bond cost
4. **Mini-Auction opens**: A compressed menu auction launches with only two clips (incumbent vs. challenger):
   - Submission window: 7 days (incumbent's response window)
   - Auction phase: **5 days total** (3-day commit + 2-day reveal) — intentionally compressed vs. standard Shot Market's 7-day auction phase
5. **Incumbency Advantage applied**: The incumbent clip starts with a **synthetic SelectionWeight equal to 50% of its original winning SelectionWeight**. The challenger must overcome this head start to trigger replacement
6. **Resolution**: Same quadratic SelectionWeight formula applies. Backers submit binary menus (valuation for incumbent, valuation for challenger). Highest total weight wins

### 4.2 Challenge Outcomes

**If Challenger Wins** (challenger's SelectionWeight > incumbent's adjusted SelectionWeight):
- Incumbent clip status -> `"archived"` (streaming revenue stops immediately)
- Challenger clip merged into Main Branch (streaming revenue starts immediately)
- Challenge Bond refunded in full + challenger's share of reward pool
- Incumbent creator's Clip NFT persists with historical/collectible value but earns no revenue

**If Incumbent Survives** (incumbent's adjusted SelectionWeight holds):
- Incumbent clip remains merged and continues earning
- Challenger receives 75% refund on Challenge Bond (loses 25 USDC)
- Incumbent creator receives **50% of the challenger's forfeited 25 USDC** (a "defense bonus" of 12.50 USDC)
- Remaining 50% of forfeited amount flows to Film Treasury

### 4.3 Challenge Cooldown (21 Days)

After any challenge resolves — regardless of outcome — the same shot **cannot be challenged again for 21 days**. This prevents:
- Harassment of incumbent creators through repeated frivolous challenges
- Exhaustion of a creator's response capacity through rapid successive challenges
- Destabilizing the auction market with constant uncertainty

The cooldown applies per-shot, not per-creator. A new shot can be challenged immediately if its own 21-day window has expired.

### 4.4 Economic Effect Analysis

The Challenge mechanism creates a healthy equilibrium:

| Participant | Behavior Incentivized |
|-------------|----------------------|
| Incumbent Creator | Maintain clip quality; proactively improve when better tools emerge |
| Challenger | Only challenge with genuinely superior work (100 USDC at stake) |
| Backers | Submit honest binary menus; menu auction removes risk of backing the wrong side |
| Director | Film quality automatically improves as better clips replace older ones |
| Audience | Benefits from continuously improving film quality |

---

## 5. Revenue Sources and Distribution

### 5.1 Film Revenue Sources

Films generate income through six channels, all flowing into the Treasury Smart Contract:

| Source | Description |
|--------|-------------|
| Pay-Per-View | Audience streaming fees per view |
| Advertising Revenue | Platform ad network revenue share |
| Copyright Licensing | Studio or streaming platform distribution deals |
| Merchandise | Official merchandise tied to Film NFT holder rights |
| Film Ticket NFTs | Crowdfunding pre-sale proceeds (one-time, at issuance) |
| Competition Tax Revenue | 5% of losing Creator Bonds from Shot Markets -> flows to Film Treasury and is redistributed via the standard 75/10/5/10 formula alongside streaming revenue |

All revenue flows into a **Treasury Smart Contract** (`Treasury.sol`) for transparent, automated distribution. No manual intervention is required for payouts.

### 5.2 Platform Fee and Treasury Flow

Before distribution, the platform takes a fee of **2-5%** of all film treasury revenue. This fee is used for $MONTAGE buyback and burn (see Section 6.3).

```
TreasuryRevenue = TotalFilmRevenue x (1 - PlatformFee)
```

The platform fee percentage is set by DAO governance within the 2-5% range.

### 5.3 Standard Revenue Distribution (No Film Ticket NFTs)

| Stakeholder | Share | Basis |
|-------------|-------|-------|
| Clip Creators | **75%** | Proportional to screen time of merged clips |
| Film NFT Holder (Director) | **10%** | Fixed director compensation for vision and management |
| Standard Library Contributors | **5%** | Split among contributors who built Standard Library assets used by the film |
| Clip Backers Pool | **10%** | Distributed to backers of currently-merged clips |
| **Total** | **100%** | |

This is the **canonical split**: 75/10/5/10. It applies to all films without Film Ticket NFTs.

### 5.4 Film Ticket NFT Adjusted Distribution

When Film Ticket NFTs are issued, the creator pool absorbs the Ticket NFT holder share:

| Stakeholder | Adjusted Share |
|-------------|---------------|
| Clip Creators | **65%** |
| Film NFT Holder (Director) | **10%** (unchanged) |
| Standard Library Contributors | **5%** (unchanged) |
| Clip Backers Pool | **10%** (unchanged) |
| Film Ticket NFT Holders | **10%** |
| **Total** | **100%** |

The director's 10% is **never** reduced by Film Ticket NFTs. Ticket NFT holders are compensated from the creator pool only.

### 5.5 Clip Creator Revenue Formula

```
Creator_Earnings = CreatorPool x (Clip_Duration_seconds / Total_Film_Duration_seconds)

Where:
  CreatorPool = TreasuryRevenue x 0.75    (or 0.65 with Film Ticket NFTs)
```

Each Clip NFT in the current Main Branch earns revenue proportional to its screen time. Revenue flows continuously while the clip remains merged. When a clip is replaced via Challenge, revenue stops immediately.

### 5.6 Clip Backer Revenue Formula

```
BackerPool = TreasuryRevenue x 0.10

For backer i on clip j (where clip j is currently merged):
  Backer_i_Earnings = BackerPool
                    x (Clip_j_Duration_seconds / Total_Film_Duration_seconds)
                    x (v_i_winner_j / TotalWinnerValuations_j)
```

Where `v_i_winner_j` is backer i's stated valuation for clip j from their menu (now recorded as their streaming revenue stake), and `TotalWinnerValuations_j` is the sum of all backer valuations for clip j.

Backer returns from streaming are modest per-clip but compound across a portfolio. An active backer with winning positions across 20 clips in 5 films builds a meaningful passive income stream that requires no ongoing work.

### 5.7 Revenue Changes When a Clip Is Replaced

| Party | Effect When Their Clip is Replaced |
|-------|---------------------------------|
| Old Creator | Streaming revenue stops immediately |
| Old Backers | Streaming revenue from that clip stops immediately |
| New Creator (challenger) | Streaming revenue begins immediately upon merge |
| New Backers | Streaming revenue begins immediately upon merge |
| Old Clip NFT | Status changes to `"archived"`; NFT persists but earns nothing |

### 5.8 Complete Revenue Example — $50,000/Month Film

**Film**: "Cyberpunk Dreams" — 90-minute runtime (5,400 seconds), 150 shots
**Monthly Revenue**: $50,000
**Platform Fee**: 3%

```
TreasuryRevenue = $50,000 x 0.97 = $48,500/month

CreatorPool     = $48,500 x 75% = $36,375/month
DirectorShare   = $48,500 x 10% = $4,850/month
StdLibShare     = $48,500 x 5%  = $2,425/month
BackerPool      = $48,500 x 10% = $4,850/month
```

**Individual Earnings**:

| Participant | Clip Duration | Monthly Earnings |
|-------------|--------------|------------------|
| Top Creator (3 clips, 5-minute total) | 300 seconds | $36,375 x (300/5400) = **$2,020.83** |
| Average Creator (1 clip, 36 seconds) | 36 seconds | $36,375 x (36/5400) = **$242.50** |
| Short Clip Creator (1 clip, 10 seconds) | 10 seconds | $36,375 x (10/5400) = **$67.36** |

**Backer with 100 USDC valuation on a 30-second winning clip** (clip had 1,000 USDC total backer valuations):
```
Backer_Earnings = $4,850 x (30/5400) x (100/1,000) = $2.69/month
```

While per-clip backer earnings are modest, a portfolio of 20 winning clips earning an average of $2.50/month each = **$50/month passive income** from a total valuation position of roughly 2,000 USDC — a 2.5% monthly yield on staked capital, plus competition payouts.

---

## 6. $MONTAGE Token

### 6.1 Role and Use Cases

$MONTAGE is the **platform governance and utility token**. It does not determine which clips win Shot Markets — that function is performed by USDC-denominated menu auctions. $MONTAGE governs the platform itself and provides economic benefits to engaged participants.

| Use Case | Description |
|----------|-------------|
| **Platform Governance** | Vote on platform fee changes, contract upgrades, treasury allocation, grant programs |
| **Fee Reduction** | Staking $MONTAGE reduces Competition Tax for the staker (from 5% down to minimum 2%) |
| **Participation Rewards** | Earned by successful creators and backers as bonus incentives for platform activity |
| **Value Accrual** | Platform fee revenue (2-5%) is used to buy back and burn $MONTAGE (deflationary pressure) |
| **Premium Access** | Priority access to premium Standard Libraries, early access to new AI generation tools |

### 6.2 Staking Fee Discount Mechanism

Creators who stake $MONTAGE receive a reduced Competition Tax rate on their losing Creator Bonds:

**Formula**:
```
EffectiveTaxRate = max(2%, 5% - StakingDiscount)

StakingDiscount = min(3%, StakedMONTAGE / 10,000 x 1%)
```

**Discount Schedule**:

| $MONTAGE Staked | Effective Tax Rate | Discount | Savings on Losing Creator Bond |
|-----------------|-------------------|----------|-------------------------------|
| 0 | 5.0% (standard) | None | -- |
| 10,000 | 4.0% | 1% | 0.50 USDC |
| 20,000 | 3.0% | 2% | 1.00 USDC |
| 30,000+ | 2.0% (floor) | 3% | 1.50 USDC |

The discount applies to the tax portion of losing Creator Bonds — the savings stay in the creator's refund rather than flowing to the Film Treasury.

### 6.3 Value Accrual Mechanism (Buyback & Burn)

```
Platform Fee Revenue -> $MONTAGE Buyback -> Permanent Burn
```

Each quarter:
1. Platform collects 2-5% of all film revenue across the entire platform
2. **80% of collected fees** are used to purchase $MONTAGE on open market
3. Purchased tokens are burned (permanently removed from supply via `burn()`)
4. **20% of collected fees** are retained for platform operations (infrastructure, development, audits)

As platform revenue grows, buyback volume increases proportionally, creating ongoing deflationary pressure on $MONTAGE supply.

### 6.4 Token Allocation

**Total Supply**: 1,000,000,000 $MONTAGE

| Allocation | Percentage | Amount | Vesting Schedule |
|------------|-----------|--------|------------------|
| Community Rewards | 30% | 300,000,000 | Released over 5 years; earned through platform activity — winning competitions, successful backing, active participation |
| Ecosystem & Grants | 20% | 200,000,000 | 3-year linear vest; governed by DAO for film projects, developer grants, Standard Library bounties |
| Team & Advisors | 15% | 150,000,000 | 4-year vest with 1-year cliff |
| Treasury Reserve | 15% | 150,000,000 | Technically unlocked but requires DAO vote to spend; excluded from circulating supply until deployed |
| Public Sale / IDO | 10% | 100,000,000 | 30% at TGE, remaining over 12 months |
| Seed & Strategic | 7% | 70,000,000 | 2-year vest with 6-month cliff |
| Liquidity Provision | 3% | 30,000,000 | At TGE for DEX liquidity pools |

### 6.5 Circulating Supply Projection

TGE is projected for Q1 2027.

| Period | New Tokens Released | Cumulative Circulating | % of Total Supply |
|--------|--------------------|-----------------------|--------------------|
| TGE (Q1 2027) | 80,000,000 | 80,000,000 | 8.0% |
| End Year 1 (2027) | 155,000,000 | 235,000,000 | 23.5% |
| End Year 2 (2028) | 175,000,000 | 410,000,000 | 41.0% |
| End Year 3 (2029) | 160,000,000 | 570,000,000 | 57.0% |
| End Year 4 (2030) | 130,000,000 | 700,000,000 | 70.0% |
| End Year 5 (2031) | 100,000,000 | 800,000,000 | 80.0% |
| Long-tail (2032+) | 200,000,000 | 1,000,000,000 | 100.0% |

**Notes**:
- TGE circulating: 30% of Public Sale (30M) + Liquidity Provision (30M) + early Community Rewards (20M) = ~80M
- Buyback & burn reduces effective circulating supply over time, partially offsetting scheduled unlocks
- Treasury Reserve (150M) is excluded from circulating supply until DAO governance authorizes deployment

---

## 7. Game Theory Analysis

### 7.1 Creator Nash Equilibrium

**Dominant strategy**: Submit the highest-quality clip achievable.

**Reasoning**: Higher quality attracts higher valuations from more backers -> higher aggregate quadratic SelectionWeight -> higher probability of winning -> Creator Bond returned in full + Reward Pool share + ongoing streaming revenue.

Low-quality submissions attract low valuations, yield low SelectionWeight, and result in a 25% bond loss. The replacement mechanic amplifies this pressure: even a merged clip can be displaced at any time by a superior submission. Creators cannot rest on past success.

**Equilibrium result**: All creators submit their best possible work.

### 7.2 Backer Nash Equilibrium

**Dominant strategy**: Submit truthful valuations reflecting genuine quality assessment.

**Reasoning**: The menu auction's truthful equilibrium (Bernheim & Whinston 1986) means that deviating from true valuations cannot improve a backer's expected payoff. Unlike single-clip backing where choosing the wrong clip means a 25% loss, the menu auction lets backers value ALL clips honestly. They pay only their stated valuation for the winner — overstating wastes capital, understating reduces streaming revenue stake and reward pool share.

The blind commit-reveal mechanism eliminates herding — a backer cannot see others' menus during the commit phase. The rank-based shortcut further encourages participation by reducing cognitive load.

**Equilibrium result**: Backers develop quality assessment skills over time; the mechanism structurally rewards honest preference revelation and penalizes misrepresentation.

### 7.3 Director Nash Equilibrium

**Dominant strategy**: Invest meaningfully in the Film Creation Bond and Standard Library.

**Reasoning**: A higher Film Creation Bond signals commitment to potential contributors, attracting more and higher-quality submissions. A well-built Standard Library (character LoRA models, style guides, consistent lighting ControlNets) produces more visually coherent submissions, leading to a higher-quality film, larger audience, and a larger 10% director revenue share.

The 50% completion refund (returned when >=80% of shots are merged) means a 10,000 USDC bond effectively costs only 5,000 USDC for a director who follows through to completion — the Treasury seed represents real infrastructure expenditure, not a sunk cost.

**Equilibrium result**: Directors invest seriously in quality infrastructure and follow through to completion.

### 7.4 Attack Vectors and Mitigations

| Attack | Description | Mitigation |
|--------|-------------|------------|
| **Whale dominance** | Wealthy actor assigns massive valuation to a single clip | Quadratic weighting: sqrt(10,000) = 100 vs. 100 x sqrt(100) = 1,000. Crowd beats whale 10:1 with identical total capital |
| **Sybil backing** | Create many wallets to maximize quadratic weight artificially | Minimum 10 USDC deposit + 7-day wallet age requirement; splitting $X across N wallets costs more in gas than backing $X from one wallet due to quadratic math |
| **Creator-backer collusion** | Creator recruits friends to inflate valuations | Blind commit-reveal prevents pre-coordination. Overstating valuations wastes capital if their clip wins. Quadratic weighting limits concentrated influence |
| **Self-backing** | Creator deposits large valuation backing their own clip | sqrt(valuation) weight means self-backing provides minimal advantage versus many genuine independent backers |
| **Quality flooding** | Submit many mediocre clips to split backer attention | Each submission costs 50 USDC Creator Bond. Backers can value the best clips highly and bad clips at 0 — menus are not zero-sum |
| **Strategic menu manipulation** | Assign high valuations to weak clips to dilute competition | Paying a high valuation for a weak clip that wins wastes capital on poor streaming revenue. Truthful menus are the dominant strategy |
| **Timing attack** | Track on-chain commit hashes to infer preferences | Commit phase hides all menu contents. Only commitment count per shot is visible |
| **Strategic non-reveal** | Commit but don't reveal to manipulate outcomes | Non-reveal forfeits entire deposit to reward pool. Revealing is always strictly better |

### 7.5 Creator Break-Even Analysis

**Costs per submission**:
- Creator Bond: 50 USDC
- Compute costs (AI generation): $5-$20
- Time: 2-4 hours

**If you win** (CB_l represents total losing creator bonds):
- Bond returned in full: 50 USDC
- Competition reward: 50% x 20% x CB_l (scales with number of competing creators)
- Ongoing streaming revenue: proportional to screen time, paid indefinitely

**If you lose**:
- Bond refund: 37.50 USDC (75%)
- Net cash loss: 12.50 USDC + compute costs (~$15-$30 total)

**Break-even scenario**: A 30-second merged clip in a film earning $50,000/month generates approximately $202/month in passive streaming revenue. Even accounting for losing 2 prior competitions (each costing ~$20 net), a single win pays back the total investment within the first month. A creator winning 1 in 3 competitions is comfortably profitable.

**Portfolio compounding**: A creator with 3 merged segments in a $50,000/month film earns approximately $600+/month — meaningful passive income that requires no ongoing work after the initial submission.

---

## 8. Anti-Gaming Mechanisms

### 8.1 Sealed Menu Commit-Reveal

The commit-reveal scheme is the primary anti-manipulation mechanism for the auction phase:

1. **Commit phase (5 days)**: Backers submit `keccak256(abi.encodePacked(clipId_1, v_1, ..., clipId_N, v_N, salt))` — a cryptographic hash of their full valuation menu. Commitment counts per shot are publicly visible on-chain, but the valuations, clip preferences, and backer identities are not visible to any participant until the reveal phase.
2. **Reveal phase (2 days)**: Backers reveal their complete menus. The contract verifies each reveal against its stored hash and checks that max(valuations) <= deposit.
3. **Non-reveal penalty**: Unrevealed deposits are forfeited entirely to the reward pool.

**Behaviors prevented**:
- **Valuation bandwagoning**: Cannot see which clips are attracting high valuations during commit phase
- **Last-minute sniping**: Cannot observe running SelectionWeights and adjust menus
- **Social signaling**: No way to broadcast your valuation preferences to influence others during commit phase
- **Strategic non-revelation**: Forfeiture penalty makes revealing always the dominant choice

### 8.2 Quadratic Weighting on Valuations

The mathematical structure of quadratic weighting on backer valuations provides structural resistance to capital concentration:

```
Whale attack (1 backer values at 10,000 USDC): Weight = sqrt(10,000) = 100
Community (100 backers value at 100 USDC each): Weight = 100 x sqrt(100) = 1,000

Crowd outweighs whale 10:1 with identical total valuation
```

**Sybil attack cost analysis**: To match 100 genuine backers with 100 USDC each (total weight 1,000), a Sybil attacker would need:
- 1,000 fake wallets at minimum 10 USDC each = 10,000 USDC + gas for 2,000 transactions (commit + reveal)
- Each wallet must be >=7 days old (requires advance planning and ongoing maintenance)
- The attack is easily detectable by on-chain analysis of wallet age distributions

### 8.3 Minimum Threshold System

| Threshold | Value | Purpose |
|-----------|-------|----------|
| Creator Bond | 50 USDC | Prevents spam submissions; signals creator confidence |
| Minimum Deposit | 10 USDC per backer per shot | Makes each Sybil wallet attack expensive |
| Maximum Deposit | 5,000 USDC per backer per shot | Caps single-backer influence beyond quadratic dampening |
| Minimum Submissions | 2 clips per shot | Ensures genuine competition exists before auction opens |
| Wallet Age | 7 days minimum | Prevents instant Sybil wallet creation at attack time; enforced at smart contract level. A separate Discord account age check (also 7 days) is applied at the bot level as an additional Sybil barrier |

### 8.4 Cooldown Period Design

**Challenge Cooldown (21 days)**: Applied after any challenge attempt on a shot, regardless of outcome.

The 21-day duration is calibrated to:
- Be long enough to prevent rapid harassment campaigns
- Be short enough to allow genuine quality improvements to be submitted and adjudicated
- Align with the natural creative cycle of AI video generation (meaningful improvements require real work time)

---

## 9. Economic Flywheel

The five reinforcing loops that drive platform growth:

```
Higher Quality Films
        |
More Audience Revenue
        |
Higher Creator Earnings + Higher Backer Returns
        |
Attracts More Skilled Creators + More Curation Capital
        |
More Competitive Shot Markets + Richer Preference Signals
        |
Higher Quality Films (cycle repeats, accelerating)
```

**Loop 1 — Creator Loop**: Higher film revenue -> higher earnings per screen-second -> top AI creators compete for merges -> better clips -> larger audience -> more revenue -> repeat.

**Loop 2 — Backer Loop**: Successful backers earn streaming revenue proportional to their winning valuations -> reinvest in more Shot Markets with informed menus -> richer preference data -> better clip selection -> better film outcomes -> more revenue -> repeat.

**Loop 3 — Director Loop**: Successful films attract more contributors -> more competition per shot -> higher clip quality -> more audience revenue -> director's 10% share grows -> incentivizes more directors to launch ambitious projects.

**Loop 4 — Standard Library Loop**: Better Standard Libraries -> more visually consistent films -> higher audience satisfaction -> more revenue -> Standard Library contributors earn their 5% share -> incentivizes building better LoRA models, ControlNets, and style guides.

**Loop 5 — Token Loop**: More platform revenue -> more $MONTAGE buyback & burn -> reduced circulating supply -> upward token value pressure -> more staking for fee discounts -> more platform engagement -> more revenue -> repeat.

---

## 10. Governance Architecture

### 10.1 Governance Philosophy

OpenMontage governance is built on four foundational principles:

**Meritocracy**: Influence in the ecosystem is earned through contributions, not purchased through connections. A first-time creator whose clip wins a Shot Market has the same economic rights as a veteran with dozens of merged clips. Quality is the currency for film-level decisions. Platform governance introduces token-weighted voting, but caps and reputation multipliers ensure that deep engagement outweighs raw capital.

**Transparency**: Every vote, every revenue distribution, every governance proposal — all are recorded on-chain. There are no backroom deals. Monthly treasury reports, quarterly audits, and real-time dashboards ensure every participant can independently verify that the system operates as documented.

**Progressive Decentralization**: OpenMontage does not launch as a fully decentralized DAO on day one. Early-stage protocols need the ability to respond quickly to bugs, exploits, and market feedback. Governance decentralizes in deliberate phases: from a founding team multi-sig, to token-weighted advisory votes, to binding on-chain governance, to a constitutional DAO with dispute courts.

**Separation of Concerns**: Film-level governance (which segment wins Shot 042) and platform-level governance (should the platform fee be raised from 3% to 4%) are fundamentally different decisions requiring different participants, different quorums, and different timelines. OpenMontage enforces this separation structurally.

### 10.2 Dual-Layer Governance Separation

| Layer | Decision Type | Mechanism | Participants |
|-------|--------------|-----------|-------------|
| Film Level | Which clip merges into each shot | Menu Auction (sealed valuation menus + quadratic SelectionWeight) | Creators, Backers |
| Platform Level | Protocol parameters, fees, upgrades | $MONTAGE token-weighted voting | $MONTAGE Holders |

A $MONTAGE whale cannot override a film community's menu auction outcome. Individual film contributors cannot unilaterally change platform economics. These layers are structurally isolated.

### 10.3 Platform Governance ($MONTAGE Holders)

**Governance Scope** — $MONTAGE holders vote on:
- Platform fee adjustments (within 2-5% range)
- New feature prioritization and roadmap direction
- Treasury allocation — grants, partnerships, ecosystem development
- Smart contract upgrades — proxy upgrades, migration proposals
- Competition Tax and Creator Bond parameter changes
- Emergency pause authority (Phase 3 and beyond)

**Proposal Process (5 Steps)**:

1. **Discussion Phase (>=7 days)**: Author posts proposal in Discord `#governance-proposals`. Community discusses, questions, and suggests amendments. Proposals must include: motivation, specification, risk analysis, and implementation plan.

2. **Signal Vote (48-72 hours)**: Off-chain vote on Snapshot.org to gauge community sentiment. Non-binding, but proposals failing the signal vote are sent back for revision rather than proceeding to on-chain execution.

3. **On-Chain Submission**: If signal vote passes, proposal is submitted to the Governor contract (OpenZeppelin Governor with timelock). Token holders vote on-chain with their staked $MONTAGE balance.

4. **Quorum and Threshold**:
   - Standard proposals: 4% of circulating supply must vote (quorum), 50%+1 majority required
   - Contract upgrades and treasury allocations above $100,000: 67% supermajority required

5. **Timelock (48 hours)**: Approved proposals enter a 48-hour timelock before execution, allowing the community to review and flag last-minute concerns.

**Emergency Actions (Phase 1 Only)**: Before full DAO governance is active, emergency actions (contract pause, critical bug fixes, security responses) are controlled by a **3-of-5 Gnosis Safe multi-sig** of founding team members. All emergency actions are retroactively ratified by the community within 7 days.

### 10.4 Film-Level Governance — Menu Auction Shot Markets

**Menu Auction as Governance Mechanism**: Each Shot Market menu auction is a film-level governance decision. The community decides which clip represents the film's creative direction for each shot by submitting sealed valuation menus. The mechanism aggregates the community's priced preferences into an efficient outcome. This is not advisory — the outcome is final (subject only to dispute resolution).

**Director Authority and Limits**:

| Director CAN | Director CANNOT |
|-------------|----------------|
| Set film parameters (shot count, competition timing, quality thresholds) | Submit valuation menus in their own film's Shot Markets (smart contract enforced) |
| Manage Standard Library (minor updates unilaterally, major updates with community vote) | Override menu auction outcomes |
| Extend deadlines for stalled shots | Selectively accept or reject auction results |
| Set Film Creation Bond amount | Change the 75/10/5/10 revenue split |
| Issue Film Ticket NFTs | Vote on platform governance without $MONTAGE tokens |

The `ShotMarket.commitBacking()` function rejects all transactions from the Film NFT holder's wallet address for their own film. This is a hard contract-level constraint, not a social rule.

**Anti-Whale Mechanism**: Quadratic weighting on valuations is the anti-whale mechanism. No additional voting power cap is needed for film-level decisions. The mathematics guarantee that broad community consensus outweighs concentrated capital at any scale.

**Dispute Resolution (7-Person Jury)**:

If a participant believes an auction outcome was unfair due to manipulation, technical glitch, or rule violation:

1. **Appeal Window**: 48 hours after auction result finalized
2. **Appeal Stake**: Appellant stakes **10 $MONTAGE** (slashed if appeal fails; returned if successful)
3. **Jury Selection**: 7 jurors randomly selected from high-reputation backers (reputation >= 75th percentile, no participation in the disputed auction)
4. **Jury Deliberation**: 72 hours to review clips, valuation records, and appellant's argument
5. **Jury Verdict**: Simple majority (4/7) to overturn; if overturned, auction is re-run with disputed clips flagged
6. **Final Escalation**: If either party contests the jury verdict, the dispute escalates to a platform-level DAO vote (binding; rare by design)

### 10.5 Standard Library Governance

The Standard Library (character LoRA models, style guides, ControlNets, voice models) is the artistic backbone of each film. Changes affect every contributor, so updates are governed by update size.

**Minor Updates (Director Authority)**

The director can unilaterally push **PATCH and MINOR** version changes (under Semantic Versioning):
- Bug fixes in LoRA models or prompt templates
- Adding new assets (e.g., a new supporting character model)
- Style guide clarifications

These updates increment the version (e.g., `1.2.0` -> `1.2.1` or `1.3.0`) and do not invalidate existing work-in-progress submissions.

**Major Updates (Community Approval Required)**

Breaking changes that would invalidate existing submissions require community consensus:
- Re-training core character LoRAs with different base weights
- Fundamental art style changes (e.g., switching from photorealistic to stylized)
- Removing or replacing ControlNet rigs

Process for major updates:
1. Director publishes a migration proposal detailing what changes and why
2. **70% approval** required from active contributors (any wallet with a submission in the last 30 days)
3. **2-week migration period**: Existing work-in-progress can be submitted under the old version; new submissions must use the new version after migration period ends
4. Version increments to next MAJOR (e.g., `1.x.x` -> `2.0.0`)

**Version Scheme (SemVer)**:

| Level | Trigger | Example |
|-------|---------|----------|
| MAJOR | Breaking changes requiring work regeneration | `1.2.3` -> `2.0.0` |
| MINOR | New assets or non-breaking enhancements | `1.2.3` -> `1.3.0` |
| PATCH | Bug fixes and documentation corrections | `1.2.3` -> `1.2.4` |

### 10.6 Roles and Participation Rights

| Role | Definition | Film-Level Rights | Platform-Level Rights |
|------|-----------|------------------|----------------------|
| **Creator** | Wallet with >=1 clip submitted | Submit clips + pay Creator Bond; submit valuation menus in other shots | None (unless also holding $MONTAGE) |
| **Backer** | Wallet with >=1 valuation menu submitted | Submit valuation menus in Shot Market auctions; claim payouts | None (unless also holding $MONTAGE) |
| **Director** | Film initiator holding the Film NFT | Set film parameters; manage Standard Library; **cannot** submit menus in own film | $MONTAGE governance (if holding tokens) |
| **$MONTAGE Holder** | Wallet staking $MONTAGE | Fee discounts on Competition Tax | Platform governance votes + delegation |
| **Platform Team** | OpenMontage founding/engineering team | No participation in any Shot Markets | Emergency multi-sig (Phase 1 only) |

### 10.7 Anti-Corruption Measures

**Conflict of Interest Rules**:
- Directors cannot submit valuation menus in their own film's Shot Markets (smart contract enforced, not just policy)
- Creators automatically contribute to their own clip's SelectionWeight via the Creator Bond — this is transparent by design, not a conflict
- Backers submit one menu per shot covering all clips — the mechanism encourages honest, complete preference revelation rather than strategic single-clip commitment

**Market Integrity**:
- All valuation records are public and auditable on-chain after reveal
- Blind commit-reveal prevents bandwagoning, sniping, and social coordination during the commit phase
- Commitment hashes cannot be changed once submitted; non-reveal forfeits the deposit entirely

**Economic Safeguards**:
- Quadratic weighting on valuations is the primary anti-whale mechanism
- Maximum single-address deposit of 5,000 USDC per shot caps concentration beyond quadratic dampening
- Non-reveal penalty makes strategic abstention always negative expected value
- Challenge Bond (2x Creator Bond) discourages frivolous incumbent challenges
- Appeal bond (10 $MONTAGE) prevents frivolous dispute appeals

**Platform Governance Delegation**:
- $MONTAGE vote delegation is allowed for platform-level governance
- Delegation is revocable at any time without penalty
- Delegates cannot re-delegate (no delegation chains beyond depth 1)
- Delegation does not apply to Shot Markets (menu submission requires direct USDC deposits)

### 10.8 Progressive Decentralization Timeline

**Phase 1: Guided Launch (Q2-Q3 2026)**
- Control: 3-of-5 founding team Gnosis Safe multi-sig
- Community role: Advisory votes via Discord polls (non-binding)
- Shot Markets: Fully operational menu auctions for all film-level clip selection
- Rationale: Rapid iteration, bug fixes, and security responses require centralized authority during the unproven early phase
- Commitments: All multi-sig actions logged publicly within 24 hours; monthly governance transparency reports published

**Phase 2: Hybrid Governance (Q4 2026)**
- Control: Multi-sig for platform operations; Shot Market menu auctions autonomous for all film-level decisions
- Community role: $MONTAGE token-weighted voting for fee structures and feature priorities; DAO treasury grants begin
- Rationale: Film-level decisions operate autonomously via menu auctions from day one. Platform-level governance transitions from team oversight to community input during mainnet stabilization.

**Phase 3: Full DAO (2027)**
- Control: On-chain Governor contract for all platform governance; multi-sig retained only for emergency pause with 24h fast-track voting (67% threshold)
- Community role: Binding votes on all platform decisions — fees, treasury, contract upgrades, parameter changes, grant programs
- Transition: Multi-sig signers publicly commit to executing all DAO-approved proposals regardless of personal opinion
- Rationale: Protocol is stable, audited, and battle-tested. Community has developed sufficient governance expertise.

**Phase 4: Constitutional Governance (2028+)**
- Control: Ratified constitution defining immutable rights and governance procedures
- Community role: Constitutional amendments require 75% supermajority + 30-day deliberation period
- Features: On-chain dispute courts (Kleros-style), elected governance council for day-to-day operations, formal separation of executive/legislative/judicial functions
- Rationale: Mature protocols benefit from stable, predictable governance that resists capture by transient token majorities

### 10.9 Transparency Requirements

**Financial Transparency**:
- Monthly treasury reports published on-chain and summarized in Discord `#treasury-reports`; includes all inflows, outflows, token burns, and grant disbursements
- Real-time revenue dashboard showing per-film revenue, creator earnings, and treasury balance — updated every block
- Quarterly independent financial review of treasury operations and fee collection accuracy

**Governance Transparency**:
- Public dashboard displaying all proposals, votes, outcomes, and execution status in real-time
- Vote explorer: any user can look up any address's voting history across all films and all platform proposals
- Multi-sig transparency: all multi-sig transactions published with signer identities and rationale within 24 hours of execution

**Technical Transparency**:
- Open-source contracts: all smart contract code is verified on-chain and published on GitHub
- Audit reports: all security audit reports published in full (no redactions except for active vulnerabilities under remediation)
- Incident reports: post-mortems for any security incident, governance failure, or unplanned downtime published within 7 days

---

## 11. Competitive Positioning and Market Analysis

### 11.1 Market Background — Three Converging Trends

**Decentralized creator economies** have matured since the 2021-2022 NFT boom. Platforms like Zora (which generated $425M in creator coin trading volume in Q2 2025) and Story Protocol (mainnet launched February 2025, backed by $134M from a16z) demonstrate sustained demand for on-chain creator infrastructure. However, video remains underserved — Glass.xyz, the most prominent video NFT platform, suspended development in September 2023, concluding there was "no sustainable demand for video NFTs" in a pure collectible model.

**AI video generation** has reached production quality. Sora 2, Runway Gen-3/Gen-4, Pika, and Luma AI now generate cinematic-quality clips from text prompts. The AI media market was valued at $26 billion in 2024 with 24.2% CAGR. Collaborative AI filmmaking tools are emerging — Martini (YC 2025) and LTX Studio offer script-to-screen pipelines — but none incorporate decentralized ownership or community governance.

**Film DAOs** have proven the concept of community-driven filmmaking. Film.io celebrated a record-breaking first year at ETH Denver 2025, while Decentralized Pictures (backed by the Coppola family) funded short films through community voting. However, these platforms focus on financing and greenlighting traditional productions, not on the collaborative creation of AI-generated content.

### 11.2 Direct Competitors

**Film.io**: The closest existing competitor in the decentralized filmmaking space. Uses a dual-token model (FAN + FILM tokens) for community voting on which projects receive financing. Key difference: Film.io is a financing and greenlighting platform — fans vote on which projects get funded, but production follows traditional workflows. It does not involve crowdsourced content creation, AI video generation, or a pull-request model. In 2025, Film.io faced tokenomics challenges including a Biconomy delisting and approved burning ~3 billion unallocated FAN tokens to address inflation.

**Decentralized Pictures (DCP)**: Co-founded by Roman Coppola and American Zoetrope, a 501(c)(3) nonprofit running a democratic film fund. Members earn TALNT tokens by reviewing pitches and scripts. In 2025, DCP moved to Base (L2) and launched the DCP+ streaming service. Key difference: DCP focuses on discovering and funding traditional filmmakers, not on collaborative AI content creation. It lacks Shot-based decomposition, pull request workflow, and automated revenue sharing.

**Mogul Productions**: A Toronto-based platform connecting filmmakers with investors through blockchain-based quadratic voting. Over 50,000 users participate in selecting which films receive financing via STARS tokens. Key difference: Like Film.io and DCP, Mogul is a financing platform. Community involvement ends at the funding decision.

**Martini (YC 2025)**: A collaborative AI-native filmmaking platform for professionals. It represents the emerging category of multi-user AI filmmaking tools. Key difference: Martini targets professional teams using centralized collaboration. It lacks decentralized governance, NFT-based ownership, or open contribution models.

### 11.3 Adjacent Platforms

**Story Protocol**: Launched mainnet in February 2025 as a purpose-built L1 blockchain for intellectual property management, backed by $134M. Its Programmable IP License (PIL) attaches usage permissions and revenue-sharing terms to on-chain IP assets. OpenMontage should evaluate building on Story Protocol's IP infrastructure for Clip NFT licensing and Standard Library asset registration rather than creating entirely custom licensing contracts. Story Protocol is media-agnostic infrastructure — it doesn't provide the filmmaking workflow, visual consistency, or community curation that OpenMontage offers.

**Royal.io**: Pioneer of fractional music royalty ownership through NFTs. Generated $7.5M in trading volume and paid out $156K in royalties before sunsetting its marketplace in 2024-2025. Key lesson: Royal validated fractional creative revenue ownership via NFTs; its challenges with token liquidity, marketplace sustainability, and regulatory complexity are directly instructive for OpenMontage's Clip NFT model. The SEC scrutiny of music royalty NFTs will likely apply to Clip NFTs with revenue rights — prepare legal strategy early.

**Zora**: Evolved from an NFT marketplace into an on-chain social network by 2025-2026. Key metrics: 1.5M Creator Coins, $425M in trading volume (Q2 2025), 2.8M active traders. Demonstrates that creator-driven token economies can achieve meaningful scale. Cautionary note: Zora's drift toward pure attention speculation (rather than actual content production) is a path to avoid.

**Nouns DAO**: Auctions one generative NFT per day with 100% of proceeds to a community treasury. NFT holders govern spending through proposal voting. Funded 150+ creative projects. Provides a proven model for community-funded creative work. The 2023 fork mechanism — allowing dissatisfied members to exit with ~$27M — is a governance stress test OpenMontage should consider replicating (exit rights for token holders build community trust).

**Mirror.xyz**: Pioneered Web3 publishing with on-chain content, NFT minting, and crowdfunding. Acquired by Paragraph in May 2024. Illustrates the challenge of sustaining standalone creator platforms, and validates Arweave for permanent content storage (directly applicable to OpenMontage's planned video storage architecture).

### 11.4 Infrastructure Layer

**Coordinape**: Peer-to-peer DAO compensation tool where members allocate tokens based on perceived contribution value. Could complement OpenMontage's menu auction curation system for evaluating Standard Library contributions or non-video work that doesn't fit the Shot-based pull request model.

**Dework**: DAO task management with Discord-integrated bounty boards, token-based rewards, and contributor profiles. Shot assignments structured as Dework bounties could provide a familiar UX for DAO-experienced contributors. OpenMontage could build on Dework's infrastructure or learn from its patterns.

**Lens Protocol and Farcaster**: Decentralized social infrastructure (Lens Protocol: $31M funding, ZK-powered chain; Farcaster: $1B valuation, Neynar acquisition 2025). Integrating with these identity layers could bootstrap creator communities without building identity infrastructure from scratch.

### 11.5 Full Competitive Comparison Table

| Platform | Category | Revenue Sharing | Consistency Solution | Governance Mechanism | Key Strength | Key Weakness |
|----------|----------|----------------|---------------------|---------------------|--------------|----------------|
| **OpenMontage** | Crowdsourced AI Filmmaking | 75% creators / 10% Film NFT / 5% StdLib / 10% backers | Standard Library (LoRAs, style guides, ControlNets) | Menu Auctions (sealed valuation menus, quadratic weighting) for film; $MONTAGE for platform | Unified pipeline with visual consistency + efficient clip selection (quadratic-weighted) | Pre-launch; unproven at scale |
| **Film.io** | Film Financing DAO | Funds projects, traditional distribution | N/A (traditional production) | FAN token governance | Active community, Sundance debut | Tokenomics challenges; financing only |
| **Decentralized Pictures** | Film Funding Nonprofit | Grant-based funding only | N/A (traditional production) | TALNT token community review | Coppola credibility, nonprofit structure | Small scale; grants limit throughput |
| **Story Protocol** | IP Infrastructure | Automated royalty distribution via Royalty Module | N/A (infrastructure layer) | $IP token governance | On-chain IP licensing standard, $134M backing | Infrastructure only — no creative workflow |
| **Zora** | Creator Social/Trading | 50% of trading fees to creators | N/A | $ZORA token | $425M volume, strong creator adoption | Drifting toward pure speculation |
| **Royal.io** | Music Royalty NFTs | Direct royalty passthrough to NFT holders | N/A (music only) | N/A | Proven royalty distribution | Marketplace sunsetted; pivoting |
| **Nouns DAO** | Community Creative Fund | Treasury funds projects | N/A | 1 Noun = 1 vote, fork mechanism | $27M+ treasury, 150+ projects, open IP | Not a production platform |
| **Mogul Productions** | Film Financing | Revenue participation via token staking | N/A (traditional production) | STARS token quadratic voting | 50K+ users, democratic selection | Financing only; limited recent traction |
| **Runway ML** | AI Video Generation | N/A (SaaS tool) | Per-project style controls | N/A (centralized SaaS) | Best-in-class AI video generation | No community ownership or governance |
| **Martini (YC 2025)** | Professional AI Filmmaking | N/A (SaaS subscription) | Centralized project controls | N/A | Professional-grade collaborative pipeline | Centralized; closed contribution model |

### 11.6 Market Gaps — OpenMontage's Unique Position

**Gap 1 — No platform bridges AI video generation with decentralized ownership.** AI video tools (Runway, Sora, Pika) are centralized SaaS products with no on-chain ownership layer. Blockchain film platforms (Film.io, DCP, Mogul) fund traditional production without touching AI-generated content. Nobody enables community members to generate, submit, and collectively curate AI video with on-chain ownership and automated revenue sharing.

**Gap 2 — Visual consistency in crowdsourced content is unsolved.** This is the fundamental technical barrier to collaborative AI filmmaking. When 100 creators generate clips for the same film, the results are visually incoherent — different character appearances, inconsistent lighting, varying styles. OpenMontage's Standard Library concept (mandatory LoRA models, style guides, prompt templates, ControlNets per film) is a novel solution with no equivalent in the market.

**Gap 3 — No granular per-contribution revenue sharing for video.** Story Protocol automates IP royalties at the asset level. Royal.io distributes music royalties. No platform distributes film revenue proportionally to screen time contribution at the Shot level, tracked through NFT provenance. OpenMontage's menu auction mechanism adds an efficient quality-selection layer (under quadratic-weighted objectives) on top of this, transforming passive curation into economic investment.

**Gap 4 — Git-style version control for creative video does not exist.** The pull request model — submit a Shot, community evaluates via menu auctions, best version merges to main branch — applies proven software development workflows to film production. This is conceptually powerful and entirely unoccupied territory.

**Gap 5 — Discord-first DAO filmmaking is unexplored.** While Dework and Coordinape provide DAO task management through Discord, no platform has built a complete filmmaking workflow — from Shot assignment through submission, menu auction evaluation, and merging — natively within Discord.

### 11.7 Strategic Implications

**Learn From**:
- **Story Protocol's PIL licensing model**: Evaluate building on Story Protocol's IP infrastructure rather than creating custom licensing smart contracts. The PIL framework for derivatives and automated royalties is directly applicable to Clip NFTs and Standard Library assets.
- **Zora's creator economics**: The 50% creator fee share drove real adoption. OpenMontage's 75% creator share is more generous; the menu auction mechanism additionally rewards quality curators with 10% streaming revenue. Emphasize this dual incentive in positioning.
- **Nouns DAO's fork mechanism**: Build exit rights into governance from day one. The ability for dissatisfied participants to exit cleanly strengthens community trust.
- **Royal.io's regulatory journey**: Music royalty NFTs faced securities scrutiny. OpenMontage's Clip NFTs with revenue rights will likely face similar questions — prepare legal strategy for SEC and global regulatory environments early.
- **Glass.xyz's failure**: "No sustainable demand for video NFTs" in a pure collectible model. OpenMontage must ensure Clip NFTs have functional utility (streaming revenue rights, menu auction participation) beyond speculation.

**Watch Closely**:
- **Story Protocol ecosystem growth**: If Story becomes the standard IP layer, building on it early provides network effects. If it stalls, OpenMontage needs its own licensing contracts.
- **AI video quality trajectory**: As Sora, Runway, and competitors improve, the quality bar for acceptable AI film clips rises. The Standard Library approach becomes more powerful as base model quality increases — better models make Standard Library adherence easier to achieve.
- **Film.io's pivot attempts**: Film.io is the nearest conceptual neighbor. If it pivots toward AI-assisted production or contributor-driven content, it becomes a more direct competitor.
- **Regulatory landscape for creative NFTs with revenue rights**: SEC and global regulatory attitudes toward fractionalized creative revenue will directly impact OpenMontage's token model.

**Build These Competitive Moats**:
1. **Standard Library ecosystem**: The collection of film-specific LoRAs, ControlNets, and style guides becomes a defensible asset as films accumulate. Each completed film's Standard Library is a unique creative artifact that cannot be replicated without rebuilding the training data.
2. **Contributor network**: A community of skilled AI video creators trained on the PR workflow and Standard Library system represents a talent moat. Platform-specific expertise creates switching costs.
3. **Film catalog with provenance**: A growing library of films with complete on-chain contribution history and NFT provenance creates compounding value that cannot be easily replicated. Each film is an auditable creative artifact.

---

## 12. Changes from Previous Model

| Aspect | v2.x (Previous Model) | v3.0 (Current Model) | Rationale |
|--------|-----------------------|---------------------|----------|
| **Winner selection** | Single-clip quadratic backing: each backer backs ONE clip | Menu Auction: backers submit valuation menus across ALL clips | Full preference revelation produces efficient outcomes under quadratic-weighted objective (adapted from Bernheim & Whinston 1986); eliminates "backing a loser" penalty |
| **Backer input** | Single (clipId, amount) commitment | Valuation menu: (clipId_1, v_1, ..., clipId_N, v_N) | Richer preference data enables better clip selection; reduces backer risk |
| **Backer payment** | Full deposit locked; losers get 75% refund (25% penalty) | Pay only valuation for winning clip; excess deposit refunded in full | Encourages participation; backers face no punitive loss for "wrong" preferences |
| **Reward Pool source** | 20% of ALL losing deposits (creators + backers) | 20% of losing Creator Bonds only | Backers no longer face punitive losses; reward pool comes from creator side |
| **Backer streaming stake** | Deposit amount = streaming revenue stake | Winning valuation = streaming revenue stake | Backer's investment scales with their assessment of the winning clip |
| **Weight formula** | BackingWeight = sqrt(CreatorBond) + SUM sqrt(deposit_i) | SelectionWeight = sqrt(CreatorBond) + SUM sqrt(v_i_c) | Same quadratic structure; v_i_c is backer i's valuation for clip c from their menu |
| **UX complexity** | Simple: pick one clip, deposit USDC | Medium: rank clips or set valuations; simplified by rank-based shortcut | Rank-based UX (`/back-shot ... rank:A,B,C`) keeps cognitive load manageable |
| **Commit hash** | keccak256(clipId, amount, salt) | keccak256(clipId_1, v_1, ..., clipId_N, v_N, salt) | Encodes full valuation menu |
| **Challenge backing** | Same as standard: single-clip backing | Binary menu: (incumbent_value, challenger_value) | Consistent with menu auction framework |
| **Academic foundation** | Quadratic voting/funding literature (Lalley & Weyl 2018) | Menu auctions (Bernheim & Whinston 1986) + quadratic weighting | Combines efficient preference aggregation with whale resistance |
| **Revenue split** | 75/10/5/10 | 75/10/5/10 (unchanged) | Split is independent of selection mechanism |
| **Creator Bond economics** | 50 USDC bond; losers get 75% refund | 50 USDC bond; losers get 75% refund (unchanged) | Creator-side economics preserved |

---

**Version**: 3.0
**Last Updated**: 2026-03-05
**Authors**: OpenMontage Core Team
