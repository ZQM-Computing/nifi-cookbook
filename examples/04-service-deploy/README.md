# 04 — Service Deploy: NiFi as a Windows Service

**Design-only — gate 1 target. Not runtime-validated.** This pattern
runs NiFi as a Windows service, covering the two fleet realities: the
Node-1 route (NSSM wrapping the JVM directly because the bootstrap is
unusable there) and the Node-4 state (installed but not running,
blocked on a port-range decision). Service deployment is where a flow
stops being a demo and starts being infrastructure: it must survive
logoff, restart on failure within policy, and answer its health probe
after a reboot.

## Prerequisites

- NiFi 2.12.0 — fleet seed version
- Java 21 runtime (fleet Node-1: JDK at `C:\java21-jdk` — confirm the
  path on the target node; it is fleet evidence for Node-1, not a
  universal path)
- NSSM (for the Node-1 route)
- Administrator rights on the target node (service install only; no
  other node state touched)
- NiFi admin credentials are an owner decision: the single-user setup
  (`nifi.cmd set-single-user-credentials`) is Alex's call — placeholders
  only here, never baked in

## Files

- `service.md` — the service build sheet: Node-1 NSSM-direct-JVM
  route, Node-4 blocked state, probe + restart policy

## Expected output

- Node-1 route: an NSSM service exists, set to Automatic, wrapping the
  JVM launch directly (bootstrap bypassed); after a service restart,
  `https://localhost:8443/nifi/` answers (200 at the UI path — the
  fleet's probe returns a full page; the exact byte count varies by
  version and is not asserted here).
- The service survives logoff: the UI stays up with no user session.
- After a host reboot (when one is permitted — never forced by this
  pattern), the service returns to Running without manual action, or
  its failure is logged with a reason.
- **Probe rule (fleet fact): probe at `https://localhost:8443/nifi/`.
  A 400 from `https://127.0.0.1:8443/nifi/` is a host-handling probe
  artifact, not an outage.**

## Failure behavior — the fleet's real states, kept honest

- **Node-1: the bootstrap is unusable, so NSSM wraps the JVM
  directly.** If the bootstrap is retried instead, it fails the same
  way; the NSSM-direct-JVM route is the fix, not a workaround to
  apologize for. The verbatim JVM command line (classpath, system
  properties, main class) is read from the live Node-1 service
  definition at validation — it is NOT guessed here.
- **Node-4: installed but NOT RUNNING.** NiFi's bootstrap
  management-port range 52020–52050 is hardcoded, and the excluded
  ranges on that boot cover it entirely. The remedy (WinNAT restart or
  next boot) needs Alex's decision; until then, "installed but not
  running" is the correct status, not a failed install.
- Service starts but UI never answers → check the NiFi log for the
  bootstrap/JVM failure first, then the port bind (8443), then the
  probe host (localhost, never bare-IP 127.0.0.1).
- Single-user auth unset → the UI's auth behavior is the owner's
  decision to make; the pattern never ships a default credential.
- To confirm at validation: the verbatim Node-1 NSSM service
  definition (command line, environment, working directory), the
  Node-4 port-range remedy decision, and the post-reboot behavior on
  each node.

## Versions note

NiFi 2.12.0 per the fleet seed. NSSM command shapes in `service.md`
are sketches with placeholders for the JVM command line; the verbatim
definition is a validation output, read from the live Node-1 service.
