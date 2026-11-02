# Concert Buddies — v1 Spec

**Status**: agreed through a full requirements interview
**Date**: 2026-11-02
**See also**: [`GLOSSARY.md`](../GLOSSARY.md) for domain terms, and [`docs/adr/`](./adr/) for the decisions behind this spec.

---

## 1. Product

Log in with Spotify, view your top artists across three time ranges, search for a discoverable user, and compare your top fifty to see your **mutual artists**.

The rebuild replaces the legacy Vue 3 client and Java Spring Boot server with a single Next.js App Router application.

## 2. Scope

**In scope**

- Spotify OAuth login and logout
- Snapshot of the logged-in user's top artists for all three time ranges
- Manual snapshot refresh
- User search (discoverable users only)
- Pairwise comparison showing mutual artists
- Discoverability toggle
- Delete-my-data

**Out of scope (v1)**

- Friend requests / friend graph
- Artist detail pages
- User detail pages
- Top tracks
- Async/background snapshot refresh
- Rank-weighted similarity scoring

## 3. Domain rules

- **Mutual Artist** — an artist present in *both* users' top 50 for the *same* time range. Rank orders the display only; it never changes whether an artist is mutual.
- **Discoverable** — a user may appear in search and comparison. Defaults on; the user can opt out.
- **Snapshot** — one user's top artists for one time range, captured at a point in time.
- **Snapshot Refresh** — all-or-nothing across all three ranges. Either every range is fetched and written, or the previous snapshot is kept untouched.
- **Needs Reauth** — set when Spotify rejects a stored token; the user must log in again before another refresh.

## 4. Architecture

- Full-stack **Next.js App Router**. Java server retired.
  See [ADR-0001](./adr/0001-full-stack-nextjs.md).
- **PostgreSQL + Drizzle ORM**, deployed on Vercel with managed Postgres. Connect through the pooler; keep the pool small.
- **Hand-rolled Spotify OAuth** — Authorization Code + PKCE + `state`; tokens stored server-side in Postgres; only a random session id in an `httpOnly` cookie; all authorization checks in a server-only data access layer.
  See [ADR-0003](./adr/0003-hand-rolled-oauth.md).
- **Persisted snapshots**, not live Spotify fetches at comparison time.
  See [ADR-0002](./adr/0002-persisted-snapshots.md).
- **Discoverable-by-default pairwise comparison**, no friend graph.
  See [ADR-0004](./adr/0004-discoverable-pairwise.md).
- **Unweighted set intersection at top 50.**
  See [ADR-0005](./adr/0005-unweighted-intersection.md).

## 5. Data model (high level)

| Table | Purpose | Key columns |
|---|---|---|
| `users` | Spotify identity and settings | `id`, `spotify_id` (unique), `display_name`, `avatar_url`, `email`, `discoverable` (default true), `needs_reauth` (default false) |
| `sessions` | Login sessions | `id`, `user_id`, `expires_at` |
| `spotify_tokens` | Server-side OAuth tokens | `user_id`, encrypted `access_token`, encrypted `refresh_token`, `expires_at`, `scope` |
| `artists` | Canonical artist metadata | `id`, `spotify_id` (unique), `name`, `image_url` |
| `user_top_artists` | Ranked snapshot rows | `user_id`, `artist_id`, `time_range`, `rank` |
| `snapshot_syncs` | Records "synced and legitimately empty" | `user_id`, `time_range`, `synced_at` |

Comparison query: join `user_top_artists` for both users on `artist_id` within the selected `time_range`, ordered by the requesting user's rank.

## 6. Data fetching and state

- **Reads**: Server Components through a server-only DAL (Drizzle → Postgres).
- **Mutations**: Server Actions (refresh snapshot, toggle discoverability, delete data, logout).
- **Route Handlers**: `/auth/login` and `/auth/callback` only.
- **State**: URL `searchParams` for `range`, `q`, `page`, and `user`; local `useState` for ephemeral UI. No client store and no client data cache.
- **Caching**: none in v1. Mutations call `revalidatePath` so the current view re-renders.

## 7. Routes

| Route | Auth | Shows / does |
|---|---|---|
| `/` | optional | Logged out: landing + "Log in with Spotify". Logged in: your top artists for the selected range, "last synced", refresh, search entry. |
| `/search?q=&page=` | required | Discoverable users only; rows link to compare. |
| `/compare/[spotifyId]?range=` | required | Other user's name and avatar, count of shared artists, and mutual artists ordered by your rank. |
| `/settings` | required | Discoverability toggle, "Delete my data", logout. |
| `/auth/login`, `/auth/callback` | — | OAuth redirect and code exchange. |

## 8. Edge-case policy

| # | Case | Behavior |
|---|---|---|
| 1 | New user / little history | Store what exists (0–50 per range); empty UI state; comparison shows no shared artists, not an error. |
| 2 | One range fails during refresh | All-or-nothing: fetch all three ranges, write in one transaction; on failure keep the previous snapshot. |
| 3 | Spotify `429` / 5xx during refresh | Bounded retry honoring `Retry-After`; a failed snapshot never blocks login. |
| 4 | Artist identity | Canonical key is the Spotify artist id; upsert metadata on each fetch; rows keyed `(user, artist, range)`. |
| 5 | Missing artist image/fields | Placeholder; never crash. |
| 6 | Comparing with yourself | Blocked: "That's you." |
| 7 | Compared user has no snapshot | "Hasn't synced yet" empty state, not an error. |
| 8 | Non-discoverable user | Excluded from search; direct URL returns not-found. |
| 9 | User opts out of discoverability | Hidden from search/comparison; data retained. |
| 10 | Delete my data | Removes user, tokens, sessions, and snapshots. |
| 11 | Search | Case-insensitive partial match on display name + Spotify id; 20 per page; excludes non-discoverable users. |
| 12 | Refresh token invalid/revoked | Mark account `needs_reauth`; prompt re-login; keep snapshot but label it stale. |
| 13 | Session expiry vs token expiry | Independent: session expiry → re-login; token expiry → silent refresh. |
| 14 | Logout | Delete session, clear cookie, keep snapshot, revoke Spotify token. |
| 15 | Concurrent refresh | Per-user single-flight (advisory lock / status guard) + idempotent upsert. |
| 16 | Staleness | Show "Last synced <relative time>"; manual refresh. |

## 9. Testing

- **Unit (Vitest)**: set intersection, refresh decision, PKCE/`state` helpers, search filters. Spotify mocked at the `fetch` boundary.
- **Integration**: DAL and comparison query against a real test Postgres; migrations applied, tables truncated between tests.
- **E2E**: deferred; add one happy-path Playwright test (login → home → compare) once the UI settles.

## 10. Deployment and configuration

- **Host**: Vercel.
- **Database**: managed Postgres (Neon or Supabase) via the pooled connection string.
- **Environment variables**: `SPOTIFY_CLIENT_ID`, `SPOTIFY_CLIENT_SECRET`, `SPOTIFY_REDIRECT_URI`, `DATABASE_URL`, a session secret, and a token-encryption key.
- **Redirect URI** must be registered per environment (localhost and production).

## 11. Defaults

- First load uses `medium_term`; the chosen range then lives in `?range=`.
- Snapshot refreshes on every login and on the manual Refresh action.
- Session lifetime is 30 days absolute; expiry requires re-login.
- Search is ordered by display name ascending, case-insensitive.
- Access and refresh tokens are stored in Postgres only; refresh tokens are encrypted at rest.
- Unauthenticated access to `/search`, `/compare`, or `/settings` redirects to `/`.
- Branding is "Concert Buddies".
