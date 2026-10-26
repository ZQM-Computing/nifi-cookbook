# 01 — First Flow: Processors, Connections, Queues

**Design-only — gate 1 target. Not runtime-validated.** This pattern
builds a first NiFi flow in the UI: a generator processor feeding a
logging processor through a connection with a queue. It teaches the
three objects every NiFi flow is made of — processors do work,
connections carry flowfiles, queues buffer between them — and the two
operations every flow needs: start and stop without losing data.

## Prerequisites

- NiFi 2.12.0 — fleet seed version
- Java 21 runtime
- A running NiFi instance with UI access (fleet: Node-1 under NSSM;
  see pattern 04 for service deployment)
- **Probe rule (fleet fact): probe NiFi at `https://localhost:8443/nifi/`.
  A 400 from `https://127.0.0.1:8443/nifi/` is a probe artifact — NiFi's
  host handling rejects the bare-IP host — not an outage.**
- NiFi admin credentials are an owner decision (placeholders only here;
  never bake real credentials into an example)

## Files

- `flow.md` — the flow definition: processors, properties, connections,
  and the start/stop/verify sequence (flows are built in the UI; this
  file is the build sheet, not a claim of a run)

## Steps

1. Open the NiFi UI at `https://localhost:8443/nifi/` (localhost, not
   127.0.0.1 — see the probe rule above).
2. Drag a GenerateFlowFile processor onto the canvas. Set its
   properties: custom text "synthetic-flowfile-001", scheduling period
   60 sec, and a single run schedule that you start by hand.
3. Drag a LogAttribute processor. Set log level INFO and attributes to
   log `uuid, filename, fileSize`.
4. Connect GenerateFlowFile's `success` relationship to LogAttribute.
   Leave the connection queue at defaults (back-pressure thresholds
   are pattern 02's subject).
5. Start GenerateFlowFile, then LogAttribute. Watch the queue count
   rise and drain, and check the NiFi log for the LogAttribute lines.

## Expected output

- The connection queue count rises as flowfiles generate, then drains
  as LogAttribute processes them; the count never grows without bound
  on a 60-second schedule.
- The NiFi log shows one LogAttribute line per flowfile with uuid,
  filename, and fileSize.
- Stopping LogAttribute while GenerateFlowFile runs leaves flowfiles
  queued (not lost); restarting LogAttribute drains them.
- Provenance (Data Provenance → search by processor) shows the
  generate and log events for each flowfile.

## Failure behavior

- Processor in `Invalid` state (missing required property) → it will
  not start; the UI names the missing property — fix the property, do
  not delete and recreate the processor.
- Unconnected relationship (e.g. `failure` with no destination) →
  NiFi warns on start; connect it or terminate it deliberately at the
  relationship, never leave it dangling silently.
- Queue grows without bound → back-pressure or a stopped downstream
  processor; stop the source, drain or inspect, then restart downstream
  first.
- To confirm at validation: the exact 2.12.0 processor property names
  (GenerateFlowFile custom-text property spelling) and the UI flow for
  terminating an unconnected relationship.

## Versions note

NiFi 2.12.0 per the fleet seed. Processor names are the NiFi 2.x
standard set; property spellings are design-only until a validation run
proves them against a live 2.12.0 UI.
