# Service build sheet (design-only, NiFi 2.12.0)

## Node-1 route: NSSM wrapping the JVM directly

Fleet fact: on Node-1 the NiFi bootstrap is unusable, so the service
wraps the JVM launch directly. NSSM command shape (sketch — the
verbatim JVM command line is read from the live Node-1 service
definition at validation, never guessed):

```powershell
nssm install ZQM-NiFi <java-exe> <jvm-args-from-live-service-definition>
nssm set ZQM-NiFi AppDirectory <nifi-home>
nssm set ZQM-NiFi AppEnvironmentExtra JAVA_HOME=<jdk-path>
nssm set ZQM-NiFi Start SERVICE_AUTO_START
nssm start ZQM-NiFi
```

Notes:

- `nssm set AppEnvironmentExtra` REPLACES the whole environment block
  — pass all variables in one call (standing fleet lesson).
- JDK path on Node-1 is `C:\java21-jdk` (fleet evidence); confirm the
  path on any other node before reuse.
- Create any log directory BEFORE the first service start: NSSM cannot
  open log files in a nonexistent directory and the service dies with
  zero logs (standing fleet lesson).

## Node-4 state: installed but NOT RUNNING (correct status)

Fleet fact: NiFi 2.12.0 is installed on Node-4 but not running. Its
bootstrap management-port range 52020–52050 is hardcoded, and the
excluded ranges on that boot cover it entirely. Remedy (WinNAT restart
or next boot) needs Alex's decision. Until then, report "installed but
not running" — not a failed install, not a running service.

## Probe + restart policy

- Probe: `https://localhost:8443/nifi/` (localhost, never bare-IP
  127.0.0.1 — the 400 there is a host-handling artifact, not an outage).
- After a host reboot (only when one is permitted — never forced by
  this pattern): the service must return to Running without manual
  action, or its failure must be logged with a reason.
- Auth: single-user credentials via `nifi.cmd set-single-user-credentials`
  are an owner decision. Placeholders only in every example; never a
  default or real credential.

## To confirm at validation

- Verbatim Node-1 NSSM service definition (command line, environment,
  working directory).
- Node-4 port-range remedy decision (WinNAT restart vs next boot).
- Post-reboot behavior on each node.
