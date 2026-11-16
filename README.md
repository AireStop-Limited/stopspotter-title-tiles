# StopSpotter title boundary tiles

Builds the UK title-boundary lookup tiles that power StopSpotter's
on-demand "Look up title" admin feature. Scrapes HM Land Registry
(England & Wales) and Registers of Scotland open data, parses it, and
publishes gzipped NDJSON shards to S3 (`airestop-title-tiles`, served via
CloudFront) that
[StopSpotter](https://github.com/AireStop-Limited/StopSpotter)'s
`lookup-title-boundary` Supabase Edge Function fetches at query time.

## Why this is a separate repo

Originally part of the `stopspotter` repo; split out so the data pipeline
is independent of the app. Both repos are now private (Nov 2026), so the
tiles are published to S3 + CloudFront rather than GitHub Release assets:
a private repo's release assets can't be fetched by the edge function's
plain unauthenticated `fetch()`. Note the ~2-hour quarterly run now uses
metered Actions minutes (private repo).

This code has zero dependency on Supabase, Postgres, or anything else
app-specific — by design, see `docs/TITLE_BOUNDARY_API_PLAN.md`.

## Usage

```bash
npm install

# England & Wales
node scripts/build-title-tiles-ew.mjs --all [--dry-run] [--out=./title-tiles-build]
node scripts/build-title-tiles-ew.mjs --council="Exeter" [--dry-run]

# Scotland
node scripts/build-title-tiles-scotland.mjs --all [--dry-run] [--out=./title-tiles-build]
node scripts/build-title-tiles-scotland.mjs --county="Lanark" [--dry-run]

# Unit tests
npm test
```

`.github/workflows/build-title-tiles.yml` runs both scripts quarterly and
publishes the result to `s3://airestop-title-tiles/title-tiles-latest-<ew|scotland>-<partition>/`
(via the `stopspotter-title-tiles-publish` OIDC role), served through
CloudFront, which `stopspotter`'s edge function reads from directly — see
`docs/TITLE_BOUNDARY_API_PLAN.md` for the full architecture and incident
history.
