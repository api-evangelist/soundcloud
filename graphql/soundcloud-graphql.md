# SoundCloud GraphQL Schema

This document describes a conceptual GraphQL schema for the SoundCloud platform API. SoundCloud provides a REST API v2 for audio streaming and music discovery, covering tracks, users, playlists, social interactions, comments, search, and OAuth authentication. This schema maps those REST resources and relationships into a strongly-typed GraphQL surface.

## Schema Source

- **Provider**: SoundCloud
- **API Version**: v2 (REST)
- **Developer Portal**: https://developers.soundcloud.com/
- **Base URL**: https://api.soundcloud.com
- **Authentication**: OAuth 2.1 with PKCE (Authorization Code and Client Credentials flows)

## Type Overview

### Tracks
Core audio content types covering track metadata, media, waveforms, statistics, and discovery attributes.

- `Track` — Core track entity with id, title, description, genre, tags, license, and sharing state.
- `TrackDetails` — Extended track metadata including BPM, key signature, duration, and recording details.
- `TrackMedia` — Audio stream URLs and transcodings (HLS, progressive) with format and quality metadata.
- `TrackTag` — Individual genre or descriptive tag attached to a track.
- `TrackGenre` — Genre classification string used for filtering and discovery.
- `TrackType` — Enumeration of track kinds: original, remix, live, recording, spoken word.
- `TrackLicense` — Creative Commons or all-rights-reserved license attached to a track.
- `TrackBPM` — Beats-per-minute value declared by the uploader.
- `TrackKeySignature` — Musical key and mode (major/minor) declared by the uploader.
- `TrackWaveform` — Waveform data URL used to render the visual audio scrubber.
- `TrackStats` — Aggregate counters: play count, likes, reposts, comments, downloads.
- `PlaybackCount` — Scoped playback counter with time-range and region breakdown.

### Social Interactions
Types representing user engagement actions on tracks and playlists.

- `Likes` — Collection of like records with pagination, filterable by entity type.
- `Reposts` — Collection of repost records for a track or playlist.
- `Comments` — Paginated list of comments on a track.
- `Downloads` — Download permission flag and download count for a track.
- `RePost` — A repost action: who reposted, what was reposted, and when.
- `Like` — A like action: actor user, target entity (track or playlist), and timestamp.
- `Comment` — Single comment with author, body, optional timestamp offset, and creation date.
- `CommentBody` — Text content of a comment.
- `CommentTimestamp` — Seconds offset into the track where a timed comment is anchored.

### Playlists
Set and ordered-collection types for grouping tracks.

- `Playlist` — A set or album with id, title, description, artwork, sharing settings, and track list.
- `PlaylistDetails` — Extended playlist metadata: release date, label name, EAN, purchase URL.
- `PlaylistType` — Enumeration of playlist kinds: playlist, album, ep, single, compilation.
- `PlaylistTracks` — Ordered list of `Track` objects within a playlist with position index.
- `PlaylistSharingType` — Visibility enum: public, private, unlisted.

### Users and Profiles
Types for SoundCloud accounts, creator tiers, and follow relationships.

- `User` — Core user entity with id, username, display name, avatar, and follower counts.
- `UserDetails` — Extended profile data: city, country, website, bio, and social links.
- `UserProfile` — Combined view of `User` and `UserDetails` used for public profile rendering.
- `FollowRelation` — Directed relationship between a follower and followee with timestamp.
- `Follower` — A user who follows the target account.
- `Following` — A user that the target account follows.
- `CreatorProfile` — Augmented profile for content creators with upload counts and spotlight tracks.
- `ArtistProfile` — Artist-tier profile with verified badge, spotlight items, and station link.
- `SoundCloudPro` — Subscription status object indicating Pro or Pro Unlimited tier perks.

### Streams and Discovery
Types for the personalised feed, charts, artist radio, and algorithmic discovery.

- `StationStream` — Artist or track radio station delivering an infinite stream of related tracks.
- `ArtistStation` — Station seeded from a specific artist with seed track and related tracks list.
- `ChartsPlaylist` — Genre or category chart with ranked tracks and a chart kind (top, trending).
- `Trending` — Trending tracks collection scoped by genre and region.
- `Activity` — A single event in the user's activity feed (track posted, reposted, liked).
- `ActivityHistory` — Paginated list of `Activity` events with a next-cursor for streaming.
- `Stream` — The authenticated user's personalised home stream of followed artists' activities.
- `Discovery` — Discovery page payload containing multiple `DiscoveryBucket` groupings.
- `DiscoveryBucket` — Named bucket of recommended tracks or playlists within the discovery surface.

### Auth and Identity
Types for OAuth credentials, tokens, and the authenticated user context.

- `OAuthToken` — Access and refresh token pair with expiry, scope, and token type.
- `APIKey` — Client ID and client secret pair used to identify an application.
- `Token` — Generic bearer token wrapper (used in responses from the token endpoint).
- `Me` — Authenticated user context: current `User` plus subscription, quota, and settings.

### Search
Types for full-text and filtered search results across tracks, playlists, users, and albums.

- `SearchResult` — Union result type returned from search queries, discriminated by kind field.

### Messaging
Types for direct messages between users.

- `MessageThread` — A conversation thread between two or more users with messages and participants.

### Webhooks
Types for event subscription and delivery.

- `Webhook` — Registered webhook endpoint with event types, secret, and delivery status.

### Subscriptions and Tiers
Types for SoundCloud Go and Pro subscription plans.

- `Subscription` — A user's active subscription record with plan, start date, and renewal date.
- `TierType` — Enumeration of subscription tiers: free, go, go_plus, pro, pro_unlimited.

## Query Examples

```graphql
# Fetch a track with waveform and stats
query GetTrack($id: ID!) {
  track(id: $id) {
    id
    title
    genre
    license { name spdxId }
    waveform { url }
    stats { playbackCount likes reposts comments }
    media { transcodings { url format { protocol mimeType } } }
  }
}

# Fetch the authenticated user's stream
query GetStream($limit: Int, $cursor: String) {
  stream(limit: $limit, cursor: $cursor) {
    items {
      ... on Track { id title user { username } }
      ... on Playlist { id title trackCount }
    }
    nextCursor
  }
}

# Search for tracks
query SearchTracks($q: String!, $limit: Int) {
  search(q: $q, kind: TRACKS, limit: $limit) {
    ... on Track { id title user { username } genre stats { playbackCount } }
  }
}
```

## Mutation Examples

```graphql
# Like a track
mutation LikeTrack($trackId: ID!) {
  likeTrack(trackId: $trackId) {
    success
    like { id createdAt user { id username } }
  }
}

# Post a comment
mutation PostComment($trackId: ID!, $body: String!, $timestamp: Float) {
  createComment(trackId: $trackId, body: $body, timestamp: $timestamp) {
    comment { id body timestamp createdAt user { username avatarUrl } }
  }
}

# Follow a user
mutation FollowUser($userId: ID!) {
  followUser(userId: $userId) {
    success
    relation { followerId followeeId createdAt }
  }
}
```

## Notes

- The SoundCloud public REST API v2 does not expose a native GraphQL endpoint. This schema is a conceptual mapping intended to illustrate how the REST resources could be represented as a typed GraphQL surface.
- Authentication uses OAuth 2.1 with PKCE. The `OAuthToken` type captures the token response structure.
- Stream and discovery endpoints are only available to authenticated users.
- Track media transcodings include both HLS (for adaptive streaming) and progressive (for direct download) formats.
