# NiFi Cookbook

<!-- ZQM-START -->
<!-- markdownlint-disable MD033 -->
<p align="center">
  <strong>ZQM Computing — Geospatial Engineering & Scientific Computing</strong><br>
  <a href="https://github.com/ZQM-Computing">ZQM Computing</a>
</p>
<!-- markdownlint-enable MD033 -->
<!-- ZQM-END -->

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

A **NiFi cookbook** for visual dataflow, ingestion, and feed-to-PostGIS pipelines. Patterns are drawn from the ZQM fleet's NiFi 2.12.0 installs (HTTPS port 8443) on Node-1 and Node-3, including the service-deployment differences between nodes and the localhost-only probe rule. It is intended for Data engineers who build ingestion flows visually and need them running as services. Validate each pattern against your own data, software versions, and operating environment before adopting it.

## Repository role and status

**Status:** Active, public — seeding. Pattern passes land in follow-up work from fleet-verified setups; the tables below list what is planned, not what is already written.

**What this repository is:** NiFi patterns — see the table below.

**What it contributes:** Part of the ZQM Computing geospatial portfolio. The umbrella index for the series is [geospatial-cookbooks](https://github.com/ZQM-Computing/geospatial-cookbooks).

**Standing:** This repository is maintained independently. Automated checks cover only the paths and environments configured in its workflows; they do not establish production readiness or operational certification.

## Who this is for

Data engineers who build ingestion flows visually and need them running as services.

## Prerequisites

- NiFi 2.12.0
- Java 21 runtime
- PostGIS-backed Postgres for the feed pass

## Patterns (planned)

| # | Example | What it will demonstrate |
|---|---------|--------------------------|
| 01 | `01-first-flow` | Building a first flow: processors, connections, and queues |
| 02 | `02-processors` | Common processors for files, HTTP, and databases |
| 03 | `03-feed-to-postgis` | A feed-to-PostGIS flow, as run on the fleet |
| 04 | `04-service-deploy` | Running NiFi as a Windows service, including the NSSM-direct-JVM route where the bootstrap is unusable |

## Repository layout

```text
nifi-cookbook/
├── examples/          # pattern passes, landing in follow-up work
├── README.md
└── LICENSE
```

## Roadmap

- First pattern pass: the first two table rows
- Second pass: the remaining rows, from fleet-verified setups

## Contributing

A contributing guide lands with the first pattern pass. Report security issues to <zqmcomputing@gmail.com> — please do not open a public issue for those.

## More from this series

- [prefect-cookbook](https://github.com/ZQM-Computing/prefect-cookbook) — Workflow orchestration, schedules, and deployments
- [dlt-cookbook](https://github.com/ZQM-Computing/dlt-cookbook) — ELT pipelines from Python into Postgres and DuckDB
- [postgis-cookbook](https://github.com/ZQM-Computing/postgis-cookbook) — Spatial indexing, joins & KNN, SRID management, vector tiles
