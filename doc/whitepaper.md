# OpenMontage Whitepaper

**Version 3.0 — Git for Movies: Decentralized AI Collaborative Filmmaking**

---

## Documentation Suite

This whitepaper provides the authoritative overview of OpenMontage. For detailed specifications, refer to the companion documents:

| Document | Contents |
|----------|---------|
| [Operations Manual](operations.md) | Discord platform, bot commands, director/creator/backer workflows, roadmap |
| [Technical Specification](technical-spec.md) | Smart contracts, storage architecture, API design, security |
| [Economics & Governance](economics-governance.md) | NFT mechanics, Menu Auction math, $MONTAGE token, governance framework |

---

## Executive Summary

OpenMontage is a decentralized platform that applies Git-style version control and open-source collaboration principles to film production. It enables global communities to collaboratively create professional-quality films using AI video generation tools, with automated, transparent revenue distribution enforced by smart contracts.

The platform addresses three structural failures that have prevented crowdsourced AI filmmaking from producing coherent, professional output: the **consistency problem** (independently generated clips look incompatible), the **incentive problem** (contributors have no economic stake in the outcome), and the **quality problem** (traditional films cannot improve after release, yet AI capabilities advance rapidly).

OpenMontage’s solution rests on four mutually reinforcing innovations:

1. **Standard Library** — A mandatory per-film dependency package (character LoRA models, style guides, prompt templates, ControlNets, voice models) that enforces visual and audio consistency across all contributors, the same way npm packages enforce API compatibility in software projects.

2. **Pull Request Workflow** — Scripts are decomposed into discrete Shots, each an independently claimable unit of work. Contributors generate video segments, submit them as Pull Requests, and earn proportional revenue based on screen time in the final film.

3. **Shot Market Menu Auctions** — Each Shot becomes a sealed menu auction where community backers submit valuation menus expressing preferences across ALL competing clips. The clip with the highest aggregate quadratic-weighted valuation (`SelectionWeight = √(CreatorBond) + Σ√(valuation_i)`) wins. Adapted from the Bernheim-Whinston (1986) menu auction framework, this produces efficient outcomes under the quadratic-weighted objective and eliminates the “backing a loser” penalty — backers pay only their stated valuation for whichever clip wins.

4. **NFT-Driven Economics** — Clip NFTs establish immutable authorship; Film NFTs represent copyright and director revenue rights; Film Ticket NFTs enable crowdfunding. Revenue is distributed automatically via smart contracts at a **75/10/5/10 split**: creators receive 75% proportional to screen time, Film NFT holders receive 10%, Standard Library contributors receive 5%, and backers of merged clips receive 10%.

**Key metrics at a glance:**

| Parameter | Value |
|-----------|-------|
| Revenue split | 75% creators / 10% Film NFT holder / 5% Standard Library / 10% backers |
| Creator Bond (submission) | 50 USDC per clip |
| Film Creation Bond | Minimum 500 USDC (50% refundable upon completion) |
| Challenge Bond | 100 USDC (2x Creator Bond) |
| Losing Creator refund | 75% of Creator Bond returned |
| Competition Tax | 5% of losing Creator Bonds → Film Treasury |
| Reward Pool | 20% of losing Creator Bonds → split between winning creator (50%) and winning backers (50%) |
| Primary network | Base (Coinbase L2) |
| Governance token | $MONTAGE — platform governance only; Shot Market menu auctions use USDC |

OpenMontage is not a film financing DAO, an NFT collectible platform, or an AI video tool. It is the **production infrastructure layer** for collaborative AI filmmaking — the Git repository, npm registry, and revenue distribution engine that makes professional crowdsourced films economically viable for the first time.

---

## 1. Problem Statement

### 1.1 The AI Video Revolution and Its Limits

AI video generation has crossed a meaningful quality threshold. Tools such as Sora, Runway Gen-3, Pika, Kling, and open-source pipelines built on Stable Diffusion Video can now produce cinematic-quality footage from text prompts in minutes. For the first time in history, an individual creator without access to cameras, crews, or production budgets can generate footage that rivals professional output.

The logical next step — crowdsourced, collaborative AI filmmaking — has not materialized. Despite the tools existing, no community has successfully produced a complete, coherent, professional-quality film through distributed AI generation. The reason is structural, not creative. Three fundamental barriers block the path from “AI can generate impressive clips” to “communities can produce complete films.”

### 1.2 Barrier One: The Consistency Problem

A film is a coherent visual narrative. Its protagonist must look the same in every scene. The lighting and color palette must feel unified across 100 different shots. The audio — character voices, ambient sound design — must match across contributors who have never coordinated directly.

When ten different creators independently generate clips for the same film using current AI tools, they produce ten incompatible versions of every element. Character faces drift between scenes. Color grading varies from naturalistic to high-contrast without motivation. Voice synthesis models produce different tonal qualities for the same character.

This is not a problem of creator skill. It is a problem of missing infrastructure. Software development faced an analogous challenge before package managers: libraries written by different developers could not be easily composed because there were no enforced interfaces, versioning, or dependency standards. npm, pip, and Cargo solved this for software. No equivalent exists for AI video production.

**The result**: Collaborative AI filmmaking produces footage libraries, not films.

### 1.3 Barrier Two: The Incentive Problem

AI video generation is not free. A high-quality 10-second clip may require $5–$20 in API costs, hours of iterative prompt engineering, and non-trivial technical expertise. A contributor who invests this effort in a crowdsourced project has no mechanism to recover their costs or earn a return proportional to their contribution’s value to the final product.

Existing platforms that have attempted crowdsourced content creation — YouTube collaborations, open video projects, fan film initiatives — rely entirely on intrinsic motivation: passion for the project, desire for recognition, community belonging. These motivations are real but insufficient at scale. Without economic returns, participation is concentrated among the most devoted fans and burns out quickly. Projects rarely reach completion.

**The result**: Most collaborative film projects stall before completion because contributors cannot justify the sustained effort without economic incentive.

### 1.4 Barrier Three: The Quality Problem

Traditional film production follows an “append-only” model. Once a film is released, its content is fixed. This made sense when reshooting footage was prohibitively expensive and time-consuming. It is a poor fit for an AI-native production model in which regenerating a scene at higher quality — using a newly released AI model — takes hours rather than months and costs dollars rather than millions.

A collaborative film produced in early 2026 using current AI tools will look dated by mid-2027 as generation models improve by an order of magnitude. There is no mechanism in traditional filmmaking, or in any existing collaborative platform, to systematically improve individual segments of a completed work as better tools become available.

**The result**: AI films are locked to the quality ceiling of the tools available at production time, even as those tools advance rapidly.

---

## 2. The Solution: Git for Movies

### 2.1 Core Philosophy

OpenMontage’s insight is that these three barriers are not unique to filmmaking — they are well-understood problems in software development, and software development has solved them.

- The consistency problem is solved by **dependency management**: enforced interfaces and shared libraries that ensure independently written modules work together.
- The incentive problem is solved by **economic participation**: ownership stakes that align contributor effort with project success.
- The quality problem is solved by **version control**: the ability to replace any component with a better version, with community consensus determining what “better” means.

OpenMontage applies these principles to filmmaking through a system of four integrated innovations. Each addresses one barrier directly; together, they create a self-reinforcing platform.

### 2.2 Innovation One: The Standard Library (Solving Consistency)

Every film on OpenMontage must publish an official **Standard Library** before accepting contributor submissions. The Standard Library is a mandatory dependency package — analogous to a project’s `package.json` and its locked dependencies — that every contributor must use when generating their clips.

A Standard Library contains:

**Character LoRA Models** — Pre-trained Stable Diffusion LoRA models for every primary character. When contributors load the protagonist’s LoRA into their generation pipeline, the resulting character retains consistent facial structure, hair, and clothing across all submissions, regardless of which creator generated it or which AI base model they used.

**Art Direction Assets** — Color palette definitions, lighting reference images, cinematography notes (lens characteristics, camera angle conventions, depth-of-field guidance). These translate the director’s visual intent into parameters that AI generation tools can implement consistently.

**Prompt Templates** — Standardized prompt structures that contributors follow when describing their scenes. Templates encode the style tokens, negative prompts, and structural conventions that produce footage matching the film’s visual language.

**ControlNet Specifications** — 3D skeletal rigs or pose control networks for character animation. These ensure that character movement and posture are consistent across the film’s action sequences.

**Voice Clone Models** — Authorized text-to-speech models for each character, trained on reference audio. Dialogue scenes across the film share consistent vocal timbre and delivery style regardless of which creator handled that shot.

The Standard Library is versioned using Semantic Versioning (SemVer). Minor updates — adding a supporting character model, refining a prompt template — can be made by the director unilaterally. Major breaking changes (retraining core character LoRAs, fundamental art style shifts) require 70% approval from active contributors (defined as any wallet with at least one submission in the last 30 days) and a two-week migration period.

The Standard Library transforms collaborative AI filmmaking from a coordination problem into a technical constraint. Contributors do not need to coordinate their visual choices — they simply load the same dependencies.

### 2.3 Innovation Two: The Pull Request Workflow (Modular Creation)

OpenMontage decomposes screenplays into **Shots** — discrete, independently producible units of the film, analogous to functions in software code. Each Shot has a defined scope: a scene excerpt, a specific camera angle, defined characters, a duration target.

The contributor workflow mirrors a Git pull request:

1. **Claim** — A creator browses available Shots in a film repository and claims one for a 72-hour exclusive work window.
2. **Pull Dependencies** — The creator downloads the film’s Standard Library assets via the `/standard-library` command.
3. **Generate** — The creator uses their preferred AI tools (Runway, ComfyUI, Sora, Kling, or any compatible pipeline), loading Standard Library assets into their generation workflow.
4. **Submit** — The creator uploads the generated video segment with a 50 USDC Creator Bond, paying to enter the shot’s competition. The submission is minted as a Clip NFT recording authorship, the Standard Library version used, and the video’s IPFS hash.
5. **Compete** — Multiple submissions for the same Shot enter a Shot Market menu auction, where community backers submit sealed valuation menus expressing preferences across all competing clips.
6. **Merge or Archive** — The winning clip is merged into the film’s Main Branch and begins earning streaming revenue proportionally to its screen time. Losing clips are archived; their creators receive a 75% bond refund.

This workflow produces a film manifest (`film.json`) — a Film NFT that dynamically references the currently-merged Clip NFT for each Shot, updating as menu auctions resolve and as replacement challenges succeed.

### 2.4 Innovation Three: Shot Market Menu Auctions (Quality Selection)

The mechanism that selects the winning clip for each Shot is OpenMontage’s most significant departure from conventional platform design. Rather than collecting token votes or requiring backers to commit to a single clip, Shot Market menu auctions let backers express valuations across ALL competing clips via sealed menus. Adapted from the Bernheim-Whinston (1986) menu auction framework, this produces efficient outcomes under the quadratic-weighted objective in truthful equilibrium.

Each Shot Market follows a **sealed menu commit-reveal** protocol:

**Submission Phase (14 days)**: Creators submit clips and pay the 50 USDC Creator Bond. Multiple submissions per Shot are encouraged; competition improves outcomes. The auction phase opens once at least two submissions exist.

**Commit Phase (5 days)**: Backers construct a **valuation menu** — a set of valuations, one per clip — and generate a commitment hash: `keccak256(clipId_1, v_1, clipId_2, v_2, ..., salt)`. They deposit USDC equal to their highest valuation (max(v_i)). During the commit phase, only commitment counts per shot are publicly visible — valuations, clip preferences, and backer identities remain hidden until reveal. This eliminates bandwagoning and herding. Every backer must independently assess all clips.

**Reveal Phase (2 days)**: Backers reveal their full valuation menus. The contract verifies each reveal matches the stored hash and that max(valuations) <= deposit. Backers who fail to reveal forfeit their deposit to the reward pool.

**Resolution**: The clip with the highest **aggregate quadratic SelectionWeight** wins:

```
SelectionWeight(clip_c) = √(CreatorBond_c) + Σ_backers √(v_i_c)
```

The square root transformation ensures that broad community support outweighs concentrated capital:

| Scenario | Total Valuation | Quadratic SelectionWeight |
|----------|----------------|---------------------------|
| 1 whale values Clip A at 10,000 USDC | 10,000 | √10,000 = **100** |
| 100 people each value Clip B at 100 USDC | 10,000 | 100 × √100 = **1,000** |
| 25 people each value Clip C at 400 USDC | 10,000 | 25 × √400 = **500** |

With identical total valuation, Clip B (100 backers) has 10x the weight of Clip A (1 whale). The clip with the broadest genuine community support wins.

**Key advantage**: Because backers value ALL clips (not just one), they pay only their stated valuation for whichever clip wins. If they valued the winner at zero, they pay nothing. There is no punitive “backing a loser” penalty — excess deposits are refunded in full. This encourages participation and honest preference revelation.

After resolution, funds are distributed deterministically:

- **Backer payments**: Each backer pays their stated valuation for the winning clip (v_i_winner). Excess deposit (deposit - v_i_winner) is refunded immediately. The payment is recorded as their streaming revenue stake.
- **Competition Tax**: 5% of losing Creator Bonds flow to the Film Treasury.
- **Loser Creator Refund**: 75% of losing Creator Bonds are returned to losing creators.
- **Reward Pool**: 20% of losing Creator Bonds are split between the winning creator (50%) and winning backers (50%, proportional to √(v_i_winner)).
- **Winning Creator Bond**: Returned in full.

This creates aligned incentives across all participants. Creators compete on quality because winning returns their bond plus a reward pool share plus ongoing streaming revenue. Backers submit truthful valuations because the menu auction’s truthful equilibrium makes honest menus the dominant strategy — overstating wastes capital, understating reduces streaming revenue stake.

### 2.5 Innovation Four: NFT-Driven Economics (Fair Distribution)

Every economic relationship in OpenMontage is encoded in transferable, auditable, on-chain assets:

**Clip NFTs (ERC-721)** represent authorship of individual video segments. Each Clip NFT records the creator’s wallet, the video’s IPFS hash, the Standard Library version used, and the clip’s current status (merged or archived). Merged clips earn streaming revenue continuously while they remain in the Main Branch. Archived clips retain their NFT provenance — historical record of contribution — but do not earn ongoing revenue.

**Film NFTs (ERC-721)** represent the film as a composite creative work. Minted when a director pays the Film Creation Bond, the Film NFT carries copyright claim, 10% of ongoing streaming revenue, and governance authority over the film’s parameters. It is fully transferable — a director can sell their film and its revenue stream to another party.

**Film Ticket NFTs** are optional crowdfunding instruments. Directors can issue limited-supply tickets that grant holders lifetime streaming access and a revenue share drawn from the creator pool (reducing it from 75% to 65%, with the 10% difference flowing to Ticket NFT holders). The Film NFT holder’s 10% share is unaffected by Film Ticket NFTs.

Revenue distribution across all these assets is automated by the Treasury smart contract, requiring no manual intervention:

```
Standard Split (no Film Ticket NFTs):
  Clip Creators:              75%  (proportional to screen time)
  Film NFT Holder (Director): 10%
  Standard Library Contrib.:   5%
  Clip Backers Pool:          10%  (proportional to stake in merged clips)

Modified Split (with Film Ticket NFTs):
  Clip Creators:              65%  (reduced by 10pp to fund Ticket holders)
  Film NFT Holder (Director): 10%  (unchanged)
  Standard Library Contrib.:   5%  (unchanged)
  Clip Backers Pool:          10%  (unchanged)
  Film Ticket NFT Holders:    10%
```

---

## 3. Participant Roles and Experience

### 3.1 Directors and Film Initiators

Directors are the architects of OpenMontage films. They define the creative vision, build the infrastructure that makes collaboration possible, and earn 10% of all streaming revenue as Film NFT holders.

**What Directors Do**:
- Decompose a screenplay into Shots with clear descriptions, duration targets, and technical requirements
- Build the Standard Library — training character LoRA models, defining art direction, creating prompt templates, recording voice clone models
- Pay the Film Creation Bond (minimum 500 USDC; recommended 50 USDC per planned shot). 50% seeds the Film Treasury for infrastructure costs; 50% is held in Completion Escrow and refunded when 80% of shots have a merged clip
- Set competition parameters: submission window duration, minimum quality thresholds, Standard Library version requirements
- Optionally issue Film Ticket NFTs to crowdfund production capital before shots are filled

**Director Constraints**: Directors cannot submit valuation menus in their own film’s Shot Markets. This conflict-of-interest prohibition is enforced at the smart contract level — the `ShotMarket.commitBacking()` function rejects transactions from the Film NFT holder’s address.

**Director Economics**: A completed film earning $50,000/month in streaming revenue generates $4,850/month for the Film NFT holder after the 3% platform fee, indefinitely. Directors who build high-quality Standard Libraries attract better contributors, which produces higher-quality films, which attracts larger audiences — a reinforcing loop.

### 3.2 AIGC Creators

Creators are the contributors who generate video segments. They are the primary revenue earners on the platform, receiving 75% of all streaming revenue proportional to their merged clip’s screen time.

**What Creators Do**:
- Browse film repositories to find shots that match their skills and interests
- Download the film’s Standard Library assets via the platform
- Generate video segments using AI tools, loading character LoRAs, following prompt templates, applying ControlNet constraints
- Submit clips with a 50 USDC Creator Bond, entering them into Shot Market menu auctions
- Optionally improve and resubmit if their clip is challenged by a successor

**Creator Economics**: A creator with a 2-minute merged clip in a film earning $50,000/month earns approximately $808/month in passive streaming revenue — while the clip remains in the Main Branch. The Creator Bond is returned in full if the clip wins; 75% is refunded if it loses, meaning net downside is 12.50 USDC plus compute costs. A creator who wins one in three competitions is profitable from streaming revenue alone within the first month.

Creators can accumulate a portfolio of merged clips across multiple films, compounding their passive income. A creator with three merged clips in a $50,000/month film earns approximately $600+/month from that film alone.

**Replacement Risk**: Merged clips can be challenged by superior submissions. Challengers pay 100 USDC (double the Creator Bond), and the incumbent clip starts with a synthetic SelectionWeight advantage equal to 50% of its original winning weight. Backers submit binary valuation menus (incumbent vs. challenger). Incumbents are notified and given a 7-day response window to improve their submission at no additional cost. A 21-day cooldown prevents harassment through repeated challenges. **Challenge outcomes**: If the challenger wins, the incumbent clip is archived and the challenger’s clip replaces it in the Main Branch. If the incumbent wins, the challenger receives a 75% refund of their Challenge Bond (losing 25 USDC), and the incumbent earns a defense bonus from the Reward Pool.

### 3.3 Backers

Backers are quality curators. They submit sealed valuation menus expressing preferences across ALL competing clips in each Shot Market auction, earning returns from streaming revenue proportional to their valuations for winning clips.

**What Backers Do**:
- Evaluate all submitted clips for a given Shot during the submission phase
- During the commit phase, construct a valuation menu covering all clips and deposit USDC equal to their highest valuation (10-5,000 USDC range)
- During the reveal phase, reveal their full valuation menu to the contract
- Claim competition payouts and ongoing streaming revenue after auction resolution

**Backer Economics**: Backers pay only their stated valuation for the winning clip — excess deposits are refunded in full. Their payment is recorded as a streaming revenue stake: 10% of the Film Treasury’s revenue flows to backers of currently-merged clips, proportional to their valuation-based stake. Backers also share in the Reward Pool (50% of 20% of losing Creator Bonds, proportional to sqrt(valuation) for the winning clip). A backer with winning positions across 20 clips in five films earns meaningful passive income that compounds over time.

There is no punitive “backing a loser” penalty. Backers who valued the winning clip at zero pay nothing and receive their full deposit back. The expected value of participation is positive for backers who can accurately assess clip quality — truthful valuations are the dominant strategy in the menu auction’s equilibrium.

**Anti-Gaming Protections**: The sealed menu commit-reveal protocol ensures backers cannot see others’ valuations during the commit phase, eliminating bandwagoning. The 10 USDC minimum deposit prevents near-zero-cost Sybil attacks. The 5,000 USDC maximum deposit caps single-address influence. The 7-day minimum wallet age requirement adds friction for freshly created wallets. Quadratic weighting on valuations ensures crowd consensus beats capital concentration.

### 3.4 Audience

Audiences experience OpenMontage films as dynamically assembled streaming content. The platform’s video assembly engine reads the current film manifest — an ordered list of merged Clip NFTs — and stitches segments into a seamless stream with fade transitions. Creator attribution is displayed throughout.

**Audience Participation Options**:
- Stream films via the web platform or mobile app
- Purchase Film Ticket NFTs for lifetime access and fractional revenue participation
- Observe Shot Market auction results to understand which clips were selected and why
- As the platform matures, use the dispute resolution system to flag technically invalid market outcomes

The audience represents OpenMontage’s revenue foundation. Every stream, advertisement impression, and licensing deal flows into the Treasury smart contract and distributes automatically to creators, backers, directors, and Standard Library contributors. More audience engagement means more revenue for all contributors — aligning every participant’s incentive toward film quality.

---

## 4. Economic Model Overview

### 4.1 NFT Asset System

OpenMontage’s economic model is built on three layers of NFT assets:

**Clip NFTs** are the atomic unit of contribution. Each video segment submitted to a Shot Market is minted as an ERC-721 NFT with immutable authorship metadata, the video’s IPFS hash, the Standard Library version used, and status tracking (merged or archived). Merged Clip NFTs earn streaming revenue continuously. Creators retain a 10% perpetual royalty on secondary market sales of their Clip NFTs. The NFT marketplace enables price discovery for high-value clips in successful films.

**Film NFTs** are the composite creative asset. The Film NFT is a dynamic pointer array — a manifest that references the currently-merged Clip NFT for each Shot, updating as menu auctions resolve. Beyond the manifest function, the Film NFT carries copyright claim over the composite work, 10% of all streaming revenue, and governance authority over film parameters. Full transferability means directors can realize the value of a successful film by selling the Film NFT along with its revenue stream.

**Film Ticket NFTs** are crowdfunding instruments. By issuing a limited supply of tickets (e.g., 10,000 tickets at 50 USDC each), directors can raise production capital before or during the filmmaking process. Ticket holders receive lifetime streaming access and 10% of film revenue drawn from the creator pool. This mechanism enables community-funded production at a scale previously requiring traditional financing.

### 4.2 Shot Market Menu Auctions: The Core Mechanism

The Shot Market Menu Auction is OpenMontage’s central quality selection mechanism. Adapted from the Bernheim-Whinston (1986) menu auction framework, it replaces single-clip backing with sealed valuation menus, producing efficient outcomes under the quadratic-weighted objective in truthful equilibrium.

The complete auction lifecycle:

```
SUBMISSION PHASE (14 days)
  Creators submit clips + 50 USDC Creator Bond
  Clips minted as Clip NFTs, enter competition
  Auction phase opens once >= 2 submissions exist
          |
          v
COMMIT PHASE (5 days)
  Backers submit: keccak256(clipId_1, v_1, ..., clipId_N, v_N, salt)
  Deposit = max(v_i) locked in contract
  Only commitment counts visible — valuations and preferences hidden until reveal
          |
          v
REVEAL PHASE (2 days)
  Backers reveal: full valuation menu + salt
  Contract verifies hash match and max(valuations) <= deposit
  Non-reveals forfeit deposit to reward pool
          |
          v
RESOLUTION
  SelectionWeight(clip_c) = sqrt(CreatorBond_c) + sum_backers sqrt(v_i_c)
  Highest weight wins
  Payouts distributed deterministically
```

The payout structure maintains full conservation — every USDC deposited is accounted for:

```
Backer settlements:
  Each backer pays v_i_winner (their valuation for the winning clip)
  Excess refund: deposit - v_i_winner (returned immediately)
  Payment recorded as streaming revenue stake

From losing Creator Bonds (CB_l):
  Competition Tax:  5% * CB_l   ->  Film Treasury
  Loser Refunds:   75% * CB_l   ->  returned to losing creators
  Reward Pool:     20% * CB_l   ->  50% to winning creator, 50% to winning backers

Winning Creator Bond: returned in full

Total Out = all deposits accounted for (funds conserved exactly)
```

### 4.3 Revenue Distribution: The 75/10/5/10 Split

All revenue generated by a film — streaming fees, advertising, licensing, Film Ticket NFT sales, Shot Market competition taxes — flows into the Treasury smart contract and is distributed automatically on every settlement cycle. Backer valuations for winning clips are recorded as their streaming revenue stakes.

The standard distribution split:

| Stakeholder | Share | Distribution Logic |
|-------------|-------|--------------------|
| Clip Creators | 75% | Proportional to screen time: `Creator_Earnings = CreatorPool * (Clip_Duration / Total_Film_Duration)` |
| Film NFT Holder | 10% | Fixed share to the current Film NFT holder (the director or whoever holds it) |
| Standard Library Contributors | 5% | Split among contributors who built the film’s Standard Library assets |
| Clip Backers Pool | 10% | Proportional to valuation-based stake in merged clips, weighted by screen time |

When Film Ticket NFTs are issued, the creator pool adjusts from 75% to 65%, with the 10% difference flowing to Ticket NFT holders. All other shares remain unchanged.

**Illustrative Example** — 90-minute film, $50,000/month revenue, 3% platform fee:

```
Treasury Revenue = $50,000 * 0.97 = $48,500/month

Film NFT Holder (Director):      $48,500 * 10%  = $4,850/month
Standard Library Contributors:   $48,500 * 5%   = $2,425/month
Clip Backers Pool:                $48,500 * 10%  = $4,850/month
Clip Creator Pool:                $48,500 * 75%  = $36,375/month

  Creator A (3 clips, 5 min total):   $36,375 * (300/5400) = $2,021/month
  Creator B (1 clip, 30 seconds):     $36,375 * (30/5400)  = $202/month
```

Earnings are passive and continuous. A merged clip earns every month it remains in the Main Branch, indefinitely.

### 4.4 The $MONTAGE Token: Platform Governance

$MONTAGE is OpenMontage’s platform governance token. It plays no role in film-level clip selection — that is handled entirely by USDC-denominated menu auctions. This separation is by design: film quality decisions should be driven by honest valuation menus from the community, not by accumulated token holdings.

**$MONTAGE use cases:**

- **Platform governance**: Token holders vote on platform fee changes, contract upgrades, treasury allocation, grant programs, and feature prioritization. Proposals require 4% quorum of circulating supply and a simple majority; contract upgrades require 67% supermajority.
- **Fee reduction**: Staking $MONTAGE reduces the effective Competition Tax rate from 5% down to a minimum of 2%, proportional to stake size.
- **Value accrual**: 80% of platform fees are used to buy back and burn $MONTAGE on the open market each quarter, creating deflationary pressure as platform revenue grows.
- **Access**: Priority access to premium Standard Library tools and new platform features.

**Total supply**: 1,000,000,000 $MONTAGE. Initial circulating supply at TGE (Q1 2027): approximately 8%. Allocation: 30% community rewards (5-year vest), 20% ecosystem grants (3-year vest), 15% team and advisors (4-year vest, 1-year cliff), 15% treasury reserve, 10% public sale, 7% seed and strategic, 3% liquidity provision.

---

## 5. Technical Architecture Overview

### 5.1 System Layers

OpenMontage is organized into four layers, each with clearly defined responsibilities:

```
APPLICATION LAYER
  Frontend (web, mobile, Discord bot)
  Film browsing, submission management,
  Menu Auction interface, wallet integration
          |
          v
API LAYER
  User authentication, video assembly engine
  Metadata indexing, search and discovery
  Revenue dashboard, Standard Library distribution
          |
     |         |
     v         v
BLOCKCHAIN      STORAGE
LAYER           LAYER
Smart contracts  IPFS / Arweave (video files, permanent)
NFT minting      Cloud CDN (delivery, adaptive bitrate)
Menu Auctions   Git LFS (Standard Library assets, LoRAs)
Treasury         Metadata indexing (The Graph subgraph)
```

### 5.2 Blockchain Layer: Base L2

OpenMontage deploys on **Base**, Coinbase’s Ethereum L2 rollup (OP Stack), as the primary chain. The rationale:

- **Sub-cent transaction costs** — NFT minting, menu auction commits, reveals, and payouts are economically viable at Base’s fee level. On Ethereum mainnet, these operations would be prohibitively expensive.
- **Creator ecosystem alignment** — Base hosts the largest on-chain creator community, including Zora and other creator-native platforms. OpenMontage’s target contributors already exist here.
- **Coinbase fiat onramp** — Filmmakers and audiences unfamiliar with crypto can purchase USDC and interact with the platform without navigating complex exchange flows.
- **Ethereum security** — Base inherits Ethereum’s security guarantees as a rollup; all state is periodically posted to Ethereum mainnet.
- **EVM compatibility** — Standard Solidity tooling (Hardhat, Foundry, OpenZeppelin) and contract portability.

**Fallback**: Arbitrum One is designated as the secondary deployment target. Both Base and Arbitrum are EVM-compatible rollups, so contract migration requires minimal changes if Base conditions change.

**Core smart contracts:**

| Contract | Responsibility |
|----------|---------------|
| `ClipNFT.sol` | ERC-721 for segment ownership; backer registry; Standard Library compliance validation |
| `FilmNFT.sol` | Film copyright, revenue rights, Film Creation Bond management, dynamic manifest |
| `Treasury.sol` | Automated 75/10/5/10 revenue distribution; Competition Tax collection; buyback |
| `ShotMarket.sol` | Submission, sealed menu commit-reveal, quadratic SelectionWeight resolution, payouts |
| `StandardLibrary.sol` | Asset versioning, dependency manifest, compliance verification |

All contracts deploy behind OpenZeppelin TransparentProxy in Phase 1 to allow bug fixes during early operation. A 3-of-5 Gnosis Safe multi-sig (founding team, hardware wallets, geographically distributed) controls upgrades and emergency pause. Proxy admin authority transfers to the DAO Governor contract in Phase 3.

Security: All core contracts undergo two independent audits (OpenZeppelin for ERC-721 compliance and access control; Trail of Bits for economic exploits and MEV risks) before mainnet deployment. Treasury.sol implements checks-effects-interactions pattern and OpenZeppelin ReentrancyGuard. Oracle-dependent revenue distributions use Chainlink Data Feeds with a 24-hour staleness check.

### 5.3 Storage Layer

**Video files** are stored on IPFS and Arweave for permanent, decentralized availability. The video’s content hash is recorded in the Clip NFT metadata, enabling any party to independently verify file integrity. Regional CDN edge nodes (Cloudflare, AWS CloudFront) cache popular content for low-latency streaming.

**Standard Library assets** — LoRA models (.safetensors), ControlNet configurations, voice clone models — are distributed via Git LFS and CDN. File sizes for LoRA models range from 50MB to 300MB; CDN distribution is essential for global creator access.

**Metadata indexing** is handled by a custom The Graph subgraph that indexes all on-chain events: clip submissions, market resolutions, revenue distributions, and Film NFT transfers. This powers the web dashboard and API layer without requiring full-node queries.

**Real-time assembly**: The video playback engine reads the current Film NFT manifest, fetches each segment’s IPFS hash from the corresponding Clip NFT, and assembles an adaptive bitrate stream (HLS/DASH) with seamless transitions between shots.

### 5.4 Discord-First Strategy

Rather than building a full web application before validating the core product loop, OpenMontage launches Phase 1 as a **Discord-first platform**. Discord is where AI creator communities and crypto-native communities already operate. Meeting users in their existing environment eliminates onboarding friction and enables a usable product to ship in weeks rather than months.

The Discord bot (MontageBot) translates user commands into smart contract interactions:

| Command | Action |
|---------|--------|
| `/create-film` | Director initiates film, pays Film Creation Bond, Film NFT minted |
| `/claim-shot film:[id] shot:[id]` | Creator claims a shot for 72-hour exclusive work window |
| `/submit film:[id] shot:[id] url:[url]` | Uploads video to IPFS, mints Clip NFT, pays 50 USDC Creator Bond |
| `/back-shot film:[id] shot:[id] deposit:[USDC] rank:[clipA,clipB,...]` | Submits sealed valuation menu (rank-based shortcut), locks deposit |
| `/back-shot-menu film:[id] shot:[id] deposit:[USDC] values:[clipA:v1,...]` | Submits sealed valuation menu (explicit valuations) |
| `/reveal-backing film:[id] shot:[id] salt:[secret]` | Reveals full valuation menu during reveal phase |
| `/claim-market film:[id] shot:[id]` | Claims winner reward or loser refund after resolution |
| `/standard-library film:[id]` | Downloads Standard Library assets from CDN |
| `/my-earnings` | Displays streaming revenue and competition rewards |

The transition from Discord-first to a full web application follows the roadmap phases. By Phase 3, Discord transitions to a notification and community role while the standalone web app handles all workflows.

---

## 6. Governance Framework Overview

### 6.1 Two-Layer Governance Separation

OpenMontage enforces a strict separation between film-level governance and platform-level governance. These are fundamentally different decisions requiring different participants, different mechanisms, and different timelines.

**Film-Level Governance — Menu Auction Shot Markets**

Clip selection for each Shot is determined by menu auctions, not by any voting process. Backers submit sealed valuation menus expressing preferences across all competing clips; the clip with the highest aggregate quadratic-weighted SelectionWeight wins. Directors cannot override auction outcomes; the contract result is final (subject to a structured dispute resolution process involving a randomly selected jury of high-reputation backers).

This design means film quality decisions are purely meritocratic — the clip that best convinces the broadest community of backers wins, independent of who produced it, who the director favors, or how much capital any single actor controls. The menu auction’s truthful equilibrium guarantees that honest valuations are the dominant strategy.

**Platform-Level Governance — $MONTAGE Token Voting**

Platform governance controls the OpenMontage protocol itself: fee structures, contract upgrades, treasury allocation, feature prioritization, grant programs. $MONTAGE token holders vote on these decisions through a structured proposal process: 7-day community discussion, off-chain signal vote on Snapshot, on-chain vote with 4% quorum and simple majority (67% supermajority for contract upgrades and large treasury allocations).

The separation is enforced structurally. A $MONTAGE whale cannot influence which clip wins a menu auction. An individual film contributor cannot unilaterally change platform economics. These concerns are handled by different mechanisms operating independently.

### 6.2 Progressive Decentralization

OpenMontage does not launch as a fully decentralized DAO. Early-stage protocols require the ability to respond quickly to bugs, exploits, and market feedback. Authority decentralizes in deliberate phases:

**Phase 1 (Q2-Q3 2026)**: 3-of-5 founding team multi-sig controls platform operations. Menu auctions operate autonomously for film-level decisions. Community advisory votes via Discord polls are non-binding. All multi-sig actions are logged publicly.

**Phase 2 (Q4 2026)**: Platform governance proposals via $MONTAGE token-weighted voting for fee structures and feature priorities. Menu auctions handle all film-level clip selection independently. Multi-sig retained for emergency actions.

**Phase 3 (2027)**: On-chain Governor contract for all platform governance. Multi-sig retained only for emergency pause, with authority transferring to DAO via fast-track voting mechanism. Film-level governance via menu auctions continues unchanged.

**Phase 4 (2028+)**: Constitutional governance with ratified charter, on-chain dispute courts, elected governance council, and formal separation of executive, legislative, and judicial functions. Constitutional amendments require 75% supermajority and 30-day deliberation.

**Standard Library Governance** occupies its own layer within film governance. Minor version updates (adding assets, fixing bugs) are at the director’s discretion. Major breaking changes (retraining character LoRAs, fundamental style shifts) require 70% approval from active contributors (defined as any wallet with at least one submission in the last 30 days) and a two-week migration period with backward compatibility guarantees.

---

## 7. Use Cases

**Independent AI Films** — Original screenplays produced entirely through global contributor communities, earning streaming revenue indefinitely. The pilot film “Genesis” (20-minute sci-fi, 100 shots) serves as the proof of concept in Phase 1.

**Fan and Community Films** — Community-driven adaptations of public domain works (Shakespeare plays, mythology, historical events) with guaranteed visual consistency. Standard Library assets give such projects a professional, unified visual identity that fan films historically have lacked.

**Educational and Documentary Content** — Universities, edtech platforms, and documentary producers can crowdsource science explainer videos, historical recreations, or training materials shot-by-shot. Menu auctions ensure quality curation without editorial bottlenecks.

**Music Videos** — Musicians with existing audiences can crowdfund visual production via Film Ticket NFTs, then open menu auctions for the video’s individual segments. The 75% creator revenue share means AI video artists earn proportionally to their contribution.

**Brand and Corporate Content** — Organizations can commission modular video production through OpenMontage’s contributor marketplace, paying only for merged segments and retaining quality control through menu auction curation. The on-chain provenance trail provides auditable attribution for content compliance.

---

## 8. Competitive Positioning

OpenMontage occupies a unique intersection that no existing platform addresses: decentralized, crowdsourced AI filmmaking with enforced visual consistency and automated revenue sharing.

| Platform | Category | Consistency Solution | Revenue to Creators | Iterative Improvement |
|----------|----------|---------------------|--------------------|-----------------------|
| **OpenMontage** | Crowdsourced AI Filmmaking | Mandatory Standard Library (LoRAs, style guides, ControlNets) | 75% streaming revenue, proportional to screen time | Menu auctions enable efficient clip selection (quadratic-weighted); films improve continuously |
| Film.io / Decentralized Pictures | Film Financing DAO | None — funds traditional production | Not direct; grant-based | No — traditional fixed releases |
| Runway / Sora / Pika | AI Video Generation Tools | Per-project style controls (no cross-creator enforcement) | None — tool vendors | No |
| YouTube Collaborations | Ad-hoc creator collab | Manual coordination, often ignored | Ad revenue (platform takes 45%) | No formal mechanism |
| Zora / Lens | Creator social platforms | None | Tips, trading fees | No |
| Story Protocol | IP Infrastructure | None — infrastructure layer | Automated royalties (asset-level) | No |

**Three structural advantages that cannot be easily replicated:**

1. **The only platform with a mandatory visual consistency system for AI-generated collaborative content.** The Standard Library concept is technically novel and becomes more defensible as each film’s library of LoRAs, ControlNets, and style assets accumulates into a unique creative artifact.

2. **The only streaming revenue model proportional to per-shot contribution.** Existing platforms distribute revenue at the film level to a small number of rights holders. OpenMontage distributes to every contributing creator proportionally to their screen time — enabling long-tail creator economics at scale.

3. **The only version-controlled film format.** Films on OpenMontage can improve over time as AI tools advance and new creators submit superior versions of existing shots. This is a structural advantage in an industry where AI model quality is improving by an order of magnitude per year.

---

## 9. Risk Analysis and Mitigations

### 9.1 Technical Risks

**Visual Consistency at Scale** — Standard Library compliance validation relies on AI-powered similarity scoring (face recognition, style matching). These models may produce false positives or negatives. Mitigation: multi-layer validation combining automated scoring with human jury review for borderline cases; Standard Library v2.0 will expand automated validation tooling.

**Blockchain Dependence** — Platform availability depends on Base’s uptime and fee environment. Mitigation: multi-sig emergency controls, Arbitrum One as designated fallback, architecture designed for EVM portability with minimal migration effort. Storage on IPFS/Arweave is independent of any single chain.

**Smart Contract Risk** — Treasury.sol, ShotMarket.sol, and ClipNFT.sol handle real economic value. Mitigation: two independent audits (OpenZeppelin, Trail of Bits) before mainnet; TransparentProxy for bug fixes in Phase 1; checks-effects-interactions pattern and ReentrancyGuard throughout; 3-of-5 multi-sig emergency pause.

**AI Generation Cost Volatility** — API costs for AI video generation fluctuate as providers update pricing. Mitigation: platform supports local generation (ComfyUI, Stable Diffusion Video) with no API dependency; negotiate subsidized access with Runway, Pika, Kling for platform creators; compute credit system funded by Film Ticket NFT sales in Phase 3.

### 9.2 Legal and Regulatory Risks

**AI Content Copyright** — The legal status of AI-generated content varies across jurisdictions and continues to evolve. AI-assisted works may face challenges in claiming copyright in some markets. Mitigation: contributor attestation system requiring creators to assert originality and legal compliance; DMCA takedown procedures; IP attorney partnerships specializing in AI-generated content; on-chain provenance trail provides stronger attribution than traditional filmmaking.

**Revenue NFT Securities Classification** — Clip NFTs and Film NFTs with revenue rights may be characterized as securities in certain jurisdictions, particularly the United States. Mitigation: legal review of token structure before TGE; modeled on lessons from Royal.io’s regulatory navigation; platform fee and governance token structure designed with regulatory considerations in mind; jurisdictional access controls as needed.

**Standard Library IP** — Character LoRA models trained on reference images may incorporate IP that third parties claim. Mitigation: director attestation of training data rights; DMCA process; automated content fingerprinting for known IP.

### 9.3 Market and Adoption Risks

**Chicken-and-Egg Cold Start** — No films without creators; no creators without films. Mitigation: Pilot Film “Genesis” produced with 50 invite-only creators in Phase 1, generating showcase content and a community nucleus before public launch. Phase 1 contributors receive $MONTAGE test token airdrops creating early loyalty.

**Quality Dilution at Scale** — As the platform opens publicly, submission quality variance increases. Mitigation: automated quality floors (resolution, format, Standard Library compliance validation) filter non-compliant submissions before they reach menu auctions; quadratic weighting on valuations ensures broad community consensus outweighs concentrated capital; directors can set minimum quality thresholds per film.

**$MONTAGE Token Volatility** — If the governance token experiences significant price volatility, it could distort platform incentives. Mitigation: creator earnings are denominated in USDC from film revenue, not in $MONTAGE; $MONTAGE governs the platform, not clip selection (menu auctions use USDC); vesting schedules prevent early selling pressure (only 8% circulating at TGE); buyback-and-burn creates structural demand.

---

## 10. Roadmap Overview

### Phase 1: Foundation (Q2–Q3 2026)

**Theme: Build the foundation, prove the concept.**

- Deploy core smart contracts on Base testnet: `ClipNFT.sol`, `FilmNFT.sol`, `Treasury.sol`, `ShotMarket.sol`
- Launch Discord server and MontageBot v0.1 with core commands: `/claim-shot`, `/submit`, `/back-shot`, `/back-shot-menu`, `/reveal-backing`, `/claim-market`
- Publish Standard Library Specification v1.0 (required asset formats, SemVer versioning, compliance criteria)
- Launch Pilot Film “Genesis”: 20-minute sci-fi short, 100 shots, 50 invite-only creators
- Complete security audit; address all critical findings
- Target: 5,000 Discord members, 80% of pilot shots filled, net-positive creator sentiment

### Phase 2: Alpha Launch (Q4 2026)

**Theme: Open the floodgates.**

- Deploy all contracts to Base mainnet with gas optimization and subgraph indexing
- Launch Web Dashboard v1: film browsing, submission management, revenue tracking, creator profiles
- Remove invite gate — any creator can submit, any director can create films (minimum Film Creation Bond: 500 USDC)
- Launch Film Ticket NFT crowdfunding for three inaugural public films
- Target: 20,000 community members, 10 active film projects, $50,000+ in Film Ticket NFT sales

### Phase 3: Ecosystem Growth (2027)

**Theme: Build the economy.**

- $MONTAGE TGE (Q1 2027) via IDO on Base-native DEX; initial circulating supply ~8%
- Full DAO governance activation — binding on-chain votes via Governor contract
- Advanced Standard Library tools: one-click LoRA training, style transfer templates, AI consistency scoring
- Creator marketplace for secondary Clip NFT trading (10% perpetual creator royalty on resales)
- Cross-film Standard Library marketplace — license assets across film projects
- DeFi integrations: Clip NFT collateral for loans, $MONTAGE staking vaults
- Mobile apps (iOS and Android)
- Submit top community films to independent film festivals
- Target: 50,000+ community members, 100+ active films, $1M+ total creator earnings distributed

### Phase 4: Scale and Legitimacy (2028+)

**Theme: Mainstream adoption and institutional credibility.**

- Festival partnerships and dedicated OpenMontage award categories
- Studio licensing framework for distributing community-created films
- VR/AR immersive viewing (Meta Quest, Apple Vision Pro)
- Platform UI localization in 25+ languages with regional community infrastructure
- Film school curriculum integrations (USC, NYU, AFI, international equivalents)
- Revenue sustainability target: platform self-funding from fees by end of 2028
- Long-term vision: 50,000+ active creators, 500+ completed films, $10M+ annual creator earnings

**Key milestones timeline:**

| Milestone | Target Date |
|-----------|-------------|
| Documentation complete | Q1 2026 (done) |
| Testnet smart contracts | Q2 2026 |
| MontageBot v0.1 | Q2 2026 |
| Pilot film “Genesis” kickoff | Q2 2026 |
| Pilot film complete | Q3 2026 |
| Security audit complete | Q3 2026 |
| Base mainnet deployment | Q4 2026 |
| Web Dashboard v1 | Q4 2026 |
| $MONTAGE TGE | Q1 2027 |
| Full DAO governance | Q2 2027 |
| Mobile app beta | Q3 2027 |

---

## 11. Conclusion

OpenMontage is the infrastructure layer that makes professional crowdsourced AI filmmaking economically viable. It solves the three structural barriers that have prevented collaborative AI film production — consistency, incentives, and quality — through a coherent system of four integrated innovations: the Standard Library, the Pull Request workflow, Shot Market Menu Auctions, and NFT-driven economics.

The platform’s economic design aligns incentives across every participant category. Directors invest in quality Standard Libraries because better infrastructure attracts better contributors and generates larger revenue streams. Creators invest effort in quality submissions because winning returns capital and generates passive income. Backers submit honest valuation menus because truthful preference revelation is the dominant strategy in the menu auction’s equilibrium. Audiences drive the entire engine through engagement, making every participant’s incentives point toward film quality.

Shot Market Menu Auctions represent the platform’s most distinctive innovation. Adapted from the Bernheim-Whinston (1986) menu auction framework and enhanced with quadratic weighting, they let backers express preferences across ALL competing clips for each shot via sealed valuation menus. The clip with the highest aggregate quadratic-weighted community valuation wins — producing efficient outcomes under the quadratic-weighted objective. By eliminating the “backing a loser” penalty and encouraging full preference revelation, the mechanism creates the first genuinely efficient creative curation system. Films improve over time as AI technology advances and new creators submit superior versions of existing shots. Earnings are continuous, transparent, and proportional to contribution — enforced by smart contracts, not by platform administrators.

This is not simply a new way to fund films, or a new NFT marketplace, or another AI video tool. It is a new model for how creative works are produced, curated, owned, and monetized in the age of AI — one in which the traditional trade-off between creative control and collaborative scale no longer applies.

**The future of cinema is collaborative, iterative, and decentralized. OpenMontage is building its infrastructure.**

---

**Website**: [openmontage.com](https://openmontage.com)
**GitHub**: [github.com/digimads-lab/OpenMontage](https://github.com/digimads-lab/OpenMontage)
**Discord**: [discord.gg/SS8BdjYH6W](https://discord.gg/SS8BdjYH6W)
**Twitter**: [@OpenMontage](https://twitter.com/openmontage)

---

**Version**: 3.0
**Last Updated**: 2026-03-05
**Authors**: OpenMontage Core Team
