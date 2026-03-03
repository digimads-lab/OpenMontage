# OpenMontage Product Mechanism (Git for Movies)

OpenMontage's core innovation is the **modularization** and **standardization** of film production. By treating a movie like a software repository, we enable truly decentralized, AI-powered collaborative filmmaking.

---

## 0. Discord-First Platform Overview

OpenMontage launches as a **Discord-first platform**. Rather than building a custom web app before validating the core product loop, all Phase 1 interactions happen inside a Discord server at [discord.gg/openmontage](https://discord.gg/openmontage).

### Why Discord

Discord is already home to the communities OpenMontage needs: AI artists, crypto-native builders, and indie filmmakers. By meeting users where they are, we eliminate onboarding friction and leverage Discord's built-in identity system, role management, and moderation tools. A Discord bot can ship in weeks; a custom web frontend takes months and risks building the wrong thing before product-market fit is proven.

### What You Can Do From Discord

- **Claim shots** — Browse `#film-listings`, then `/claim-shot` to reserve a shot to work on
- **Download Standard Libraries** — `/standard-library film:[id]` fetches character LoRAs, style guides, and prompt templates
- **Submit segments** — `/submit film:[id] shot:[id] url:[video-url]` uploads your work and mints a Clip NFT
- **Vote on submissions** — `/vote submission:[id] score:[1-5]` to curate quality
- **Track earnings** — `/my-earnings` shows your revenue across all films
- **Create films** — Directors use `/create-film` to launch projects with a guided setup wizard

### Transition Path

Discord is primary in Phase 1 (Q2-Q3 2026). In Phase 2 (Q4 2026), a read-only web dashboard launches for browsing films and viewing stats. By Phase 3 (2027+), a full web app handles all workflows and the Discord bot transitions to a notification role.

For the complete Discord server architecture, channel structure, and bot command reference, see the [Discord Platform Design](discord-platform.md).

---

## 1. The Script Tree (The Repository)

The initiator (e.g., director or screenwriter) creates a "Film Repository" and uploads the screenplay. The script is rigorously decomposed into:

- **Scenes**: Major narrative segments
- **Shots**: Individual camera setups within each scene

Each Shot includes:
- Detailed textual description
- Target duration
- Reference visuals or storyboards

---

## 2. The Standard Library (Visual & Audio Consistency)

**The critical innovation**: To solve the fatal consistency problem in crowdsourced AI-generated content (character appearance, art style, voice), the initiator **must** establish an **Official Standard Library** when creating the repository.

### What's Included:

- **Character LoRA Models**: Pre-trained LoRA files for all main characters to ensure consistent facial features and appearance across all submissions
- **Style Reference Images**: Official art direction guides (color palette, lighting style, cinematography references)
- **Prompt Template Dictionary**: Standardized prompt structures for consistent AI generation
- **ControlNet Specifications**: 3D skeletal rigs or pose control networks for character animation consistency
- **Voice Clone Models**: Authorized voice synthesis models for each character to maintain audio coherence

**Why This Matters**: Just like software dependencies ensure code compatibility, the Standard Library guarantees visual and auditory unity across hundreds of independent contributors — solving the #1 barrier to crowdsourced AI filmmaking.

---

## 3. Contributing Segments (Pull Requests)

### Process:

1. **Claim a Shot**: Any AIGC creator can claim an unassigned Shot from the repository
2. **Pull Dependencies**: Download the Standard Library (LoRA models, style guides, prompts, ControlNets, voice models)
3. **Generate Content**: Create a video segment using:
   - AI video generation tools (Sora, Runway, Pika, etc.)
   - Mandatory adherence to Standard Library specifications
   - Personal creativity within the established framework
4. **Submit PR**: Upload the video as a "Pull Request" to that Shot's submission pool

**Key Constraint**: All submissions **must** use the official Standard Library assets to pass initial validation.

---

## 4. Version Control & Merging

### Community Governance:

- **Voting Rights**: Community members or token holders vote on submitted segments
- **Selection Criteria**:
  - Adherence to Standard Library specifications (visual/audio consistency)
  - Technical quality (lighting, motion smoothness, composition)
  - Creative interpretation within guidelines
  
### Merge Process:

1. **Initial Selection**: Highest-voted submission is merged into the Main Branch
2. **Dynamic Updates**: If a better version is submitted later (e.g., using more advanced AI models with superior lighting/motion), the community can vote to replace the existing segment
3. **Version History**: All previous versions remain archived, enabling rollbacks if needed

**This enables continuous improvement** — the film naturally evolves as AI technology advances and new talent joins.

---

## 5. Seamless Playback (Render)

### Viewer Experience:

- **Real-Time Assembly**: The platform frontend sequences all latest Main Branch Shots in script order
- **Unified Post-Production**: Automatic application of:
  - Consistent color grading
  - Transition effects between shots
  - Unified soundtrack and sound effects
  - Subtitles in multiple languages
- **Live Viewing**: Audiences always see the latest "evolved" version of the film
- **Attribution Display**: Creator credits shown dynamically during or after playback

---

## 6. Comparison to Traditional Filmmaking

| Aspect | Traditional | OpenMontage (Web) | OpenMontage (Discord Phase 1) |
|--------|-------------|-------------------|-------------------------------|
| **Collaboration** | Centralized crew | Global decentralized contributors | Same — coordinated via Discord channels |
| **Consistency** | Single production designer | Standard Library enforces unity | Standard Library distributed via bot commands |
| **Iteration** | Expensive re-shoots | Vote-driven segment replacement | Voting via `/vote` slash commands |
| **Accessibility** | High equipment/skill barrier | AI tools + Standard Library = low barrier | Even lower — no new app to learn, just join Discord |
| **Evolution** | Fixed after release | Continuous improvement via new submissions | Same mechanism, Discord notifications for new votes |

---

## 7. Technical Stack Requirements

### Repository Layer
- Git-based version control (GitHub/GitLab/self-hosted)
- Large file storage (Git LFS) for video submissions

### Standard Library Storage
- Cloud CDN for LoRA models, ControlNets, and reference assets
- Versioned dependency management (like npm/pip for packages)

### Validation Pipeline
- Automated checks for Standard Library compliance
- AI-powered consistency scoring (face recognition, style matching)

### Rendering Engine
- Real-time segment assembly
- Frame-accurate transition generation
- On-the-fly color grading and audio mixing

---

## 8. Key Advantages

1. **Scalability**: Unlimited contributors can work in parallel on different Shots
2. **Quality Assurance**: Standard Library + community voting ensures consistent high quality
3. **Technology Agnostic**: Works with any AI video generation tool as long as Standard Library is followed
4. **Cost Efficiency**: No need for expensive equipment or location shoots
5. **Eternal Evolution**: Films improve over time as technology and talent advance

---

## 9. Example Workflow

### Scenario: Creating an indie sci-fi film

1. **Director uploads script** → Breaks into 240 Shots
2. **Creates Standard Library**:
   - LoRA models for 5 main characters
   - Cyberpunk art style guide (color palette, lighting references)
   - Prompt templates for cityscape backgrounds
   - ControlNets for character movement
   - Voice clones for all speaking roles
3. **Community claims Shots** → 150 contributors worldwide start generating
4. **First submissions** → Shot 001 receives 8 versions
5. **Community votes** → Version by Creator_A (best lighting) is merged
6. **3 months later** → Creator_B submits new version using GPT-6 with better motion
7. **Re-vote + merge** → Shot 001 updated; film now 5% better
8. **6 months in** → All 240 Shots have at least one merged version
9. **Public premiere** → Film streams with real-time assembly, credits all 150 contributors

---

## 10. The Director's Journey

Creating a film on OpenMontage is different from traditional filmmaking. This section walks through the complete lifecycle of an OpenMontage film project, from concept to premiere.

### Step 1: Concept Development

Not every script works well on OpenMontage. The best candidates are films with **discrete, visually-describable shots** that can be independently generated by different creators. Think: anthology stories, episodic narratives, music videos, sci-fi with distinct environments. Films requiring frame-perfect continuity across long takes (e.g., single-shot dialogues) are harder to crowdsource — though not impossible with strong ControlNet specifications.

### Step 2: Script Writing for AI Generation

Write with AI generation in mind. Each shot description should read like a detailed image/video prompt: specific about visual elements, lighting, camera angle, and character action. Avoid vague directions like "they exchange a meaningful look" — instead, write "close-up of PROTAGONIST, tears forming, warm backlighting, slight head tilt left." The more precise your shot descriptions, the better your submissions will be.

### Step 3: Shot Decomposition

Break your script into individual shots. Guidelines for optimal shot design:

- **Duration**: 5–15 seconds per shot. Shorter shots are easier to generate consistently; longer shots compound AI artifacts.
- **Visual clarity**: Each shot should have one primary action or visual focus.
- **Character limits**: Avoid complex multi-character interactions in a single shot. Two characters maximum per shot yields the best results with current AI tools.
- **Transitions**: Plan for cuts between shots. AI-generated content works best with hard cuts; complex transitions (dissolves, wipes) are applied in post-production.

A typical 90-minute film decomposes into 200–400 shots.

### Step 4: Building the Standard Library

This is the most important step. Your Standard Library determines the visual quality ceiling for the entire film. You need:

- **Character LoRA models**: Train using Kohya SS or similar tools. Budget approximately 3–5 hours of training time per character. Provide 50+ reference images with diverse angles, lighting conditions, and expressions. The LoRA is what keeps your protagonist looking like the same person across 200 shots from 150 different creators.
- **Style guide document**: Define your film's visual identity — color palette, lighting approach (e.g., "desaturated noir with teal highlights"), camera language, and reference films/images.
- **Prompt template dictionary**: Create standardized prompt structures that contributors plug their shot-specific details into. This ensures consistent AI generation parameters.
- **Voice clone models** (if dialogue): Use Eleven Labs or similar services to create authorized voice models for each speaking character.
- **ControlNet specifications** (optional): For shots requiring precise character poses or camera movements, provide skeletal rigs or depth maps.

### Step 5: Repository Setup on Discord

Use the `/create-film` command in Discord to launch the project wizard. You'll configure: film metadata (title, genre, logline), upload your Standard Library assets, set Treasury parameters (confirm the 80/15/5 revenue split or customize), and optionally price Film Ticket NFTs for crowdfunding.

### Step 6: Launching the Film

Announce your project in `#film-listings`. Post a compelling pitch: logline, visual references, sample generations using your Standard Library, and the number of shots available. First impressions matter — creators choose projects based on the quality of the Standard Library and the director's vision.

### Step 7: Ongoing Management

Your job doesn't end at launch. Active directors produce better films:

- Answer creator questions in your film's `#director-chat` channel
- Monitor submission quality and provide constructive feedback
- Update the Standard Library if consistency issues emerge (version bump)
- Provide constructive feedback on submissions in `#director-chat` (note: directors cannot vote on their own film's segment selection to prevent conflicts of interest — see governance framework)
- Promote your film to attract top-tier AI creators

---

## 11. Standard Library Best Practices

The Standard Library is the single most important factor in a film's success. A great library attracts top creators and produces visually coherent results; a weak library produces inconsistent submissions that frustrate everyone.

### What Makes a Good Standard Library

**Specificity beats generality.** A style guide that says "cinematic lighting" is useless. A style guide that says "high-contrast noir lighting, key light 45° camera-left, rim light from behind, deep shadows on right side of face, color temperature 4500K" produces consistent results. The more specific your constraints, the more coherent your film looks.

### Character LoRA Quality Guidelines

- **Training data**: Minimum 50 reference images per character. Include: frontal, 3/4 view, profile, varied expressions, multiple lighting conditions, different clothing/settings.
- **Resolution**: Training images should be at least 512x512, ideally 1024x1024.
- **Consistency in training data**: All reference images must depict the same character design. Mixed references produce unstable LoRAs.
- **Testing**: Before publishing, generate 20+ test images across varied prompts and verify the character is recognizable in all of them.
- **Versioning**: Start at v1.0.0. Bump minor version for refinements, major version for redesigns. Contributors should always pull the latest version.

### Prompt Template Format

Templates should be structured with clear placeholders:

```
[CHARACTER_LORA] [ACTION_DESCRIPTION], [LOCATION], [LIGHTING_STYLE], [CAMERA_ANGLE], [STYLE_TOKENS]
```

Example template for a dialogue shot:
```
[PROTAGONIST_LORA] speaking to [ANTAGONIST_LORA], indoor office setting, noir high-contrast lighting with teal accent, medium close-up shot, cinematic 35mm film grain, shallow depth of field
```

### Style Guide Components

**Mandatory**: Color palette (hex codes), lighting approach (with reference images), camera language (preferred focal lengths, angles), mood/tone references (link 3–5 reference films or images).

**Optional but recommended**: Title card typography, transition style preferences, sound design direction, aspect ratio preference (16:9, 2.39:1, etc.).

### Common Mistakes to Avoid

- **Under-training LoRAs**: Fewer than 30 reference images produces inconsistent characters. Invest the time.
- **Vague style guides**: "Make it look cool" helps no one. Be specific.
- **Ignoring negative prompts**: Include negative prompt templates to suppress common AI artifacts.
- **No test generations**: Always test the full Standard Library end-to-end before opening the film for contributions.
- **Forgetting voice models**: If your film has dialogue, voice consistency matters as much as visual consistency.

---

## Conclusion

OpenMontage transforms filmmaking from a centralized, capital-intensive process into a **decentralized, AI-powered collaborative art form**. By combining version control principles from software development with mandatory Standard Libraries for consistency, we enable anyone to contribute to professional-quality films — while maintaining the artistic coherence traditionally requiring top-down control.

This is **Git for Movies** — where films are living artifacts that evolve through community creativity.

---

**Version**: 1.1
**Last Updated**: 2026-03-03
**Authors**: OpenMontage Core Team
