# Hand-rolled Spotify OAuth with database sessions

**Status**: accepted

We implement Spotify's Authorization Code flow with PKCE and `state`, store access and refresh tokens server-side in Postgres, and keep only a random session id in an `httpOnly` cookie. Authorization checks live in a server-only data access layer. This gives explicit control over Spotify's refresh-token rotation and avoids depending on a beta auth library against this unfamiliar Next.js version.

## Considered Options

- Auth.js/NextAuth with the Spotify provider and a database adapter.
- A hosted provider (Clerk, Better Auth, etc.).

Auth.js still requires custom access-token refresh code for Spotify and carries version-compatibility risk; hosted providers may not expose the raw Spotify refresh token the manual refresh needs.
