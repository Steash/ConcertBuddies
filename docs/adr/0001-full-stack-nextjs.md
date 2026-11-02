# Full-stack Next.js, retiring the Java server

**Status**: accepted

The legacy app is a Vue 3 client plus a Java Spring Boot server. We are rebuilding as one Next.js App Router application: UI, Spotify calls, persistence, comparison, and auth all live in the same deployable. This removes a second deployable, a second auth story, and a network hop on every comparison, and the server's responsibilities (calling Spotify, storing snapshots, intersecting artists) fit Server Components and Server Actions.

## Considered Options

- Keep the Java server and replace only the Vue client.
- Hybrid: Next.js owns auth and reads, Java keeps stored data.

Both were rejected because they preserve the cross-service hop and split ownership of the same data.
