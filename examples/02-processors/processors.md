# Processor catalog (design-only, NiFi 2.12.0)

Property spellings are the NiFi 2.x standard surface; confirm each
against a live 2.12.0 UI at validation. Placeholders only — never real
credentials, never real private endpoints.

## GetFile

- Purpose: one flowfile per file in an input directory, with offset
  state so re-runs do not duplicate consumed files.
- Key properties: Input Directory (design-only spelling), Keep Source
  File (false for consume, true for copy), file filter pattern.
- Failure: missing directory → error + retry on schedule; the flow
  does not fail.

## PutFile

- Purpose: write flowfile content to a directory (landing zone,
  dead-letter, archive).
- Key properties: Directory, Conflict Resolution Strategy (replace /
  ignore / fail / rename).
- Failure: unwritable directory → `failure` relationship; never
  silently drop.

## InvokeHTTP

- Purpose: call an HTTP endpoint; response body becomes flowfile
  content, status code becomes an attribute.
- Key properties: HTTP Method, Remote URL (placeholder only),
  connection/read timeouts.
- Failure: 4xx/5xx → `failure`/`retry` relationships; bound retries
  with penalization, never an unbounded retry loop.

## RouteOnAttribute

- Purpose: route flowfiles by attribute expression (e.g. filename
  matching `*.json`).
- Key properties: one dynamic property per route (name = route,
  value = expression); `unmatched` relationship catches the rest.
- Failure: no matching route and no `unmatched` destination → design
  error; connect or terminate deliberately.

## LogAttribute

- Purpose: log flowfile attributes (and optionally content) for
  verification.
- Key properties: Log Level, Attributes to Log.
- Failure: none data-affecting; logging never alters the flowfile.

## PutDatabaseRecord

- Purpose: write records to a database via a record reader/writer and
  a DB connection pool (used in pattern 03).
- Key properties: Record Reader, Record Writer, Statement Type, DBCP
  Connection Pool (placeholder credentials via parameter context).
- Failure: DB error → `failure` relationship; the fleet's honest
  answer when DB auth is unavailable is the file landing-zone fallback
  (pattern 03), not a faked write.
