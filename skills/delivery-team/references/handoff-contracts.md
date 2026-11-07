# Handoff contracts

These contracts are the single source for dispatch messages. Paths are relative to the target project root, not to this repository. The Lead verifies the absolute project root and each actual input before dispatch, passes the exact paths and approved revision or content identity, and records the producer and review verdict. Each recipient reads the inputs and reports what it used. A missing, stale or inaccessible input blocks dependent work until resolved or an explicit replacement is agreed; expected paths and summaries are not evidence that files exist. Architecture must be approved before requirements, requirements before implementation, and integrated implementation before test execution. The Lead copies the Definition of Done of a stage into the dispatch message verbatim and checks the same list when the deliverables arrive.

## Stage 1 — role-business-technologist

Inputs:
- The business request from the user or the Lead (text, ticket, or a path to a document).
- `docs/business/**`
- `README.md`, `docs/**`
- Stakeholder list, if provided.

Deliverables:
- `docs/business/process-<name>.md`
- `docs/business/business-requirements.md`

Definition of Done:
- [ ] Every to-be step has an actor, an input and an output.
- [ ] Every business requirement has a priority and a source.
- [ ] Every KPI has a formula and a target value or an explicit `unknown` with an owner.
- [ ] Open questions list is non-empty or explicitly states "нет открытых вопросов".
- [ ] Documents are in Russian; identifiers and file names are in English.

## Stage 2 — role-enterprise-architect

Inputs:
- `docs/business/business-requirements.md`, `BRQ-...`
- `docs/business/process-*.md`
- `docs/architecture/**`
- Inventory of existing systems (list, registry export or document), if provided by the user or the Lead.
- `README.md`, `docs/**`

Deliverables:
- `docs/architecture/enterprise-context.md`
- `docs/architecture/adr/0000-template.md` — ADR template

Definition of Done:
- [ ] Every system in the landscape has an owner, a purpose and a status.
- [ ] Every business capability from the processes is mapped to a system or marked `missing`.
- [ ] Every principle has a rationale and at least one implication for design.
- [ ] Every constraint names its source.
- [ ] The roadmap covers every `BRQ-...` with priority Must or Should, or explains why not.
- [ ] Open questions list is non-empty or explicitly states "нет открытых вопросов".

## Stage 3 — role-solution-architect

Inputs:
- `docs/architecture/enterprise-context.md`
- `docs/business/business-requirements.md`, `BRQ-...`
- `docs/architecture/adr/*`, `0000-template.md`
- Existing code structure of the target project: top-level directories, build files, dependency manifests, configuration.
- `README.md`, `docs/**`

Deliverables:
- `docs/architecture/solution-design.md`
- `docs/architecture/adr/NNNN-<slug>.md` — one file per significant decision; `NNNN` is a 4-digit sequence starting at `0001`, `<slug>` is a kebab-case English slug
- `docs/architecture/spikes/<slug>/` — optional small prototypes that prove a decision; each with a `README.md` that states the question and the result.

Definition of Done:
- [ ] Context and container views exist as Mermaid diagrams and match the enterprise landscape.
- [ ] Every integration row has protocol, direction, data and SLA, or an explicit `TBD` with an owner.
- [ ] Every `NFR-...` has a metric and a target, or `TBD` with an owner.
- [ ] Every technology, store, protocol and hosting choice is backed by an ADR with status `Принято`.
- [ ] Every `BRQ-...` with priority Must is addressed by a container or component, or listed as an open question.
- [ ] Open questions list is non-empty or explicitly states "нет открытых вопросов".

## Stage 4 — role-system-analyst

Inputs:
- `docs/business/business-requirements.md`, `BRQ-...`, `BR-...`
- `docs/architecture/solution-design.md`
- `docs/requirements/**`
- Existing API contracts of the target project (OpenAPI, GraphQL, protobuf), if present.
- `README.md`, `docs/**`

Deliverables:
- `docs/requirements/srs.md`
- `docs/requirements/user-stories.md` — one block per story `US-001...`: title, «Как / Я хочу / Чтобы», linked `FR-...`, acceptance criteria as Gherkin scenarios in Russian (Дано / Когда / Тогда).
- `docs/requirements/api/openapi.yaml` — OpenAPI 3.1 skeleton in English: `info`, `paths` with operations and `operationId`, `components/schemas` for every entity of the data model, and a shared `Error` schema (`code`, `message`, `details`) for error responses.
- `docs/requirements/traceability.md` — table with columns `BRQ`, `FR`, `US`, `TC`; the `TC` column stays `TBD` until the tester fills it.

Definition of Done:
- [ ] Every `BRQ-...` maps to at least one `FR-...` in the traceability matrix or is an open question.
- [ ] Every `FR-...` has a priority, a source and a testable acceptance statement.
- [ ] Every `US-...` has at least two Gherkin scenarios and links to an `FR-...`.
- [ ] `openapi.yaml` declares `openapi: 3.1.0` and every schema name matches an entity of the data model.
- [ ] Open questions list is non-empty or explicitly states "нет открытых вопросов".

## Stage 5 — role-developer

Inputs:
- `docs/requirements/srs.md`, `FR-...`
- `docs/requirements/user-stories.md`, `US-...`
- `docs/requirements/api/openapi.yaml`
- `docs/architecture/solution-design.md`, `docs/architecture/adr/*`
- Project conventions: `AGENTS.md`, `CONTRIBUTING.md`, lint and format configs, existing tests and project layout.
- Existing files of the target project that relate to the task.

Deliverables:
- Source code and tests placed in the layout of the target project (its `src/`, `tests/` or equivalent directories).
- `docs/dev/implementation-notes.md`

Definition of Done:
- [ ] Every FR and US in scope is listed in Реализованные требования with its files and tests.
- [ ] The full test suite, linters and formatters pass with the commands written in the notes.
- [ ] Every new behaviour has a unit test; no test was skipped, deleted or weakened to pass.
- [ ] Commits follow Conventional Commits and contain no secrets or credentials.
- [ ] Contradictions with requirements or architecture are reported to the Lead, not resolved silently.

## Stage 5p — role-tester (test plan and cases, parallel with stage 5)

This stage prepares tests only; executing tests and reporting results belong to stage 6 after implementation is integrated. Coordinate file ownership with developers to avoid overlapping writes.

Inputs:
- `docs/requirements/srs.md`, `FR-...`
- `docs/requirements/user-stories.md`, `US-...`
- `docs/requirements/traceability.md`

Deliverables:
- `docs/test/test-plan.md`
- `docs/test/test-cases.md` with a table `TC-001...`: id, FR/US, предусловия, шаги, ожидаемый результат, приоритет.

Definition of Done:
- [ ] Every FR and US in scope has at least one `TC` and the `TC` column of traceability is filled.
- [ ] Each test case has preconditions, steps and an expected result.
- [ ] The test plan states execution environment and exit criteria.

## Stage 6 — role-tester (test execution)

Inputs:
- `docs/requirements/srs.md`, `FR-...`
- `docs/requirements/user-stories.md`, `US-...`
- `docs/requirements/traceability.md`
- `docs/dev/implementation-notes.md`
- `docs/test/test-plan.md`
- `docs/test/test-cases.md` with a table `TC-001...`: id, FR/US, предусловия, шаги, ожидаемый результат, приоритет.

Deliverables:
- Automated tests in the test framework and test directory of the target project.
- `docs/test/test-report.md`
- The `TC` column of `docs/requirements/traceability.md` filled for every requirement in scope.

Definition of Done:
- [ ] Every FR and US in scope has at least one `TC` and the `TC` column of traceability is filled.
- [ ] Every `TC` was executed; command, environment and date of each run are recorded.
- [ ] Every failed or blocked test has a `DEF` with severity, steps, expected, actual and evidence.
- [ ] The verdict in the report follows the exit criteria of the test plan.
- [ ] Automated tests run with one documented command in the project's framework.

## Stage 7 — role-devops-engineer

Inputs:
- `docs/architecture/solution-design.md`
- `docs/dev/implementation-notes.md`
- Existing CI/CD configuration and infrastructure files of the target project (`.github/workflows/`, `.gitlab-ci.yml`, `Dockerfile`, compose files, Terraform, Helm charts and similar).
- The list of target environments with their purpose, from the Lead.
- Existing files of the target project that relate to the task.

Deliverables:
- Pipeline configuration files in the layout of the project's CI platform.
- Infrastructure files: `Dockerfile`, compose file, Terraform or Helm, as appropriate to the project.
- `docs/ops/deployment.md`
- `docs/ops/runbook.md`

Definition of Done:
- [ ] The pipeline covers lint, test, build, package and deploy for every environment from the Lead's list.
- [ ] Build and test commands in the pipeline equal the commands from the implementation notes.
- [ ] No secret value appears in the repository or in the documents; every secret is named with its storage location.
- [ ] The rollback procedure has exact commands and a verification step.
- [ ] Local verification (image build, test run, pipeline lint) was executed and its results are recorded.

## Gate — role-reviewer

Inputs:
- The artifact or artifacts under review: paths given by the Lead (documents, code, tests, pipeline files) and the commit or revision to review.
- The Definition of Done of the stage, from the handoff message or the `delivery-team` references.
- The upstream documents the artifact must conform to: `docs/business/**`, `docs/architecture/**`, `docs/requirements/**`, as relevant to the stage.
- Project conventions: `AGENTS.md`, `CONTRIBUTING.md`, lint and format configs, existing code style.
- Existing files of the target project that relate to the task.

Deliverables:
- `docs/review/review-<YYYY-MM-DD>-<scope>.md`, where `<scope>` is an English kebab-case slug of the stage or artifact (for example `srs`, `developer-stage`, `pipeline`)

Definition of Done:
- [ ] Every artifact in scope was read fully; for code, build and tests were executed and results recorded.
- [ ] Every finding has an id, a severity, a location, a description and a recommendation.
- [ ] Every DoD item of the reviewed stage has a `pass` or `fail` with a linked finding for each `fail`.
- [ ] The verdict follows the rule: `request-changes` with any blocker or major, `approve` otherwise.
- [ ] The review file is in Russian with English identifiers, paths and code excerpts.

Verdict rule: `request-changes` if any `blocker` or `major` finding exists; `approve` otherwise.

## Write scopes

One role per subagent; two subagents never share a write scope at the same time. Everything outside the allowed scope is read-only for that role.

| Role | Allowed write scope |
|---|---|
| `business-technologist` | `docs/business/` |
| `enterprise-architect` | `docs/architecture/enterprise-context.md`, `docs/architecture/adr/0000-template.md` |
| `solution-architect` | `docs/architecture/` |
| `system-analyst` | `docs/requirements/` |
| `developer` | source and test directories of the project plus `docs/dev/` |
| `tester` | test directories plus `docs/test/` and the `TC` column of `docs/requirements/traceability.md` |
| `devops-engineer` | CI/CD and infrastructure files plus `docs/ops/` |
| `reviewer` | `docs/review/` only |
| Lead | `docs/delivery/` |
