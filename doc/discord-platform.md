# OpenMontage Discord-First Platform Design

**Version 1.0 — Building on Discord Before Building from Scratch**

---

## 1. Strategy: Why Discord-First

### The Audience Is Already There

The AI creator community lives on Discord. Midjourney's server has 19M+ members. Runway, Pika, and Sora communities each host hundreds of thousands of active creators who already know how to generate video from prompts, load LoRAs, and iterate on AI output. These are exactly the people OpenMontage needs — and they are already logged in.

Building a standalone web app from day one means competing for attention against platforms creators already use daily. A Discord-first approach meets them where they are.

### Lower Friction, Faster Adoption

- **No new account**: Creators authenticate with their existing Discord identity
- **Familiar interface**: Channels, threads, embeds, and slash commands are second nature to this audience
- **Built-in community features**: Voice channels for director-creator calls, threaded discussions per shot, role-based permissions, notification controls
- **Mobile-ready**: Discord's mobile app gives creators on-the-go access to claim shots, check earnings, and vote — without building native apps

### Faster Iteration

Discord bots can be deployed and updated in hours. A slash command interface lets the team test workflows, gather feedback, and iterate on UX before committing to expensive frontend development. The bot *is* the MVP.

### Transition Path

Discord-first does not mean Discord-only. The roadmap:

| Phase | Primary Interface | Discord Role |
|-------|-------------------|--------------|
| Phase 1 (Q2-Q3 2026) | Discord bot | Full platform — all actions happen here |
| Phase 2 (Q4 2026) | Web dashboard launches | Discord as notification + quick-action layer |
| Phase 3 (2027+) | Web + mobile apps | Discord as community hub + alerts |

Discord remains a permanent part of the stack — it transitions from primary interface to companion channel, never disappearing entirely.

---

## 2. Server Architecture

The Discord server is organized into five categories, each serving a distinct function. Film-specific categories are created dynamically by the bot when a director launches a new project.

### Category: ANNOUNCEMENTS

| Channel | Purpose | Permissions |
|---------|---------|-------------|
| `#announcements` | Platform updates, partnerships, milestones | Read-only (Admin post) |
| `#changelog` | Bot updates, new features, bug fixes | Read-only (Admin post) |
| `#governance-proposals` | Token governance proposals and discussion | @Verified-Voter can discuss; Admin posts proposals |

### Category: FILM PROJECTS

| Channel | Purpose | Permissions |
|---------|---------|-------------|
| `#film-listings` | Bot posts new films seeking contributors — browsable catalog | Read-only (Bot post) |

When a director creates a film via `/create-film`, the bot auto-generates a dedicated category:

**Category: [Film Title] (e.g., SPACE-ODYSSEY-2077)**

| Channel | Purpose | Permissions |
|---------|---------|-------------|
| `#film-overview` | Pinned: script link, Standard Library download, technical specs, treasury parameters | Read-only (@Director pins) |
| `#shots-available` | Bot-maintained list of unclaimed shots with descriptions and durations | Read-only (Bot-managed) |
| `#submissions` | All submitted segments posted as embeds with voting buttons | @Creator can submit; all can vote |
| `#main-branch` | Current merged film — progress tracker showing completion %, timeline of merges | Read-only (Bot-managed) |
| `#director-chat` | Q&A between director and contributors — threaded per topic | @Director + @Creator |

### Category: CREATOR HUB

| Channel | Purpose |
|---------|--------|
| `#tools-help` | Troubleshooting AI tools, LoRA loading, ComfyUI workflows |
| `#standard-library-workshop` | Discussion on building and using Standard Libraries |
| `#showcase` | Completed/merged segments — creators share their wins |
| `#ai-tools-discussion` | News and discussion about Sora, Runway, Pika, ComfyUI, etc. |

### Category: TOKENOMICS

| Channel | Purpose |
|---------|--------|
| `#earnings-dashboard` | Bot responds to `/my-earnings` — personal revenue tracking |
| `#nft-marketplace` | Clip NFT listings, secondary sales, trading discussion |
| `#governance-voting` | Active $MONTAGE governance votes with reaction-based polling |
| `#faucet` | Request testnet $MONTAGE tokens during Phase 1 |

### Category: COMMUNITY

| Channel | Purpose |
|---------|--------|
| `#introductions` | New members introduce themselves |
| `#general` | Off-topic and general discussion |
| `#feedback` | Platform feedback and feature requests |
| `#verify-wallet` | Connect wallet via Collab.Land for on-chain identity verification |
| `#role-select` | Self-assign Creator, Curator, Director, Developer roles via bot reactions |
| `#bot-support` | Report MontageBot issues and command errors |

---

## 3. Bot Command Design

The OpenMontage bot (named **MontageBot**) handles all platform operations through slash commands. Every command provides immediate feedback via Discord embeds.

### Core Commands

#### `/create-film`

```
/create-film title:[name] script:[url]
```

Starts the film setup wizard. The bot DMs the director a step-by-step flow: upload script, upload Standard Library assets, set treasury parameters, configure shot durations. On completion, the bot creates the film's channel category and posts to `#film-listings`.

**Example embed posted to `#film-listings`:**

```
┌─────────────────────────────────────────────┐
│  🎬  NEW FILM: Space Odyssey 2077           │
│─────────────────────────────────────────────│
│  Director: @CosmicDirector                  │
│  Genre: Sci-fi / Drama                      │
│  Shots: 240 total (240 available)           │
│  Avg Shot Duration: 6-12 seconds            │
│  Standard Library: v1.0.0 (5 characters,    │
│    cyberpunk style, 3 voice models)         │
│  Revenue Split: 80% creators / 15%          │
│    director / 5% curators                   │
│                                             │
│  📜 Script: [View Script]                   │
│  📦 Standard Library: [Download]            │
│                                             │
│  [👉 Browse Shots]  [📖 Read Overview]      │
└─────────────────────────────────────────────┘
```

#### `/claim-shot`

```
/claim-shot film:[film-id] shot:[shot-id]
```

Locks the shot for 72 hours. The bot DMs the creator with Standard Library download links, the shot's script excerpt, target duration, and technical requirements. If the shot is already claimed, the bot notifies and shows when the lock expires.

**Claim Expiration**:
- When a 72-hour claim lock expires without a submission, the shot is automatically released back to `#shots-available`.
- The creator receives a DM notification 12 hours before expiration and again at expiration.
- The same creator may re-claim the shot immediately unless they have hit the 5-claim limit.
- Repeated claim expirations (3+ in 30 days) trigger a temporary claim cooldown of 48 hours.

**Example DM to creator:**

```
┌─────────────────────────────────────────────┐
│  🔒  Shot Claimed: #042 — Chase Scene       │
│─────────────────────────────────────────────│
│  Film: Space Odyssey 2077                   │
│  Duration: 8-12 seconds                     │
│  Resolution: 1080p minimum                  │
│  Lock expires: 2026-03-06 14:00 UTC         │
│                                             │
│  Script excerpt:                            │
│  "The protagonist sprints through neon-lit  │
│  alleyways, pursuers close behind..."       │
│                                             │
│  📦 Standard Library v1.0.0:                │
│  • protagonist.safetensors [Download]       │
│  • antagonist.safetensors [Download]        │
│  • cyberpunk-style-guide.png [Download]     │
│  • prompt-templates.json [Download]         │
│  • protagonist-voice.model [Download]       │
│                                             │
│  Submit with: /submit film:space-odyssey    │
│    shot:042 url:[your-video] notes:[desc]   │
└─────────────────────────────────────────────┘
```

#### `/submit`

```
/submit film:[film-id] shot:[shot-id] url:[video-url] notes:[description]
```

The bot downloads and validates the video (see Section 6: Quality Validation Pipeline). If validation passes, the submission is minted as a Clip NFT and posted to the film's `#submissions` channel as a voting embed. If validation fails, the bot DMs the creator with specific errors.

**Example embed posted to `#submissions`:**

```
┌─────────────────────────────────────────────┐
│  📹  New Submission: Shot #042              │
│─────────────────────────────────────────────│
│  Creator: @NeonArtist                       │
│  Duration: 9.2s | Resolution: 1080p         │
│  AI Model: Runway Gen-3 + protagonist LoRA  │
│  Std Library: v1.0.0 ✅                     │
│  Clip NFT: 0xa3f...8c2                      │
│                                             │
│  Notes: "Went for a Blade Runner-inspired   │
│  lighting approach with heavy fog."         │
│                                             │
│  ▶️ [Watch Video]                            │
│                                             │
│  ⏳ Voting open for 72 hours                │
│  Current votes: 0                           │
│                                             │
│  Rate this submission:                      │
│  [⭐1] [⭐2] [⭐3] [⭐4] [⭐5]               │
└─────────────────────────────────────────────┘
```

#### `/vote`

```
/vote submission:[submission-id] score:[1-5]
```

Casts a vote weighted by the voter's $MONTAGE stake. Votes are anonymous until the 72-hour voting period closes. Alternatively, voters can use the reaction buttons on submission embeds. The bot DMs a confirmation and current vote count (without revealing scores).

#### `/film-status`

```
/film-status film:[film-id]
```

**Example response:**

```
┌─────────────────────────────────────────────┐
│  🎬  Space Odyssey 2077 — Status            │
│─────────────────────────────────────────────│
│  Progress: ████████░░ 78% (187/240 shots)   │
│  Active claims: 23 shots                    │
│  Open shots: 30 shots                       │
│  Submissions awaiting vote: 12              │
│  Total contributors: 94                     │
│  Estimated runtime: 38m 22s                 │
│                                             │
│  Recent merges:                             │
│  • Shot #187 → @PixelMage (2h ago)          │
│  • Shot #042 → @NeonArtist (5h ago)         │
│  • Shot #103 → @CinemaAI (1d ago)           │
└─────────────────────────────────────────────┘
```

#### `/my-earnings`

```
/my-earnings
```

**Example response (DM):**

```
┌─────────────────────────────────────────────┐
│  💰  Your Earnings — @NeonArtist            │
│─────────────────────────────────────────────│
│  Active Clip NFTs: 4 (merged)               │
│  Archived Clip NFTs: 1                      │
│                                             │
│  Monthly revenue estimate: $342.18          │
│                                             │
│  Breakdown:                                 │
│  • Space Odyssey — Shot #042 (9.2s) $128.44 │
│  • Space Odyssey — Shot #119 (6.8s)  $95.02 │
│  • Neon Dreams — Shot #007 (11.1s)   $78.40 │
│  • Neon Dreams — Shot #033 (5.4s)    $40.32 │
│                                             │
│  Lifetime earnings: $1,204.55               │
│  Wallet: 0x7a2...f91                        │
└─────────────────────────────────────────────┘
```

#### `/leaderboard`

```
/leaderboard film:[film-id]
```

Shows top contributors ranked by total merged screen time for the given film.

#### `/standard-library`

```
/standard-library film:[film-id]
```

Returns download links for all Standard Library assets for the specified film, including version number and file sizes.

---

## 4. Complete Workflow: From Script to Film

### Director Flow

**Step 1 — Create the film**

The director runs `/create-film title:"Space Odyssey 2077" script:https://...` in `#film-listings`. MontageBot DMs them a setup wizard.

**Step 2 — Upload script and decompose**

The bot parses the uploaded script and presents a proposed shot breakdown. The director reviews, adjusts shot boundaries and durations, and confirms. The bot creates the film's channel category with all five channels.

**Step 3 — Upload Standard Library**

The director uploads LoRA models, style guides, prompt templates, ControlNets, and voice models through the DM wizard. The bot validates file formats and sizes, assigns version `v1.0.0`, and stores assets on CDN with download links.

**Step 4 — Configure treasury**

The bot walks the director through revenue split confirmation (default: 80/15/5), platform fee acknowledgment, and wallet address verification. On confirmation, the bot deploys the Treasury smart contract.

**Step 5 — Announce**

MontageBot posts the film listing embed to `#film-listings`, pins the overview in `#film-overview`, and populates `#shots-available` with all shot descriptions. The film is live.

### Creator Flow

**Step 1 — Discover**

A creator browses `#film-listings` and finds Space Odyssey 2077. They click through to the film's `#film-overview` to read the script and style direction.

**Step 2 — Claim a shot**

They run `/claim-shot film:space-odyssey shot:042`. MontageBot locks the shot for 72 hours and DMs them the Standard Library download links plus the shot's script excerpt and requirements.

**Step 3 — Generate video**

The creator loads the protagonist LoRA into their pipeline (Runway, ComfyUI, etc.), follows the prompt templates, applies ControlNet constraints, and generates the video segment.

**Step 4 — Submit**

They run `/submit film:space-odyssey shot:042 url:https://... notes:"Blade Runner-inspired lighting, heavy fog"`. MontageBot validates the submission (format, resolution, duration, Standard Library version). On pass, it mints a Clip NFT and posts the voting embed to `#submissions`.

**Step 5 — Voting**

Community members with $MONTAGE stake vote over 72 hours using `/vote` or the reaction buttons on the embed. Votes are anonymous until the period closes. Voting requires a minimum of 3 submissions per shot; if a shot doesn't reach this threshold within 30 days, the director may trigger an early vote or auto-merge a single submission (see governance framework for details).

**Step 6 — Merge or archive**

If the submission wins the vote, MontageBot updates the film's `#main-branch` channel, changes the Clip NFT status to `"merged"`, and announces the merge. If a different submission wins, the creator's NFT is archived — they can iterate and resubmit.

**Step 7 — Earn**

The creator runs `/my-earnings` anytime to see their active Clip NFTs and estimated monthly revenue from streaming.

### Curator/Voter Flow

**Step 1 — Get notified**

MontageBot pings the `@Curator` role in the film's `#submissions` channel whenever a new submission is posted and needs votes.

**Step 2 — Review**

Curators watch the submitted video via the embed's "Watch Video" link and compare against the Standard Library style guide and shot requirements.

**Step 3 — Vote**

They use `/vote submission:0xa3f8c2 score:4` or click the star reaction buttons on the embed. Their vote is weighted by $MONTAGE stake.

**Step 4 — Earn**

After the voting period closes, curators who participated receive their share of the 5% curation pool, distributed proportionally to voting activity.

---

## 5. Wallet Integration

### Verification via Collab.Land

OpenMontage uses [Collab.Land](https://collab.land), the industry-standard Discord-to-wallet verification bot already used by thousands of Web3 communities. The flow:

1. New member joins the server and visits `#verify-wallet` (a channel in the COMMUNITY category)
2. Collab.Land prompts them to connect their wallet (MetaMask, WalletConnect, Phantom, etc.)
3. The user signs a gasless message to prove ownership
4. Collab.Land assigns the `@Verified-Voter` role on Discord
5. MontageBot reads the Collab.Land role to associate the Discord user ID with their wallet address

### Role Gating

| Discord Role | Requirement | Capabilities |
|-------------|-------------|______________|
| `@Verified-Voter` | Wallet connected via Collab.Land | Vote on submissions, view earnings |
| `@Creator` | Wallet verified + first `/claim-shot` | Submit segments, claim shots |
| `@Curator` | Wallet verified + minimum $MONTAGE stake | Weighted voting, curation rewards |
| `@Director` | Wallet verified + created a film | Manage film channels, pin notes |

### On-Chain Mapping

The bot maintains a mapping table (stored in the application database, not on-chain) linking:

```
Discord User ID  <->  Wallet Address  <->  On-Chain Identity
```

All on-chain transactions (NFT minting, revenue distribution, voting) use the wallet address. Discord is the interface layer — the blockchain is the source of truth.

### Security Considerations

- Wallet signatures are verified cryptographically — no passwords stored
- Users can re-verify to update their wallet address
- Multi-wallet support planned for Phase 2 (web dashboard)
- The bot never holds private keys or funds

---

## 6. Quality Validation Pipeline

When a creator runs `/submit`, MontageBot performs a series of automated checks before accepting the submission. This prevents low-quality or non-compliant content from reaching the voting stage.

### Automated Checks

**1. Video Format**
- Container: MP4 (H.264 or H.265 codec)
- Minimum resolution: 1920x1080 (1080p)
- Frame rate: 24fps or 30fps
- Audio: AAC or no audio (dialogue added in post)

**2. Duration**
- Must fall within the shot's specified range (e.g., 8-12 seconds)
- Tolerance: +/- 0.5 seconds

**3. Standard Library Version**
- Submission metadata must reference the correct Standard Library version tag
- Bot cross-checks against the film's current library version
- Submissions using outdated library versions are rejected with upgrade instructions

**4. File Integrity**
- File is downloadable and not corrupted
- File size within platform limits (max 500MB per segment)

**5. AI Consistency Scoring (Future — Phase 2)**
- Face recognition matching against character LoRA reference images
- Style similarity scoring against art direction references
- Automated flagging of visual inconsistencies

### Validation Results

**Pass**: The bot mints a Clip NFT, posts the voting embed to `#submissions`, and DMs the creator a confirmation with their NFT ID.

**Fail**: The bot DMs the creator with specific error details and remediation steps:

```
┌─────────────────────────────────────────────┐
│  ❌  Submission Failed — Shot #042          │
│─────────────────────────────────────────────│
│  Issues found:                              │
│                                             │
│  1. Duration: 14.3s (max allowed: 12.5s)    │
│     → Trim to 8-12 seconds                  │
│                                             │
│  2. Standard Library: v0.9.0 (current: v1.0)│
│     → Download latest: /standard-library    │
│       film:space-odyssey                    │
│                                             │
│  Resubmit with: /submit film:space-odyssey  │
│    shot:042 url:[fixed-video] notes:[desc]  │
│                                             │
│  Your shot lock remains active until        │
│  2026-03-06 14:00 UTC                       │
└─────────────────────────────────────────────┘
```

Failed validations do not consume the creator's shot lock — they can fix and resubmit within the 72-hour window.

---

## 7. Moderation Design

### Role Hierarchy

| Role | Scope | Capabilities |
|------|-------|-------------|
| `@Admin` | Server-wide | Full server management, bot configuration, ban/kick |
| `@Director` | Per-film channels | Pin notes in film channels, flag submissions, manage Standard Library updates |
| `@Curator` | Per-film submissions | Weighted voting, flag inappropriate content |
| `@Creator` | Per-film submissions | Claim shots, submit segments |
| `@Verified-Voter` | Per-film submissions | Basic voting rights |

### Spam Prevention

- **Rate limiting**: Max 3 submissions per shot per creator per 24 hours
- **Claim limits**: Max 5 active shot claims per creator across all films
- **Slowmode**: `#submissions` channels have 30-second slowmode to prevent spam discussion
- **New account restrictions**: Accounts less than 7 days old cannot claim shots (prevents throwaway abuse)

### Content Reporting

- Any user can right-click a submission embed and use Discord's built-in report flow
- MontageBot adds a `[🚩 Report]` button on submission embeds for platform-specific reports
- Reports are sent to a private `#mod-queue` channel visible only to `@Admin`
- Three or more independent reports auto-hide a submission pending admin review

### Rights Violations

- Directors can flag submissions that violate Standard Library terms or use unauthorized assets
- Flagged submissions are temporarily hidden from voting and sent to `#mod-queue`
- Confirmed violations result in Clip NFT burn and temporary creator suspension
- Repeat offenders lose `@Creator` role

### Director Authority

- Directors can pin "official notes" as threaded replies on any submission in their film's channels
- Directors cannot delete submissions or override votes — governance remains community-driven
- Directors can update the Standard Library (with version bump), triggering bot notifications to all active contributors

---

## 8. Phase 2 Transition: Discord to Web

### When to Transition

The Discord-first approach is designed for Phase 1 (Q2-Q3 2026). The transition to a web dashboard begins in Phase 2 (Q4 2026) when:

- The platform has validated core workflows with 3+ completed pilot films
- User feedback identifies Discord's limitations (complex dashboards, portfolio views, video playback)
- The team has sufficient data on usage patterns to design an optimal web UX

### What Moves to Web

| Feature | Discord (Phase 1) | Web Dashboard (Phase 2) |
|---------|-------------------|------------------------|
| Film browsing & discovery | `#film-listings` channel | Rich catalog with filters, search, thumbnails |
| Shot claiming | `/claim-shot` command | Visual shot board (kanban-style) |
| Video submission | `/submit` + URL | Drag-and-drop upload with inline preview |
| Voting | Embed reactions / `/vote` | Side-by-side comparison viewer |
| Earnings tracking | `/my-earnings` DM | Real-time dashboard with charts, payout history |
| Film playback | External links | Integrated video player with segment-by-segment navigation |
| Standard Library | File download links | Visual asset browser with version diffing |
| NFT management | Basic text display | Full portfolio with marketplace integration |

### What Stays on Discord

Discord remains the **community and notification layer** even after the web dashboard launches:

- **Notifications**: Bot pings for new films, vote results, earnings milestones, shot replacements
- **Quick actions**: `/claim-shot`, `/vote`, `/my-earnings` continue to work for convenience
- **Community discussion**: `#general`, `#tools-help`, `#ai-tools-discussion`, director Q&A channels
- **Voice channels**: Creator calls, director AMAs, community events
- **Governance discussion**: Proposal debates before formal on-chain votes

### Technical Integration

The web dashboard and Discord bot share the same backend API and database. Actions taken on either surface are reflected in both:

- Claim a shot on web → `#shots-available` updates on Discord
- Submit via Discord → appears in web dashboard immediately
- Vote on web → vote count updates on Discord embed

This dual-interface approach ensures no user is forced to migrate — power users can use the web dashboard for complex tasks while continuing to get notified and take quick actions through Discord.

---

**Version**: 1.0
**Last Updated**: 2026-03-03
