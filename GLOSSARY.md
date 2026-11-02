# Concert Buddies

An app that finds the musical overlap between two Spotify listeners — the "insights Spotify doesn't" give you. This glossary fixes the language the product and the code share.

## Language

**Music Buddy**:
A user who shares at least one mutual artist with another user for a chosen time range.
_Avoid_: Friend, match, twin

**Discoverable**:
A user who may appear in search and in pairwise comparisons. Discoverability defaults on and can be turned off.
_Avoid_: Public, visible, listed

**Snapshot**:
A stored copy of one user's top artists for one time range, captured at a point in time.
_Avoid_: Cache, dump, profile

**Snapshot Refresh**:
Replacing a user's snapshot with a newly fetched copy. A refresh either fully succeeds or leaves the previous snapshot untouched.
_Avoid_: Sync, update, pull

**Time Range**:
One of Spotify's three windows for top artists: short term (about four weeks), medium term (about six months), or long term (several years).
_Avoid_: Period, timeframe, duration

**Top Artist**:
An artist in a user's top fifty for a given time range, carrying a rank.
_Avoid_: Favourite, most-played

**Mutual Artist**:
An artist present in both users' top fifty for the same time range. Rank does not affect whether an artist is mutual.
_Avoid_: Shared artist, common artist, overlap

**Comparison**:
The pairwise view of two users' mutual artists for one time range.
_Avoid_: Match, diff, overlap report

**Core Loop**:
Log in with Spotify, see your top artists, search for a discoverable user, and open a comparison.
_Avoid_: Flow, journey

**Needs Reauth**:
The account state after Spotify rejects a stored token. The user must log in again before another snapshot refresh.
_Avoid_: Expired, disconnected, broken
