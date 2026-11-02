# Unweighted set intersection at top fifty

**Status**: accepted

A mutual artist is any artist present in both users' top fifty for the same time range; rank orders the result but does not affect whether an artist matches. The comparison deliberately avoids a rank-weighted similarity score because an unweighted intersection is explainable, testable, and matches the legacy behavior. A score can be added alongside the set later without changing the set semantics.

## Consequences

- Popular artists make matches common at N=50; this is accepted for v1.
