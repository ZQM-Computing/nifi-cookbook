# 02 — Processors: Files, HTTP, and Databases

**Design-only — gate 1 target. Not runtime-validated.** This pattern
covers the processors a working ingestion flow actually uses: getting
files, calling HTTP endpoints, routing on attributes, and writing to a
database — plus the demo flow that ties them together. Processor choice
is the flow's architecture: pick the processor whose contract matches
the data (streaming vs batch, at-least-once vs exactly-once), not the
one whose name sounds closest.

## Prerequisites

- NiFi 2.12.0 — fleet seed version
- Pattern 01's first flow built (processors, connections, queues)
- A directory of synthetic input files and a placeholder HTTP endpoint
  (never a real private endpoint; never real credentials)

## Files

- `processors.md` — processor catalog: purpose, key properties, and
  failure behavior for each processor in the demo flow
- `demo.md` — the demo flow build sheet: GetFile → RouteOnAttribute →
  LogAttribute, with a dead-letter branch

## The demo flow

GetFile (input directory of synthetic files, one flowfile per file) →
RouteOnAttribute (route on `filename` matching `*.json` vs everything
else) → LogAttribute on each branch. Unmatched files go to a PutFile
dead-letter directory instead of being dropped.

## Expected output

- Each synthetic file becomes exactly one flowfile (GetFile keeps
  offset state; re-running does not duplicate files already consumed).
- `.json` files route to the JSON branch; all other files route to the
  dead-letter branch; nothing is silently dropped.
- LogAttribute lines on each branch show the routed filename and branch.
- The dead-letter directory contains exactly the non-JSON files.

## Failure behavior

- GetFile on a missing directory → processor logs the error and
  retries on schedule; it does not fail the flow.
- RouteOnAttribute with no matching route → the flowfile goes to the
  `unmatched` relationship; an unmatched flowfile with no destination
  is a design error — connect it or terminate it deliberately.
- InvokeHTTP (used in pattern 03) on a 4xx/5xx → the flowfile routes
  to `failure`/`retry` relationships per configuration; a retry loop
  without back-off is a design error — penalize and bound retries.
- PutDatabaseRecord (used in pattern 03) on a DB error → the flowfile
  routes to `failure`; the fleet's landing-zone fallback (pattern 03)
  is the honest answer when DB auth is unavailable.
- To confirm at validation: the exact 2.12.0 property spellings in
  `processors.md` and the GetFile state behavior on the fleet's
  filesystem.

## Versions note

NiFi 2.12.0 per the fleet seed. Processor names are the NiFi 2.x
standard set; property spellings are design-only until a validation run
proves them against a live 2.12.0 UI.
