# Demo flow build sheet (design-only, NiFi 2.12.0)

GetFile → RouteOnAttribute → LogAttribute, with a dead-letter branch.

## Processors

| Processor | Key properties |
|-----------|---------------|
| GetFile | Input Directory: synthetic input dir; Keep Source File: false |
| RouteOnAttribute | `json` route: filename matches `*.json` (expression spelling to confirm) |
| LogAttribute (JSON branch) | Log Level INFO; Attributes: uuid, filename |
| LogAttribute (other branch) | Log Level INFO; Attributes: uuid, filename |
| PutFile (dead-letter) | Directory: dead-letter dir; Conflict Resolution: rename |

## Connections

| From | Relationship | To |
|------|--------------|-----|
| GetFile | `success` | RouteOnAttribute |
| RouteOnAttribute | `json` | LogAttribute (JSON branch) |
| RouteOnAttribute | `unmatched` | LogAttribute (other branch), then PutFile (dead-letter) |

## Verify

1. Drop 2 synthetic `.json` files and 1 synthetic `.txt` file into the
   input directory.
2. Expect: 2 LogAttribute lines on the JSON branch, 1 on the other
   branch, and exactly 1 file in the dead-letter directory.
3. Re-run GetFile: no duplicates (offset state held).
4. Nothing is silently dropped: every input file is accounted for on
   exactly one branch.
