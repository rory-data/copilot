---
title: Airflow DAG CICD Options Analysis
description: Focused comparison of three release strategies for a multi-squad Airflow 2.10 deployment — parallel folder separation, tag-based release, and formalised feature toggles
author: Platform Engineering
date: 2026-04-27
status: Draft for Review
tags:
  - airflow
  - cicd
  - release-strategy
  - platform-engineering
version: "1.3"
---
---

**Version:** 1.3 | **Status:** Draft for Review | **Owner:** Platform Engineering

> **Change log v1.2 → v1.3**
> - Option C: Staging auto-validation sub-section added — Jenkins-executable assertions for toggle-off and toggle-on execution paths, DAG parse health, Variable initialisation, and cleanup ticket existence; Staging validation gate added as guard on Production release action
> - Option C delivery flow (Scenario B): explicit `ASSERT:` lines added to Staging step mirroring Options A and B flow style
> - Option C: Vault-backed feature flags sub-section added — operational differences, task-level access patterns, audit trail improvement, local dev friction, and Secrets Backend coupling risk; new row in Toggle Patterns table
> - Option C C1 rationale updated to note Staging auto-validation partially closes the gap; score unchanged (◑)
> - Option C C4 rationale updated to note Vault audit log as materially better audit trail than plain Variables when Vault is already in environment

> **Change log v1.1 → v1.2**
> - Option B: RC assembly rewritten to use previous Production release as base and file-level checkout of tagged files only (eliminates Staging scope contamination; removes "Staging receives all of main" weakness)
> - Option B: C1 score corrected from ◑ to ◕ with updated rationale; weighted score updated from 3.28 to 3.50
> - Option B: delivery flow updated to match corrected assembly model
> - Option B: RC assembly complexity claim corrected — gap vs Option A is smaller than previously stated
> - Option A: `dag_folder` configuration approach made explicit (repo-root with `.airflowignore`)
> - Option C: Airflow 2.x DAG serialisation risk from parse-time `Variable.get()` added
> - Option C: partial audit trail mitigations documented (git tag at enable time, Airflow event log)
> - Branch by Abstraction: added as named pattern in shared assumptions and Option C utility section
> - New section: Other Considered Approaches (image-based deployment, dynamic DAG factory pattern)

---

## Context and Problem Statement

Multiple product squads share a single Airflow 2.10 deployment and a single DAG monorepo. The current ad-hoc release process creates deployment coupling between squads — a squad whose DAG is not yet ready for production must either block other squads or accept a risk of premature deployment. The goal of this analysis is to identify the release strategy that best satisfies:

- **Squad autonomy**: each squad can release its DAGs independently, without blocking or being blocked by other squads
- **Production stability**: a failed DAG change in one squad does not impact running DAGs from other squads
- **Operational simplicity**: the overhead imposed on individual developers and the platform team is proportionate to the benefit
- **Traceability**: there is a clear record of what was deployed, when, and by whom

Three strategies have been identified and are evaluated below. This document does not evaluate tooling choices beyond the current stack (Bitbucket, Jenkins, Astronomer-hosted Airflow 2.10).

---

## Evaluation Criteria

| ID | Criterion | Weight | Description |
| --- | --- | --- | --- |
| C1 | Pre-production validation | 22% | Confidence that the change has been exercised in a production-equivalent environment before the production deployment action is taken |
| C2 | Rollback capability | 25% | Speed, completeness, and reversibility of a rollback; whether rollback requires a new deployment or can be effected at the control plane |
| C3 | Pipeline complexity | 14% | Overhead imposed on developers and the platform team: number of manual steps, tool integrations, and failure modes in the pipeline |
| C4 | Audit trail | 25% | Traceability: who approved what, when the change was enabled in production, whether the trail is tamper-evident |
| C5 | Multi-squad blast radius | 14% | Whether a failed or misbehaving change from one squad can impact the production availability of other squads' DAGs |

**Scoring scale:** ○ Absent / ◑ Partial / ◕ Mostly present / ● Full

**Numeric key:** ○ = 1.0 · ◑ = 2.0 · ◕ = 3.0 · ● = 4.0

---

## Shared Assumptions

- Airflow 2.10 deployed on Astronomer (self-hosted or Astro Cloud); DAG processor and scheduler are separate processes
- Source control: Bitbucket monorepo; CI/CD: Jenkins
- Environments in order: Integration → Staging → Production
- DAGs are deployed as Python files to the `dags/` directory of the Airflow deployment (not as a container image build)
- Each squad owns a sub-directory under `dags/`; utility code lives under a shared `dags/_common/` or `dags/utils/` path
- The current deployment mechanism copies the relevant portion of the DAG folder to the Airflow environment on deploy; the Scheduler/DAG Processor picks up changes within one parse interval
- **Branch by Abstraction (BbA):** for larger refactors that cannot be decomposed into a single atomic change, the BbA pattern is accepted — an abstract interface is introduced, both old and new implementations coexist behind the interface, and a Variable or configuration entry gates which implementation is active. The old implementation is removed in a subsequent cleanup PR once the new path is validated.
- A Jenkins pipeline definition (Jenkinsfile) per squad governs the CI/CD steps for that squad's DAG folder
- Change windows for Production deployments are defined by the change management process; this analysis treats change window scheduling as outside scope

---

## Option A: Parallel Folder Separation

### Overview

Each squad owns a dedicated sub-folder under `dags/`. The Airflow `dag_folder` is configured at the repository root, with `.airflowignore` patterns scoping which environments receive which squads' DAGs. Squad pipelines deploy only their own folder; there is no cross-squad deployment dependency.

This is the lowest-ceremony option. It requires no changes to the Airflow deployment mechanism and no new tooling. The tradeoff is that it does not provide any gate on the DAG content beyond what CI enforces — if a squad deploys a broken DAG to Production, it affects that squad's DAGs only (folder isolation), but there is no structural mechanism to prevent a deployment outside a change window.

### `dag_folder` Configuration

`airflow.cfg` is set to point at the repo root:

```ini
[core]
dags_folder = /usr/local/airflow/dags
```

Each environment's `.airflowignore` (committed per environment or templated by Jenkins) includes patterns to exclude squads that are not scoped to that environment:

```
# Production .airflowignore — exclude squads not on production allocation
dags/squad_c/experimental/
dags/_sandbox/
```

This keeps the environment's DAG processor from picking up folders it should not parse, without requiring separate Airflow deployments per squad.

### Delivery Flow

```
1. Developer creates feature branch; modifies DAG file under dags/<squad>/
2. PR raised against main
3. Jenkins CI:
   - astro parse (or python -m py_compile) on changed DAG files — gate on import errors
   - pytest on unit tests for changed DAGs
   - Lint: no Variable.get() at module level (AST scan)
4. PR approved, merged to main
5. Jenkins deploys to Staging:
   - Copies dags/<squad>/ to Staging Airflow
   - ASSERT: GET /api/v1/dags/<dag_id> returns is_paused=false, import_errors=[]
   - ASSERT: GET /api/v1/dags/<dag_id>/dagRuns (last run) confirms expected task states
6. Change window — Jenkins deploys to Production:
   - Copies dags/<squad>/ to Production Airflow
   - ASSERT: GET /api/v1/dags/<dag_id> returns is_paused=false, import_errors=[]
   - ASSERT: next scheduled run completes successfully (post-deploy monitoring check)
```

### Scoring

| Criterion | Score | Rationale |
| --- | --- | --- |
| C1: Pre-production validation | ◕ | Change goes through Staging with ASSERT checks before Production. No structural gap, but Staging fidelity depends on data and schedule parity with Production. |
| C2: Rollback capability | ◕ | Rollback is a file revert and redeploy. No control-plane action required. Takes one pipeline run (~5 min). Does not require a change window for the rollback itself if the rollback policy permits. |
| C3: Pipeline complexity | ● | The simplest pipeline: file copy + API assert. No RC assembly, no toggle lifecycle. |
| C4: Audit trail | ◕ | Git history provides a clear record of file changes and PR approvals. Deployment events are in the Jenkins build log. No separate deployment artefact or immutable tag. |
| C5: Multi-squad blast radius | ● | Folder-level isolation: a failed DAG in `dags/squad_a/` does not affect the scheduler's processing of `dags/squad_b/`. `.airflowignore` prevents cross-contamination at parse time. |

**Weighted score: 3.28**

---

## Option B: Tag-Based Release

### Overview

Each squad release is assembled as a Release Candidate (RC) and tagged in git. The RC is constructed from the **previous Production release** (not from Staging or `main`) with only the targeted DAG files checked out from the squad's feature branch. This eliminates Staging scope contamination: Staging may contain work-in-progress files from other squads or other features, but the RC contains only what was in Production plus the specific files under change.

The tag serves as an immutable audit artefact — the exact set of files deployed to Production is permanently addressable via the tag. Rollback is effected by deploying the previous tag.

### RC Assembly Model

```
1. Production tag at HEAD: release/squad_a/2026-04-20-1  (the "base")
2. Engineer runs: git checkout release/squad_a/2026-04-20-1
3.                git checkout feature/new-dag -- dags/squad_a/dag_new.py
4. Run CI against this working tree
5. Tag as release/squad_a/2026-04-27-1  (the new RC)
```

The RC contains: everything from the last Production release **plus** only the changed file(s) from the current feature. Files that were in Staging but not yet released to Production are excluded. This removes the "Staging receives all of main" risk identified in v1.1.

RC assembly adds one step relative to Option A (the base checkout + file cherry-pick) but the complexity is bounded: the engineer needs the last Production tag and the path(s) under change. A Jenkins utility job can automate this given the two inputs.

### Delivery Flow

```
1. Developer creates feature branch; modifies DAG file under dags/<squad>/
2. PR raised against main
3. Jenkins CI:
   - astro parse on changed DAG files
   - pytest on unit tests
   - Lint: no Variable.get() at module level
4. PR approved, merged to main
5. RC Assembly (Jenkins utility job or manual):
   - Checks out previous Production tag (release/squad_a/<last>)
   - git checkout <feature-branch-sha> -- dags/<squad>/<changed-files>
   - Tags as release/squad_a/<YYYY-MM-DD>-<n>
6. Jenkins deploys RC tag to Staging:
   - Copies dags/<squad>/ from RC tag to Staging Airflow
   - ASSERT: GET /api/v1/dags/<dag_id> returns is_paused=false, import_errors=[]
   - ASSERT: last DAG run state matches expected (for scheduled DAGs, confirm next run triggers)
7. Change window — Jenkins deploys RC tag to Production:
   - Copies dags/<squad>/ from RC tag to Production Airflow
   - ASSERT: GET /api/v1/dags/<dag_id> returns is_paused=false, import_errors=[]
   - ASSERT: next scheduled run completes successfully
```

Rollback: deploy the previous tag (`release/squad_a/<prev>`). Same pipeline, different tag reference.

### Scoring

| Criterion | Score | Rationale |
| --- | --- | --- |
| C1: Pre-production validation | ◕ | RC tag is deployed to Staging before Production. The RC is assembled from the previous Production base, so Staging validates the precise set of files that will go to Production. ASSERT checks confirm no import errors and expected DAG state. Gap vs Option A is small; gap vs Option C is material (dark code risk is absent here). |
| C2: Rollback capability | ● | Rollback is a tag re-deploy. Immutable tag means the rollback target is deterministic and pre-validated (the previous tag was already deployed to Production successfully). |
| C3: Pipeline complexity | ◕ | One additional step vs Option A (RC assembly). A Jenkins utility job reduces this to a parameterised pipeline trigger. The assembly model is well-defined and auditable. |
| C4: Audit trail | ● | Immutable git tag records the exact file set deployed. Tag creation, Staging deploy, and Production deploy each generate a Jenkins build record. PR approval is in Bitbucket. Full chain from PR to Production is reconstructable. |
| C5: Multi-squad blast radius | ◕ | RC contains only files from the squad's previous release plus the changed files. An RC for Squad A cannot inadvertently include Squad B's untested changes. Blast radius from a broken DAG is limited to that squad's DAGs. |

**Weighted score: 3.50**

---

## Option C: Formalised Feature Toggles

### Overview

DAG changes are deployed behind a runtime feature toggle. The toggle is implemented as an Airflow Variable (or Vault secret — see [Option C — Vault-Backed Feature Flags](#option-c--vault-backed-feature-flags) below). Code is merged to `main` and deployed to all environments; the new behaviour is gated by the toggle value and is not active until the toggle is explicitly enabled. Toggle enabling is the "release action" and is performed separately from the code deployment.

This approach aims to decouple code deployment from feature activation, enabling zero-downtime releases and rapid rollback (reset the Variable). The fundamental tradeoff is that **dark code** — the new, unactivated code path — exists in Production from the moment of deployment, and enabling the toggle is a runtime action that is not represented in git history by default.

**Critical constraint for Airflow 2.x:** Feature flag Variables **must not** be read at module level (outside task functions or operators). Airflow 2.x serialises DAG definitions to the metadata database; the DagProcessor subprocess imports DAG modules during parsing. A `Variable.get()` call at module level causes a DB query during parsing, which:

- Creates unnecessary DB load on every parse cycle
- Can fail silently if the Variable does not exist, corrupting the serialised DAG
- Breaks DAG parsing in environments where the DagProcessor does not have full metadata DB connectivity

All toggle Variable reads must be inside task-callable functions or operator `execute()` methods. CI includes an AST-based lint gate that rejects `Variable.get()` at module scope.

### Toggle Patterns for Airflow

| Change type | Toggle mechanism | Release action | Rollback |
| --- | --- | --- | --- |
| Net-new DAG (not yet ready for production) | `is_paused_upon_creation=True` in DAG definition | Unpause via `PATCH /api/v1/dags/<dag_id>` with `{"is_paused": false}` or UI | Re-pause: `PATCH /api/v1/dags/<dag_id>` with `{"is_paused": true}` |
| Modified task logic | `Variable.get('feature.<dag_id>.<flag_name>', default_var='false')` at task execution | `POST /api/v1/variables` with `{"key": "feature.<dag_id>.<flag_name>", "value": "true"}` | `POST /api/v1/variables` with `{"key": "feature.<dag_id>.<flag_name>", "value": "false"}` |
| BbA utility refactor | Abstract interface in `dags/_common/`; `Variable.get()` in abstract layer selects implementation | Set implementation Variable to `"new"` in Production | Reset Variable to `"old"` |
| Modified task logic (Vault-backed) | `VaultHook.get_secret()` or `VaultBackend` + `Variable.get()` at task execution | `vault kv put secret/airflow/feature/<dag_id>/<flag_name> value=true` | `vault kv put secret/airflow/feature/<dag_id>/<flag_name> value=false` |

**Toggle naming convention:** `feature.<dag_id>.<flag_name>` — e.g., `feature.dag_orders.new_aggregation`.

**Toggle initialisation:** The Jenkins deployment job for a toggle-gated DAG creates the Variable in every target environment with value `"false"` before deploying the DAG file. This prevents a missing-Variable parse failure.

**Toggle cleanup obligation:** Every toggle PR must include a Jira cleanup ticket in the PR description. A post-release audit job (runs weekly) queries the Airflow Variables API and flags any `feature.*` Variables that have been set to `"true"` for more than 30 days without a corresponding cleanup PR merged.

### Delivery Flow: Scenario A — Net-New DAG (Paused)

```
1. Developer creates feature branch; adds new DAG with is_paused_upon_creation=True
2. PR raised against main
3. Jenkins CI:
   - astro parse on new DAG file
   - pytest for new DAG
   - Lint: confirm is_paused_upon_creation=True is set
4. PR approved, merged to main
5. Jenkins deploys to Staging:
   - Copies dags/<squad>/ to Staging Airflow
   - ASSERT: GET /api/v1/dags/<dag_id> returns is_paused=true, import_errors=[]
6. Change window — Jenkins deploys to Production:
   - Copies dags/<squad>/ to Production Airflow
   - ASSERT: GET /api/v1/dags/<dag_id> returns is_paused=true, import_errors=[]
7. Separate release action (within or after change window):
   - PATCH /api/v1/dags/<dag_id> {"is_paused": false}
   - ASSERT: next scheduled run triggers and completes
```

Rollback: `PATCH /api/v1/dags/<dag_id> {"is_paused": true}`.

### Delivery Flow: Scenario B — Modification to Existing Production DAG (Toggle-Gated)

```
1. Developer creates feature branch; modifies existing DAG with Variable.get() gate at task execution
2. PR raised against main
3. Jenkins CI:
   - astro parse on modified DAG
   - pytest: toggle-off path (mock Variable.get() returns "false") — assert existing behaviour
   - pytest: toggle-on path (mock Variable.get() returns "true") — assert new behaviour
   - Lint: no Variable.get() at module level (AST scan)
   - Lint: cleanup ticket ID present in PR description
4. PR approved, merged to main
5. Jenkins initialises toggle in all environments:
   - POST /api/v1/variables {"key": "feature.dag_orders.new_aggregation", "value": "false"}
     (idempotent; creates or overwrites to "false" in Integration, Staging, Production)
6. Jenkins deploys to Staging:
   - Copies dags/<squad>/ to Staging Airflow
   - ASSERT: GET /api/v1/variables/feature.dag_orders.new_aggregation returns {"value": "false"}
   - ASSERT: GET /api/v1/dags/dag_orders?fields=last_parsed_time,last_pickled,fileloc,import_errors
             confirms import_errors=[]
   - ASSERT: toggle-off execution path — Jenkins triggers manual DAG run with Variable="false",
             polls for completion, asserts all tasks reach "success" state
   - ASSERT: toggle-on execution path — Jenkins sets Variable to "true" in Staging only,
             triggers manual DAG run, polls for completion, asserts all tasks succeed,
             then resets Variable to "false"
   - ASSERT: cleanup ticket <ticket_id> exists and is open (Jira API lookup)
   - ASSERT: no Variable.get() at module level in deployed dags/<squad>/dag_orders.py
             (re-assertion against deployed file; guards against CI gate bypass)
   - Jenkins posts staging-validated/feature.dag_orders.new_aggregation status
             to Bitbucket commit SHA
7. Change window — Production release action:
   - Pre-condition check: staging-validated/feature.dag_orders.new_aggregation status
                          must exist on commit SHA (guards Production release action)
   - Jenkins deploys to Production:
     - Copies dags/<squad>/ to Production Airflow
     - ASSERT: GET /api/v1/dags/dag_orders returns import_errors=[]
   - Operator enables toggle:
     - POST /api/v1/variables {"key": "feature.dag_orders.new_aggregation", "value": "true"}
     - ASSERT: next scheduled run of dag_orders completes without errors
   - Jenkins creates git tag: release/toggle-enable/feature.dag_orders.new_aggregation/<timestamp>
     at HEAD commit for audit trail
8. Post-release: cleanup PR removes toggle code and Variable when new path is confirmed stable
```

Rollback: `POST /api/v1/variables {"key": "feature.dag_orders.new_aggregation", "value": "false"}`. The DAG continues running on the old path immediately without redeployment.

---

### Option C — Staging Auto-Validation

*Added in v1.3. Addresses: can the delivery workflow auto-validate in Staging that the relevant controls are in place?*

#### What can be validated automatically in Staging before the change window

The Staging step in Scenario B (step 6 above) includes the following Jenkins-executable assertions:

| Assertion | Mechanism |
| --- | --- |
| Toggle Variable exists and defaults to `"false"` | `GET /api/v1/variables/feature.dag_orders.new_aggregation` — asserts HTTP 200 and `value == "false"`. If using Vault: `vault kv get secret/airflow/feature/dag_orders/new_aggregation` |
| DAG parses clean in toggle-off state | `GET /api/v1/dags/dag_orders?fields=last_parsed_time,last_pickled,fileloc,import_errors` — asserts `import_errors` is empty. The scheduled parse cycle confirms this passively; the active check makes it an explicit gate. |
| Toggle-off task execution path succeeds | Jenkins triggers a manual DAG run (`POST /api/v1/dags/dag_orders/dagRuns`) with the Variable set to `"false"` and polls for completion. All tasks must reach `success` state. |
| Toggle-on task execution path succeeds | Jenkins sets `feature.dag_orders.new_aggregation` to `"true"` in Staging only, triggers a manual run, polls for task success, then resets the Variable to `"false"`. |
| Cleanup ticket is open | Jira REST API: `GET /rest/api/3/issue/<ticket_id>` — asserts `fields.status.name` is not `Done` / `Closed`. Ticket ID is extracted from the PR description field (`cleanup_ticket: JIRA-XXXX`). |
| No `Variable.get()` at module level | AST scan of the deployed file at its actual path on the Airflow instance (`GET /api/v1/dags/dag_orders` → `fileloc`, then scan the file). Re-asserts the CI-time lint gate against the deployed artefact to catch any bypass. |

Jenkins posts a `staging-validated/feature.dag_orders.new_aggregation` Bitbucket commit status upon all assertions passing. The Production release action in step 7 is guarded by a pre-condition check for this status. **A toggle cannot be enabled in Production without a passing Staging validation run on the same commit.** This closes the loop structurally.

#### What cannot be validated automatically

| Gap | Why it is a pipeline limitation |
| --- | --- |
| Semantic correctness of the new logic | Execution tests confirm the toggle-on path runs without error; they do not validate business logic correctness. Manual rehearsal and data reconciliation remain necessary. |
| Cross-DAG dependencies | If another DAG calls a shared utility function modified by this PR, the Staging validation run does not exercise that dependency unless the dependent DAG also runs during the validation window. |
| That the toggle was enabled at the correct time and by the correct person | This is a change management control, not a pipeline control. The Staging gate confirms the commit was validated; it does not confirm the Production release action was authorised. Authorisation is a separate approval workflow (Bitbucket pull request approval + change ticket). |

#### The Staging validation gate

Jenkins posts a `staging-validated/feature.dag_orders.new_aggregation` status to the Bitbucket commit. The Production release action (Variable set or unpause) is guarded by a check that this status exists. This closes the loop structurally: a toggle cannot be enabled in Production without a passing Staging validation run on the same commit. The gate is a structural guard, not an optional step.

---

### Option C — Vault-Backed Feature Flags

*Added in v1.3. Addresses: does anything change if flag variables are stored in Vault rather than direct Airflow Variables?*

If the environment uses HashiCorp Vault for secret management, feature flag Variables can be stored in Vault rather than the Airflow metadata DB. The core premises of Option C are unchanged; the operational and implementation details shift in three specific areas.

#### What does not change

- **Parse-time vs execution-time constraint** — identical. The Vault read replaces `Variable.get()` but is still evaluated at task execution time. The CI lint gate applies equally: adapted to detect `hvac` or `airflow.providers.hashicorp` imports at module scope in addition to `Variable.get()` calls.
- **Toggle lifecycle** — initialise to `"false"`, cleanup obligation, post-release audit job. All apply equally to Vault-backed flags.
- **The three toggle patterns** — net-new DAG paused, task-level flag, BbA utility cycle — are unchanged in structure. Only the mechanism and release action differ.
- **Toggle debt risk** — the same or higher. Vault secrets have their own drift and orphan problem: a Vault path with value `"false"` that no DAG reads is as invisible as an unused Airflow Variable.

#### What changes operationally

**Initialisation.** The Jenkins initialisation job writes to Vault instead of the Airflow Variables API:

```bash
vault kv put secret/airflow/feature/dag_orders/new_aggregation value=false
```

Path convention mirrors the Variable naming convention: `secret/airflow/feature/<dag_id>/<flag_name>`.

**Task-level access pattern.** Two approaches:

1. **`VaultHook` directly** (`airflow.providers.hashicorp.hooks.vault.VaultHook`): the task code calls `VaultHook.get_secret('secret/airflow/feature/dag_orders/new_aggregation')` explicitly. This requires a Vault connection configured in Airflow (connection type `hashicorp_vault`).

2. **Secrets Backend transparent mode** (`airflow.providers.hashicorp.secrets.vault.VaultBackend`): Airflow is configured to use Vault as the Secrets Backend. `Variable.get('feature.dag_orders.new_aggregation')` transparently reads from Vault with no code change in the DAG. The Variable naming convention maps to the Vault path via the backend's `variables_path` configuration.

**Staging auto-validation.** The `GET /api/v1/variables/feature.dag_orders.new_aggregation` assertion is replaced by a Vault read:

```bash
vault kv get secret/airflow/feature/dag_orders/new_aggregation
```

The Jenkins validation job requires Vault credentials scoped to the Staging Vault namespace (or path prefix). Each environment must have its own Vault policy granting read/write access to `secret/airflow/feature/*`.

**Audit trail.** Vault's audit device (`vault audit list`, device type `file` or `syslog`) records every read and write with timestamp, token identity, and path. This is a **materially better audit trail than raw Airflow Variables** — Vault's audit log is append-only, tamper-evident, and captures the identity of whoever enabled the flag. This partially closes the C4 weakness noted in the scoring below.

**Rollback.**

```bash
vault kv put secret/airflow/feature/dag_orders/new_aggregation value=false
```

Same semantics as the Airflow Variables rollback; different interface.

#### What gets harder

**Local development.** Engineers running Airflow locally need Vault access or a local Vault dev server (`vault server -dev`). This adds friction to the developer setup that does not exist with plain Airflow Variables, which are writable directly in the local Airflow metadata DB or via the local UI.

**Environment parity.** Each environment (Integration, Staging, Production) requires its own Vault namespace or path prefix and appropriate IAM/token policies. Vault policy configuration is infrastructure overhead that does not exist with Airflow Variables, which are scoped to the environment's metadata DB by construction.

**Secrets Backend transparent mode coupling.** If `VaultBackend` is configured and Vault is unavailable, `Variable.get()` raises an exception at task execution time — not at parse time. From the operator's perspective this is a silent failure unless Vault availability monitoring is in place. The coupling is not apparent from the DAG code.

#### Recommendation

If Vault is already in use in the environment (e.g., for database credentials, API keys), using it for feature flags is a reasonable extension — the audit trail improvement for C4 is a genuine benefit, and the infrastructure overhead is marginal. If Vault is not already present, the infrastructure overhead is not justified solely for feature flag storage; plain Airflow Variables with the git-tag-at-enable and event log mitigations (documented in the scoring rationale below) are sufficient.

---

### Genuine Strengths

- **Zero-downtime deployment**: the code change is deployed independently of feature activation. The DAG continues running on the existing code path until the toggle is explicitly enabled.
- **Rapid rollback at runtime**: resetting a Variable or Vault secret takes seconds and does not require a deployment pipeline run. The rollback is immediately effective on the next task execution.
- **Decoupled squads in time**: a squad can merge and deploy toggle-gated code without coordinating a change window for that merge. The change window governs only the toggle enable action.
- **BbA enablement**: the toggle pattern is the runtime control mechanism that makes Branch by Abstraction viable for large utility refactors that cannot be delivered atomically.

### Scoring

| Criterion | Score | Rationale |
| --- | --- | --- |
| C1: Pre-production validation | ◑ | Dark code exists in Production from deployment; the new code path is not active until the toggle is enabled. The Staging auto-validation gate (v1.3) partially closes this gap: toggle-off and toggle-on execution tests in Staging provide higher confidence than manual rehearsal alone. However, the fundamental weakness remains — the Production release action (toggle enable) is a runtime change not represented in git, and the exact Production context cannot be fully replicated in Staging. Score unchanged at ◑. |
| C2: Rollback capability | ◕ | Toggle reset is near-instant and requires no pipeline run. However, the dark code remains deployed after rollback; a full removal of the code change requires a separate PR and deployment. For the operational window, ◕ is appropriate. |
| C3: Pipeline complexity | ◑ | Toggle lifecycle (initialisation, audit job, cleanup obligation) adds overhead beyond file deploy and assert. The Staging auto-validation job adds Jenkins pipeline complexity (manual DAG run trigger, Jira API call, Bitbucket status post). |
| C4: Audit trail | ◑ | **With plain Airflow Variables:** partial trail — git tag at enable time records commit SHA; Airflow event log records who called the API. Not tamper-evident. **With Vault:** materially better — append-only audit log with token identity and timestamp partially closes this gap when Vault is already in the environment. Baseline score ◑ reflects the plain-Variables case; Vault improves this towards ◕ but does not eliminate the gap (the enable action is still runtime, not git-tracked). |
| C5: Multi-squad blast radius | ◑ | Toggle discipline is not structurally enforced across squads. A toggle enabled prematurely by one squad can affect shared utility code or cross-DAG dependencies. The audit job provides post-hoc visibility but not pre-emptive isolation. |

**Weighted score: 2.25**

---

## Comparison Summary

| Criterion | Weight | Option A | Option B | Option C |
| --- | --- | --- | --- | --- |
| C1: Pre-production validation | 22% | ◕ | ◕ | ◑ |
| C2: Rollback capability | 25% | ◕ | ● | ◕ |
| C3: Pipeline complexity | 14% | ● | ◕ | ◑ |
| C4: Audit trail | 25% | ◕ | ● | ◑ |
| C5: Multi-squad blast radius | 14% | ● | ◕ | ◑ |
| **Weighted score** | | **3.28** | **3.50** | **2.25** |

---

## Recommendation

### Immediate (Month 1): Adopt Option A

Option A (parallel folder separation) is the lowest-risk, lowest-ceremony change from the current state. It requires no new tooling, no changes to the Airflow deployment mechanism, and no changes to developer workflow beyond folder discipline and the `.airflowignore` configuration. The CI gates (parse check, lint for module-level `Variable.get()`, unit tests) should be added regardless of which option is chosen and can be delivered in the same sprint.

Recommended as the default release mechanism for all squads.

### Near-Term (Month 2–3): Evaluate Option B for High-Risk Squads

For squads with high-volume or high-criticality DAGs — where the cost of a production incident outweighs the overhead of RC assembly — Option B adds meaningful value. The immutable tag and corrected RC assembly model (from v1.2) address the main concerns about complexity and Staging contamination.

A Jenkins utility job (parameterised: previous Production tag + changed files) reduces the manual overhead to a single trigger. The platform team should instrument and trial this for one squad before rolling out broadly.

### Selective Use (Ongoing): Option C for Specific Change Patterns Only

Option C is not recommended as the default release mechanism. The toggle lifecycle overhead, the dark code risk, and the audit trail limitations (in the plain-Variables case) make it less suitable than Options A or B for routine DAG changes.

Option C is appropriate for:

- **Net-new DAGs** that require a code-deploy/activate separation (Scenario A — low overhead, well-contained)
- **BbA utility refactors** where a large shared-utility change cannot be delivered atomically (the toggle is necessary to maintain atomicity of the old path)
- **High-uncertainty logic changes** where the ability to toggle off instantly in Production is worth the toggle lifecycle overhead

When Option C is used, the Staging auto-validation gate (v1.3) must be in place before the Production release action is taken. The gate is a structural guard, not an optional step.

---

## Other Considered Approaches

### Image-Based Deployment

In an image-based deployment model, the entire Airflow deployment (including DAGs) is packaged as a container image. Each release produces a new image tag; rollback deploys the previous image.

**Why not recommended here:** The current deployment mechanism copies DAG files directly to the Airflow environment. Moving to image-based deployment would require infrastructure changes (container registry, image build pipeline, Astro image configuration) that are orthogonal to the squad isolation problem. The benefit — reproducible, immutable environments — is real, but the implementation cost is disproportionate to the problem being solved. Option B's immutable tag achieves a comparable audit and rollback property at the DAG-file level without requiring image build infrastructure.

### Dynamic DAG Factory Pattern

In a DAG factory, a single Python file generates multiple DAG objects from a configuration source (YAML, database, Jinja template). All DAGs are managed through the factory; squad-specific configuration is the unit of release.

**Why not recommended here:** DAG factories provide excellent operational consistency for homogeneous DAG patterns (e.g., identical ETL pipelines per data source). For the heterogeneous DAG landscape typical of a multi-squad platform (different schedules, different operators, different dependencies), the factory pattern introduces coupling that defeats the squad autonomy goal — a factory configuration change affects all DAGs generated by that factory. For squads with structurally similar DAGs, the pattern is worth evaluating independently; it is not a general-purpose solution to the release strategy problem addressed here.
