# ChordKita — Technical Architecture Document

**Version:** 2.0 (Draft)
**Date:** 17 Agustus 2026
**Phase:** Sprint 0 — System Design
**Related doc:** `ChordKita_PRD.md`

---

## 1. Purpose & Scope

This document defines the technical architecture for ChordKita's MVP using a **self-hosted stack**: Flutter mobile client, Go (Gin) backend, MySQL database, JWT auth, optional Redis caching, containerized with Docker and deployed via GitHub Actions CI/CD.

---

## 2. Architecture Principles

1. **Clear separation of concerns** — feature-layered structure on mobile, layered (handler/service/repository) structure on backend.
2. **Own your infra, keep it simple.** Self-hosted stack means more control but also more ops responsibility — keep the initial deployment topology minimal (single VPS/small cluster is enough for MVP).
3. **Data-cost aware.** Indonesian users are sensitive to mobile data usage — paginate, compress, cache.
4. **Content is the product.** Backend must make it easy to ingest/update song & chord content independently of app releases.
5. **Stateless backend.** JWT auth + no server-side session storage (except optional Redis cache) keeps horizontal scaling simple later.

---

## 3. Tech Stack

| Layer | Choice | Why |
|---|---|---|
| **Mobile app** | **Flutter** (feature-layered folder structure) | Single codebase, opens the door to iOS later without a rewrite; feature-layered structure keeps each feature self-contained and testable |
| **State management** | **Riverpod** (recommended) | Type-safe, minimal boilerplate, plays well with feature-layered architecture; Bloc is a valid alternative if the team prefers stricter event-driven patterns |
| **Backend** | **Go + Gin** | Fast, low memory footprint (good for a lean VPS deployment), strong concurrency model, simple routing/middleware via Gin |
| **Database** | **MySQL** | Relational fit for songs/artists/users/favorites with clear foreign-key relationships; mature tooling, easy to self-host via Docker |
| **Auth** | **JWT (access + refresh token)**, custom implementation | Stateless, simple to reason about, no third-party auth dependency; password hashing via bcrypt |
| **Cache** | **Redis (optional, recommended for MVP+)** | Cache popular/trending songs and search results to reduce MySQL load; can be deferred to post-launch if traffic is low initially |
| **Search** | **MySQL FULLTEXT index** (MVP) → evaluate **Meilisearch/Typesense** later if typo-tolerance/relevance becomes a problem | Avoids adding another service on day 1; MySQL FULLTEXT is "good enough" for a curated few-hundred-song launch catalog |
| **Media storage** | **Object storage** (e.g., S3-compatible — MinIO self-hosted or a cloud provider bucket) | Chord diagram images; keeps binary data out of MySQL |
| **Payments/Subscription** | **Google Play Billing Library** (client) + backend **receipt validation** via Google Play Developer API | Required for Android IAP; backend must be the source of truth for subscription status, never trust the client alone |
| **Ads (free tier)** | **Google AdMob** | Standard Android ads SDK, client-side only |
| **CI/CD** | **GitHub Actions** + **Docker** (backend + MySQL, and Redis if used) | Automated test/build/deploy pipeline; Docker Compose for local dev parity with production |
| **Deployment target** | Single VPS (Docker Compose) for MVP → migrate to orchestration (e.g., managed Kubernetes) only once scale demands it | Keeps ops overhead low for a small team |
| **Monitoring/Logging** | Basic structured logging (Go `zerolog`/`zap`) + optional Prometheus/Grafana later | Start minimal; add observability as traffic grows |

---

## 4. System Architecture Overview

```
┌───────────────────────────────────────────────────────────────┐
│                      FLUTTER MOBILE APP                        │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  features/{search, chord_viewer, favorites, auth, ...}  │   │
│  │     presentation → domain → data  (per feature)         │   │
│  │  core/ (network client, theme, shared widgets, utils)   │   │
│  └───────────────────────────┬────────────────────────────┘   │
└──────────────────────────────┼─────────────────────────────────┘
                                │  HTTPS (REST + JWT bearer token)
                                ▼
┌───────────────────────────────────────────────────────────────┐
│                     BACKEND (Go + Gin) — Docker container       │
│  ┌───────────┐   ┌────────────┐   ┌───────────┐   ┌─────────┐  │
│  │  Handler   │→ │  Service    │→ │Repository │→ │  Models  │  │
│  │(routes,    │  │(business    │  │(DB access,│  │(structs) │  │
│  │ middleware,│  │ logic)      │  │ queries)  │  │          │  │
│  │ JWT auth)  │  │             │  │           │  │          │  │
│  └───────────┘   └────────────┘   └─────┬─────┘   └─────────┘  │
└─────────────────────────────────────────┼───────────────────────┘
                                           │
                  ┌────────────────────────┼─────────────────────┐
                  ▼                        ▼                     ▼
         ┌─────────────────┐    ┌──────────────────┐   ┌──────────────────┐
         │  MySQL (Docker)  │    │  Redis (optional, │   │ Object Storage    │
         │ songs, users,    │    │  Docker) — cache   │   │ (chord diagram     │
         │ favorites, subs  │    │  hot queries        │   │  images)           │
         └─────────────────┘    └──────────────────┘   └──────────────────┘

         External: Google Play Billing (subscription validation)
                   Google AdMob (client-side ads, free tier)
```

**Deployment topology (MVP):** one VPS running Docker Compose with 3 services — `backend`, `mysql`, `redis` (optional) — behind a reverse proxy (e.g., Caddy/Nginx) for TLS termination.

---

## 5. Mobile App Structure (Flutter — Feature-Layered)

```
lib/
├── core/
│   ├── network/          # Dio client, interceptors (JWT attach/refresh), API error handling
│   ├── theme/             # colors, typography, spacing
│   ├── widgets/           # shared/reusable widgets
│   ├── utils/             # chord transpose logic, formatters
│   └── constants/
│
├── features/
│   ├── auth/
│   │   ├── data/           # datasources (API), repository impl, DTO models
│   │   ├── domain/         # entities, repository interface, use cases
│   │   └── presentation/   # screens, widgets, Riverpod providers/controllers
│   │
│   ├── song_search/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │
│   ├── chord_viewer/       # chord+lyric display, auto-scroll, transpose, diagram popup
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │
│   ├── favorites/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │
│   └── subscription/       # paywall, Play Billing integration
│       ├── data/
│       ├── domain/
│       └── presentation/
│
├── app.dart                # MaterialApp, routing setup
└── main.dart
```

> **Convention:** each feature is self-contained (own data/domain/presentation). Cross-feature reuse goes into `core/`. This keeps features independently testable and lets you parallelize work across contributors later without merge conflicts across unrelated features.

---

## 6. Backend Structure (Go + Gin)

```
cmd/
└── api/
    └── main.go              # entrypoint, wiring dependencies

internal/
├── handler/                 # HTTP handlers (Gin routes), request/response DTOs
│   ├── auth_handler.go
│   ├── song_handler.go
│   ├── favorite_handler.go
│   └── subscription_handler.go
│
├── service/                 # business logic, orchestrates repositories
│   ├── auth_service.go
│   ├── song_service.go
│   ├── favorite_service.go
│   └── subscription_service.go
│
├── repository/               # DB access layer (MySQL queries, Redis cache access)
│   ├── song_repository.go
│   ├── user_repository.go
│   └── favorite_repository.go
│
├── model/                    # DB structs / entities
│   ├── song.go
│   ├── user.go
│   └── favorite.go
│
├── middleware/                # JWT auth middleware, logging, rate limiting
│   └── jwt_auth.go
│
└── config/                    # env config loading

migrations/                    # SQL migration files (e.g., golang-migrate)
pkg/                            # shared utilities (jwt helper, password hash, response wrapper)

docker/
├── Dockerfile
└── docker-compose.yml         # backend + mysql (+ redis)

.github/workflows/
└── ci-cd.yml
```

---

## 7. Data Model (MySQL Schema)

```sql
-- artists
CREATE TABLE artists (
  id            BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  name          VARCHAR(150) NOT NULL,
  image_url     VARCHAR(255),
  created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- songs
CREATE TABLE songs (
  id                BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  title             VARCHAR(200) NOT NULL,
  artist_id         BIGINT UNSIGNED NOT NULL,
  genre             VARCHAR(100),               -- e.g. "Pop Indonesia", "Religi"
  original_key      VARCHAR(10),                -- e.g. "C"
  lyrics_chords     JSON NOT NULL,               -- structured blocks: [{text, chord, position}]
  popularity_score  INT DEFAULT 0,
  content_status    ENUM('pending_review','verified') DEFAULT 'pending_review',
  created_at        TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at        TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (artist_id) REFERENCES artists(id),
  FULLTEXT (title)
);

-- chord_diagrams (reference data)
CREATE TABLE chord_diagrams (
  id                BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  chord_name        VARCHAR(20) NOT NULL UNIQUE,  -- e.g. "Am7", "Dsus4"
  finger_positions  JSON NOT NULL,
  image_url         VARCHAR(255)
);

-- users
CREATE TABLE users (
  id                   BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  display_name         VARCHAR(100),
  email                VARCHAR(150) UNIQUE NOT NULL,
  password_hash        VARCHAR(255),               -- null if OAuth-only, if added later
  subscription_status  ENUM('free','premium') DEFAULT 'free',
  subscription_expires_at TIMESTAMP NULL,
  created_at           TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- favorites
CREATE TABLE favorites (
  user_id     BIGINT UNSIGNED NOT NULL,
  song_id     BIGINT UNSIGNED NOT NULL,
  added_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (user_id, song_id),
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (song_id) REFERENCES songs(id) ON DELETE CASCADE
);

-- refresh_tokens (for JWT refresh flow)
CREATE TABLE refresh_tokens (
  id            BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id       BIGINT UNSIGNED NOT NULL,
  token_hash    VARCHAR(255) NOT NULL,
  expires_at    TIMESTAMP NOT NULL,
  revoked       BOOLEAN DEFAULT FALSE,
  created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

> **Design note:** `lyrics_chords` is stored as JSON (structured blocks) rather than raw text with inline chord tags, so transpose can be computed client-side without re-parsing strings, and the backend doesn't need per-request text processing.

---

## 8. Auth Flow (Simple JWT)

1. **Register/Login** → backend verifies credentials (bcrypt-compared password hash) → issues:
   - **Access token** (JWT, short-lived, e.g., 15 min) — sent as `Authorization: Bearer <token>` on each request.
   - **Refresh token** (long-lived, e.g., 30 days) — stored hashed in `refresh_tokens` table, returned to client for secure storage (Flutter secure storage).
2. **Authenticated requests** → Gin middleware validates JWT signature + expiry, extracts `user_id` into request context.
3. **Token refresh** → client calls `/auth/refresh` with refresh token before access token expires; backend checks it's not revoked, issues new access token.
4. **Logout** → refresh token marked `revoked = true`.
5. **Guest mode** → app functions without login (local-only favorites); JWT issued only after sign-up, at which point local favorites can be synced to backend.

> Keep it simple for MVP: no OAuth/social login required yet, but the `password_hash` nullable column leaves room to add Google Sign-In later without a schema rewrite.

---

## 9. Feature → Architecture Mapping

| PRD Feature (Epic) | Technical Components Involved |
|---|---|
| Song & Chord Search | `song_handler` → `song_service` → MySQL FULLTEXT query (cached in Redis for popular queries if enabled) |
| Chord + Lyric Display | `GET /songs/:id` → returns structured `lyrics_chords` JSON → Flutter `chord_viewer` feature renders aligned chord/lyric UI |
| Auto-Scroll | Client-side only — Flutter `ScrollController` driven by a timer/animation, no backend involvement |
| Chord Diagram Popup | `GET /chords/:name` → `chord_diagrams` table, cached locally on-device after first fetch |
| Transpose | Pure client-side logic in `core/utils` — shifts chord values in-memory using music theory transposition rules |
| Favorites/Bookmarks | `favorite_handler` → `favorites` table; guest mode falls back to local Flutter storage (e.g., Hive/SharedPreferences) until user signs up |
| Account System | `auth_handler` (JWT issue/refresh/logout), `users` table |
| Freemium Paywall | Backend exposes feature-limit config (e.g., `/config/limits`); `subscription_service` validates Play Billing receipts server-side, updates `subscription_status` |
| Genre/Category Browse | `GET /songs?genre=...` with MySQL indexed query |

---

## 10. Content Pipeline (MVP Approach)

1. Curate initial ~500 songs manually (verified for chord/lyric accuracy) in a structured format (spreadsheet or JSON files).
2. An **admin-only backend endpoint or CLI script** (Go) bulk-inserts/updates rows into `songs` via the repository layer — reuses the same validation logic as the API.
3. `content_status` field gates unverified content out of public search/list queries (`WHERE content_status = 'verified'`).
4. Post-MVP: build a lightweight internal admin web tool once content volume outgrows manual scripts.

> ⚠️ **Reminder from PRD:** song lyrics/chords are copyrighted content — licensing review should happen before scaling past the initial curated batch. The `content_status` gate exists precisely to keep unlicensed content out of production.

---

## 11. Non-Functional / Cross-Cutting Concerns

| Concern | Approach |
|---|---|
| **Performance on mid-range devices** | Flutter: lazy-loaded lists, compressed images (WebP); Backend: Redis cache for hot queries (trending songs, popular searches) |
| **Data usage** | Paginate all list endpoints, lazy-load chord diagram images, cache aggressively on-device |
| **Security** | JWT signature validation on every protected route; bcrypt password hashing; parameterized SQL queries (no raw string concatenation) to prevent SQL injection; HTTPS enforced via reverse proxy |
| **Privacy (UU PDP compliance)** | Store minimal PII (name, email, hashed password); provide account deletion endpoint; document data usage in Privacy Policy |
| **Subscription integrity** | Play Billing purchase receipts always validated server-side via Google Play Developer API — client-reported status is never trusted directly |
| **Crash/error resilience** | Structured backend logging (zerolog/zap); Flutter crash reporting (Firebase Crashlytics is fine to keep even with a self-hosted backend, since it's mobile-client-only) |
| **Backups** | Automated MySQL backups (e.g., nightly `mysqldump` to object storage) — critical since this is now self-managed, not a managed DB service |

---

## 12. CI/CD Pipeline (GitHub Actions + Docker)

**Backend pipeline (`.github/workflows/ci-cd.yml`):**
1. **On PR:** run `go vet`, `go test ./...`, lint (`golangci-lint`).
2. **On merge to `main`:**
   - Build Docker image (`docker build -f docker/Dockerfile .`)
   - Push image to a container registry (GitHub Container Registry / Docker Hub)
   - Run MySQL migrations (`golang-migrate`) against target environment
   - Deploy: SSH to VPS → `docker compose pull && docker compose up -d` (or trigger a deploy webhook)

**Mobile pipeline:**
1. **On PR:** `flutter analyze`, `flutter test`
2. **On merge to `main`/release branch:** build APK/AAB, upload to Play Console internal testing track (via `fastlane` or GitHub Actions Play publishing action)

**Local dev parity:** `docker-compose.yml` spins up `backend + mysql (+ redis)` together so local development matches production environment closely.

---

## 13. Environment & Repo Setup (Sprint 0 Checklist)

- [ ] Monorepo or two repos (`chordkita-mobile`, `chordkita-backend`) — recommend **two repos** since mobile/backend release cycles differ
- [ ] `docker-compose.yml` for local dev (backend + MySQL + Redis)
- [ ] `.env.example` documented (DB credentials, JWT secret, Redis URL) — never commit real secrets
- [ ] SQL migration tooling set up (`golang-migrate` or similar) with initial schema migration committed
- [ ] GitHub Actions workflows for both repos (lint/test on PR, build/deploy on merge)
- [ ] VPS provisioned with Docker + reverse proxy (Caddy/Nginx) for TLS
- [ ] Play Console app entry created (internal testing track)
- [ ] AdMob account linked (mobile client)
- [ ] Automated MySQL backup job configured

---

## 14. Open Questions / Decisions Needed Before Sprint 1

1. **Redis: include from day 1 or defer?** — Recommend deferring until you have real traffic data showing MySQL query load justifies it; the schema/service layer should still be written cache-friendly (i.e., repository methods abstracted so adding a cache-aside pattern later doesn't require a rewrite).
2. **Search quality** — MySQL FULLTEXT may not handle typos/partial matches well for Indonesian song titles; budget time to evaluate this with real query patterns early, with Meilisearch as a fallback plan.
3. **Content licensing model** — affects whether `content_status` gating is a launch blocker (see PRD Risks).
4. **VPS provider & sizing** — needs a decision (e.g., DigitalOcean/Linode/local ID provider) based on budget and latency to Indonesian users.
5. **Monorepo vs. two repos** — recommended two repos above, but confirm based on team preference/tooling familiarity.

---

*This document should evolve alongside the PRD as technical spikes resolve the open questions above.*