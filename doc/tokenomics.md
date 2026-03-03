# OpenMontage Tokenomics (Crypto Economic Model)

> **Executive Summary**
>
> | Metric | Value |
> |--------|-------|
> | **Total $MONTAGE Supply** | 1,000,000,000 (1B tokens) |
> | **Initial Circulating at TGE** | ~8% |
> | **Platform Fee** | 2–5% of film revenue (used for buyback & burn) |
> | **Revenue Split** | 80% creators / 15% director + Standard Library / 5% curators |
> | **Primary Network** | Base (Coinbase L2) |

Open-source initiatives without incentives often struggle to sustain themselves — especially when video generation requires significant computational costs. The crypto economic model solves the fundamental question: **"Who pays, and how do we distribute rewards?"**

---

## 1. Asset Ownership (NFT-ization)

### Clip NFT (Segment NFT)

Every video segment submitted by a creator is minted as an independent NFT containing:

**Metadata**:
- Creator wallet address
- Generation timestamp
- Standard Library version used
- AI model/tool information
- Technical specifications (resolution, duration, format)
- IPFS hash or storage pointer to video file

**Purpose**: Establishes immutable proof of authorship and contribution to the collaborative work.

### Root NFT (Movie NFT)

The complete film exists as a special "Composite NFT" that:

- **Does not contain video data directly**
- **Contains pointers** to all currently merged Clip NFTs in the Main Branch
- **Updates dynamically** when community voting replaces segments
- **Represents ownership** of the complete work as an evolving artifact

**Structure Example**:
```json
{
  "movie_id": "space-odyssey-2077",
  "title": "Space Odyssey 2077",
  "root_nft_address": "0x...",
  "segments": [
    {"shot_id": "001", "clip_nft": "0xabc...", "creator": "0x123...", "duration": 8.5},
    {"shot_id": "002", "clip_nft": "0xdef...", "creator": "0x456...", "duration": 12.3},
    ...
  ],
  "total_runtime": 7200,
  "last_updated": "2026-02-24T14:30:00Z"
}
```

---

## 2. Value Flow & Incentive Distribution

### Revenue Sources (Treasury)

Films can generate income through multiple channels:

1. **Pay-Per-View**: Audience pays to stream the film
2. **Advertising Revenue**: Platform ad share from free viewers
3. **Copyright Sales**: Licensing the complete work to studios/platforms
4. **Merchandise**: Official merch tied to film NFT holders
5. **Crowdfunding**: Pre-sale of "Film Ticket NFTs" to fund initial production

All revenue flows into a **Treasury Smart Contract** for transparent, automated distribution.

---

## 3. Dynamic Revenue Split Mechanism

### Automatic Distribution via Smart Contract

Every revenue event triggers proportional payouts across three stakeholder groups:

### A. Initiator & Standard Library Contributors (15%)

**Who**: The director/producer who created the repository and built the official Standard Library (character LoRAs, style guides, ControlNets, voice models)

**Why**: Rewards the foundational work that makes consistent collaborative creation possible

**Distribution**:
- Film initiator: 10%
- Standard Library asset creators: 5% (split proportionally if multiple contributors)

---

### B. Curators & Voters (5%)

**Who**: Community members who participate in voting and quality curation

**Why**: Incentivizes active participation in selecting the best segments

**Distribution Model**:
- **Voting Weight**: Based on tokens staked or reputation score
- **Reward Pool**: Distributed proportionally to voters who supported merged segments
- **Anti-Gaming**: Bonus rewards for early detection of high-quality submissions

**Example**: If you voted for Clip A before it became popular and got merged, you earn more than late voters.

---

### C. Segment Creators (80%) — **The Core Innovation**

**Revenue Pool**: 80% of all income flows into a **per-second streaming payment pool**

**Payment Mechanism**:
- Each Clip NFT in the current Main Branch earns revenue proportional to its **screen time**
- Formula: `Creator_Earnings = (Clip_Duration / Total_Film_Duration) × 80% × Total_Revenue`
- **Real-time streaming**: Revenue flows continuously while the segment remains active

**Example**:
- Film generates $10,000/month
- Your 30-second clip in a 90-minute film = (30s / 5400s) × $8,000 = **$44.44/month**
- Another creator's 2-minute clip = (120s / 5400s) × $8,000 = **$177.78/month**

---

## 4. Iterative Competition ("Benevolent Arms Race")

### Replacement Dynamics

**What happens when your segment is replaced?**

1. Community votes to merge a better version of Shot 042
2. Your Clip NFT is **removed from Main Branch** (but remains archived)
3. **Revenue stream stops** — you no longer earn from that shot
4. New creator's Clip NFT starts earning immediately
5. You can re-compete by submitting an even better version later

### Why This Creates Healthy Competition

**Incentives**:
- ✅ Continuous improvement: Creators race to use latest AI models (Sora v3 → v4)
- ✅ Quality obsession: Better lighting/motion = more voting support = revenue security
- ✅ Long-tail monetization: Even older segments earn if they remain best-in-class
- ✅ Meritocracy: Payment directly tied to on-screen presence, not seniority

**Result**: Films naturally evolve toward perfection as technology advances and talent competes.

---

## 5. Advanced Tokenomics Features

### 5.1 Film Ticket NFTs (Crowdfunding)

**Pre-Production Funding**:
- Project sells limited "Ticket NFTs" before production starts
- Holders get:
  - Lifetime streaming access
  - Voting rights on segment selection
  - Profit share (e.g., 10% of future revenue pool)
  - Exclusive behind-the-scenes content

**How Ticket NFT revenue share integrates with the 80/15/5 split**: When a film has Film Ticket NFT holders, the 10% profit share comes out of the creator pool. The modified distribution becomes: 70% creators / 15% initiator & Standard Library / 5% curators / 10% Ticket NFT holders = 100%. Films without Ticket NFTs retain the standard 80/15/5 split.

**Example**: 10,000 Ticket NFTs sold at $50 each = $500,000 production budget

---

### 5.2 Governance Token ($MONTAGE)

**Platform-Level Token** for cross-film participation:

**Use Cases**:
- **Staking for voting power**: Lock tokens to increase influence on segment selection
- **Treasury governance**: Vote on platform fee structure, feature development
- **Creator rewards**: Bonus token distributions for high-quality contributions
- **Access gating**: Premium tools or early access to new Standard Libraries

**Value Accrual**: Platform takes 2-5% fee from all film revenue, which buys back and burns $MONTAGE tokens.

---

### 5.3 Royalty Streaming

**Secondary Market Sales**:
- Clip NFTs can be traded on marketplaces
- Original creator earns **perpetual royalty** (e.g., 10%) on all future sales
- Even if your segment is replaced, your NFT retains collectible value

---

## 6. Token Allocation & Distribution

### 6.1 $MONTAGE Token Allocation

| Allocation | % | Amount | Vesting |
|------------|---|--------|---------|
| Community Rewards | 30% | 300,000,000 | Released over 5 years, earned through platform activity (contributing, voting, curating) |
| Ecosystem & Grants | 20% | 200,000,000 | 3-year linear vest, governed by DAO, for film projects, developer grants, partnerships |
| Team & Advisors | 15% | 150,000,000 | 4-year vest, 1-year cliff |
| Treasury Reserve | 15% | 150,000,000 | Unlocked but requires DAO vote to spend |
| Public Sale / IDO | 10% | 100,000,000 | 30% at TGE, remaining over 12 months |
| Seed & Strategic | 7% | 70,000,000 | 2-year vest, 6-month cliff |
| Liquidity Provision | 3% | 30,000,000 | At TGE for DEX liquidity |

### 6.2 Circulating Supply Projection

The table below projects year-by-year token unlocks based on the vesting schedules above. TGE is projected for Q1 2027.

| Year | New Tokens Released | Cumulative Circulating | % of Total Supply |
|------|--------------------|-----------------------|-------------------|
| TGE (Q1 2027) | 80,000,000 | 80,000,000 | 8.0% |
| End Year 1 (2027) | 155,000,000 | 235,000,000 | 23.5% |
| End Year 2 (2028) | 175,000,000 | 410,000,000 | 41.0% |
| End Year 3 (2029) | 160,000,000 | 570,000,000 | 57.0% |
| End Year 4 (2030) | 130,000,000 | 700,000,000 | 70.0% |
| End Year 5 (2031) | 100,000,000 | 800,000,000 | 80.0% |
| Long-tail (2032+) | 200,000,000 | 1,000,000,000 | 100.0% |

**Notes**:
- TGE circulating comprises: 30% of Public Sale (30M) + Liquidity Provision (30M) + early Community Rewards (20M) = ~80M tokens
- Buyback & burn from platform fees reduces effective circulating supply over time, partially offsetting new unlocks
- Treasury Reserve tokens (150M) are technically unlocked but require DAO governance votes and are excluded from circulating supply until deployed

---

## 7. Game Theory & Economic Sustainability Analysis

### Why Creators Compete (Not Collude)

The replacement mechanic ensures that each creator's revenue is directly proportional to their screen time — a zero-sum dynamic relative to the total creator pool. If your 30-second clip is in a 90-minute film, you earn exactly (30/5400) of the 80% creator pool. The only way to earn more is to hold more screen time with higher-quality work. Colluding on mediocre submissions would be self-defeating: it would lower overall film quality, reduce audience revenue, and shrink everyone's earnings. Meanwhile, any single defector who submits genuinely excellent work captures the slot and profits. The Nash equilibrium is honest competition.

The replacement mechanic amplifies this further. Even after a clip is merged, it can be displaced by a superior submission at any time. This creates constant competitive pressure — creators cannot rest on past success. The result is a "benevolent arms race" where each new AI model release (Sora v4, Runway Gen-4) triggers a wave of improved submissions, and films organically improve over time.

### Why Curators Vote Honestly

Curators earn 5% of a film's total revenue. Their earnings are directly tied to the film's commercial success — which depends on audience satisfaction — which depends on segment quality. A curator who votes for low-quality segments is voting to reduce their own income. This alignment is reinforced by the reputation system: curators who consistently back winning segments (those that remain merged longest and generate the most revenue) earn reputation bonuses that increase their voting weight. Conversely, coordinated bad voting triggers stake slashing, making collusion economically irrational. The blind voting period prevents social signaling and bandwagoning, forcing curators to evaluate quality independently.

### Minimum Viable Revenue Analysis

For the platform to sustain contributor engagement, the economics must be compelling at the individual level. Consider the cost structure for a typical AI creator:

- **Cost to generate one shot**: $5–$20 in compute costs (Runway/Sora API fees, GPU time for LoRA-based generation)
- **Time investment**: 2–4 hours including prompt iteration, Standard Library integration, and quality refinement
- **Break-even scenario**: A 30-second clip in a film earning $5,000/month generates approximately $44/month in passive income
- **Payback period**: 2–4 months to recover compute and time costs

This creates a positive ROI even for modestly successful films. For a creator with 3 segments in a $50,000/month film, monthly earnings reach $2,200+ — a meaningful income stream from a portfolio of contributions. The passive, ongoing nature of the earnings (as long as the segment remains merged) compounds the value proposition far beyond one-time freelance work.

### Network Effects

OpenMontage exhibits strong positive network effects across multiple dimensions. More quality films attract more audience, which generates more revenue per creator, which attracts more top creators, which produces higher quality films. This virtuous cycle is the core flywheel. Standard Libraries compound the effect: as directors build increasingly sophisticated character LoRAs, style guides, and ControlNets, these assets become valuable infrastructure with secondary market potential (licensing Standard Libraries across films). Finally, $MONTAGE staking rewards tie token demand directly to platform usage — as more creators and curators participate, demand for voting power grows, creating organic buy pressure that benefits all token holders.

---

## 8. Anti-Gaming Mechanisms

### Sybil Resistance
- **Identity verification** (optional) for higher voting weight
- **Reputation system** rewards consistent quality curation

### Collusion Prevention
- **Blind voting periods**: Submissions anonymous until voting closes
- **Stake slashing**: Voters who consistently support low-quality work lose influence

### Quality Floors
- Minimum technical standards enforced by smart contract (resolution, format, Standard Library compliance)

---

## 9. Economic Flywheel

```
More Revenue → Higher Creator Earnings → Attracts Better Talent → Higher Quality Films → More Audience → More Revenue
```

**Network Effects**:
1. Successful films attract top AI creators seeking revenue streams
2. High-quality Standard Libraries become valuable shared infrastructure
3. Platform reputation grows, enabling larger crowdfunding rounds
4. Token value increases as platform volume grows

---

## 10. Comparison: Traditional vs. OpenMontage

| Aspect | Traditional | OpenMontage |
|--------|-------------|-------------|
| **Payment** | Upfront fee or salary | Revenue share + streaming royalties |
| **Ownership** | Studio owns everything | Creators retain Clip NFT ownership |
| **Replaceability** | Final cut is permanent | Continuous improvement via replacement |
| **Barrier to Entry** | Industry connections | Open submission + quality merit |
| **Revenue Timeline** | One-time payment | Ongoing earnings while active |

---

## 11. Case Study: Revenue Distribution Example

### Film: "Cyberpunk Dreams" (90 minutes, 150 segments)

**Monthly Revenue**: $50,000

**Distribution**:
- Film Initiator: $5,000 (10%)
- Standard Library Creators: $2,500 (5%)
- Voters: $2,500 (5%)
- **Segment Creators: $40,000 (80%)**

**Top Earner**: 
- Creator_A has 3 segments totaling 5 minutes (5.6% of runtime)
- Earnings: $2,240/month

**Average Creator**:
- 1 segment, 30 seconds (0.6% of runtime)
- Earnings: $240/month

**Passive income** as long as segment remains in Main Branch!

---

## 12. Future Enhancements

- **DeFi Integration**: Clip NFTs as collateral for loans
- **Fractional Ownership**: Split Clip NFT ownership among team members
- **Streaming Derivatives**: Futures market for betting on segment replacements
- **Cross-Film Rewards**: Bonus for creators whose segments appear in multiple films

---

## Conclusion

OpenMontage's tokenomics transforms filmmaking from a zero-sum, gatekeeper-controlled industry into a **dynamic, meritocratic marketplace**. By NFT-izing contributions and implementing streaming revenue distribution, we align incentives for continuous quality improvement while rewarding all stakeholders transparently.

**Key Innovation**: The 80% creator pool with per-second revenue streaming creates the first truly **"living film"** economic model — where movies improve over time and contributors earn proportionally to their on-screen presence.

This is not just Git for movies — it's **Airbnb for film production**, where anyone can rent out their creative talent and earn from their work as long as it remains the best.

---

**Version**: 1.1
**Last Updated**: 2026-03-03
**Authors**: OpenMontage Core Team
