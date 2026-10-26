# Flow 03 — Feed-to-PostGIS build sheet (design-only, NiFi 2.12.0)

Two branches, one gate. The database branch runs when DB auth works;
the landing-zone branch runs when it does not. The run reports which
branch it took — never a database write claimed for a file landing.

## Processors

| Processor | Key properties (design-only spellings) |
|-----------|----------------------------------------|
| InvokeHTTP | Method GET; Remote URL: placeholder public feed URL; schedule per feed cadence |
| EvaluateJsonPath / ConvertRecord | Shape each record: id, title, geometry, observed time (reader/writer pair to confirm) |
| RouteOnAttribute | `db` route: `db.available` parameter true; `landing` route: otherwise |
| PutDatabaseRecord (db branch) | Record Reader/Writer; Statement Type INSERT; DBCP pool (parameter-context credentials, placeholders only); target PostGIS table + geometry column (to confirm) |
| PutFile (landing branch) | Directory: landing zone; one file per record (or one batch file — decide at validation and record it) |
| LogAttribute (both branches) | Log Level INFO; Attributes: uuid, filename, branch |

## Connections

| From | Relationship | To |
|------|--------------|-----|
| InvokeHTTP | `success` | EvaluateJsonPath / ConvertRecord |
| EvaluateJsonPath / ConvertRecord | `success` | RouteOnAttribute |
| RouteOnAttribute | `db` | PutDatabaseRecord → LogAttribute |
| RouteOnAttribute | `landing` | PutFile → LogAttribute |
| InvokeHTTP | `retry` | Bounded retry + penalization, then run-failure report |
| ConvertRecord | `failure` | Preserved for inspection (bad record never kills the batch) |

## Verify

1. Feed record count N (from the feed itself).
2. Database branch: PostGIS row count after the run == N; geometries
   validate.
3. Landing-zone branch: file count == N; run report states explicitly
   that the database branch was skipped for auth.
4. Every record accounted for on exactly one branch.

## Fleet reality (kept in the pattern, not hidden)

NiFi flows on Node-1 and Node-3 were auth-blocked with a file
landing-zone fallback delivered honestly. Until a validation run proves
the database branch against a live PostGIS with working auth, the
landing-zone branch is the proven fleet behavior.

## Probe rule (fleet fact)

Probe NiFi at `https://localhost:8443/nifi/`. A 400 from
`https://127.0.0.1:8443/nifi/` is a host-handling probe artifact,
not an outage.
