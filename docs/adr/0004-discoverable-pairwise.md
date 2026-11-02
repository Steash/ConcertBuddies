# Discoverable-by-default pairwise comparison, no friend graph

**Status**: accepted

The legacy "music buddies" feature compared you only against accepted friends, which also acted as a consent boundary. v1 drops the friend-request graph and replaces it with a single `discoverable` flag (on by default, opt-out): discoverable users appear in search and can be compared pairwise, while non-discoverable users are hidden and return not-found. One boolean preserves a consent mechanism without rebuilding the social graph.

## Consequences

- Any logged-in user can compare against any discoverable user.
- "Delete my data" is the explicit path for removing a user's snapshots; opting out only hides them.
