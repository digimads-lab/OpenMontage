# OpenMontage Technical Specification

**Version 2.0 — System Architecture & Implementation Specification**

---

## Table of Contents

1. [System Architecture Overview](#1-system-architecture-overview)
2. [Blockchain Layer](#2-blockchain-layer)
3. [Storage Layer](#3-storage-layer)
4. [Application Layer](#4-application-layer)
5. [Frontend Layer](#5-frontend-layer)
6. [Wallet Integration](#6-wallet-integration)
7. [Security Design](#7-security-design)
8. [Standard Library Technical Specification](#8-standard-library-technical-specification)
9. [Data Flow and Integration](#9-data-flow-and-integration)

---

## 1. System Architecture Overview

### 1.1 Layered Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        FRONTEND LAYER                               │
│                                                                     │
│  Phase 1: Discord Bot (MontageBot)                                  │
│  Phase 2: Web Dashboard (React/Next.js)                             │
│  Phase 3: Mobile Apps (iOS/Android) + Discord as notification layer │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ REST API / WebSocket
┌───────────────────────────▼─────────────────────────────────────────┐
│                      APPLICATION LAYER                              │
│                                                                     │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐ │
│  │   REST API      │  │  Discord Bot    │  │  Video Assembly     │ │
│  │  (Auth, Films,  │  │  Backend        │  │  Engine             │ │
│  │   Shots, Users) │  │  (Node.js)      │  │  (HLS/DASH)         │ │
│  └────────┬────────┘  └────────┬────────┘  └──────────┬──────────┘ │
│           │                   │                       │            │
│  ┌────────▼───────────────────▼───────────────────────▼──────────┐ │
│  │          Shared Backend Services                               │ │
│  │  - User/Wallet mapping DB   - Quality validation pipeline     │ │
│  │  - Metadata indexer         - Event listener & notification   │ │
│  └────────────────────────────────────────────────────────────────┘ │
└────────────┬────────────────────────────────────────────────────────┘
             │
     ┌───────┴────────────────────────┐
     │                                │
┌────▼────────────────┐  ┌────────────▼──────────────────────────────┐
│   BLOCKCHAIN LAYER  │  │           STORAGE LAYER                   │
│                     │  │                                           │
│  Base L2 (primary)  │  │  ┌──────────────┐  ┌───────────────────┐ │
│  Arbitrum One       │  │  │ IPFS/Arweave │  │  Git LFS          │ │
│  (fallback)         │  │  │ (Video files)│  │  (LoRA/ControlNet)│ │
│                     │  │  └──────────────┘  └───────────────────┘ │
│  ClipNFT.sol        │  │  ┌──────────────────────────────────────┐ │
│  FilmNFT.sol        │  │  │  Cloud CDN (Cloudflare/CloudFront)   │ │
│  Treasury.sol       │  │  │  Global delivery + edge caching      │ │
│  ShotMarket.sol     │  │  └──────────────────────────────────────┘ │
│  StandardLibrary.sol│  │  ┌──────────────────────────────────────┐ │
│                     │  │  │  The Graph / Custom Indexer           │ │
│  Chainlink Oracles  │  │  │  (On-chain event indexing)           │ │
│  Gnosis Safe Multisig│ │  └──────────────────────────────────────┘ │
└─────────────────────┘  └───────────────────────────────────────────┘
```

### 1.2 Component Responsibility Matrix

| Component | Owns | Does NOT Own |
|-----------|------|--------------|
| `ClipNFT.sol` | Clip authorship, merge/archive status, backer registry | Revenue calculation, market logic |
| `FilmNFT.sol` | Film copyright, director rights, creation bond escrow | Clip selection, revenue math |
| `Treasury.sol` | Revenue splitting, claimable balances, competition tax | Market resolution, NFT minting |
| `ShotMarket.sol` | Market phases, sealed menu commit-reveal, quadratic SelectionWeight resolution | Revenue distribution, NFT metadata |
| `StandardLibrary.sol` | Asset versioning, compliance hash registry | Storage, CDN delivery |
| Discord Bot Backend | Discord ↔ wallet mapping, command handling | Private keys, fund custody |
| Video Assembly Engine | Segment sequencing, HLS/DASH packaging | Blockchain state |
| CDN | Fast delivery, edge caching | Permanence (deferred to IPFS/Arweave) |
| IPFS/Arweave | Content-addressed permanent storage | Speed (deferred to CDN) |

### 1.3 Phase 1 vs Phase 2+ Architecture Differences

| Dimension | Phase 1 (Q2–Q3 2026) | Phase 2+ (Q4 2026+) |
|-----------|----------------------|----------------------|
| Primary UI | Discord Bot (MontageBot) | Web Dashboard (React/Next.js) |
| Chain | Base Testnet | Base Mainnet |
| Smart contracts | TransparentProxy behind team multisig | Same pattern; governance migrates to DAO in Phase 3 |
| Wallet connection | Collab.Land verification + WalletConnect relay | Full MetaMask/WalletConnect/Coinbase Wallet browser integration |
| Video validation | Format + resolution + duration checks | + AI consistency scoring (face recognition, style matching) |
| Metadata indexing | Custom indexer | The Graph subgraph + custom indexer |
| Governance | Team multisig (3-of-5 Gnosis Safe) | $MONTAGE DAO Governor (Phase 3) |
| Notification | Discord embeds | Discord + web dashboard + push notifications |

---

## 2. Blockchain Layer

### 2.1 Chain Selection and Rationale

**Primary Chain: Base (Coinbase L2)**

Base is an OP Stack rollup on Ethereum. It is selected as OpenMontage's primary deployment target for the following reasons:

| Factor | Base Advantage |
|--------|----------------|
| Transaction fees | Sub-cent costs make frequent Clip NFT minting, backing commits, and reveals economically viable |
| Creator ecosystem | Base hosts the largest on-chain creator community (Zora, Farcaster), matching OpenMontage's target users |
| Fiat on-ramp | Coinbase's onramp lets non-crypto filmmakers and audiences fund accounts without prior Web3 experience |
| Ethereum security | Inherits Ethereum's L1 finality guarantees as an OP Stack rollup |
| EVM compatibility | Standard Solidity tooling: Hardhat, Foundry, OpenZeppelin contracts work without modification |

**Fallback Chain: Arbitrum One**

If Base throughput or ecosystem conditions change materially, Arbitrum One serves as the secondary deployment target. Both chains are EVM-compatible rollups; migrating the same Solidity contracts requires only configuration changes (RPC endpoint, chain ID, block explorer). Arbitrum uses Nitro (not OP Stack), but Solidity bytecode is identical.

**Chain IDs and RPC:**

| Network | Chain ID | Role |
|---------|----------|------|
| Base Mainnet | 8453 | Primary production |
| Base Sepolia (testnet) | 84532 | Phase 1 testing |
| Arbitrum One | 42161 | Fallback production |

### 2.2 Core Smart Contracts

Five contracts form the complete on-chain system. Each is deployed behind an OpenZeppelin `TransparentUpgradeableProxy` in Phase 1–2, with the team's 3-of-5 Gnosis Safe as the proxy admin.

| Contract | Purpose |
|----------|---------|
| `ClipNFT.sol` | ERC-721 for video segment ownership; tracks merge status and backer registry |
| `FilmNFT.sol` | ERC-721 for film copyright; manages Film Creation Bond and completion escrow |
| `Treasury.sol` | Automated 75/10/5/10 revenue distribution; holds claimable balances |
| `ShotMarket.sol` | Shot market lifecycle: submission, sealed menu commit-reveal, quadratic SelectionWeight resolution, payouts |
| `StandardLibrary.sol` | Standard Library asset versioning and compliance hash registry |

#### 2.2.1 ClipNFT.sol — ERC-721 Segment Ownership

`ClipNFT.sol` mints one ERC-721 token per submitted video segment. The token is the immutable proof of authorship. Its `status` field tracks whether the clip is currently merged into the film's Main Branch or has been archived.

**On-chain metadata per token:**

```solidity
struct ClipMetadata {
    uint256 filmId;
    uint256 shotId;
    address creator;
    uint256 timestamp;
    bytes32 videoHash;       // keccak256 of IPFS/Arweave CID
    string  videoCID;        // IPFS or Arweave content identifier
    string  stdLibVersion;   // SemVer string, e.g. "1.2.0"
    string  generationMethod; // e.g. "Runway Gen-3 + protagonist LoRA v1.2.0"
    uint256 durationMs;      // clip duration in milliseconds
    uint256 creatorBond;     // USDC amount deposited at submission (wei units)
    ClipStatus status;       // Submitted | Merged | Archived
}

enum ClipStatus { Submitted, Merged, Archived }
```

**Backer registry** — each Clip NFT holds a read-only record of all backer valuations after market resolution:

```solidity
struct BackerRecord {
    address backer;
    uint256 valuationUsdc;  // v_i_winner paid by this backer
    bool    isWinner;
}
// clipId => BackerRecord[]
mapping(uint256 => BackerRecord[]) public backerRegistry;
```

The backer registry is written once at market resolution and is thereafter immutable.

**Secondary sale royalty:** Creators receive a 10% perpetual royalty on secondary sales, implemented via ERC-2981.

#### 2.2.2 FilmNFT.sol — Film Copyright and Bond Management

`FilmNFT.sol` mints one ERC-721 token when a director pays the Film Creation Bond. The token represents the film as a composite creative work.

**On-chain metadata per film:**

```solidity
struct FilmMetadata {
    string  title;
    address director;
    uint256 creationBond;      // total USDC bonded (in USDC decimals)
    uint256 treasurySeed;      // 50% of bond, sent to Treasury
    uint256 completionEscrow;  // 50% of bond, locked until completion
    string  stdLibVersion;     // current Standard Library version
    uint256 totalShots;        // declared shot count
    uint256 mergedShots;       // currently merged shots
    FilmStatus status;         // Active | Completed | Abandoned
    uint256 lastActivityAt;    // unix timestamp, for abandonment detection
}

enum FilmStatus { Active, Completed, Abandoned }
```

**Bond allocation (immutable at minting):**
- `treasurySeed` = `creationBond * 50 / 100` — transferred to `Treasury.sol` immediately
- `completionEscrow` = `creationBond * 50 / 100` — locked in `FilmNFT.sol` until `mergedShots >= totalShots * 80 / 100`

**Abandonment condition:** If `block.timestamp - lastActivityAt > 90 days`, any caller may trigger `declareAbandoned()`, which distributes `completionEscrow` pro-rata to all creators who submitted clips for that film.

**Transferability:** Full ERC-721 transfer is enabled. The new token holder inherits all director rights and the 10% revenue share recorded in `Treasury.sol`. `Treasury.sol` reads `FilmNFT.ownerOf(filmId)` at claim time; no manual update is required on transfer.

#### 2.2.3 Treasury.sol — Automated Revenue Distribution

`Treasury.sol` is the financial settlement layer. It accumulates revenue from all sources and maintains a per-participant claimable balance. It does not push funds proactively; participants pull their balance by calling `claimRevenue()`.

**Revenue split constants (immutable per deployment, adjustable by governance):**

```solidity
uint256 public constant CREATOR_SHARE_BPS  = 7500; // 75% (basis points)
uint256 public constant DIRECTOR_SHARE_BPS = 1000; // 10%
uint256 public constant STDLIB_SHARE_BPS   = 500;  // 5%
uint256 public constant BACKER_SHARE_BPS   = 1000; // 10%
uint256 public constant BPS_DENOMINATOR    = 10000;
```

**Revenue attribution logic:**

For each merged Clip NFT in a film, the Treasury tracks:
- `clipDurationMs` — used to compute screen-time fraction
- `totalFilmDurationMs` — sum of all merged clip durations
- `backerTotalValuations[clipId]` — sum of all winning backer valuations (v_i_winner) for that clip

On each revenue deposit (`depositRevenue(filmId, amount)`):

```
creatorPool   = amount × CREATOR_SHARE_BPS / BPS_DENOMINATOR
directorShare = amount × DIRECTOR_SHARE_BPS / BPS_DENOMINATOR
stdlibShare   = amount × STDLIB_SHARE_BPS / BPS_DENOMINATOR
backerPool    = amount × BACKER_SHARE_BPS / BPS_DENOMINATOR

For each merged clip j:
  screenFraction_j = clipDurationMs_j / totalFilmDurationMs
  Creator of clip j earns: creatorPool × screenFraction_j
  Backer i of clip j earns:
    backerPool × screenFraction_j × (v_i_winner_j / backerTotalValuations_j)
    where v_i_winner_j is backer i's stated valuation for clip j from their menu

Director of film earns: directorShare (distributed to FilmNFT.ownerOf(filmId))
StdLib contributors earn: stdlibShare / stdlibContributorCount (equal split or weighted by asset count)
```

**Film Ticket NFT integration:** When a film has Film Ticket NFTs, `CREATOR_SHARE_BPS` drops to 6500 (65%) and a `TICKET_SHARE_BPS` of 1000 (10%) is added. All other shares remain unchanged.

**Competition Tax:** `ShotMarket.sol` calls `depositCompetitionTax(filmId, amount)` after each market resolution; this amount is deposited into the film's general revenue pool and distributed on the next `distributeRevenue()` call. Competition tax (5% of losing Creator Bonds) flows into the Film Treasury as general operating revenue. It is treated as a revenue source alongside streaming fees and is redistributed via the standard 75/10/5/10 formula on the next `distributeRevenue()` call. This circular flow is intentional — the tax strengthens the Treasury while maintaining the same distribution principles for all revenue sources.

#### 2.2.4 ShotMarket.sol — Menu Auction Mechanism

`ShotMarket.sol` implements the full Shot Market Menu Auction lifecycle, adapted from the Bernheim-Whinston (1986) menu auction framework:

```
Submission Phase (14 days default)
  → Commit Phase (5 days) — backers submit sealed valuation menus
    → Reveal Phase (2 days) — backers reveal full menus
      → Resolution — highest aggregate quadratic SelectionWeight wins
        → Payout Claims
```

**Market state machine:**

```solidity
enum MarketPhase {
    Inactive,       // market not yet opened
    Submission,     // creators may submit clips
    Commit,         // backers submit sealed menu commitment hashes
    Reveal,         // backers reveal full valuation menus
    Resolved        // winner determined; claims open
}

struct Market {
    uint256 filmId;
    uint256 shotId;
    MarketPhase phase;
    uint256 submissionDeadline;
    uint256 commitDeadline;
    uint256 revealDeadline;
    uint256 winningClipId;
    uint256 totalLosingCreatorBonds; // CB_l — used for reward pool calculation
    bool    resolved;
}
```

**Commitment structure (per backer):**

```solidity
struct MenuCommitment {
    bytes32 commitHash;   // keccak256(abi.encodePacked(clipId_1, v_1, ..., clipId_N, v_N, salt))
    uint256 depositUsdc;  // max(v_i) — locked USDC at commit time
    bool    revealed;
}

struct RevealedValuation {
    uint256 clipId;
    uint256 valuation;    // backer's stated valuation for this clip winning
}

// shotId => backer => MenuCommitment
mapping(uint256 => mapping(address => MenuCommitment)) public commitments;

// shotId => backer => clipId => valuation (populated at reveal time)
mapping(uint256 => mapping(address => mapping(uint256 => uint256))) public revealedMenus;
```

**Quadratic SelectionWeight computation (view function):**

```solidity
function computeSelectionWeight(uint256 shotId, uint256 clipId)
    public view returns (uint256 weight)
{
    // Creator bond contributes sqrt(creatorBond)
    uint256 creatorBond = clips[clipId].creatorBond;
    weight = sqrt(creatorBond);

    // Each revealed backer contributes sqrt(valuation_for_this_clip)
    for (uint i = 0; i < backerList[shotId].length; i++) {
        address backer = backerList[shotId][i];
        if (commitments[shotId][backer].revealed) {
            uint256 valuation = revealedMenus[shotId][backer][clipId];
            if (valuation > 0) {
                weight += sqrt(valuation);
            }
        }
    }
}
```

Note: Integer square root via Newton's method (no floating-point; all values in USDC wei units). A single backer contributes to the SelectionWeight of ALL clips they valued > 0.

**Payout distribution after resolution:**

```
Backer settlements:
  Each backer pays v_i_winner (valuation for the winning clip)
  Excess refund: deposit - v_i_winner → returned immediately
  v_i_winner recorded as streaming revenue stake in Treasury

Creator Bond settlements (from losing Creator Bonds CB_l):
  Competition Tax  = 5%  × CB_l  → Treasury.depositCompetitionTax()
  Loser Refund     = 75% × CB_l  → refunds to losing creators
  Reward Pool      = 20% × CB_l
    Winning Creator share: 50% × Reward Pool
    Winning Backers share:  50% × Reward Pool (proportional to sqrt(v_i_winner))
```

**Challenge market:** Separate market type `ChallengeMarket` inherits `Market` and adds:
- `challengeBond`: 100 USDC (2x standard Creator Bond)
- `incumbentClipId`: the currently-merged clip
- `syntheticWeight`: The incumbent clip begins the mini-auction with a pre-loaded synthetic SelectionWeight equal to 50% of its original winning SelectionWeight. New backer valuations add to this via the standard quadratic formula: `effectiveWeight = syntheticWeight + Σ√(v_i_incumbent)`. The challenger's clip starts at 0 and must accumulate weight purely from new backer valuations.
- `cooldownUntil`: `block.timestamp + 21 days` after any resolution; rejects new challenges before cooldown expires
- Challenge auctions use binary menus: backers submit (incumbent_valuation, challenger_valuation)
- **Challenge resolution payouts:**
  - *Challenger wins:* Challenge Bond refunded in full; incumbent clip archived; challenger clip merged. Standard reward pool distribution applies.
  - *Incumbent survives:* Challenger receives 75% refund on Challenge Bond (loses 25 USDC). Incumbent creator receives a **defense bonus** of 50% of the forfeited amount (12.50 USDC). Remaining 50% (12.50 USDC) flows to Film Treasury. Emits `ChallengeDefenseBonus`. The existing `claimWinnerReward()` handles the defense bonus payout for the incumbent creator.

> **Auction phase timing:** Challenge mini-auctions use a compressed schedule:
> ```
> // Challenge auction phase is compressed: 3-day commit + 2-day reveal = 5 days total
> // (vs. standard Shot Market: 5-day commit + 2-day reveal = 7 days)
> // Rationale: challenges should resolve faster to minimize disruption to merged clips
> ```

#### 2.2.5 StandardLibrary.sol — Dependency Versioning and Compliance

`StandardLibrary.sol` maintains a registry of approved Standard Library versions per film and a set of content hashes that identify compliant assets.

```solidity
struct LibraryVersion {
    string  semver;           // e.g. "1.2.0"
    bytes32 manifestHash;     // keccak256 of the dependency manifest JSON
    uint256 publishedAt;
    address publisher;        // must be film director or authorized contributor
    bool    deprecated;
}

// filmId => semver string => LibraryVersion
mapping(uint256 => mapping(string => LibraryVersion)) public versions;

// filmId => current semver
mapping(uint256 => string) public currentVersion;
```

**Compliance check at submission:**

`ShotMarket.submitClip()` calls `StandardLibrary.verifyCompliance(filmId, stdLibVersion)` before minting. The check ensures the declared `stdLibVersion` exists for the film and is not deprecated.

Extended validation (off-chain, enforced by bot before calling contract):
- Content hash of submitted video is compared against a style similarity oracle score (Phase 2)
- Videos scoring below 70% similarity to the Standard Library reference images are rejected before the transaction is sent

### 2.3 Complete Contract Interfaces (Solidity)

```solidity
// ═══════════════════════════════════════════════════════════════════
// ClipNFT.sol
// ═══════════════════════════════════════════════════════════════════

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IClipNFT {

    // ── Events ──────────────────────────────────────────────────────

    event ClipSubmitted(
        uint256 indexed filmId,
        uint256 indexed shotId,
        uint256 indexed clipId,
        address creator,
        string  videoCID,
        string  stdLibVersion
    );

    event ClipMerged(
        uint256 indexed filmId,
        uint256 indexed shotId,
        uint256 indexed clipId,
        uint256 previousClipId  // 0 if first merge for this shot
    );

    event ClipArchived(
        uint256 indexed clipId,
        uint256 indexed replacedByClipId
    );

    event BackerRegistered(
        uint256 indexed clipId,
        address indexed backer,
        uint256 valuationUsdc,
        bool    isWinner
    );

    // ── State-changing functions ─────────────────────────────────────

    /// @notice Submit a new clip. Caller must have approved 50 USDC to this contract.
    /// @param filmId  Film token ID
    /// @param shotId  Shot index within the film
    /// @param videoCID  IPFS CID or Arweave TX ID of the video file
    /// @param videoHash keccak256 of the raw video bytes (integrity check)
    /// @param stdLibVersion SemVer string of the Standard Library used
    /// @param generationMethod Human-readable AI tool description
    /// @param durationMs Video duration in milliseconds
    /// @return clipId The minted Clip NFT token ID
    function submitClip(
        uint256 filmId,
        uint256 shotId,
        string  calldata videoCID,
        bytes32 videoHash,
        string  calldata stdLibVersion,
        string  calldata generationMethod,
        uint256 durationMs
    ) external returns (uint256 clipId);

    /// @notice Mark a clip as merged (called by ShotMarket after resolution)
    /// @param shotId  Shot index
    /// @param clipId  Winning clip token ID
    function mergeClip(uint256 shotId, uint256 clipId) external;

    /// @notice Archive a clip that has been replaced (called by ShotMarket)
    /// @param clipId  Clip token ID to archive
    /// @param replacedByClipId  The new winning clip token ID
    function archiveClip(uint256 clipId, uint256 replacedByClipId) external;

    /// @notice Register backer records after market resolution (called by ShotMarket)
    /// @param clipId  Clip token ID
    /// @param backers Array of backer addresses
    /// @param valuations Array of USDC valuation amounts (v_i_winner) corresponding to backers
    /// @param isWinnerClip Whether this clip won the market
    function registerBackers(
        uint256 clipId,
        address[] calldata backers,
        uint256[] calldata valuations,
        bool isWinnerClip
    ) external;

    // ── View functions ───────────────────────────────────────────────

    function getClipMetadata(uint256 clipId)
        external view returns (
            uint256 filmId,
            uint256 shotId,
            address creator,
            uint256 timestamp,
            bytes32 videoHash,
            string  memory videoCID,
            string  memory stdLibVersion,
            uint256 durationMs,
            uint256 creatorBond,
            uint8   status   // 0=Submitted, 1=Merged, 2=Archived
        );

    function getBackerRegistry(uint256 clipId)
        external view returns (
            address[] memory backers,
            uint256[] memory deposits,
            bool[] memory   isWinner
        );

    function getCurrentMergedClip(uint256 filmId, uint256 shotId)
        external view returns (uint256 clipId);
}

// ═══════════════════════════════════════════════════════════════════
// FilmNFT.sol
// ═══════════════════════════════════════════════════════════════════

interface IFilmNFT {

    // ── Events ──────────────────────────────────────────────────────

    event FilmCreated(
        uint256 indexed filmId,
        address indexed director,
        uint256 bond,
        uint256 totalShots,
        string  stdLibVersion
    );

    event FilmCompleted(
        uint256 indexed filmId,
        uint256 mergedShots,
        uint256 totalShots,
        uint256 escrowRefunded
    );

    event FilmAbandoned(
        uint256 indexed filmId,
        uint256 escrowDistributed,
        uint256 claimantsCount
    );

    event FilmUpdated(
        uint256 indexed filmId,
        uint256 mergedShots,
        string  newStdLibVersion  // empty if not updated
    );

    // ── State-changing functions ─────────────────────────────────────

    /// @notice Create a film and mint a Film NFT. Caller must approve `bondAmount` USDC.
    /// @param title  Film title (stored in NFT metadata)
    /// @param totalShots  Number of shots declared for this film
    /// @param stdLibVersion  Initial Standard Library SemVer
    /// @param bondAmount  USDC to bond (minimum 500e6 in 6-decimal USDC)
    /// @return filmId The minted Film NFT token ID
    function createFilm(
        string  calldata title,
        uint256 totalShots,
        string  calldata stdLibVersion,
        uint256 bondAmount
    ) external returns (uint256 filmId);

    /// @notice Record a shot merge; updates mergedShots counter and lastActivityAt.
    ///         Called by ShotMarket after market resolution.
    function recordMerge(uint256 filmId) external;

    /// @notice Director claims completion escrow refund.
    ///         Requires mergedShots >= totalShots * 80 / 100.
    /// @return refunded USDC amount returned to director
    function claimBondRefund(uint256 filmId) external returns (uint256 refunded);

    /// @notice Anyone can trigger abandonment check after 90 days of inactivity.
    ///         Distributes completion escrow pro-rata to clip submitters.
    function declareAbandoned(uint256 filmId) external;

    /// @notice Director updates Standard Library version for the film.
    function updateStdLibVersion(uint256 filmId, string calldata newVersion) external;

    // ── View functions ───────────────────────────────────────────────

    function getFilmMetadata(uint256 filmId)
        external view returns (
            string  memory title,
            address director,
            uint256 creationBond,
            uint256 treasurySeed,
            uint256 completionEscrow,
            string  memory stdLibVersion,
            uint256 totalShots,
            uint256 mergedShots,
            uint8   status,           // 0=Active, 1=Completed, 2=Abandoned
            uint256 lastActivityAt
        );

    function getBondStatus(uint256 filmId)
        external view returns (
            uint256 bonded,
            uint256 escrowRemaining,
            bool    completionEligible,
            bool    refundClaimed
        );
}

// ═══════════════════════════════════════════════════════════════════
// Treasury.sol
// ═══════════════════════════════════════════════════════════════════

interface ITreasury {

    // ── Events ──────────────────────────────────────────────────────

    event RevenueDeposited(
        uint256 indexed filmId,
        uint256 amount,
        address depositor
    );

    event RevenueClaimed(
        uint256 indexed filmId,
        address indexed claimant,
        uint256 creatorAmount,
        uint256 backerAmount,
        uint256 directorAmount,
        uint256 stdlibAmount
    );

    event CompetitionTaxDeposited(
        uint256 indexed filmId,
        uint256 indexed shotId,
        uint256 amount
    );

    // ── State-changing functions ─────────────────────────────────────

    /// @notice Deposit external revenue (streaming fees, ad revenue, licensing)
    ///         into a film's treasury. Caller must approve USDC amount.
    function depositRevenue(uint256 filmId, uint256 amount) external;

    /// @notice Called by ShotMarket after resolution to deposit competition tax.
    function depositCompetitionTax(
        uint256 filmId,
        uint256 shotId,
        uint256 amount
    ) external;

    /// @notice Trigger revenue distribution calculation for a film.
    ///         Iterates merged clips and updates claimable balances.
    ///         Can be called by anyone; typically called by a keeper bot.
    function distributeRevenue(uint256 filmId) external;

    /// @notice Pull-based claim: participant withdraws their claimable balance.
    /// @return totalClaimed Total USDC transferred to caller
    function claimRevenue(uint256 filmId) external returns (uint256 totalClaimed);

    // ── View functions ───────────────────────────────────────────────

    function getClaimable(uint256 filmId, address participant)
        external view returns (
            uint256 creatorRevenue,
            uint256 backerRevenue,
            uint256 directorRevenue,
            uint256 stdlibRevenue
        );

    function getTreasuryBalance(uint256 filmId)
        external view returns (uint256 unclaimedUsdc);

    function getFilmRevenueStats(uint256 filmId)
        external view returns (
            uint256 totalDeposited,
            uint256 totalDistributed,
            uint256 totalClaimed,
            uint256 competitionTaxCollected
        );
}

// ═══════════════════════════════════════════════════════════════════
// ShotMarket.sol — Menu Auction Mechanism
// ═══════════════════════════════════════════════════════════════════

interface IShotMarket {

    // ── Events ──────────────────────────────────────────────────────

    event MarketOpened(
        uint256 indexed filmId,
        uint256 indexed shotId,
        uint256 submissionDeadline,
        uint256 commitDeadline,
        uint256 revealDeadline
    );

    event ClipEnteredMarket(
        uint256 indexed shotId,
        uint256 indexed clipId,
        address creator,
        uint256 creatorBond
    );

    event MenuCommitted(
        uint256 indexed shotId,
        address indexed backer,
        bytes32 commitment,
        uint256 depositUsdc   // max(valuations) locked
    );

    event MenuRevealed(
        uint256 indexed shotId,
        address indexed backer,
        uint256 clipCount     // number of clips valued > 0
    );

    event MenuForfeited(
        uint256 indexed shotId,
        address indexed backer,
        uint256 forfeitedAmount  // added to reward pool
    );

    event MarketResolved(
        uint256 indexed filmId,
        uint256 indexed shotId,
        uint256 winningClipId,
        uint256 winningWeight,
        uint256 totalLosingCreatorBonds,
        uint256 competitionTax,
        uint256 rewardPool
    );

    event BackerSettled(
        uint256 indexed shotId,
        address indexed backer,
        uint256 valuationPaid,   // v_i_winner
        uint256 excessRefunded   // deposit - v_i_winner
    );

    event WinnerRewardClaimed(
        uint256 indexed shotId,
        address indexed claimant,
        uint256 amount,
        bool    isCreator
    );

    event LoserCreatorRefundClaimed(
        uint256 indexed shotId,
        address indexed creator,
        uint256 refundAmount
    );

    event ChallengeOpened(
        uint256 indexed filmId,
        uint256 indexed shotId,
        uint256 challengerClipId,
        uint256 incumbentClipId,
        uint256 challengeBond,
        uint256 cooldownAfter
    );

    event ChallengeResolved(
        uint256 indexed filmId,
        uint256 indexed shotId,
        uint256 winningClipId,
        bool    incumbentSurvived
    );

    event ChallengeDefenseBonus(
        uint256 indexed shotId,
        address indexed incumbentCreator,
        uint256 bonusAmount           // 50% of challenger's forfeited bond (12.50 USDC)
    );

    event AutoMerged(
        uint256 indexed filmId,
        uint256 indexed shotId,
        uint256 clipId
    );

    // ── State-changing functions ─────────────────────────────────────

    /// @notice Open a Shot Market for a specific shot.
    ///         Callable by anyone once the submission deadline has passed and >=2 clips are submitted.
    /// @param filmId  Film token ID
    /// @param shotId  Shot index
    /// @param submissionDays  Length of submission phase in days (7-30)
    function openMarket(
        uint256 filmId,
        uint256 shotId,
        uint256 submissionDays
    ) external;

    /// @notice Submit a clip to an open market. Transfers 50 USDC Creator Bond.
    ///         Calls ClipNFT.submitClip() and StandardLibrary.verifyCompliance().
    /// @param shotId  Shot index
    /// @param videoCID  IPFS or Arweave CID
    /// @param videoHash  keccak256 of raw video bytes
    /// @param stdLibVersion  SemVer string
    /// @param generationMethod  Human-readable description
    /// @param durationMs  Duration in milliseconds
    /// @return clipId  Minted Clip NFT ID
    function submitClip(
        uint256 shotId,
        string  calldata videoCID,
        bytes32 videoHash,
        string  calldata stdLibVersion,
        string  calldata generationMethod,
        uint256 durationMs
    ) external returns (uint256 clipId);

    /// @notice Commit a sealed valuation menu during the commit phase.
    ///         Caller must approve `depositUsdc` USDC to this contract.
    ///         depositUsdc must equal max(valuations) in the backer's menu.
    /// @param shotId  Shot index
    /// @param commitment  keccak256(abi.encodePacked(clipId_1, v_1, ..., clipId_N, v_N, salt))
    /// @param depositUsdc  USDC to lock = max valuation in menu (10-5000 USDC)
    function commitBacking(
        uint256 shotId,
        bytes32 commitment,
        uint256 depositUsdc
    ) external;

    /// @notice Reveal full valuation menu during the reveal phase.
    ///         Must match the previously submitted commitment hash.
    /// @param shotId     Shot index
    /// @param clipIds    Array of clip IDs in the menu
    /// @param valuations Array of USDC valuations corresponding to each clip
    /// @param salt       The random salt used at commit time
    function revealMenuBacking(
        uint256 shotId,
        uint256[] calldata clipIds,
        uint256[] calldata valuations,
        bytes32 salt
    ) external;

    /// @notice Resolve a market after the reveal deadline.
    ///         Determines winner by highest aggregate quadratic SelectionWeight.
    ///         Settles backer payments (v_i_winner) and refunds (deposit - v_i_winner).
    ///         Calls ClipNFT.mergeClip() and Treasury.depositCompetitionTax().
    /// @param shotId  Shot index
    /// @return winningClipId  Token ID of the winning Clip NFT
    function resolveMarket(uint256 shotId) external returns (uint256 winningClipId);

    /// @notice Claim winner reward (Creator Bond refund + share of reward pool).
    ///         Can be called by winning creator or winning backers.
    function claimWinnerReward(uint256 shotId) external returns (uint256 amount);

    /// @notice Claim losing creator refund (75% of Creator Bond).
    function claimLoserCreatorRefund(uint256 shotId) external returns (uint256 amount);

    /// @notice Auto-merge the sole clip when only one submission exists at deadline.
    ///         Callable by the director. Skips commit/reveal phases entirely.
    ///         Refunds the Creator Bond in full (no competition tax).
    /// @param shotId  Shot index with exactly 1 submitted clip
    function autoMerge(uint256 shotId) external;

    /// @notice Open a challenge auction for an already-merged shot.
    ///         Challenger pays 100 USDC Challenge Bond.
    ///         Challenge uses binary menus: backers value incumbent vs. challenger.
    /// @param shotId  Shot index to challenge
    /// @param videoCID  IPFS CID of the challenger's clip
    /// @param videoHash  keccak256 of raw video bytes
    /// @param stdLibVersion  SemVer string
    /// @param generationMethod  Human-readable description
    /// @param durationMs  Duration in milliseconds
    function openChallenge(
        uint256 shotId,
        string  calldata videoCID,
        bytes32 videoHash,
        string  calldata stdLibVersion,
        string  calldata generationMethod,
        uint256 durationMs
    ) external;

    // ── View functions ───────────────────────────────────────────────

    function getMarketPhase(uint256 shotId)
        external view returns (uint8 phase);  // MarketPhase enum value

    function getSelectionWeight(uint256 shotId, uint256 clipId)
        external view returns (uint256 weight);

    function getCommitmentCount(uint256 shotId)
        external view returns (uint256 totalCommits, uint256 totalReveals);

    function getMarketResult(uint256 shotId)
        external view returns (
            uint256 winningClipId,
            uint256 winningWeight,
            uint256 totalLosingCreatorBonds,
            uint256 rewardPool
        );

    function getBackerMenuValuation(uint256 shotId, address backer, uint256 clipId)
        external view returns (uint256 valuation);

    function isChallengeCooledDown(uint256 shotId)
        external view returns (bool cooledDown, uint256 cooldownUntil);
}

// ═══════════════════════════════════════════════════════════════════
// StandardLibrary.sol
// ═══════════════════════════════════════════════════════════════════

interface IStandardLibrary {

    // ── Events ──────────────────────────────────────────────────────

    event LibraryVersionPublished(
        uint256 indexed filmId,
        string  semver,
        bytes32 manifestHash,
        address publisher
    );

    event LibraryVersionDeprecated(
        uint256 indexed filmId,
        string  semver,
        address deprecatedBy
    );

    event ContributorRegistered(
        uint256 indexed filmId,
        address contributor,
        string  assetType    // "lora", "controlnet", "voice", "style"
    );

    // ── State-changing functions ─────────────────────────────────────

    /// @notice Publish a new Standard Library version for a film.
    ///         Only callable by the film's director (Film NFT holder).
    /// @param filmId       Film token ID
    /// @param semver       SemVer string (must be strictly greater than current version)
    /// @param manifestHash keccak256 of the dependency manifest JSON
    /// @param contributors Array of contributor wallet addresses for this version
    /// @param assetTypes   Array of asset type strings parallel to contributors
    function publishVersion(
        uint256 filmId,
        string  calldata semver,
        bytes32 manifestHash,
        address[] calldata contributors,
        string[]  calldata assetTypes
    ) external;

    /// @notice Deprecate a Standard Library version.
    ///         Submissions referencing deprecated versions will be rejected.
    function deprecateVersion(uint256 filmId, string calldata semver) external;

    /// @notice Verify that a declared Standard Library version is valid for a film.
    ///         Called by ShotMarket.submitClip() before minting.
    /// @return valid True if the version exists and is not deprecated
    function verifyCompliance(uint256 filmId, string calldata semver)
        external view returns (bool valid);

    // ── View functions ───────────────────────────────────────────────

    function getCurrentVersion(uint256 filmId)
        external view returns (string memory semver, bytes32 manifestHash);

    function getVersionDetails(uint256 filmId, string calldata semver)
        external view returns (
            bytes32 manifestHash,
            uint256 publishedAt,
            address publisher,
            bool    deprecated,
            address[] memory contributors,
            string[]  memory assetTypes
        );

    function getContributors(uint256 filmId)
        external view returns (address[] memory contributors);
}
```

### 2.4 Contract Interaction Flow

```
DIRECTOR: createFilm()
──────────────────────────────────────────────────────────────────────
  FilmNFT.createFilm()
    → mints Film NFT to director
    → transfers 50% bond → Treasury.depositRevenue() (treasury seed)
      // FilmNFT.createFilm() calls Treasury.depositRevenue() for the treasury seed (50% of bond)
      // The Treasury contract address is set once at FilmNFT deployment via constructor and immutable thereafter
    → locks 50% bond in FilmNFT (completion escrow)
  StandardLibrary.publishVersion()
    → stores manifest hash for v1.0.0
  ShotMarket.openMarket()  [for each shot]
    → initializes Market struct with phase deadlines

CREATOR: submitClip()
──────────────────────────────────────────────────────────────────────
  ShotMarket.submitClip()
    → StandardLibrary.verifyCompliance()  [reverts if invalid]
    → transfers 50 USDC Creator Bond from creator → ShotMarket
    → ClipNFT.submitClip()  [mints Clip NFT to creator]
    → emits ClipEnteredMarket

BACKER: commitBacking() + revealMenuBacking()
──────────────────────────────────────────────────────────────────────
  ShotMarket.commitBacking()
    → stores commitment hash
    → transfers max(valuations) USDC from backer → ShotMarket (locked)
  ShotMarket.revealMenuBacking()
    → verifies keccak256(clipId_1, v_1, ..., clipId_N, v_N, salt) == stored hash
    → verifies max(valuations) <= deposit
    → records backer's full valuation menu across all clips

ANYONE: resolveMarket()  [after reveal deadline]
──────────────────────────────────────────────────────────────────────
  ShotMarket.resolveMarket()
    → computes aggregate quadratic SelectionWeight for each clip
    → determines winner (highest SelectionWeight)
    → settles backer payments: each backer pays v_i_winner
    → refunds excess deposits: deposit - v_i_winner → backer
    → records v_i_winner as streaming revenue stake in Treasury
    → ClipNFT.mergeClip(winningClipId)
    → ClipNFT.archiveClip() for losers
    → ClipNFT.registerBackers() for winning clip
    → FilmNFT.recordMerge()
    → Treasury.depositCompetitionTax() [5% of CB_l]
    → emits MarketResolved

WINNERS/LOSERS: claimWinnerReward() / claimLoserCreatorRefund()
──────────────────────────────────────────────────────────────────────
  ShotMarket.claimWinnerReward()
    → checks-effects-interactions pattern
    → marks claim as processed
    → transfers (Creator Bond + reward share) → winning creator
    → transfers reward pool share → winning backers (proportional to sqrt(v_i_winner))

  ShotMarket.claimLoserCreatorRefund()
    → marks refund as processed
    → transfers 75% of Creator Bond → losing creator

KEEPER BOT: distributeRevenue()  [periodic, e.g. daily]
──────────────────────────────────────────────────────────────────────
  Treasury.distributeRevenue()
    → reads FilmNFT.getFilmMetadata() for merged shots
    → reads ClipNFT.getClipMetadata() for each merged clip (duration)
    → reads ClipNFT.getBackerRegistry() for backer proportions
    → updates claimable balances per participant

PARTICIPANTS: claimRevenue()  [anytime]
──────────────────────────────────────────────────────────────────────
  Treasury.claimRevenue()
    → checks claimable balance > 0
    → zeros balance (effect before transfer)
    → transfers USDC → caller
```

### 2.5 Gas Optimization Strategies

| Strategy | Applied To | Description |
|----------|-----------|-------------|
| Pull-based payments | Treasury | Participants pull their balance; no iteration over all claimants per distribution |
| Bitmap storage | ShotMarket | Claim status packed into uint256 bitmaps to minimize SSTORE operations |
| Events over storage | ClipNFT | Video metadata emitted in events; only the minimal on-chain state stored |
| Calldata arrays | Registration | `registerBackers()` uses calldata arrays; processed in a single transaction by ShotMarket |
| Unchecked arithmetic | Weight computation | Integer square root inner loop uses `unchecked {}` where overflow is impossible |
| SLOAD caching | Distribution loop | `distributeRevenue()` caches film duration and clip count in memory variables before the loop |
| ERC-721A batch minting | Future Film Tickets | Batch-mint Film Ticket NFTs in a single transaction using ERC-721A |

---

## 3. Storage Layer

### 3.1 Video File Storage — IPFS and Arweave

**Why content-addressed storage:**

Video files cannot be stored on-chain due to size. The smart contracts store only a content identifier (CID on IPFS or transaction ID on Arweave) and the `keccak256` hash of the raw bytes. Any node can verify that the file at the CID matches the on-chain hash, making the storage trustless.

**IPFS:**
- Content is addressed by its SHA2-256 multihash (the CID)
- Files are split into a Merkle DAG; partial retrieval is possible
- Used for hot storage — files pinned by OpenMontage's pinning infrastructure
- Interoperability: any IPFS node worldwide can serve the content

**Arweave:**
- Permanent storage with a one-time fee (stored in the permaweb)
- Arweave TX IDs are stable forever — no re-pinning required
- Used for cold archival of all merged clips and Standard Library assets
- More expensive per byte than IPFS, but eliminates pinning maintenance

**Pinning strategy:**

| Asset Type | IPFS Pinning | Arweave |
|------------|-------------|---------|
| Submitted clips (all) | Yes — pinned by OpenMontage for market duration | No |
| Merged clips | Yes — pinned indefinitely | Yes — permanent backup |
| Archived clips | Pinned for 90 days post-archival, then unpinned | Yes — preserved for historical record |
| Standard Library assets | Yes — pinned indefinitely | Yes — permanent backup |
| Film manifest (film.json) | Yes | Yes |

**Content integrity flow:**

```
Creator uploads video file
         │
         ▼
Bot backend computes keccak256(rawBytes)  →  stored as videoHash on-chain
Bot backend uploads to IPFS               →  CID returned
Bot backend uploads to Arweave            →  TX ID returned
         │
         ▼
ShotMarket.submitClip(shotId, CID, videoHash, ...)
         │
         ▼
At playback time:
  Fetch file from CDN (IPFS gateway or Arweave gateway)
  Verify: keccak256(fetchedBytes) == on-chain videoHash
  Render clip if valid
```

### 3.2 Standard Library Asset Distribution

**Asset types and storage:**

| Asset | Format | Max Size | Storage |
|-------|--------|----------|---------|
| Character LoRA models | `.safetensors` | 2 GB per file | Git LFS + CDN |
| ControlNet weights | `.safetensors` + config `.json` | 1 GB per model | Git LFS + CDN |
| Voice clone models | `.pth`, `.onnx`, `.bin` | 500 MB per model | Git LFS + CDN |
| Style reference images | `.png`, `.jpg` (min 512×512) | 10 MB per image | CDN |
| Prompt templates | `.json` | 1 MB | CDN |
| Art direction guide | `.md` or `.pdf` | 5 MB | CDN |

**Git LFS for large binary assets:**

Standard Library binary files (LoRAs, ControlNets, voice models) are stored in a Git repository using Git Large File Storage (LFS). The Git history provides version lineage, and LFS pointers keep the repository lightweight. The actual binary data is stored on an LFS server backed by object storage (S3-compatible).

```
standard-library-repo/
├── .lfsconfig               # LFS server URL
├── manifest.json            # Dependency manifest (current version)
├── characters/
│   ├── protagonist.safetensors  # LFS pointer
│   └── protagonist.json         # Metadata (training data, version)
├── style/
│   ├── art-direction.png
│   └── color-palette.json
├── prompts/
│   └── templates.json
├── controlnets/
│   └── character-rig.safetensors  # LFS pointer
└── voices/
    └── protagonist-voice.pth      # LFS pointer
```

**CDN global delivery:**

All Standard Library assets and merged video clips are served through a CDN (Cloudflare CDN for the global edge network, AWS CloudFront as fallback). Edge caching ensures low-latency downloads for creators worldwide.

```
Asset request flow:

Creator requests protagonist.safetensors
         │
         ▼
CDN edge node (nearest PoP)
   └─ Cache hit?  ─── Yes ──→  Return from edge cache (< 50ms)
         │ No
         ▼
CDN origin (S3-backed object storage)
   └─ Fetch and cache at edge
         │
         ▼
Creator receives file; verifies against manifest hash
```

**Dependency manifest format (JSON schema):**

```json
{
  "$schema": "https://openmontage.com/schemas/stdlib-manifest-v1.json",
  "filmId": "space-odyssey-2077",
  "version": "1.2.0",
  "publishedAt": "2026-04-01T12:00:00Z",
  "publisher": "0x123...",
  "manifestHash": "0xabc...",
  "assets": {
    "characters": [
      {
        "name": "protagonist",
        "description": "Lead character, 25-year-old female, cyberpunk aesthetic",
        "file": "characters/protagonist.safetensors",
        "hash": "sha256:abc123...",
        "sizeBytes": 148897612,
        "cdnUrl": "https://cdn.openmontage.com/films/space-odyssey-2077/v1.2.0/characters/protagonist.safetensors",
        "ipfsCID": "QmXyZ...",
        "loraWeight": 0.8,
        "triggerWords": ["[PROTAGONIST]"]
      }
    ],
    "style": [
      {
        "name": "art-direction",
        "file": "style/art-direction.png",
        "hash": "sha256:def456...",
        "sizeBytes": 2345678,
        "cdnUrl": "https://cdn.openmontage.com/films/space-odyssey-2077/v1.2.0/style/art-direction.png"
      }
    ],
    "controlnets": [
      {
        "name": "character-rig",
        "file": "controlnets/character-rig.safetensors",
        "hash": "sha256:ghi789...",
        "sizeBytes": 987654321,
        "cdnUrl": "https://cdn.openmontage.com/films/space-odyssey-2077/v1.2.0/controlnets/character-rig.safetensors",
        "controlnetType": "openpose"
      }
    ],
    "voices": [
      {
        "name": "protagonist-voice",
        "file": "voices/protagonist-voice.pth",
        "hash": "sha256:jkl012...",
        "sizeBytes": 256000000,
        "cdnUrl": "https://cdn.openmontage.com/films/space-odyssey-2077/v1.2.0/voices/protagonist-voice.pth",
        "modelType": "RVC",
        "sampleRate": 44100
      }
    ],
    "prompts": {
      "file": "prompts/templates.json",
      "hash": "sha256:mno345...",
      "cdnUrl": "https://cdn.openmontage.com/films/space-odyssey-2077/v1.2.0/prompts/templates.json"
    }
  },
  "technicalRequirements": {
    "videoFormat": "mp4",
    "codec": ["h264", "h265"],
    "minimumResolution": "1920x1080",
    "frameRate": [24, 30],
    "maxFileSizeBytes": 524288000,
    "audioCodec": "aac",
    "audioRequired": false
  },
  "compatibleTools": ["ComfyUI", "Runway Gen-3", "Pika 2.0", "Kling"]
}
```

### 3.3 Metadata Indexing

On-chain events from all five contracts are indexed for efficient querying. Two complementary approaches are used:

**The Graph subgraph (Phase 2+):**

A subgraph indexes events from all five contracts and exposes a GraphQL API. Example entity schema:

```graphql
type Film @entity {
  id: ID!           # filmId as string
  title: String!
  director: Bytes!  # address
  mergedShots: Int!
  totalShots: Int!
  clips: [Clip!]! @derivedFrom(field: "film")
  stdLibVersion: String!
  status: String!
}

type Clip @entity {
  id: ID!                 # clipId as string
  film: Film!
  shotId: Int!
  creator: Bytes!
  videoCID: String!
  durationMs: Int!
  status: String!         # Submitted | Merged | Archived
  backers: [BackerRecord!]! @derivedFrom(field: "clip")
  creatorBond: BigInt!
  timestamp: Int!
}

type BackerRecord @entity {
  id: ID!           # clipId-backerAddress
  clip: Clip!
  backer: Bytes!
  valuationUsdc: BigInt!
  isWinner: Boolean!
}

type ShotMarket @entity {
  id: ID!           # shotId as string
  film: Film!
  phase: String!
  commitDeadline: Int!
  revealDeadline: Int!
  winningClip: Clip
  totalLosingCreatorBonds: BigInt!
  rewardPool: BigInt!
}
```

**Custom indexer (Phase 1 and fallback):**

A lightweight Node.js service polls Base RPC for new blocks, processes relevant contract events, and writes to a PostgreSQL database. This is simpler to operate during Phase 1 before The Graph subgraph is production-ready.

### 3.4 Standard Library Version Control

**SemVer conventions:**

| Increment | Meaning | Example |
|-----------|---------|---------|
| Major (`x.0.0`) | Incompatible change — character LoRA retrained from scratch; style guide fully revised | 1.0.0 → 2.0.0 |
| Minor (`x.y.0`) | Backward-compatible addition — new character added, new voice model | 1.0.0 → 1.1.0 |
| Patch (`x.y.z`) | Bug fix — corrected trigger word, fixed corrupted file | 1.0.0 → 1.0.1 |

**Policy rules:**
- A major version bump deprecates all prior versions for new submissions
- Minor and patch versions do not auto-deprecate prior versions
- The director may manually deprecate any version by calling `StandardLibrary.deprecateVersion()`
- Submissions that reference a deprecated version are rejected at the contract level
- Existing merged clips retain their declared version in metadata even after that version is deprecated (no retroactive effect on already-merged work)

**Validation flow:**

```
Creator submits clip with stdLibVersion = "1.2.0"
         │
         ▼
Bot backend: downloads manifest for v1.2.0
             verifies asset hashes match CDN files
             runs format/resolution/duration checks
             (Phase 2) runs AI consistency score
         │
         ▼
ShotMarket.submitClip(..., "1.2.0", ...)
         │
         ▼
StandardLibrary.verifyCompliance(filmId, "1.2.0")
  → checks version exists for filmId
  → checks version is not deprecated
  → returns true / reverts with VersionInvalid error
```

---

## 4. Application Layer

### 4.1 REST API Design

All endpoints require JWT authentication (issued after wallet signature verification) except public read endpoints.

**Base URL:** `https://api.openmontage.com/v1`

**Authentication:**

```
POST /auth/challenge
  Response: { nonce: string, message: string }

POST /auth/verify
  Body:    { address: string, signature: string, nonce: string }
  Response: { token: string (JWT), expiresAt: number }
```

**Films:**

```
GET  /films
  Query: ?status=active&page=1&limit=20
  Response: Film[] with pagination metadata

GET  /films/:filmId
  Response: Film metadata + merged shot list

POST /films
  Auth: required (Director role)
  Body: { title, totalShots, stdLibVersion, bondAmount }
  Response: { filmId, txHash }

GET  /films/:filmId/shots
  Query: ?status=open|claimed|merged
  Response: Shot[] with current status and claim info

GET  /films/:filmId/shots/:shotId
  Response: Shot details + all submissions + market phase
```

**Clips:**

```
POST /films/:filmId/shots/:shotId/submit
  Auth: required (Creator role)
  Body: multipart/form-data { video: File, notes: string, stdLibVersion: string, generationMethod: string }
  Flow: validate → upload to IPFS → call contract → return clipId
  Response: { clipId, clipNFT, ipfsCID, txHash }

GET  /clips/:clipId
  Response: Clip NFT metadata + video URL + backer list (post-resolution)
```

**Shot Markets:**

```
GET  /markets/:shotId
  Response: { phase, commitDeadline, revealDeadline, clips: [...], commitmentCount, revealCount }

POST /markets/:shotId/commit
  Auth: required (Backer role)
  Body: { commitment: bytes32 (hex), depositUsdc: number }
  Response: { txHash, commitDeadline }

POST /markets/:shotId/reveal
  Auth: required
  Body: { clipIds: number[], valuations: number[], salt: bytes32 (hex) }
  Response: { txHash, currentLeaderboard: [...] }

POST /markets/:shotId/resolve
  Auth: required (any)
  Response: { winningClipId, rewardPool, txHash }

POST /markets/:shotId/claim
  Auth: required
  Response: { claimedAmount, txHash }
```

**Earnings:**

```
GET  /earnings
  Auth: required
  Response: {
    activeClips: [...],
    estimatedMonthlyRevenue: number,
    claimableNow: { creator: number, backer: number, director: number },
    lifetimeEarned: number
  }

POST /earnings/claim/:filmId
  Auth: required
  Response: { claimedAmount, txHash }
```

**Standard Library:**

```
GET  /stdlib/:filmId
  Response: { currentVersion, manifestHash, assets: [...] with cdnUrls }

GET  /stdlib/:filmId/:version
  Response: Version-specific manifest

POST /stdlib/:filmId/publish
  Auth: required (Director role)
  Body: { semver, manifestHash, contributors: [...] }
  Response: { txHash }
```

**Error format (all endpoints):**

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Video duration 14.3s exceeds shot maximum of 12.5s",
    "details": { "durationMs": 14300, "maxDurationMs": 12500 }
  }
}
```

### 4.2 Discord Bot Backend Architecture

**Technology stack:** Node.js 20 LTS, discord.js v14, ethers.js v6, PostgreSQL (user/wallet mapping, market state cache), Redis (rate limiting, session cache).

**Service layout:**

```
discord-bot/
├── src/
│   ├── commands/          # slash command handlers (one file per command)
│   ├── events/            # Discord event listeners (interactionCreate, guildMemberAdd, ...)
│   ├── services/
│   │   ├── web3.ts        # ethers.js provider, contract instances, tx building
│   │   ├── wallet.ts      # Discord ↔ wallet mapping CRUD
│   │   ├── ipfs.ts        # IPFS upload via Pinata/web3.storage
│   │   ├── arweave.ts     # Arweave upload via Bundlr/Turbo
│   │   ├── validation.ts  # video format, resolution, duration, stdlib checks
│   │   └── embedBuilder.ts # Discord embed factory
│   ├── listeners/
│   │   └── contractEvents.ts  # blockchain event listener → Discord notifications
│   └── index.ts
├── prisma/schema.prisma   # ORM schema
└── package.json
```

**Discord → on-chain transaction translation layer:**

Every Discord command that triggers an on-chain action follows this flow:

```
User runs /submit in Discord
         │
         ▼
Command handler (commands/submit.ts)
  1. Look up user's wallet address via wallet mapping DB
  2. Validate video file (format, resolution, duration, stdlib version)
  3. Upload video to IPFS → get CID
  4. Upload video to Arweave → get TX ID
  5. Compute keccak256(rawVideoBytes) → videoHash
  6. Build tx: ShotMarket.submitClip(shotId, CID, videoHash, stdLibVersion, method, durationMs)
  7. If user has WalletConnect session:
       → send tx through WalletConnect relay (user signs in their wallet)
     Else (custodial relay, Phase 1):
       → bot-controlled relay key signs and submits tx on behalf of user
       → user must have pre-authorized the relay address in their wallet settings
  8. Wait for tx confirmation (1 block on Base)
  9. Fetch minted clipId from ClipSubmitted event
  10. Post submission embed to #submissions channel
  11. DM creator confirmation with Clip NFT ID and IPFS link
```

**Discord User ID to wallet address mapping:**

The mapping is stored in the application database (PostgreSQL), not on-chain. Collab.Land performs the cryptographic wallet ownership verification; the bot reads the resulting Discord role (`@Verified-Voter`) as the signal that a wallet address is authenticated.

```
Database table: user_wallets
  discord_user_id  VARCHAR(20)  PRIMARY KEY
  wallet_address   VARCHAR(42)  NOT NULL
  verified_at      TIMESTAMP    NOT NULL
  collab_session   JSONB        (Collab.Land verification receipt)
  discord_tag      VARCHAR(100)

Lookup by command: wallet_address = getWallet(interaction.user.id)
```

**Wallet-less relay (Phase 1 fallback):**

In Phase 1, users who cannot maintain a WalletConnect session may delegate transaction submission to a bot-controlled relay wallet. The relay wallet address is disclosed publicly; it has no withdrawal permissions — it can only call specific whitelisted contract functions (submit, commit, reveal, claim). Users authorize the relay in a one-time `/authorize-relay` flow that emits an on-chain approval event.

**Blockchain event listener → Discord notifications:**

A persistent listener (`contractEvents.ts`) subscribes to contract events via WebSocket RPC (Alchemy WebSockets on Base). On each event, it posts a formatted embed to the appropriate Discord channel:

| Contract Event | Discord Notification |
|---------------|---------------------|
| `ClipSubmitted` | Post submission embed to film's `#submissions` |
| `BackingCommitted` | Update commitment count in `#shot-markets` embed |
| `MarketResolved` | Announce winner in `#main-branch`; DM all participants |
| `RevenueClaimed` | DM claimant with claimed amount and transaction link |
| `FilmCompleted` | Announce in `#announcements` and film's `#film-overview` |
| `ClipArchived` | DM incumbent creator; update `#main-branch` |
| `ChallengeOpened` | Ping incumbent creator; open challenge thread in `#shot-markets` |

### 4.3 Video Assembly Engine

The video assembly engine produces a complete, streamable film from the current set of merged Clip NFTs.

**Real-time sequence assembly:**

```
1. Frontend requests /stream/:filmId
2. Assembly engine fetches film manifest:
     - Query ShotMarket.getCurrentMergedClip(filmId, shotId) for each shot
     - OR read from indexed database (updated by event listener on each MarketResolved)
3. Build ordered playlist:
     [{ shotId: 1, clipId: 0xabc, videoCID: "Qm...", durationMs: 8500 },
      { shotId: 2, clipId: 0xdef, videoCID: "Qm...", durationMs: 12300 }, ...]
4. For each segment:
     - Construct CDN URL from IPFS CID
     - Generate HLS segment entry (.ts file reference)
5. Return HLS master playlist (m3u8) to frontend player
```

**HLS playlist structure:**

```
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:15
#EXT-X-MEDIA-SEQUENCE:0

#EXTINF:8.500, Shot 001 by 0x456...
https://cdn.openmontage.com/clips/QmXyZ.../720p.ts
#EXTINF:12.300, Shot 002 by 0x789...
https://cdn.openmontage.com/clips/QmAbc.../720p.ts
...
#EXT-X-ENDLIST
```

**Adaptive bitrate variants:**

| Profile | Resolution | Bitrate | Use Case |
|---------|-----------|---------|---------|
| 1080p | 1920×1080 | 8 Mbps | Desktop, high-bandwidth |
| 720p | 1280×720 | 4 Mbps | Default web playback |
| 480p | 854×480 | 2 Mbps | Mobile, low-bandwidth |
| 360p | 640×360 | 1 Mbps | Very low-bandwidth fallback |

**Creator attribution overlay:**

Each HLS segment's metadata includes the creator's address. The player renders a semi-transparent overlay at the bottom of each segment:

```
Shot 042 — @NeonArtist (0xa3f...8c2)  |  Runway Gen-3 + protagonist LoRA v1.2.0
```

**Segment caching:**

Popular film segments are pre-transcoded to all four HLS profiles and cached at CDN edge nodes. Cache invalidation occurs when `ClipMerged` or `ClipArchived` events are detected — the old segment's CDN cache is purged and the new winner's files are pre-warmed.

### 4.4 Quality Validation Pipeline

Validation runs on the bot backend before any on-chain call is made. A failed validation returns an error embed to the creator without spending any gas.

**Phase 1 automated checks:**

| Check | Rule | Error Code |
|-------|------|-----------|
| Container format | Must be MP4 | `FORMAT_INVALID` |
| Video codec | H.264 or H.265 | `CODEC_INVALID` |
| Audio codec | AAC or no audio track | `AUDIO_CODEC_INVALID` |
| Resolution | Minimum 1920×1080 | `RESOLUTION_TOO_LOW` |
| Frame rate | 24 fps or 30 fps (±0.1 fps tolerance) | `FRAMERATE_INVALID` |
| Duration | Within shot's min/max range (±0.5s tolerance) | `DURATION_OUT_OF_RANGE` |
| File size | Maximum 500 MB | `FILE_TOO_LARGE` |
| File integrity | File is downloadable and not corrupted (ffprobe check) | `FILE_CORRUPT` |
| Stdlib version | Must match film's current (or non-deprecated) version | `STDLIB_VERSION_INVALID` |

**Phase 2 AI consistency scoring:**

An AI scoring service runs after basic format checks pass:
- **Face consistency**: Runs character LoRA inference on a set of extracted keyframes; compares facial embedding distance to LoRA reference embeddings. Clips with < 70% face similarity score are flagged and rejected.
- **Style consistency**: Computes CLIP embedding distance between clip keyframes and the art direction reference images. Clips below a configurable style similarity threshold are flagged.
- **Scoring output**: A `consistency_score` (0–100) is stored in the Clip NFT metadata. Clips below 70 are rejected; 70–85 are accepted with a warning watermark in the submission embed; 85+ are accepted with a "High Consistency" badge.

---

## 5. Frontend Layer

### 5.1 Phase 1 — Discord Bot (MontageBot)

**Architecture:**

```
┌────────────────────────────────────────────────────────────────────┐
│                        Discord (User UI)                           │
│  Slash commands │ Embeds │ DMs │ Buttons │ Threads                 │
└──────────────────────────┬─────────────────────────────────────────┘
                           │ Discord Gateway (WebSocket)
┌──────────────────────────▼─────────────────────────────────────────┐
│                    MontageBot Backend (Node.js)                     │
│  discord.js command router                                          │
│  ↓                                                                  │
│  Business logic (validation, IPFS upload, wallet lookup)           │
│  ↓                                                                  │
│  Web3 service (ethers.js v6)                                        │
│  ↓                                                                  │
│  Base L2 RPC (Alchemy / Infura WebSocket)                          │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
         ┌─────────────────┴──────────────────┐
         ▼                                    ▼
┌────────────────────┐            ┌───────────────────────┐
│  Smart Contracts   │            │  IPFS / Arweave       │
│  (Base L2)         │            │  (Video + Assets)     │
└────────────────────┘            └───────────────────────┘
```

**Command processing flow (example: `/back-shot`):**

```
1. User: /back-shot film:space-odyssey shot:042 deposit:100 rank:clipA,clipB,clipC
2. discord.js fires interactionCreate event
3. Router dispatches to commands/backShot.ts
4. Handler:
   a. Defer reply (gives 15 seconds for async processing)
   b. Look up wallet address for interaction.user.id
   c. Check market phase: must be in Commit phase
   d. Validate deposit: 10 <= 100 <= 5000
   e. Generate valuation menu from rank: clipA=100, clipB=50, clipC=25 (geometric decay)
   f. Compute commitment: keccak256(abi.encodePacked(clipA, 100e6, clipB, 50e6, clipC, 25e6, saltBytes))
   g. Check backer's USDC balance >= 100 USDC (deposit = max valuation)
   h. Build tx: USDC.approve(ShotMarket, 100e6) + ShotMarket.commitBacking(shotId, commitment, 100e6)
   i. Submit via WalletConnect relay or custodial relay
   j. Wait for confirmation
   k. Store commitment + menu in local DB for reveal
   l. Build confirmation embed
5. editReply(embed) — shown in channel
6. DM backer: commitment hash + menu summary + reveal deadline + reminder instructions
```

**Embed rendering standards:**

All embeds follow a consistent visual structure:
- Color: `#7C3AED` (violet) for success states; `#EF4444` (red) for errors; `#F59E0B` (amber) for warnings
- Footer: `OpenMontage • openmontage.com`
- Timestamp: always included
- Action buttons: labeled clearly (e.g., `[Watch Video]`, `[Claim Now]`, `[View on Explorer]`)

### 5.2 Phase 2 — Web Dashboard

**Technology stack:** React 18, Next.js 14 (App Router), TypeScript, Tailwind CSS, wagmi v2 (wallet integration), viem (Ethereum types and calls), TanStack Query (data fetching/caching), The Graph (on-chain data queries).

**Core pages and components:**

| Page | Route | Key Components |
|------|-------|---------------|
| Film catalog | `/films` | Film cards with progress bar, sort/filter, search |
| Film detail | `/films/:filmId` | Shot board (kanban), Standard Library panel, film player |
| Shot detail | `/films/:filmId/shots/:shotId` | Side-by-side clip comparison, backing UI |
| Shot Market | `/markets/:shotId` | Menu commit/reveal form, live SelectionWeight leaderboard, payout calculator |
| Creator dashboard | `/dashboard/creator` | Active clips, merged clips, revenue chart, claim button |
| Backer dashboard | `/dashboard/backer` | Portfolio of backing positions, streaming revenue, competition history |
| Director dashboard | `/dashboard/director` | Film management, Standard Library version control, treasury view |
| Film player | `/watch/:filmId` | HLS video player with segment attribution overlay |
| Standard Library | `/stdlib/:filmId` | Asset browser, version history, download links |

**Dual-interface synchronization:**

The web dashboard and Discord bot share the same backend API and PostgreSQL database. Changes made via Discord are immediately reflected in the web dashboard and vice versa:

- Shot claim via `/claim-shot` on Discord → `#shots-available` list updates + web shot board updates (via WebSocket push from backend)
- Clip submission via web dashboard → `#submissions` Discord embed posted automatically
- Backing committed on web → commitment count in Discord `#shot-markets` embed updates

**Wallet connection:** wagmi v2 supports MetaMask, WalletConnect v2, and Coinbase Wallet out of the box. No private keys are stored. All transactions are signed in the user's wallet and broadcast from the frontend directly to Base L2.

### 5.3 Phase 3 — Mobile Applications

**Target platforms:** iOS 16+ (Swift/SwiftUI), Android 12+ (Kotlin/Jetpack Compose). Cross-platform framework (React Native or Flutter) is under evaluation to reduce development cost.

**Core features for mobile:**
- Film browsing and HLS video playback
- Shot claiming and status tracking
- `/my-earnings` equivalent — revenue dashboard with push notifications
- Clip submission via URL input (generation happens on desktop AI tools; submission link is pasted)
- Backing via WalletConnect deep links (mobile wallet apps)

**Push notifications (Phase 3):**
- Shot Market resolution results
- Revenue claim available
- Clip challenged or replaced
- New film posted matching user's genre preferences

Firebase Cloud Messaging (FCM) handles Android push; APNs for iOS. The backend sends push tokens when users opt in via the mobile app.

---

## 6. Wallet Integration

### 6.1 Collab.Land Discord-Wallet Verification Flow

Collab.Land is the industry-standard bot for Discord-to-wallet verification, used by thousands of Web3 communities. It handles the cryptographic ownership proof so OpenMontage's bot does not need to implement wallet signing infrastructure.

```
Step 1: User joins OpenMontage Discord server
Step 2: User navigates to #verify-wallet channel
Step 3: User clicks [Connect Wallet] button (Collab.Land interaction)
Step 4: Collab.Land DMs user with a verification link
Step 5: User opens link in browser; connects MetaMask/WalletConnect/Coinbase Wallet
Step 6: User signs a gasless EIP-712 message to prove ownership:
  {
    "domain": { "name": "Collab.Land", "version": "1" },
    "message": { "nonce": "...", "discordUserId": "...", "timestamp": ... }
  }
Step 7: Collab.Land verifies signature, adds @Verified-Voter role to user
Step 8: MontageBot reads the role assignment event, stores Discord ID → wallet address in DB
Step 9: User can now use all platform commands requiring wallet
```

### 6.2 Role Gating Mechanism

Roles are assigned by Collab.Land or MontageBot based on on-chain and in-platform conditions:

| Role | Assignment Trigger | Revocation Condition |
|------|-------------------|---------------------|
| `@Verified-Voter` | Wallet verified via Collab.Land | Wallet disconnected or verification expired |
| `@Creator` | First successful `/claim-shot` | Repeated claim expirations (3+ in 30 days) trigger temporary suspension |
| `@Backer` | Wallet verified + USDC balance > 10 USDC on Base | Automatic; re-checked on each backing attempt |
| `@Director` | Film Creation Bond deposited + Film NFT minted | Automatic; tied to Film NFT ownership |

### 6.3 Supported Wallets

| Wallet | Phase 1 (Discord) | Phase 2 (Web) | Phase 3 (Mobile) |
|--------|------------------|---------------|-----------------|
| MetaMask | Via Collab.Land web verification | Browser extension + MetaMask SDK | MetaMask Mobile deep links |
| WalletConnect | WalletConnect relay session | wagmi/WalletConnect v2 | WalletConnect v2 |
| Coinbase Wallet | Via Collab.Land | Coinbase Wallet SDK (wagmi) | Coinbase Wallet app |
| Phantom | Via Collab.Land (EVM mode) | wagmi | — |
| Ledger | Via MetaMask Ledger bridge | Via wagmi | — |

### 6.4 On-Chain Identity Mapping

```
Layer 1 — Discord Identity:
  Discord User ID  (e.g., 123456789012345678)
  Provided by Discord; stable identifier

Layer 2 — Application Mapping (PostgreSQL):
  discord_user_id  ──→  wallet_address
  Established by Collab.Land verification
  Maintained in application database (not on-chain)

Layer 3 — On-Chain Identity:
  wallet_address  ──→  Clip NFTs owned (ClipNFT.balanceOf)
                  ──→  Film NFTs owned (FilmNFT.balanceOf)
                  ──→  Claimable balance (Treasury.getClaimable)
                  ──→  Active market commitments (ShotMarket.commitments)

Source of Truth: Blockchain
  The blockchain holds all financial state.
  The application database holds only the Discord-to-wallet mapping.
  If the database is lost, users can re-verify via Collab.Land to restore the mapping.
```

### 6.5 Security Considerations

- No private keys are stored by the platform in any database or environment variable accessible to application code. The custodial relay wallet (Phase 1) is held exclusively by the bot operator and authorized only for specific whitelisted functions.
- Wallet signatures are verified using `ethers.verifyMessage()` or `ecrecover` — the address recovered from the signature must match the claimed address.
- Users may update their wallet address by re-running the Collab.Land verification flow. The old mapping is replaced; no funds are affected (on-chain balances are tied to the wallet, not the Discord ID).
- The application never requests a transaction signature for anything other than whitelisted contract functions. Embeds and DMs never ask for seed phrases or private keys.
- Multi-wallet support (Phase 2): Users may link multiple wallets to one Discord ID. Revenue claims must be made from the specific wallet that holds the relevant NFTs.

---

## 7. Security Design

### 7.1 Smart Contract Audit Strategy

All five core contracts undergo two independent audits before mainnet deployment:

**Audit 1 — OpenZeppelin:**
- Focus: ERC-721 compliance, access control, upgrade proxy patterns, OpenZeppelin library integration correctness
- Deliverable: Full audit report with severity classifications (Critical / High / Medium / Low / Informational)
- Timeline: 4 weeks before mainnet deployment

**Audit 2 — Trail of Bits:**
- Focus: Adversarial economic exploit analysis, MEV risks, commit-reveal correctness, reentrancy, integer overflow/underflow, oracle manipulation, governance attacks
- Deliverable: Full audit report + Echidna fuzzing results
- Timeline: 4 weeks before mainnet deployment (may overlap with OpenZeppelin)

**Remediation policy:**
- Critical findings: Must be resolved before deployment. Deployment is blocked until all Critical findings have a verified fix.
- High findings: Must be resolved or have a documented accepted risk with mitigation rationale before deployment.
- Medium findings: Tracked in a public GitHub issue with remediation timeline. May deploy with Medium findings if mitigated by defense-in-depth.
- Low / Informational: Addressed at team discretion; all tracked publicly.

All audit reports are published in full to the OpenMontage GitHub repository.

### 7.2 Reentrancy Attack Protection

`Treasury.sol` presents the highest reentrancy risk because it transfers ERC-20 (USDC) tokens. All payout functions implement:

**Checks-Effects-Interactions pattern (mandatory):**

```solidity
function claimRevenue(uint256 filmId) external nonReentrant returns (uint256) {
    // CHECKS
    uint256 claimable = claimableBalances[filmId][msg.sender];
    require(claimable > 0, "Nothing to claim");

    // EFFECTS — zero the balance before any external call
    claimableBalances[filmId][msg.sender] = 0;
    totalClaimed[filmId] += claimable;

    // INTERACTIONS — external transfer is last
    USDC.safeTransfer(msg.sender, claimable);

    emit RevenueClaimed(filmId, msg.sender, claimable, 0, 0, 0);
    return claimable;
}
```

**Defense-in-depth:** All external-facing functions in `Treasury.sol` and `ShotMarket.sol` also inherit OpenZeppelin's `ReentrancyGuard` and use the `nonReentrant` modifier. This catches cases where the checks-effects-interactions pattern is accidentally violated during future contract modifications.

### 7.3 Oracle Risk Management

Off-chain data enters the system in two places:

**ETH/USD price conversion:**
- Used to display USDC-denominated values in ETH or USD equivalents in the frontend
- Source: Chainlink Data Feeds (`ETH/USD` on Base)
- Staleness check: if `block.timestamp - updatedAt > 24 hours`, revenue distribution pauses and the keeper bot raises an alert

**Revenue event reporting (custom Chainlink Functions):**
- Streaming fees and advertising revenue are reported from off-chain systems
- A Chainlink Functions DON executes a verified computation that fetches revenue data from the platform's reporting API and writes it on-chain
- Multiple Chainlink nodes must agree (DON consensus) before the value is accepted
- Manual override: Team multisig can pause automated reporting and submit revenue data manually if the Chainlink Functions integration fails

### 7.4 Contract Upgrade Strategy

| Phase | Upgrade Pattern | Admin | Governance |
|-------|----------------|-------|-----------|
| Phase 1 (testnet) | OpenZeppelin TransparentUpgradeableProxy | 3-of-5 Gnosis Safe | Team only |
| Phase 2 (mainnet early) | Same TransparentProxy | 3-of-5 Gnosis Safe | Team + community proposal |
| Phase 3 (DAO) | Option A: Lock proxies permanently (immutable) | N/A | DAO vote required to choose |
| Phase 3 (DAO) | Option B: Migrate to new contracts with 30-day window | N/A | DAO vote required to choose |

**Upgrade process (Phase 1–2):**

1. New implementation contract deployed
2. Security review of the diff (internal + optional mini-audit for significant changes)
3. 3-of-5 multisig proposes upgrade via Gnosis Safe transaction
4. 48-hour timelock before execution (gives community time to react)
5. After timelock: remaining signers confirm, upgrade executes

**Storage layout:** All upgradeable contracts use a fixed storage layout with reserved gap slots to prevent storage collisions between versions:

```solidity
// Reserved storage gap — do not remove
uint256[50] private __gap;
```

### 7.5 Emergency Pause Mechanism

All five contracts inherit OpenZeppelin's `Pausable`. The `pause()` and `unpause()` functions are restricted by the `onlyPauser` modifier.

```solidity
modifier onlyPauser() {
    require(
        hasRole(PAUSER_ROLE, msg.sender),
        "Caller is not a pauser"
    );
    _;
}
```

**Pause authority timeline:**

| Phase | Who Can Pause | Who Can Unpause |
|-------|--------------|----------------|
| Phase 1–2 | 3-of-5 Gnosis Safe (any one signer for pause; 3-of-5 for unpause) | 3-of-5 Gnosis Safe |
| Phase 3 | DAO Governor contract | DAO Governor contract (requires proposal + vote) |

**When pause is triggered:**
- Any contract can be paused independently (e.g., pause `Treasury.sol` while `ShotMarket.sol` continues)
- Paused contracts reject all state-changing calls but continue to serve view functions
- Event `Paused(address account)` is emitted for transparency; Discord bot alerts are triggered automatically

**Unpause process:**
- Team publicly announces the reason for pause and remediation plan
- Code fix is reviewed and deployed to a new implementation
- Proxy upgrade executed (with timelock)
- Contract unpaused after upgrade confirmation

### 7.6 Key Management

**3-of-5 Gnosis Safe multisig:**

The Gnosis Safe at [gnosis-safe address, to be published] controls all privileged operations in Phase 1–2:

| Operation | Authorization Required |
|-----------|----------------------|
| Proxy upgrades | 3-of-5 signatures |
| Emergency pause | 1-of-5 signature (any signer) |
| Emergency unpause | 3-of-5 signatures |
| Treasury parameter changes | 3-of-5 signatures |
| Standard Library admin | 3-of-5 signatures |
| New multisig signer addition/removal | 4-of-5 signatures |

**Signer requirements:**
- 5 signers from the founding team and trusted advisors
- Geographically distributed (no two signers in the same country)
- Each signer uses a hardware wallet (Ledger or Trezor) — no hot wallets
- Signer identities are publicly disclosed in the repository

**Key rotation:** If a signer's key is compromised, the remaining signers execute a signer removal + replacement transaction on Gnosis Safe. The compromised key's signing rights are removed before any malicious transaction can gather the threshold.

---

## 8. Standard Library Technical Specification

### 8.1 Asset Format Requirements

**Character LoRA Models:**

| Property | Requirement |
|---------|------------|
| Format | `.safetensors` only (no `.pt`, `.ckpt`, `.bin` for security) |
| Maximum file size | 2 GB per file |
| Metadata file | Required `.json` sidecar with training data provenance, base model compatibility, LoRA weight recommendation, trigger words |
| Base model compatibility | Must specify compatible base model(s) (e.g., SDXL 1.0, SD 1.5, Wan 2.1) |
| Training data attestation | Director must attest (on-chain, via `StandardLibrary.publishVersion()`) that training data is licensed or original |

**ControlNet Configurations:**

| Property | Requirement |
|---------|------------|
| Format | `.safetensors` (weights) + `.json` (config) |
| Maximum file size | 1 GB per model |
| Supported types | openpose, depth, normal, lineart, canny, scribble |
| Config schema | Must include `controlnet_type`, `resolution`, `guidance_scale` |

**Voice Clone Models:**

| Property | Requirement |
|---------|------------|
| Formats | `.pth` (RVC), `.onnx` (ONNX export), `.bin` (Bark/XTTS) |
| Maximum file size | 500 MB per model |
| Sample rate | 22050 Hz or 44100 Hz |
| Language tags | ISO 639-1 language codes for multi-language support |
| Voice rights | Director must attest voice is licensed or synthesized (not cloned from real person without consent) |

**Style Reference Images:**

| Property | Requirement |
|---------|------------|
| Formats | `.png`, `.jpg` |
| Minimum resolution | 512×512 pixels |
| Maximum file size | 10 MB per image |
| Color space | sRGB |
| Usage | At least one image per major visual category: character design, environment/location, lighting reference |

**Prompt Templates:**

```json
{
  "$schema": "https://openmontage.com/schemas/prompts-v1.json",
  "version": "1.0",
  "templates": {
    "character_shot": {
      "positive": "[PROTAGONIST_LORA] in [LOCATION], [ACTION], cinematic lighting, [STYLE_TOKENS], ultra detailed, 4k",
      "negative": "blurry, low quality, different face, wrong hair color",
      "variables": ["LOCATION", "ACTION", "STYLE_TOKENS"],
      "required_loras": ["protagonist"]
    },
    "environment_shot": {
      "positive": "[LOCATION_DESCRIPTION], [STYLE_TOKENS], establishing shot, wide angle, cinematic",
      "negative": "people, characters, blurry",
      "variables": ["LOCATION_DESCRIPTION", "STYLE_TOKENS"]
    }
  }
}
```

### 8.2 Dependency Manifest Schema (JSON)

Full schema with validation rules:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://openmontage.com/schemas/stdlib-manifest-v1.json",
  "title": "OpenMontage Standard Library Manifest",
  "type": "object",
  "required": ["filmId", "version", "publishedAt", "publisher", "manifestHash", "assets", "technicalRequirements"],
  "properties": {
    "filmId": { "type": "string" },
    "version": {
      "type": "string",
      "pattern": "^(0|[1-9]\\d*)\\.(0|[1-9]\\d*)\\.(0|[1-9]\\d*)$"
    },
    "publishedAt": { "type": "string", "format": "date-time" },
    "publisher": { "type": "string", "pattern": "^0x[0-9a-fA-F]{40}$" },
    "manifestHash": { "type": "string", "pattern": "^0x[0-9a-fA-F]{64}$" },
    "assets": {
      "type": "object",
      "properties": {
        "characters": {
          "type": "array",
          "items": {
            "type": "object",
            "required": ["name", "file", "hash", "sizeBytes", "cdnUrl", "triggerWords"],
            "properties": {
              "name": { "type": "string" },
              "description": { "type": "string" },
              "file": { "type": "string", "pattern": "\\.safetensors$" },
              "hash": { "type": "string", "pattern": "^sha256:[0-9a-f]{64}$" },
              "sizeBytes": { "type": "integer", "maximum": 2147483648 },
              "cdnUrl": { "type": "string", "format": "uri" },
              "ipfsCID": { "type": "string" },
              "loraWeight": { "type": "number", "minimum": 0, "maximum": 1 },
              "triggerWords": { "type": "array", "items": { "type": "string" } }
            }
          }
        },
        "controlnets": { "type": "array" },
        "voices": { "type": "array" },
        "style": { "type": "array" },
        "prompts": { "type": "object" }
      }
    },
    "technicalRequirements": {
      "type": "object",
      "required": ["videoFormat", "codec", "minimumResolution", "frameRate", "maxFileSizeBytes"],
      "properties": {
        "videoFormat": { "type": "string", "enum": ["mp4"] },
        "codec": { "type": "array", "items": { "enum": ["h264", "h265"] } },
        "minimumResolution": { "type": "string", "pattern": "^\\d+x\\d+$" },
        "frameRate": { "type": "array", "items": { "type": "number" } },
        "maxFileSizeBytes": { "type": "integer", "maximum": 524288000 },
        "audioCodec": { "type": "string", "enum": ["aac"] },
        "audioRequired": { "type": "boolean" }
      }
    }
  }
}
```

### 8.3 Compliance Validation Standards

**Similarity scoring (Phase 2):**

| Score Range | Outcome | Notes |
|-------------|---------|-------|
| ≥ 85 | Accepted with "High Consistency" badge | Displayed in submission embed |
| 70–84 | Accepted with consistency warning in embed | Backers can see score |
| < 70 | Rejected | Creator receives specific feedback (which character failed, which style element) |

**Rejection criteria (Phase 1 — rule-based):**

- Submission references a non-existent Standard Library version for the film
- Submission references a deprecated Standard Library version
- Video metadata declares incorrect `stdLibVersion` string
- File hash does not match the file contents (corrupted upload)

**Rejection criteria (Phase 2 — AI-assisted):**

- Character face embedding distance > threshold for any main character appearing in the clip
- Style CLIP embedding distance > threshold from art direction references
- Detected generation artifacts indicating asset was not used (e.g., character appears with different hair when protagonist LoRA forces specific hair)

### 8.4 Standard Library CDN Architecture

```
Director uploads assets to Standard Library
         │
         ▼
Bot backend:
  1. Validate file formats and sizes
  2. Compute sha256 hash per asset
  3. Upload to origin object storage (S3-compatible, e.g., Cloudflare R2)
  4. Register manifest hash on-chain via StandardLibrary.publishVersion()
         │
         ▼
CDN Configuration:
  - Cloudflare CDN with 200+ global PoPs
  - Cache-Control: public, max-age=31536000, immutable
    (asset URLs include version in path; never mutate)
  - URL structure:
    https://cdn.openmontage.com/stdlib/{filmId}/{version}/{assetPath}
  - Fallback: AWS CloudFront with same URL structure (CNAME swap)
         │
         ▼
Creator downloads assets:
  1. /standard-library film:{filmId} → returns manifest with cdnUrls
  2. Creator fetches directly from CDN (< 50ms at edge)
  3. Creator verifies: sha256(downloadedFile) == manifest.assets[*].hash
```

---

## 9. Data Flow and Integration

### 9.1 Complete Data Flow — From Submission to On-Chain to Distribution

```
[CREATOR ACTION]
Creator generates clip using Standard Library assets
         │
         ▼
[BOT VALIDATION]
/submit command triggers quality validation pipeline:
  ✓ MP4 format, H.264/H.265 codec
  ✓ ≥ 1080p resolution
  ✓ 24 or 30 fps
  ✓ Duration within shot range (±0.5s)
  ✓ File size ≤ 500 MB
  ✓ File integrity (ffprobe)
  ✓ Standard Library version valid
  (Phase 2) ✓ AI consistency score ≥ 70
         │ All checks pass
         ▼
[STORAGE]
Bot backend:
  → Compute keccak256(rawBytes)  [videoHash]
  → Upload to IPFS              [CID]
  → Upload to Arweave           [TX ID]
         │
         ▼
[BLOCKCHAIN — SUBMISSION]
ShotMarket.submitClip(shotId, CID, videoHash, stdLibVersion, method, durationMs)
  → StandardLibrary.verifyCompliance(filmId, stdLibVersion) [internal call]
  → Deduct 50 USDC Creator Bond from creator
  → ClipNFT.submitClip(...) → mints Clip NFT #clipId to creator
  → emits ClipSubmitted, ClipEnteredMarket
         │
         ▼
[EVENT INDEXING]
Custom indexer / The Graph detects ClipSubmitted event
  → Updates shot's submission list in database
  → Bot posts submission embed to #submissions
         │
         ▼
[BACKER ACTIONS — COMMIT PHASE, 5 days]
For each backer:
  1. Backer constructs valuation menu off-chain: {clipA: v_A, clipB: v_B, ...}
  2. Backer computes commitment: keccak256(abi.encodePacked(clipA, v_A, clipB, v_B, ..., salt))
  3. ShotMarket.commitBacking(shotId, commitment, max(v_i))
     → max(v_i) USDC transferred from backer to ShotMarket
     → Hash stored on-chain; no valuations or preferences visible
     → On-chain: commitment counts per shot are publicly readable.
       Valuations, clip preferences, and backer identity are cryptographically
       hidden until reveal. The bot displays commitment counts to show
       participation level without revealing preference data.
         │
         ▼
[BACKER ACTIONS — REVEAL PHASE, 2 days]
For each backer:
  ShotMarket.revealMenuBacking(shotId, clipIds[], valuations[], salt)
    → Contract verifies: keccak256(encodePacked(clipIds, valuations, salt)) == stored hash
    → Verifies max(valuations) <= deposit
    → Valuations recorded for each clip in the menu
  Non-reveals: deposit forfeited to reward pool
         │
         ▼
[RESOLUTION — after reveal deadline]
Anyone calls ShotMarket.resolveMarket(shotId):
  For each clip:
    weight = sqrt(creatorBond) + Σ sqrt(backer_valuation_i_for_this_clip)
  Winner = clip with maximum SelectionWeight
  Ties broken by submission timestamp (earlier wins)
  → Settle backer payments: each backer pays v_i_winner
  → Refund excess: deposit - v_i_winner → backer
  → Record v_i_winner as streaming revenue stake
  → ClipNFT.mergeClip(winningClipId)
  → ClipNFT.archiveClip(losingClipId) for each loser
  → ClipNFT.registerBackers(...) for winning clip
  → FilmNFT.recordMerge(filmId)
  → Treasury.depositCompetitionTax(filmId, shotId, 5% × CB_l)
  → emits MarketResolved
         │
         ▼
[PAYOUT CLAIMS]
Winners and losers call claim functions:
  ShotMarket.claimWinnerReward(shotId) → returns Creator Bond + reward share
  ShotMarket.claimLoserCreatorRefund(shotId) → returns 75% of Creator Bond
         │
         ▼
[STREAMING REVENUE — ONGOING]
External revenue deposited periodically:
  Treasury.depositRevenue(filmId, amount)
Keeper bot triggers distribution:
  Treasury.distributeRevenue(filmId)
    → Reads merged clip list from ClipNFT
    → Allocates 75% Creator / 10% Director / 5% StdLib / 10% Backers
    → Updates claimableBalances per participant
Participants pull their earnings:
  Treasury.claimRevenue(filmId)
    → USDC transferred to caller
         │
         ▼
[CDN INVALIDATION & PLAYBACK UPDATE]
Event listener detects ClipMerged:
  → Invalidates old clip's CDN cache entries
  → Pre-warms new winner's HLS segments at CDN edge
  → Updates film's HLS master playlist
  → film.json manifest updated in IPFS (new CID published on-chain)
Frontend player:
  → Fetches updated m3u8 on next load or after 60-second TTL
```

### 9.2 Shot Market State Machine

```
         ┌─────────────────────────────────────────────────────────┐
         │                     INACTIVE                            │
         │   Market not yet opened for this shot                   │
         └─────────────────────┬───────────────────────────────────┘
                               │ Director calls openMarket()
                               ▼
         ┌─────────────────────────────────────────────────────────┐
         │                    SUBMISSION                           │
         │   Duration: director-configured (7–30 days, default 14) │
         │   Entry: creators call submitClip()                     │
         │   Exit conditions:                                      │
         │     A) submissionDeadline reached AND ≥2 clips exist    │
         │     B) submissionDeadline reached AND 1 clip → director │
         │        may call autoMerge() (no competition)            │
         │     C) submissionDeadline reached AND 0 clips → stays   │
         │        open until 1 clip submitted                      │
         └─────────────────────┬───────────────────────────────────┘
                               │ Condition A met
                               ▼
         ┌─────────────────────────────────────────────────────────┐
         │                     COMMIT                              │
         │   Duration: 5 days                                      │
         │   Entry: backers call commitBacking()                   │
         │   Min backing: 10 USDC | Max backing: 5,000 USDC        │
         │   Constraint: one menu per backer per shot              │
         │   Edge case: 0 backers at commit deadline →            │
         │     all clips have equal SelectionWeight (√50);        │
         │     earliest submission (lowest clipId) wins           │
         └─────────────────────┬───────────────────────────────────┘
                               │ commitDeadline reached
                               ▼
         ┌─────────────────────────────────────────────────────────┐
         │                     REVEAL                              │
         │   Duration: 2 days                                      │
         │   Entry: backers call revealMenuBacking()               │
         │   Non-reveals: deposit forfeited to reward pool         │
         └─────────────────────┬───────────────────────────────────┘
                               │ revealDeadline reached
                               ▼
         ┌─────────────────────────────────────────────────────────┐
         │                    RESOLVED                             │
         │   resolveMarket() callable by anyone                    │
         │   Winner: highest SelectionWeight(clip)                 │
         │   Actions:                                              │
         │     → Settle backer payments (v_i_winner)               │
         │     → Refund excess deposits (deposit - v_i_winner)     │
         │     → ClipNFT.mergeClip(winner)                         │
         │     → ClipNFT.archiveClip(losers)                       │
         │     → Treasury.depositCompetitionTax()                  │
         │     → Claims open for winners and losers                │
         └─────────────────────────────────────────────────────────┘
                               │
                    ┌──────────┴────────────────┐
                    │                           │
                    ▼                           ▼
       Normal shot: market closed    Shot challenged after 21-day cooldown:
                                     openChallenge() starts
                                     CHALLENGE MARKET (compressed timeline)
                                     Submission: 7 days (incumbent may improve)
                                     Commit: 3 days
                                     Reveal: 2 days
                                     Resolution: same quadratic formula +
                                       incumbent's synthetic weight advantage
```

### 9.3 Error Handling and Retry Strategy

**On-chain transaction failures:**

| Failure Type | Detection | Recovery |
|-------------|-----------|---------|
| Insufficient USDC balance | Pre-flight balance check before building tx | Bot informs user; provides Coinbase onramp link |
| Gas estimation failure | ethers.js `estimateGas()` throws | Bot retries with 10% higher gas limit; fails after 3 attempts |
| Nonce conflict | Transaction reverts with `nonce too low` | ethers.js `NonceManager` handles nonce sequencing automatically |
| Contract revert (business logic) | Receipt status = 0; parse revert reason | Bot reads revert message (using ethers.js `getContractError`) and returns human-readable explanation |
| Network congestion / timeout | Transaction not mined within 60 seconds | Bot bumps gas price 10% and rebroadcasts (EIP-1559 replacement tx) |
| RPC node failure | Connection error | Automatic failover to secondary RPC endpoint (Infura if Alchemy fails) |

**Storage failures:**

| Failure | Recovery |
|---------|---------|
| IPFS upload timeout | Retry 3× with exponential backoff; fallback to Pinata if web3.storage fails |
| Arweave upload failure | Mark for retry in background queue; clip submission proceeds with IPFS CID only (Arweave upload dequeued for retry) |
| CDN cache purge failure | Automatic retry by CDN provider; stale content has max 60-second TTL so self-corrects |

**Bot command failures:**

If a command fails after all retries, the bot:
1. Returns a descriptive error embed to the user in Discord
2. Logs the full error (stack trace, tx data, user ID) to the platform's centralized logging service
3. Increments an error counter metric; PagerDuty alert fires if error rate exceeds threshold

### 9.4 Monitoring and Alerting

**On-chain monitoring:**

| Metric | Tool | Alert Condition |
|--------|------|----------------|
| Contract event emission | Custom event listener | Any `Paused` event → immediate PagerDuty alert |
| Treasury USDC balance | Alchemy monitor | Balance < 10,000 USDC → warning |
| Market resolution lag | Custom checker | Market not resolved within 6 hours of revealDeadline → alert |
| Oracle staleness | Chainlink staleness check | Price feed > 24 hours old → pause revenue distribution |
| Failed transactions | Alchemy webhooks | Error rate > 1% in 5-minute window → alert |

**Application monitoring:**

| Metric | Tool | Alert Condition |
|--------|------|----------------|
| Bot response latency | Prometheus + Grafana | P95 latency > 5 seconds → warning |
| Bot error rate | Prometheus | Error rate > 0.5% → PagerDuty |
| IPFS upload success rate | Custom metrics | Success rate < 99% → alert |
| Discord WebSocket connection | discord.js events | Disconnect → auto-reconnect + alert if disconnected > 30 seconds |
| Database connection pool | pg-pool metrics | Pool exhausted → auto-scale + alert |
| CDN cache hit rate | Cloudflare Analytics | Cache hit rate < 90% → investigate origin load |

**Business metrics dashboard (Grafana):**

- Active films count and status distribution
- Daily submissions, merges, and challenges
- USDC volume in active Shot Markets
- Revenue distributed per day
- Creator count, backer count, director count (weekly active)
- Top films by viewer count and revenue

---

**Version**: 1.0
**Last Updated**: 2026-03-05
**Authors**: OpenMontage Core Team
