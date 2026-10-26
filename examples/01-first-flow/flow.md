# Flow 01 — First flow build sheet (design-only, NiFi 2.12.0)

Flows are built in the NiFi UI. This file is the build sheet: every
processor, property, connection, and verification step, so the flow can
be rebuilt exactly. It is not a claim that this flow ran.

## Processors

| Processor | Key properties (design-only spellings) | Notes |
|-----------|----------------------------------------|-------|
| GenerateFlowFile | Custom Text: `synthetic-flowfile-001`; Scheduling Period: `60 sec`; Run Schedule: manual start | Generates one synthetic flowfile per run |
| LogAttribute | Log Level: `INFO`; Attributes to Log: `uuid, filename, fileSize` | Logs each flowfile; proves delivery |

## Connections

| From | Relationship | To | Queue |
|------|--------------|-----|-------|
| GenerateFlowFile | `success` | LogAttribute | Defaults (back-pressure is pattern 02) |

## Start/stop/verify sequence

1. Start GenerateFlowFile, then LogAttribute.
2. Watch the connection queue count rise and drain.
3. Check the NiFi log for LogAttribute lines (uuid, filename, fileSize).
4. Stop LogAttribute with the source running: flowfiles queue, none lost.
5. Restart LogAttribute: queue drains.
6. Data Provenance search by processor shows generate + log events.

## Probe rule (fleet fact)

Probe at `https://localhost:8443/nifi/`. A 400 from
`https://127.0.0.1:8443/nifi/` is a host-handling probe artifact,
not an outage. Never diagnose an outage from the bare-IP probe.
