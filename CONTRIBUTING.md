# Contributing to OpenMontage

OpenMontage is building the infrastructure for collaborative AI filmmaking — think GitHub for movies. Whether you make AI videos, direct films, curate quality, or write code, there's a place for you here.

This guide walks you through everything you need to start contributing.

---

## Ways to Contribute

- **AI Video Creators** — Generate video segments for active film projects and earn revenue from your work
- **Directors / Producers** — Launch film projects with your vision and build the Standard Libraries that make collaboration possible
- **Curators** — Vote on segment quality and earn curation rewards (5% of film revenue)
- **Developers** — Help build the platform: Discord bot, smart contracts, web dashboard
- **Community Builders** — Grow and moderate the Discord community

---

## Getting Started

### Step 1: Join the Discord

All OpenMontage activity happens on Discord during Phase 1.

- Join at [discord.gg/openmontage](https://discord.gg/openmontage)
- Introduce yourself in `#introductions` — tell us what you do and what interests you
- Pick your role(s) in `#role-select`: Creator, Director, Curator, Developer, or Community

### Step 2: Verify Your Wallet

- Use the Collab.Land bot in the `#verify-wallet` channel
- Connect your Ethereum or Base wallet
- Once verified, you'll receive the **Verified** role, which unlocks submission and voting channels

### Step 3: Get Test Tokens

- During Phase 1 (testnet), request test `$MONTAGE` tokens from `#faucet`
- These tokens let you participate in voting and governance on the test network
- No real money required — everything runs on testnet until mainnet launch

### Step 4: Explore Active Films

- Browse `#film-listings` to see all active film projects
- Each listing includes: logline, genre, number of open shots, and Standard Library quality preview
- Download a Standard Library for any film you want to contribute to
- Ask questions in the film's `#director-chat` channel before starting work

---

## Creator Guide (Video Segment Contributor)

### Tools You Need

**AI Video Generation** (any of these):
- Runway Gen-3 or later
- Pika 1.5 or later
- Kling
- Sora
- ComfyUI with video generation workflows

**Image Generation** (for storyboarding and reference):
- Midjourney
- SDXL / Flux
- Any tool that supports LoRA loading

**Basic Video Editing**:
- DaVinci Resolve (free)
- CapCut
- Any tool that can export MP4/H.264

### Understanding the Standard Library

The Standard Library is the dependency package for each film. It's what keeps characters, lighting, and style consistent across hundreds of creators. Before creating anything:

1. Download it from Discord using `/standard-library film:[film-id]`
2. Load character LoRA files into your generation pipeline
3. Read the style guide carefully — pay attention to color palette, lighting direction, and camera language
4. Study the prompt templates and understand the placeholder structure
5. **Test your setup** with a simple generation before starting the real shot — verify the character LoRA produces recognizable results in your pipeline

### Claiming a Shot

1. Browse `#shots-available` in the film's channel category
2. Use `/claim-shot film:[id] shot:[id]` to reserve a shot
3. You have **72 hours** to submit (or release the claim with `/release-shot`). If the lock expires without a submission, the shot is automatically released back to `#shots-available`. You'll receive a DM reminder 12 hours before expiration.
4. The bot DMs you: shot description, technical requirements, duration target, and Standard Library download links

### Creating Your Segment

**Technical Requirements:**
- **Format**: MP4, H.264 codec
- **Resolution**: Minimum 1080p (1920x1080), 4K preferred
- **Duration**: Must match the shot specification within ±0.5 seconds
- **Frame rate**: 24fps for cinematic look, 30fps acceptable
- **Audio**: Include if the shot requires dialogue or sound effects; use the provided voice models for character speech

**Quality Guidelines:**
- Follow the prompt templates from the Standard Library — they exist for a reason
- **Character LoRA adherence is the most important factor**. Voters heavily weight visual consistency. If your protagonist doesn't look like the protagonist, the submission will lose regardless of how good the cinematography is.
- Lighting should match the film's style guide. Check reference images.
- Smooth motion: avoid AI artifacts like melting faces, warping limbs, or flickering textures
- Composition: follow any storyboard references in the shot description. Match camera angles and framing.

### Submitting Your Work

1. Upload your video to a publicly accessible URL (IPFS preferred, cloud storage acceptable)
2. Use `/submit film:[id] shot:[id] url:[video-url] notes:[brief description of your approach]`
3. The bot automatically validates format, resolution, duration, and Standard Library version
4. **If approved**: your Clip NFT is minted on-chain and your segment enters the voting pool
5. **If rejected**: the bot DMs you the specific validation error with instructions to fix and resubmit

### The Voting Period

- A **72-hour voting window** opens once a shot has at least 3 submissions
- All votes are anonymous until the voting period closes — no one knows who submitted what
- The winning submission's Clip NFT status changes to **"merged"** and you start earning revenue immediately
- Non-winning submissions remain **"archived"** — you keep your Clip NFT and can improve and resubmit anytime
- If a better version is submitted later, the community can vote to replace the current segment

### Tracking Your Earnings

- Use `/my-earnings` in Discord to see revenue across all your merged segments
- Revenue flows in real-time as films are streamed by audiences
- Claim your earnings weekly via the Treasury smart contract using `/claim-earnings`

---

## Curator Guide (Voter)

### How Voting Works

- Stake `$MONTAGE` tokens to set your voting weight
- Vote on submissions using `/vote submission:[id] score:[1-5]`
- Votes are weighted by: `stake amount x reputation score`
- **Blind voting**: all submissions are anonymous until the voting period ends — judge the work, not the creator

### How to Evaluate Submissions

Rate each submission on three dimensions:

1. **Standard Library Adherence** (40% of evaluation weight): Does the character look right? Is the art style consistent with the film's visual language? Are prompt templates followed?
2. **Technical Quality** (35%): Smooth motion, accurate lighting, no AI artifacts (melting, flickering, warping), clean audio if applicable
3. **Creative Execution** (25%): How well does this segment interpret the shot description? Does the composition tell the story effectively?

### Curator Rewards

- Curators share **5% of total film revenue**, distributed among voters who supported winning segments
- **Early detection bonus**: If you vote for a submission that wins before most other voters do, you earn a larger share
- Your **reputation score** increases with accurate curation history, which in turn increases your future voting weight

---

## Director Guide

### Is OpenMontage Right for Your Film?

**Best candidates:**
- Short-to-medium films (20–120 minutes of final runtime)
- Clear visual style that can be encoded in a Standard Library
- Scripts with discrete, visually-describable shots
- Stories where creative variation between shots is acceptable or even desirable

**Less suitable:**
- Films requiring extremely tight frame-level continuity across long takes
- Documentary style (talking heads, real-world locations)
- Projects where the director needs granular control over every detail of every frame

### Creating Your Film Project

1. Write your script with AI generation in mind — discrete shots, vivid visual descriptions
2. Build your Standard Library: character LoRAs, style guide, prompt templates, voice models
3. Use `/create-film` in Discord to launch the setup wizard
4. Configure your Treasury parameters: confirm the 80/15/5 revenue split, set Film Ticket NFT pricing if crowdfunding
5. Announce your project in `#film-listings` with a compelling pitch and sample generations

### Standard Library Requirements

**Mandatory:**
- Character LoRA models for all speaking and recurring characters
- Style guide document with color palette, lighting references, and camera language
- Prompt template dictionary with clear placeholder structure

**Optional but recommended:**
- ControlNet specifications for complex action shots or precise character poses
- Voice clone models for dialogue-heavy films
- Location and environment reference packs

For detailed Standard Library guidelines, see [Product Mechanism: Standard Library Best Practices](doc/product-mechanism.md#11-standard-library-best-practices).

---

## Code of Conduct

- **Respect all contributors** regardless of experience level or background
- **No plagiarism** — all submissions must be your own original AI-generated work
- **Copyright attestation** is required on every submission
- **No coordinated voting** — voting rings will result in stake slashing and potential ban
- **Constructive feedback** in `#director-chat` channels. Critique the work, not the person.
- **No NSFW content** unless the film project is explicitly tagged and age-gated

---

## Getting Help

| Topic | Channel |
|-------|---------|
| AI tool setup, LoRA loading, generation pipelines | `#tools-help` |
| Standard Library questions and troubleshooting | `#standard-library-workshop` |
| Discord bot issues or command errors | `#bot-support` |
| General questions about OpenMontage | `#general` |
| Direct help from the team | DM any member with the `@Admin` role |

---

## For Developers

The repository is currently documentation-only (pre-alpha). We are actively seeking developers starting Q2 2026 for:

- **Discord bot** — Node.js or Python, handles all user commands and blockchain interactions. See the [Discord Platform Design](doc/discord-platform.md) for bot command specs and architecture.
- **Smart contracts** — Solidity, deploying on Base L2 (Coinbase). Core contracts: ClipNFT, MovieNFT, Treasury, Voting, StandardLibrary. See [Whitepaper Section 6](doc/whitepaper.md#6-technical-architecture) for smart contract interfaces and chain selection rationale.
- **Web dashboard** — React / Next.js, initially read-only film browser and stats, later full workflow support (Phase 2+)

For the full development timeline and technical milestones, see the [Roadmap](doc/roadmap.md).

### Contributing to Documentation

Documentation contributions are welcome now. If you find errors, unclear explanations, or missing information:

1. Fork the repository
2. Make your changes — keep docs consistent across all files (the [whitepaper](doc/whitepaper.md) is the authoritative source)
3. Submit a pull request with a clear description of what you changed and why

### Developer Channels

- Watch `#announcements` for developer calls and contributor onboarding
- Join `#dev-chat` for technical discussion
- Check the [Roadmap](doc/roadmap.md) for upcoming development milestones

---

## Thank You

Every contributor — whether you generate a single shot, vote on one submission, or build an entire film — is helping prove that collaborative filmmaking works. We're building something new here, and we're glad you're part of it.

Questions? Drop them in `#general` on [Discord](https://discord.gg/openmontage) or open a GitHub issue.

---

**Version**: 1.0
**Last Updated**: 2026-03-03
**Authors**: OpenMontage Core Team
