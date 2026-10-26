# 03 — Feed to PostGIS: Ingestion With an Honest Fallback

**Design-only — gate 1 target. Not runtime-validated.** This pattern is
the fleet's feed-ingestion flow: pull a feed over HTTP, shape its
records, and write them to PostGIS — with the fallback the fleet
actually used when database auth was unavailable. The honest version of
this pattern has two branches and a gate between them, not a single
happy path: the database branch runs when DB auth works; the file
landing-zone branch runs when it does not, and the run reports which
branch it took.

## Prerequisites

- NiFi 2.12.0 — fleet seed version
- PostGIS-backed Postgres for the database branch (fleet: Postgres 17
  / PostGIS 3.6.2 on the Windows nodes)
- A public feed endpoint (placeholder URL in the build sheet; use a
  genuinely public feed, never a private or authenticated one)
- DB credentials via NiFi parameter context (placeholders only; the
  fleet's Postgres auth decisions are owner decisions, never baked in)

## Files

- `flow.md` — the flow build sheet: both branches, the auth gate, and
  the verification sequence

## The flow

InvokeHTTP (GET the public feed on a schedule) → EvaluateJsonPath /
ConvertRecord (shape each record: id, title, geometry, observed time)
→ RouteOnAttribute (DB branch vs landing-zone branch, gated on a
`db.available` parameter) → PutDatabaseRecord (database branch) or
PutFile (landing-zone branch). Both branches end in LogAttribute so
every record is accounted for.

## Expected output

- Database branch (when DB auth works): records land in the target
  PostGIS table; a row count query after the run matches the feed's
  record count; geometries validate as PostGIS geometries.
- Landing-zone branch (when DB auth does not work): one file per
  record (or one batch file, per the build sheet) lands in the landing
  directory; a file count matches the feed's record count; the run
  report states explicitly that the database branch was skipped for
  auth, not that it succeeded.
- Every feed record is accounted for on exactly one branch; none is
  silently dropped, and none is claimed written to the database when it
  only landed in a file.

## Failure behavior — the fleet's real story, kept honest

- **Fleet fact: NiFi flows on Node-1 and Node-3 were auth-blocked with
  a file landing-zone fallback delivered honestly.** This pattern
  documents that outcome as the designed fallback, not as a failure to
  hide: when the DB connection pool cannot authenticate, the flow
  takes the landing-zone branch and says so.
- Feed unreachable → InvokeHTTP routes to `retry` with bounded
  retries and penalization; after the bound, the run reports the feed
  failure instead of looping forever.
- Malformed feed record → ConvertRecord routes it to `failure` with
  the record preserved for inspection; one bad record never kills the
  batch.
- To confirm at validation: the exact 2.12.0 processor property
  spellings, the record reader/writer pair for the feed format, the
  PostGIS table + geometry column the fleet writes to, and the
  parameter-context shape for the `db.available` gate.

## Versions note

NiFi 2.12.0 + Postgres 17 / PostGIS 3.6.2 per the fleet seed. The feed
URL in the build sheet is a placeholder for a genuinely public feed.
The database branch is design-only until a validation run proves it
against a live PostGIS with working auth; until then, the
landing-zone branch is the proven fleet behavior.
