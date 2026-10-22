# Design context

> Copy this file to the root of the audited repository as `design-context.md` and fill in what you know. Every section is optional. Anything left empty will be inferred by the skills and reported as an assumption.

## System overview

<!-- What the system does, in two or three sentences, using business vocabulary. -->

## Architecture and layers

<!-- Intended layering and allowed dependency directions. Example:
- Layers (outer → inner): `web/` → `application/` → `domain/`; `infrastructure/` implements ports declared in `domain/`
- `domain/` must not import `web/` or `infrastructure/`
- `shared/` may be used by every layer
- CQRS read side (`queries/`) may read the database directly
-->

## Intended design style

<!-- Tick or describe what applies. -->
- [ ] Rich domain model (DDD-style entities, value objects, aggregates)
- [ ] Transaction scripts / CRUD (anaemic models are intended)
- [ ] Functional core, imperative shell
- [ ] Hexagonal / ports and adapters
- [ ] Other:

## Domain vocabulary

<!-- Key business terms and their meaning; synonyms to avoid; terms that differ per context. -->

| Term | Meaning | Used in context(s) |
|---|---|---|
| | | |

## Test conventions

<!-- How to tell test levels apart. Example:
- Unit: `src/**/__tests__/*.test.ts`
- Integration: `tests/integration/**`, uses Testcontainers
- E2E: `e2e/**`, Playwright
-->

## History analysis

- Window: <!-- default: last 12 months -->
- Additional paths to exclude: <!-- e.g. `docs/**`, `infra/terraform/**` -->
- Expected high-churn files (not to report as hotspots): <!-- e.g. `CHANGELOG.md`, `routes.ts`, feature-flag config -->

## Deliberate exceptions

<!-- Known, accepted design choices the skills should not report. -->

| Path or pattern | Issue ID | Reason |
|---|---|---|
| | | |

## Out of scope

<!-- Paths the audit should ignore entirely (generated code, vendored libraries, legacy areas scheduled for deletion). -->
