# OpenMontage Roadmap

**Version 1.0 — March 2026**

---

## Vision Statement

By 2029, OpenMontage aims to become the leading platform for collaborative AI filmmaking — with **50,000+ active creators**, **500+ completed films**, and **$10M+ in annual creator earnings** distributed transparently via smart contracts. We envision a world where any screenplay can become a professional film through global collaboration, where creators earn fairly based on merit, and where films continuously improve as AI technology advances.

This roadmap outlines the concrete milestones, success criteria, and risk mitigations that will take us from concept to that reality.

---

## Phase 1: Foundation (Q2–Q3 2026)

**Theme: Build the foundation, prove the concept**

Phase 1 is about validating the core thesis — that crowdsourced AI filmmaking with enforced visual consistency and economic incentives can produce a watchable, coherent film. Everything in this phase serves the Pilot Film.

### Q2 2026 Goals

**Community Infrastructure**
- Launch the OpenMontage Discord server (target: 1,000 members by end of Q2)
  - Set up server architecture: #announcements, #film-listings, #general, #creator-showcase, #voting, #support, per-film channels
  - Deploy welcome flow with role selection (Creator, Curator, Viewer)
  - Begin community building via AI filmmaking Twitter/X spaces, Reddit posts, and partnerships with AI art communities
  - Recruit 10–15 community moderators

**Smart Contract Development**
- Deploy core smart contracts on Base testnet:
  - `ClipNFT.sol` — ERC-721 for segment ownership with metadata storage
  - `MovieNFT.sol` — Composite NFT with dynamic pointer array
  - `Treasury.sol` — Automated revenue distribution (80/15/5 split)
  - `Voting.sol` — Weighted voting with blind voting periods
- Internal testing and iteration on testnet
- Publish contract ABIs and developer documentation

**Discord Bot v0.1**
- Core commands: `/claim-shot`, `/submit`, `/vote`, `/my-clips`, `/film-status`
- Basic wallet connection via Discord OAuth
- Submission upload flow (video file + metadata)
- Voting interface with emoji reactions or slash command ratings

**Standard Library Specification**
- Publish Standard Library Specification v1.0
  - Required asset formats (LoRA .safetensors, ControlNet configs, voice model specs)
  - Versioning scheme (SemVer)
  - Compliance validation criteria
  - Distribution method (Git LFS + CDN)
- Create template repository for directors to bootstrap new film projects

**Pilot Film Project Kickoff**
- Title: **"Genesis"** (20-minute sci-fi short film)
  - 100 shots total, decomposed from an original screenplay
  - Genre: sci-fi (strong match for current AI video generation capabilities — futuristic environments, stylized lighting, non-photorealistic characters reduce uncanny valley)
  - Open to 50 invite-only creators selected from community applications
- Director builds Standard Library v1.0:
  - 4 character LoRAs (protagonist, antagonist, 2 supporting)
  - Art style guide (color palette, lighting reference, cinematography notes)
  - 15 prompt templates covering dialogue, action, and environment shots
  - 2 ControlNet rigs for primary characters
  - 2 voice clone models for lead characters

### Q3 2026 Goals

**Discord Bot v0.5**
- Wallet integration (connect MetaMask/Coinbase Wallet via Discord)
- Automated voting tallies with transparent result posting
- Leaderboard: top creators by merged clips, top curators by accuracy
- Notification system for new submissions, voting periods, and merge events
- `/compare` command to view side-by-side submissions for a shot

**Pilot Film Completion**
- Complete "Genesis" — first end-to-end test of the full workflow
  - All 100 shots receive at least one submission
  - Community voting selects the best version for each shot
  - Final assembly into a watchable 20-minute film
  - Public premiere on Discord with live watch party
- Publish detailed learnings documentation:
  - What worked and what didn't in the Standard Library approach
  - Creator feedback survey results
  - Voting dynamics analysis (participation rates, consensus patterns)
  - Technical post-mortem (latency, storage costs, bot reliability)

**Security & Partnerships**
- Smart contract security audit by an external firm (e.g., OpenZeppelin, Trail of Bits, Cyfrin)
- Address all critical/high findings before mainnet
- Establish partnerships with 2–3 AI tool providers (Runway, Pika, Kling) for potential subsidized API access or co-marketing

**Community Growth**
- Target: 5,000 Discord members
- Weekly community calls (creator showcases, roadmap updates)
- Creator onboarding guide and video tutorials

### Phase 1 Success Criteria

| Criteria | Target |
|----------|--------|
| Creators who submitted ≥ 1 clip | ≥ 50 |
| Pilot film shots filled | ≥ 80% (80 of 100) |
| Smart contract security audit | Pass with no unresolved critical issues |
| Community sentiment | Net positive (measured via survey) |
| Discord members | ≥ 5,000 |
| Standard Library compliance rate | ≥ 90% of submissions pass validation |

---

## Phase 2: Alpha Launch (Q4 2026)

**Theme: Open the floodgates**

Phase 2 transitions from a controlled pilot to a public platform. The invite-only gate comes down, mainnet contracts go live, and real economic value starts flowing.

### Mainnet Deployment
- Deploy all smart contracts to Base mainnet
- Gas optimization pass (batch minting, efficient storage patterns)
- Deploy subgraph for indexing on-chain events (The Graph or custom indexer)

### Web Dashboard v1
A companion web interface to Discord (not a replacement — Discord remains the primary community hub):
- **Film browsing**: Discover active projects, browse shots, watch current cuts
- **Submission management**: Upload clips, track submission status, view voting results
- **Revenue tracking**: Real-time earnings dashboard, claim revenue, transaction history
- **Profile pages**: Creator portfolio, curation stats, reputation score
- **Wallet connection**: MetaMask, Coinbase Wallet, WalletConnect

### Discord Bot v1.0 (Full Feature Set)
- All v0.5 features plus:
  - Revenue claiming directly from Discord
  - Film creation workflow for directors (guided setup)
  - Advanced voting: quadratic voting option, delegation
  - Automated Standard Library compliance checking on submission
  - Per-film private channels with role-gated access

### Film Ticket NFT Crowdfunding
- Launch Film Ticket NFT system for 3 inaugural public films
- Ticket holders receive: lifetime streaming access, voting rights, revenue share (10% of pool)
- Smart contract escrow with milestone-based release

### Platform Opens Publicly
- No invite required — any creator can browse films, claim shots, and submit clips
- Any director can create a new film repository (with minimum Standard Library requirements)
- $MONTAGE test token distribution to Phase 1 contributors as a reward for early participation

### Phase 2 KPIs

| KPI | Target |
|-----|--------|
| Discord members | 20,000+ |
| Active film projects | 10 |
| Registered creators | 500+ |
| Film Ticket NFT crowdfunding raised | $50,000+ |
| Films with complete first cuts | ≥ 3 |
| Average time to first submission per shot | < 7 days |

---

## Phase 3: Ecosystem Growth (2027)

**Theme: Build the economy**

Phase 3 launches the $MONTAGE token, activates full governance, and builds the tooling and marketplace infrastructure that transforms OpenMontage from a platform into an ecosystem.

### H1 2027

**$MONTAGE Token Launch**
- Token Generation Event (TGE) via IDO on a Base-native DEX
- Initial circulating supply: ~80M tokens (8% of total)
- DEX liquidity pool seeded with 30M tokens from Liquidity Provision allocation
- CEX listing strategy (target 2–3 mid-tier exchanges within 3 months of TGE)

**Full Governance Activation**
- Token-weighted voting replaces testnet governance
- DAO proposal system: any holder with ≥ 10,000 $MONTAGE can submit proposals
- First governance votes: platform fee adjustment, grant allocations, feature priorities
- Delegate voting for passive holders

**Advanced Standard Library Tools**
- One-click LoRA training pipeline: directors upload reference images, receive trained LoRA model
- Style transfer templates: pre-built style packages (noir, anime, photorealistic, retro)
- AI consistency scoring: automated face recognition and style matching to validate submissions against Standard Library before human review
- Standard Library versioning dashboard with migration guides

**Creator Marketplace**
- Secondary market for Clip NFTs (buy/sell merged and archived clips)
- Royalty enforcement: original creators earn 10% perpetual royalty on resales
- Price discovery for different shot types, genres, and film popularity levels

### H2 2027

**Cross-Film Standard Library Marketplace**
- Directors can license Standard Library assets from other films
- Revenue sharing for Standard Library creators across films
- "Remix" capability: adapt characters/styles from one film to another with permission
- Trending Standard Libraries and featured collections

**DeFi Integration**
- Clip NFT collateral: borrow against merged Clip NFTs (value derived from streaming revenue)
- $MONTAGE staking vaults with tiered rewards
- Revenue streaming derivatives (experimental): futures on segment replacement probability

**Mobile App**
- iOS and Android apps for film viewing, voting, and basic submission management
- Push notifications for voting periods, merge events, and revenue claims
- Optimized mobile video player with creator attribution overlay

**Festival Strategy**
- Submit top OpenMontage films to indie film festivals (Sundance Shorts, SXSW, Tribeca)
- Establish "Made with OpenMontage" branding for festival entries
- Document the collaborative creation process as part of festival submissions

**AI Tool Provider Integrations**
- API partnerships with Runway, Pika, Kling for direct-to-platform submission
- Embedded generation within Discord bot (generate + submit in one flow)
- Standardized output format for cross-tool compatibility

### Phase 3 KPIs

| KPI | Target |
|-----|--------|
| Discord members / registered users | 50,000+ |
| Active film projects | 100+ |
| Total creator earnings distributed | $1,000,000+ |
| $MONTAGE token holders | 10,000+ |
| Standard Library marketplace listings | 50+ |
| Films accepted to festivals | ≥ 1 |

---

## Phase 4: Scale & Legitimacy (2028+)

**Theme: Mainstream adoption and institutional credibility**

Phase 4 moves OpenMontage from a crypto-native community into broader recognition as a legitimate filmmaking platform.

### Film Festival Partnerships
- Establish dedicated OpenMontage award category at partner festivals
- Annual "OpenMontage Film Awards" — community-voted, on-chain, with $MONTAGE prize pool
- Red carpet premiere events for top community films

### Studio Licensing Exploration
- Licensing framework for studios to acquire distribution rights to OpenMontage films
- "Studio Edition" workflow: professional post-production pipeline (color grading, sound mixing) applied to community-created films
- Revenue sharing between community creators and studio distribution

### VR/AR Viewing Experiences
- Immersive viewing mode for VR headsets (Meta Quest, Apple Vision Pro)
- 360-degree AI-generated environments as viewing spaces
- Spatial audio integration

### Localization & Global Expansion
- Platform UI in 25+ languages
- Localized Standard Library templates for regional storytelling styles
- Regional community managers and Discord servers
- Subtitling and dubbing pipeline for cross-language film distribution

### Educational Partnerships
- Film school curriculum integration (USC, NYU, AFI, and international equivalents)
- Student filmmaker track with reduced fees and mentorship programs
- Academic research partnerships on AI-assisted collaborative creation

### Potential Funding
- Series A funding round or major DAO treasury raise
- Strategic investors from entertainment, gaming, and AI sectors
- Revenue sustainability target: platform self-funding from fees by end of 2028

---

## Technical Milestones Timeline

| Milestone | Target Date | Status |
|-----------|-------------|--------|
| Whitepaper & documentation | Q1 2026 | Complete |
| Testnet smart contracts | Q2 2026 | Upcoming |
| Discord bot v0.1 | Q2 2026 | Upcoming |
| Standard Library Spec v1.0 | Q2 2026 | Upcoming |
| Pilot film "Genesis" kickoff | Q2 2026 | Upcoming |
| Discord bot v0.5 | Q3 2026 | Planned |
| Pilot film complete | Q3 2026 | Planned |
| Security audit complete | Q3 2026 | Planned |
| Mainnet deployment (Base) | Q4 2026 | Planned |
| Web dashboard v1 | Q4 2026 | Planned |
| Discord bot v1.0 | Q4 2026 | Planned |
| $MONTAGE token launch (IDO) | Q1 2027 | Planned |
| Full DAO governance | Q2 2027 | Planned |
| Creator marketplace | Q2 2027 | Planned |
| Mobile app beta | Q3 2027 | Planned |
| Cross-film Standard Library marketplace | Q4 2027 | Planned |

---

## Community Milestones

| Milestone | Target |
|-----------|--------|
| Discord server launch | Q2 2026 |
| 1,000 Discord members | End Q2 2026 |
| 50 active creators (pilot) | Q3 2026 |
| Pilot film "Genesis" complete | Q3 2026 |
| 5,000 Discord members | End Q3 2026 |
| 500 registered creators | Q4 2026 |
| 10,000 community members | Q4 2026 |
| 20,000 community members | End Q4 2026 |
| $1M total creator earnings | H2 2027 |
| 50,000 community members | End 2027 |
| 100+ active film projects | End 2027 |
| 500+ completed films | 2029 |

---

## Key Risks and Mitigations

### Chicken-and-Egg Problem
**Risk**: No films without creators, no creators without films.
**Mitigation**: Produce the Pilot Film "Genesis" with a hand-selected group of 50 invite-only creators before opening publicly. This proves the concept, generates showcase content, and creates a community nucleus. Phase 1 contributors receive $MONTAGE test token airdrops, creating early loyalty.

### AI Tool API Costs
**Risk**: Video generation is expensive ($5–$20 per shot in API fees). Creators may not invest if returns are uncertain.
**Mitigation**: Start with community-provided compute (local GPU generation via ComfyUI/Stable Diffusion). Negotiate partnerships with Runway, Pika, and Kling for subsidized API access for OpenMontage creators. As platform revenue grows, explore a compute credit system funded by Film Ticket NFT sales. The minimum viable revenue analysis (Section 7 of Tokenomics) shows break-even at 2–4 months for even modestly successful films.

### Legal & IP Uncertainty
**Risk**: AI-generated content ownership remains legally murky. Copyright status of AI-assisted works is evolving across jurisdictions.
**Mitigation**: Build a contributor attestation system where creators attest to the originality and legality of their submissions. Partner with IP attorneys specializing in AI-generated content. Implement DMCA takedown procedures. Follow and adapt to evolving regulation (EU AI Act, US Copyright Office guidance). The decentralized, on-chain provenance trail provides stronger attribution than traditional filmmaking.

### Blockchain Scalability
**Risk**: If Base experiences congestion, high fees, or ecosystem issues, the platform could be disrupted.
**Mitigation**: Architecture designed for L2 portability — all smart contracts are standard Solidity on EVM. Arbitrum One serves as the designated fallback chain (Base uses OP Stack and Arbitrum uses Nitro, but both are EVM-compatible). Migration requires minimal contract changes. Multi-chain deployment is a Phase 3+ consideration if demand warrants it.

### Quality Control at Scale
**Risk**: As the platform opens publicly, submission quality may drop, overwhelming curators.
**Mitigation**: Automated quality floors (resolution, format, Standard Library compliance) filter out clearly non-compliant submissions before they reach human voters. AI consistency scoring provides a pre-screen layer. Reputation-weighted voting ensures experienced curators have more influence. Directors retain the ability to set minimum quality thresholds per film.

### Token Price Volatility
**Risk**: $MONTAGE price volatility could destabilize voting incentives and creator earnings.
**Mitigation**: Creator earnings are denominated in stablecoin (USDC) from film revenue — not in $MONTAGE. The token is used for governance and voting weight, not as a payment currency. Buyback & burn creates structural demand. Treasury reserve provides a stability buffer. Vesting schedules prevent early dump pressure (only ~8% circulating at TGE).

---

**Version**: 1.0
**Last Updated**: 2026-03-03
**Authors**: OpenMontage Core Team
