<!-- Authors private promises, explores faults, and preserves immutable regression evidence through MCP. -->
# Architecture promises and regression cases

Read `get_authoring_guide` with `topic: "promises"` and the live tool schemas before using this workflow. Tools missing from discovery may mean an older deployment or insufficient grants; report that limitation. Never broaden a key's permissions automatically.

## Author and evaluate

1. Select the diagram through `list_diagrams`; read focused graph context and the saved journey through `get_scenario_draft`.
2. Read `get_promise_suite`. Choose stable supported targets from the graph: `handler_response` with node/action identity, supported database effects, or the interaction targets advertised by the schema. Do not count attempted calls as successful effects. Rolled-back writes are not effects.
3. Write `update_promise_suite` with `document` and the suite's own `version_token`. Use `occurrence_limit`, `required_ordering`, or `completion_deadline` and a scalar business-key JSON pointer. One suite holds at most 32 promises. Read the exact template fields from the tool schema; do not use labels as IDs.
4. Verify the returned saved document and suite token, then reread the suite and run `run_scenario_draft`, pinning `graph_version_token` and `promise_version_token` to the selected sources. Evaluation uses complete evidence before trace filtering.

Distinguish `passed`, `violated`, `not_exercised`, and `inconclusive`. A completed simulation is not automatically a passing promise. Missing/null/non-scalar keys, unsupported targets, incomplete evidence, and execution guards cannot pass. Preserve a proven violation even if later execution is interrupted. Ordering uses event sequence, not timestamp ties. A deadline includes its boundary; completion matching must respect the authored business-key/request-lineage mode.

## Break this design

Call `run_duplicate_challenge` with the saved journey's `version_token`, `graph_version_token`, `promise_version_token`, and inline `challenge`. The compatibility tool name covers all four fault kinds:

- `challenge.v1`, `kind: "duplicate_delivery"`: `businessKey`, `stopOnFirstViolation`. Requires exactly one original ingress and a scalar key. Candidates repeat its payload with an independent request root, at 0 ms, 1 ms, and 1 ms after baseline completion. Offsets are relative to original ingress arrival; preserve the returned schedule rather than recomputing it.
- `challenge.v2`, `kind: "failure_before_execution"`: `interactionId`, one-based `invocationOccurrence`, `stopOnFirstViolation`. The selected callee performs no work.
- `challenge.v2`, `kind: "delayed_response"`: the same target fields plus positive `delayMs`. Callee effects remain; the caller's modeled timeout/retries still apply.
- `challenge.v2`, `kind: "lost_response"`: the same target fields, with a timeout already modeled on the interaction. Callee effects remain, but delivery is suppressed.

Use current opaque interaction references in v2. Invocation occurrences include retries in scheduler order. Never invent a retry policy or mutate the saved graph to inject the fault. A baseline violation stops exploration. Every case starts from fresh fixtures; deliveries share state only within their case. Report tested, untested, and unexercised coverage and the first violation. “No violation found in these cases” is bounded evidence, never a correctness guarantee.

Execution requires `challenges:run`, journey read, confirmed full-value access, and a workspace that has not been explicitly disabled. Workspaces are enabled by default. Budgets are eight executions including baseline, 40,000 configured events, and a 15-second watchdog. Results remain transient and cannot be approved or published through these tools.

## Preserve and compare

Save a selected tested case using `save_regression_case`. Supply all three source tokens, the exact challenge and execution `versions`, and a stable `case_key`. For duplicate cases retain the returned `offsetMs`; for dependency cases use `offsetMs: null` and the returned `faultPlan`. Baseline has `offsetMs: null` and no fault plan. Reuse `case_key` only for an identical retry. A version conflict requires rereading and rerunning; never relabel old evidence as current.

A successful save returns its case reference; verify it with `get_regression_case`. These suite/case tools do not advertise a `committed` flag. `challenges:write` authorizes saved definitions, not publication. The server captures authoritative snapshots; do not send repository bodies or event artifacts. Limits are 100 cases per diagram and 16 MiB per snapshot.

Use `list_regression_cases` and `get_regression_case` to reopen a saved definition. `run_regression_case` with `mode: "original"` uses its preserved inputs. `mode: "current"` also requires the current `graph_version_token` and substitutes only the graph, preserving the original journey, promises, and fault schedule. Both return a separate result with `persisted: false`. Saved cases survive deletion of their original journey. Unknown historical execution versions remain readable but unsupported for replay; do not change their version strings or silently reinterpret them. Repair missing targets in the current model and preserve the original case.

Promise editing requires `promises:write`; reads require journey/full-value access. Cloned journeys receive independent expectations. Checkpoints pin immutable suite references; a manifest pin is authored provenance, not observed success. Private archives and workspace moves preserve definitions and exact case snapshots; shared/public views do not grant private-case access.

## Completion

Prove the requested journey, evaluate all requested promises, account for untested cases, and compare the saved original with the changed graph when a fix is requested. Report blockers, unsupported history, stale sources, and unexercised promises explicitly. Never claim completion from a green ordinary run alone.
