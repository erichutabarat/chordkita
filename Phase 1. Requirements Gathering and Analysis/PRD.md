# Product Requirements Document (PRD)
## ChordKita — Aplikasi Chord Gitar untuk Musisi Indonesia

**Version:** 1.0 (Draft)
**Date:** 17 Agustus 2026
**Author:** Eric Hutabarat
**Status:** PRD
**Phase:** Sprint 0 — Requirements Gathering & Analysis

---

## 1. Product Overview

ChordKita is a mobile application (Android, MVP) that gives Indonesian guitar players a fast, reliable, and localized way to find chords and lyrics for songs, with auto-scrolling playback so they can play hands-free. The long-term vision is to become the go-to chord companion app for Indonesian musicians — combining the depth of song libraries seen in apps like Chordtela/JagoGitar with the polish and UX quality of global apps like Ultimate Guitar.

### 1.1 Vision Statement
"To be the most trusted and easiest-to-use guitar chord app for Indonesian musicians — from angkringan jam sessions to church worship teams to bedroom singer-songwriters."

### 1.2 Product Goals
- Give users instant access to accurate chords + lyrics for Indonesian and international songs.
- Let users play along hands-free via auto-scroll.
- Build a sustainable freemium business (free core value, paid convenience/depth).
- Establish a foundation (data model, content pipeline) that can scale to tuner, transpose, chord diagrams, and offline mode in later phases.

---

## 2. Problem Statement

Indonesian guitar players currently rely on:
- **Scattered websites** (Chordtela, KapanLagi, etc.) — full of ads, pop-ups, and inconsistent formatting, not built for in-app playing experience.
- **Global apps** (Ultimate Guitar, Chordify) — weak on Indonesian/local repertoire (dangdut, pop Indonesia, religi, daerah), and premium pricing not localized for Indonesian purchasing power.
- **Local Android apps** (JagoGitar, Chord Guitar Full Offline) — large song libraries but dated UI/UX, unreliable content quality, and weak monetization/retention design.

**Gap ChordKita fills:** a clean, modern, mobile-first experience purpose-built for Indonesian repertoire and reading/playing habits (auto-scroll while singing), monetized in a way that fits local pricing expectations.

---

## 3. Target Users & Personas

| Persona | Description | Key Need |
|---|---|---|
| **Andi, 22, Mahasiswa** | Plays guitar casually at kost/kampus gatherings | Find chords fast for trending songs, easy to read on small screen |
| **Kak Rani, 28, Worship Leader** | Leads music at gereja/pengajian weekly | Reliable chord + lyric accuracy, transpose to match vocalist's key, auto-scroll while singing |
| **Bagas, 17, Pemula** | Just started learning guitar | Simple chord diagrams, beginner-friendly songs, no clutter |
| **Sari, 30, Content Creator** | Covers songs for social media | Fast search, accurate lyrics timing, wants ad-free experience (potential premium buyer) |

---

## 4. Competitive Landscape (Quick Scan)

| App | Strength | Weakness |
|---|---|---|
| **Ultimate Guitar** | <cite index="5-1">Massive global library (2M+ songs), synced lyrics, transpose, practice mode</cite> | Weak Indonesian repertoire, premium pricing not localized |
| **Chordtela (website)** | <cite index="8-1">Large Indonesian song/lyric database across many genres and regional languages</cite> | Web-only, ad-heavy, not app-native, no auto-scroll UX |
| **JagoGitar / Chord Guitar Full Offline** | <cite index="4-1">Large offline chord library with auto-scroll lyrics feature</cite>, <cite index="6-1">80,000+ chords from 8,000 local/international artists with transpose via capo feature</cite> | Dated UI, unclear content curation/quality, weak monetization strategy |
| **Chordify** | <cite index="3-1">Automatic chord detection from audio across guitar, piano, ukulele</cite> | Not song-lyric focused, less relevant for sing-along use case |

**ChordKita's differentiation:** Indonesian-first content curation + modern UX/UI + auto-scroll done well + fair freemium pricing for the local market.

---

## 5. Goals & Success Metrics (MVP)

| Goal | Metric | Target (3 months post-launch) |
|---|---|---|
| Prove core value (chord search) | Weekly Active Users (WAU) | 5,000 |
| Retention | D7 retention rate | ≥ 20% |
| Content sufficiency | Song search success rate (found vs. not found) | ≥ 85% |
| Monetization validation | Free-to-premium conversion rate | ≥ 2% |
| Engagement | Avg. sessions per user per week | ≥ 3 |
| Quality | Crash-free session rate | ≥ 99% |
| Store standing | Play Store rating | ≥ 4.3 |

---

## 6. Scope

### 6.1 MVP Scope (In Scope)
1. **Song & Chord Search** — search by song title or artist name.
2. **Chord + Lyric Display** — chords displayed inline above lyrics, standard notation.
3. **Auto-Scroll** — adjustable scroll speed, play/pause control.
4. **Chord Diagram Popup** — tap a chord to see finger placement diagram.
5. **Transpose** — shift key up/down (+/- semitone), common in worship/vocal-matching use case.
6. **Favorites / Bookmarks** — save songs for quick access.
7. **Basic Account System** — email or Google sign-in (for saving favorites + premium status).
8. **Freemium Paywall** — free tier with limits (e.g., ads, limited transpose/bookmark count); premium tier removes ads + unlocks full features.
9. **Genre/Category Browse** — Pop Indonesia, Dangdut, Religi, Pop Barat, Daerah, etc.

### 6.2 Explicitly Out of Scope for MVP (Backlog for later phases)
- Offline mode / song caching
- Audio chord detection (Chordify-style AI)
- Built-in guitar tuner
- User-submitted/community chord editing
- iOS version
- Social features (sharing, following other users)
- Practice mode / gamified learning
- Backing tracks / Tonebridge-style audio effects

> Keeping AI chord detection, tuner, and community editing out of MVP is intentional — these require heavier tech investment and content moderation processes best tackled once core retention is proven.

---

## 7. Epics & User Stories (Agile Format)

### Epic 1: Song Discovery
- **US-1.1:** As Andi, I want to search songs by title or artist so I can quickly find the chord I need.
  - *AC:* Search returns results within 2 seconds; partial-match and typo-tolerant search supported.
- **US-1.2:** As Bagas, I want to browse songs by genre/category so I can discover new songs to learn.
  - *AC:* Genre list visible on home screen; tapping shows paginated song list.

### Epic 2: Chord & Lyric Viewing
- **US-2.1:** As Kak Rani, I want to see chords aligned above lyrics so I can play and sing accurately.
  - *AC:* Chord position matches lyric syllable; readable on screens from 5"–6.5".
- **US-2.2:** As Bagas, I want to tap a chord and see its finger diagram so I can learn how to play it.
  - *AC:* Diagram popup shows finger position, string names, and open/muted strings.

### Epic 3: Hands-Free Playing
- **US-3.1:** As Kak Rani, I want auto-scroll with adjustable speed so I can play without touching my phone.
  - *AC:* Scroll speed slider (min/max bounds); play/pause/reset controls accessible during scroll.

### Epic 4: Key Matching
- **US-4.1:** As Kak Rani, I want to transpose a song's key so it matches my vocalist's range.
  - *AC:* +/- buttons shift all chords consistently; original key indicator always visible.

### Epic 5: Personalization
- **US-5.1:** As Sari, I want to save songs to favorites so I can access my set list quickly.
  - *AC:* Favorite icon toggle; favorites list accessible from bottom nav.
- **US-5.2:** As a user, I want to sign in with Google so my favorites sync across sessions.
  - *AC:* One-tap Google Sign-In; guest mode allowed with local-only favorites.

### Epic 6: Monetization
- **US-6.1:** As a free user, I want to understand what premium unlocks so I can decide whether to upgrade.
  - *AC:* Paywall screen clearly lists free vs. premium features and price (in IDR).
- **US-6.2:** As a premium user, I want an ad-free experience so my playing isn't interrupted.
  - *AC:* No banner/interstitial ads shown once subscription is active.

---

## 8. Non-Functional Requirements

| Category | Requirement |
|---|---|
| **Performance** | App cold start ≤ 3s; search response ≤ 2s on 4G |
| **Compatibility** | Android 8.0 (API 26) and above; optimize for mid-range devices (common in ID market) |
| **Localization** | Full Bahasa Indonesia UI; English as secondary language |
| **Data cost sensitivity** | Minimize data usage per session (important for prepaid data users) — lazy load images/diagrams |
| **Availability** | Backend uptime ≥ 99.5% |
| **Security** | User data (email, favorites) encrypted at rest; comply with Indonesia's PDP Law (UU PDP) for personal data handling |
| **Content Legal** | Chord/lyric licensing approach must be reviewed (see Risks, Section 10) |
| **Accessibility** | Minimum readable font size 14sp; high-contrast mode for outdoor/stage use |

---

## 9. Monetization Model Detail

**Model:** Freemium subscription

| Tier | Price (indicative) | Features |
|---|---|---|
| **Free** | Rp 0 | Full song search & view, banner ads, transpose limited to ±2, 10 favorites max |
| **Premium** | ~Rp 25.000–39.000/bulan or Rp 199.000/tahun (validate via pricing research) | No ads, unlimited transpose, unlimited favorites, priority access to new songs, offline mode (post-MVP) |

*Note: exact pricing should be validated with willingness-to-pay research in Sprint 0 backlog refinement — Indonesian freemium apps typically price premium tiers between Rp 15,000–50,000/month.*

---

## 10. Assumptions, Constraints & Risks

### Assumptions
- Users primarily play on Android phones with mid-range specs.
- Majority of usage happens with active internet connection (offline is post-MVP).
- Indonesian repertoire (pop, dangdut, religi, daerah) drives majority of search volume.

### Constraints
- Small initial team; MVP must be lean and content-pipeline-light.
- Budget-conscious infrastructure (avoid expensive audio-processing services for MVP).

### Key Risks
| Risk | Impact | Mitigation |
|---|---|---|
| **Chord/lyric content copyright** (song lyrics are copyrighted works) | High — legal exposure | Research licensing options (e.g., partnering with a licensing body like WAMI/LMKN in Indonesia, or building own transcriptions from public-domain-safe sources); consult legal counsel before scaling content |
| **Content quality/accuracy** at launch | Medium — poor reviews | Start with curated, manually-verified song set (e.g., top 500 songs) rather than mass-scraping |
| **Low differentiation vs. incumbents** | Medium | Double down on UX polish + auto-scroll experience + local pricing |
| **Slow content growth** | Medium | Define a clear content pipeline/team process before Sprint 1 |

> ⚠️ **Important flag for Sprint 0:** Song lyrics and chord transcriptions can be subject to copyright. Before scaling the content library, this needs legal review — this is a business/legal task to prioritize alongside engineering, not just a technical requirement.

---

## 11. Next Steps (Agile Process)

1. **Stakeholder review** of this PRD (you + any co-founders/team).
2. **Legal/content licensing research** (parallel workstream — don't block engineering).
3. Break Epics above into a **Product Backlog** in your tracking tool (Jira/Trello/Linear).
4. **Backlog refinement & estimation** (story points) with the team.
5. Define **Sprint 1 goal** — recommend starting with Epic 1 (Song Discovery) + Epic 2 (Chord/Lyric Viewing) as the thinnest usable slice.
6. Move to **Phase 2: System/UX Design** — wireframes, information architecture, tech stack decision (e.g., Kotlin native vs. Flutter, backend choice).

---

*This is a living document — expect it to evolve as we validate assumptions with real users and stakeholders.*