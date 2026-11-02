# Persist top-artist snapshots instead of live Spotify fetches

**Status**: accepted

Comparisons read stored per-user, per-time-range snapshots taken at login and on manual refresh, rather than fetching both users' top artists from Spotify at comparison time. Snapshots keep comparisons fast and independent of the other user's token validity and of Spotify rate limits. The cost is staleness, bounded by the manual refresh action.

## Consequences

- A failed refresh must not corrupt data: all three ranges are fetched first, then written in one transaction.
- Spotify-derived data is retained after token revocation until the user deletes it; deletion is an explicit action.
