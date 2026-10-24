# Phase 1 — Repository and Architecture Audit

**Audit date:** 2026-10-24  
**Scope:** Phase 1 only, based on the workspace available at `C:\Users\shrey\disaster_pbl` and the supplied project specification.

## Executive finding

The available workspace is not the application repository. A full recursive listing found only `.freebuff/project-id` and `.freebuff/.gitignore` (three filesystem entries including the directory). `git rev-parse --show-toplevel` reports that this directory is not a Git repository. No source files, application configuration, docs, migrations, tests, or `handoff.md` are present.

Consequently, this audit cannot verify an existing React/FastAPI/PostgreSQL/PostGIS/Kotlin/Three.js/A* implementation, describe its actual data flow, or guarantee preservation of current behavior. The architecture and file-change areas below are a **proposed target based on the supplied specification**, not claims about code found here. Phase 1 remains incomplete until the application checkout is available and these findings are reconciled against it.

## Repository evidence and feature inventory

| Area | Evidence in available workspace | Finding |
|---|---|---|
| React frontend | No `package.json`, JS/TS, or frontend source | Not verifiable; no frontend files found |
| FastAPI/Python backend | No Python source or dependency manifest | Not verifiable; no API routes found |
| PostgreSQL/PostGIS | No database config, models, migrations, or schema | Not verifiable; extension/configuration status unknown |
| Kotlin/Android | No Gradle files or Kotlin source | Not verifiable; no Android screens or routing found |
| 3D twin | No scene/rendering source or assets | Not verifiable; framework and geometry unknown |
| A* evacuation routing | No graph or routing code | Not verifiable; route representation unknown |
| Alerts/simulation | No alert, scheduler, WebSocket, or simulation code | Not verifiable; existing functionality unknown |
| FCM | No Firebase configuration or notification provider | Not verifiable; setup and delivery status unknown |
| Git/history | Not a Git repository | No branch, commit, or change history available for audit |
| Docs/tests | No README, handoff, test files, or project manifests | No project conventions or test commands can be identified |

The only readable metadata is `.freebuff/project-id` (project identifier) and `.freebuff/.gitignore` (excludes that metadata). Neither describes the application.

## Proposed target architecture and data flow

This is a planning model derived from the project brief; it must be checked against existing code before implementation.

```text
Authorized external feeds (USGS/NCS, INCOIS, IMD/CWC where permitted)
            │
            ▼
Backend source adapters ──► normalized, timestamped source observations
                                    │
                         GIS/building intersection
                                    │
                                    ▼
                      model-based hazard assessments
                                    │
          ┌─────────────────────────┴─────────────────────────┐
          ▼                                                   ▼
Scenario engine (simulation clock)                 real-world incident workflow
          │                                                   │
          └─────────────► persisted lifecycle ◄───────────────┘
                                    │
                       alert/scheduler service
                        ┌───────────┴───────────┐
                        ▼                       ▼
                 REST/WebSocket          FCM provider interface
                  React dashboard          Android client
                        │                       │
                        └───────┬───────────────┘
                                ▼
              3D building twin + affected zones + A* routes
```

PostgreSQL/PostGIS is the proposed persistence and spatial-query layer. Source observations, model assessments, hypothetical simulation state, and authorized emergency declarations need separate provenance and controls. A simulation clock must remain isolated from real-world alert times and public emergency workflows. WebSocket updates serve connected clients; FCM is a background delivery channel whose provider acceptance must not be presented as proof that a person saw an alert. Android should use its A* graph for indoor routing; outdoor routing requires mapped pedestrian data and verified safe destinations. Neither route data nor building footprints should imply complete indoor geometry where it is missing.

## Dependency map (proposed)

| Component | Depends on | Provides |
|---|---|---|
| Source adapters | Authorized access, source-specific formats, freshness/validation policy | Provenanced observations and ingestion records |
| GIS/building layer | PostGIS, CRS policy, building footprints, streets/entrances | Bounded spatial queries and building/feature intersections |
| Assessment/scenario layer | Observations or explicit hypothetical inputs, building geometry/connectivity | Separate risk assessments and reproducible scenario state |
| Persistence/API layer | Existing auth, schema/migration conventions, PostgreSQL | Validated records, role-checked actions, client-facing endpoints |
| Scheduler | Durable scenario/alert state, restart/recovery policy | Idempotent transitions, execution history, cancellation/resolution |
| 3D twin | Building/floor/zone geometry and scenario state | Visualized affected zones and explicit model assumptions |
| Evacuation routing | Room/corridor/stair/exit graph, blocked-edge state | A* route snapshots or explicit no-route outcome |
| React dashboard | API, GIS layers, WebSocket events, 3D twin | Manager configuration, review, and simulation controls |
| Kotlin app | Alert API, WebSocket, FCM tokens/provider, local A* graph | Civilian alerts, acknowledgement, history, route guidance |
| Audit/delivery records | Alert lifecycle, provider results, acknowledgement events | Inspectable operational history; delivery state bounded by provider evidence |

## Proposed file-change plan (paths are placeholders)

No matching files exist yet. Once the actual source tree is supplied, adapt these to its conventions rather than creating parallel frameworks.

| Work area | Likely files/locations to inspect or extend | Main purpose |
|---|---|---|
| Repository baseline | `README.md`, `handoff.md`, `docs/architecture.md` | Setup, architecture, phase status, limitations |
| Backend data/API | Existing FastAPI app, models/schemas, route modules, auth | Extend existing interfaces; separate observations, assessments, simulations, alerts |
| Database | Existing Alembic migrations or migration convention | Add constrained models, foreign keys, audit records, PostGIS geometry/indexes |
| GIS ingestion/API | Backend GIS modules and import scripts/docs | Bounded GeoJSON and reproducible regional data import with attribution |
| Hazard sources | Source-adapter modules and ingestion tests | Independently validate source freshness, provenance, duplicates, and failures |
| Scheduling/scenarios | Existing background-job setup and scenario modules | Durable, recoverable, idempotent simulated timeline and alert lifecycle |
| 3D twin | Existing React scene/components and building assets | Render scenario impact without replacing the rendering framework |
| Routing | Existing A* implementation and graph model (possibly Android) | Apply supported blocked edges and return alternatives/no-route honestly |
| Android | Existing Kotlin screens, API client, routing and notification code | Alerts/history/acknowledgement and provider-backed push integration |
| Frontend | Existing React dashboard/map/components | Manager workflows, provenance, simulation separation, delivery records |
| Verification/docs | Existing test layout and setup docs | Mocked end-to-end demonstration and reproducible setup instructions |

## Phase boundaries and acceptance criteria

Implementation stays phase-by-phase as requested. The following criteria are taken from the supplied brief and are proposed gates; they are not satisfied by this audit.

1. **Phase 1 — Audit:** inspect the real repository and existing behavior; map actual data flow, building/floor/room graph, A*, alert/simulation, PostGIS/FCM/WebSocket setup, migrations, and tests; document architecture, feature inventory, file plan, dependencies, and handoff. No existing functionality removed.
2. **Phase 2 — Data model:** migrations run; records persist across restart; spatial intersection, foreign keys, and state constraints have tests; lifecycle, evidence, and severity remain distinct.
3. **Phase 3 — GIS:** bounded, validated GeoJSON for chosen study region; mapped-data gaps reported; attribution/licensing documented.
4. **Phase 4 — Ingestion:** independent adapters; source, timestamps, quality/freshness, and failures recorded; no unsupported forecast or declaration claims.
5. **Phase 5 — Assessment:** building impact assessment explicitly labels model assumptions and stays distinct from source facts and authorized declarations.
6. **Phase 6 — Alerts/scheduler:** scheduled state persists and recovers; transition history is recorded; cancellation/resolution and role controls work safely.
7. **Phase 7 — Simulation:** complete configurable T−24h-to-activation demo runs on accelerated simulated time; real timestamps remain separate; each transition persists.
8. **Phase 8 — 3D twin:** existing renderer is extended; state/affected zones update; assumptions and simulated visuals are clear.
9. **Phase 9 — Routing:** A* responds to justified graph changes; tests cover blocked path, alternative, supported multi-floor route, and no valid route; simulated future hazard alone does not instruct civilians to evacuate.
10. **Phase 10 — Android/FCM:** live connected updates and persisted acknowledgements work; FCM is testable with supplied credentials; delivery claims match provider evidence.
11. **Phase 11 — Dashboard:** manager can configure, inspect, run, and review scenarios and records; real and simulated alerts are distinguishable; existing UI conventions preserved.
12. **Phase 12 — Demonstration:** repeatable mock workflow and requested failure cases are documented and tested; no genuine public notification is sent during tests.

## Commands and verification performed

- `Get-ChildItem -Force` and recursive listing: confirmed the workspace contains only Freebuff metadata.
- `rg --files --hidden` with project/source/document manifest patterns: found only `.freebuff/project-id` and `.freebuff/.gitignore`.
- `git status --short --branch`, `git rev-parse --show-toplevel`, and `git log -1 --oneline`: each reports this is not a Git repository.
- Read `.freebuff/.gitignore` and `.freebuff/project-id` to identify the metadata purpose.
- No tests were run: no source tree, test suite, dependency manifest, or runnable application is present.

## Limitations and next step

This workspace does not contain the application that the brief says to extend. The architecture, feature inventory, and proposed file map therefore cannot yet be reconciled against existing components, and preservation of existing functionality cannot be assessed. Provide/open the actual project checkout in this workspace (including its source and Git metadata if available); then repeat the repository audit before starting Phase 2. No external integration, source access, running application, or credential was tested or assumed.
