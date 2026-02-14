# OpenMontage Whitepaper

## Abstract

OpenMontage is a decentralized, community-driven platform for collaborative filmmaking. Inspired by the open-source movement, it enables creators worldwide to contribute video segments, vote on quality, and collectively produce complete films through iterative improvement.

---

## 1. Introduction

### 1.1 Motivation

In the age of AI-generated content, video creation is democratized. However, producing a complete, high-quality film still requires significant coordination, resources, and expertise. OpenMontage solves this by enabling:

- **Distributed creation**: Anyone can contribute segments
- **Quality curation**: Community voting ensures the best work rises to the top
- **Iterative improvement**: Films evolve over time as better contributions arrive

### 1.2 Vision

A future where any script can become a polished film through the collective creativity of a global community.

---

## 2. Core Concepts

### 2.1 Roles

**Contributors**
- Create and submit video segments
- May use AI tools, traditional filming, animation, etc.
- Retain attribution for their work

**Voters**
- Evaluate and rate submitted segments
- Influence which versions make it into the final assembly
- May overlap with contributors

**Viewers (Audience)**
- Watch the final assembled film for entertainment
- The ultimate consumers of the collaborative work
- May provide feedback for future iterations

### 2.2 Objects & Metadata

All entities in OpenMontage are represented as **objects** with associated **metadata**:

#### Film (Root Object)
- **Metadata**: Title, synopsis, genre, target runtime, license, creation date
- **Structure**: Contains multiple Segments
- **Version history**: Tracks assembly changes over time

#### Segment
- **Metadata**: Segment ID, script text, duration range, visual/audio requirements
- **Versions**: Multiple contributor submissions per segment
- **Selected version**: The currently highest-rated submission

#### Submission (Segment Version)
- **Metadata**: Contributor, creation method (AI model / filming / animation), resolution, format, upload date
- **Content**: Video file + audio track
- **Ratings**: Community votes and scores

#### Asset Objects (Reusable Elements)
- **Character**: Name, description, visual reference, voice profile
- **Prop**: Type, appearance, usage context
- **Scene/Location**: Description, environmental details, reference images
- **Music/SFX**: Audio assets with usage rights

These assets enable consistency across segments contributed by different creators.

### 2.3 Repository Structure

Each film is organized as a **project repository**:

```
/my-film-project/
├── film.json                 # Root manifest: segment list with metadata paths
├── README.md                 # Film overview
├── script.md                 # Full screenplay
├── segments/
│   ├── 001-opening/
│   │   ├── segment.md        # Segment description (script, requirements)
│   │   ├── segment.json      # Segment metadata (selected version, video hash, URL)
│   │   └── submissions/
│   │       ├── contributor-a.json  # Video hash, URL, ratings
│   │       ├── contributor-b.json
│   │       └── ...
│   ├── 002-introduction/
│   │   ├── segment.md
│   │   ├── segment.json
│   │   └── submissions/
│   │       └── ...
│   └── ...
├── assets/
│   ├── characters/
│   │   ├── protagonist.json
│   │   └── antagonist.json
│   ├── props/
│   ├── locations/
│   └── audio/
└── votes.json                # Voting records
```

---

## 3. Workflow

### 3.1 Film Initialization

1. A **creator** uploads a script and breaks it into segments
2. Each segment is defined with:
   - Script text
   - Duration target
   - Scene requirements (characters, props, location)
3. Assets are documented for consistency

### 3.2 Contribution Process

Contributors can provide:

**Video Segments**
1. Browse segments needing submissions
2. Create a video following segment requirements (AI generation, filming, animation, etc.)
3. Submit via pull request or platform upload

**Assets**
- Characters (visual references, descriptions)
- Props (3D models, reference images)
- Locations (environment descriptions, reference images)
- Audio (music, sound effects)

### 3.3 Voting & Selection

1. **Lightweight voting**: Users directly vote on submissions via `votes.json`
2. **Real-time ranking**: Highest-rated submission becomes the active version for each segment
3. **Dynamic selection**: As votes change, the "current best" may shift over time

### 3.4 Dynamic Playback

Instead of pre-assembling a complete video file:

1. **Viewer app** reads `film.json` and current vote tallies
2. **Real-time assembly**: App streams the highest-rated segment for each part in sequence
3. **Live updates**: As votes change, the next playback reflects new selections
4. **Attribution overlay**: Credits displayed dynamically during or after playback

---

## 4. Technical Architecture

### 4.1 Platform Components

- **Repository host**: Git-based storage (GitHub, GitLab, or custom)
- **Metadata layer**: JSON schemas for all objects
- **Viewer/Voting app**: Combined interface where users watch films, vote on segments, and browse submissions
  - Real-time playback based on current vote tallies
  - Integrated voting UI (viewers and voters are the same audience)
  - Dynamic assembly engine that selects highest-rated segments
- **Web interface**: Browse films, segments, submissions, and assets

### 4.2 Storage & Delivery

- **Video submissions**: Cloud storage (S3-compatible) with CDN delivery
- **Assets**: Stored in repository or linked externally
- **Metadata**: Stored in repository for version control and transparency

---

## 5. Use Cases

- **Student films**: Collaborative projects across schools/countries
- **Indie experiments**: Creative communities iterating on avant-garde scripts
- **Educational content**: Science explainers built segment-by-segment
- **Fan films**: Community-driven adaptations of public domain works

---

## 6. Roadmap

_(To be defined)_

---

## 7. Conclusion

OpenMontage reimagines filmmaking as a collaborative, iterative process inspired by the open-source movement. By leveraging AI-enabled creation and community curation, it empowers anyone to contribute to high-quality films — turning scripts into movies through the collective effort of a global creative community.

---

**Version**: 0.1.0 (Draft)  
**Last Updated**: 2026-02-14
