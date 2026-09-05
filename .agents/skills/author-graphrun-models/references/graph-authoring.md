<!-- Graph authoring patterns that keep MCP diagrams executable. -->
# Graph authoring

Use `get_graph_operation_contract` for the exact branch before calling `apply_graph_operations`. Use `dry_run` only if the live tool input schema advertises it; otherwise omit it.

Prefer one atomic operation batch with `local:<name>` aliases for newly-created components, interactions, edges, handlers, and actions. Echo the complete top-level `reference_bindings` map whenever the batch uses a returned `ref:<name>`.

## Components and ports

- Discover types with `search_component_types`; inspect a selected type with `get_component_type`.
- Treat instance ports and the response-scoped `reference_bindings` in filtered `get_graph_context` as authoritative for focused operations. Request only the necessary sections and use `relationship_mode: "references"` when relationship identities are needed. Echo the entire map unchanged into each graph operation batch that uses `ref:<name>`; when truncation is reported, unbound identifiers remain usable in expanded `mfref2.*` form or through a narrower context read.
- A semantic alias is `<component-type>.<catalog-port-id>`, for example `service.publishOut` or `database.queryIn`.
- If an alias is rejected, reread the component type/version and use the returned current opaque port reference.

## Executable behavior

- Port roles determine edge kinds. A `terminal` source always produces an `error` edge, including `queue.dlqOut` to an event input. Queue enqueue and dispatch routes use `asynchronous_message`; Topic delivery to Queue enqueue uses `event_publish`.
- Synchronous request: arrival handler owns each `respond` or `fail` outcome edge.
- Message producer: an executable handler owns `emit` for the selected asynchronous or event-publish edge.
- Message consumer: the handler arrives on the consume port and owns acknowledgement/failure outcomes.
- Data access: the handler-owned request and all selected result outcomes must be explicit.
- Object Store `getIn`: use one `add_interaction` with the hit as its primary response and `additional_responses` for `objectstore.missingOut`; all return edges share the same interaction atomically.
- Topic broker relay: canonical `topic.deliverOut` to `queue.enqueueIn` uses `event_publish`, not `asynchronous_message`. Reserve `asynchronous_message` for delivery such as Queue-to-worker dispatch.
- Handler action and catch `when` conditions use `expression.v1`. Graph edge `when` filters use `expression.v2` and are supported only on Topic `deliverOut` subscriptions.
- A degraded catch completes through a response-compatible response, ACK, or NACK edge. A rejected catch requires an outgoing `error` edge. Service has no terminal output port. Remove the rejected catch to propagate the failure through the waiting call frame, or use a degraded response. Let a Gateway with `errorOut` shape a terminal API rejection.

If the requested behavior does not distinguish empty success, degraded response, propagated failure, or terminal rejection and more than one fits the current graph, inspect existing contracts and journeys first. Ask which externally visible outcome is intended before writing when that evidence remains ambiguous; do not select an outcome merely to reuse an available edge.

### Literal payload assignment

Graph.v7 defines an ordered `assign` action for provider-neutral payload shaping:

```json
{
  "id": "mark-ready",
  "kind": "assign",
  "assignments": [
    { "pointer": "/status", "value": "ready" },
    { "pointer": "/attempt", "value": 1 }
  ]
}
```

Use 1–16 RFC 6901 pointers with literal JSON values. Assignments run in list order, and later actions, `when` clauses, and `for_each` selectors see the updated payload. Intermediate path segments must already exist; a final object property may be created. Array pointers must select an existing index. Never use `__proto__`, `constructor`, or `prototype` as a path segment. Treat any graph schema other than graph.v7 as invalid. Pre-v7 graphs have no compatibility or migration path: stop on `graph.version.unsupported` and do not attempt conversion through graph operations or complete-document replacement.

### Database operations

Current `database@1.3` has independent `readOperation` (`get | query`) and `writeOperation` (`insert | update | upsert | delete | atomic-batch`) settings. Configure both when one Database instance receives both `database.queryIn` and `database.writeIn`; this avoids the legacy single-operation port mismatch. Existing database@1.2 nodes keep their `operation` field until explicitly upgraded. On upgrade, preserve the legacy operation in the matching read or write field and use the catalog default for the other side. Atomic batch is already part of the current database@1.3 catalog.

Database reads merge into the current payload: `get` preserves request fields and writes its result at `/record`; `query` preserves them and writes the array at `/records`. Branch on those result pointers, not on a top-level field from the returned record.

Do not author handlers on Database `queryIn`, Cache `lookupIn`, or Object Store `getIn`. These intrinsic read handlers own their result envelopes and return routes. Put result branching, payload shaping, and HTTP status responses on the calling Service or Gateway. If an intrinsic handler was authored accidentally, use `remove_port_handler`; the MCP restores the canonical empty `catalog_default` handler even when the input port is optional. Database single-write, Cache write, and Object Store write handlers remain customizable. Database batch `writeIn` rejects custom actions and catches.

### Atomic database batches

Read `get_component_type` for `database` and `get_graph_operation_contract` for `update_component`. Set its `configuration` to include the following fields (preserve unrelated settings):

```json
{
  "collection": "orders",
  "keyPath": "/id",
  "writeOperation": "atomic-batch",
  "batchOperations": [
    { "id": "insert-order", "operation": "insert", "recordPath": "/newOrder" },
    { "id": "confirm-order", "operation": "update", "recordPath": "/confirmedOrder" }
  ]
}
```

Supply 1–32 rows with unique nonempty IDs. Each row has only `id`, `operation` (`insert | update | upsert | delete`), and `recordPath`, an RFC 6901 JSON Pointer into the incoming payload. An empty pointer selects the whole payload. Every row reuses the configured collection and `keyPath`, evaluated inside the selected record. Delete rows must select an object containing that key. An update replaces the stored record; send the complete intended record, not a partial patch.

Use `add_interaction` with explicit `service.commandOut` → `database.writeIn` and `database.writtenOut` → `service.responseIn` endpoints. The calling service owns the `call` and final `respond`. Batch `writeIn` uses the intrinsic handler with empty actions and catches; use `remove_port_handler` to clear an existing override in the same graph-operation batch when switching modes. Validation, later calls, and recovery belong in the caller.

Rows execute in order against staged records, so an insert followed by an update to the same key succeeds. All rows commit in one scheduler operation or all are discarded at the first failure. The incoming payload is preserved. The caller's catch receives the existing database code, such as `database.record_exists` or `database.record_missing`; do not invent a generic transaction failure code. There are no intermediate calls, waits, or visible partial state. No Begin/Commit/Rollback actions, multi-call transactions, multiple collections, or isolation levels are supported.

An MCP operation batch atomically saves authoring changes. A Database atomic batch executes writes during a journey. These are separate mechanisms. Declare the runtime batch's contract as described in [contracts.md](contracts.md), then prove commit and rollback with [journeys.md](journeys.md).

### Gateway round-robin calls

For a Gateway handler, use `{ "id": "balance", "kind": "call_next_backend" }`. It has no edge or await-port setting; read `get_graph_operation_contract` for `upsert_handler_action` or `set_port_handler`. Connect at least one synchronous request from `gateway.requestOut`, each with a valid paired response into `gateway.responseIn`. Conditional backend edges are rejected.

The action selects exactly one backend, waits for its response, and continues the handler. Backends are ordered by ascending edge priority, then stable edge ID. A per-gateway cursor cycles A/B/C/A and resets each run. Retries stay on the selected backend; the next action execution advances selection. Keep a final `respond` to the original caller. Explicit `call` remains available, and no second routing-mode switch is needed. Potential routes include every eligible backend; actual route evidence identifies only the selected connection.

Handler conditions and value references use `payload` for the current mutable value. Use `input` for the immutable payload captured when that handler first arrived; it remains stable across call/await, retry, catch, and resume.

A connected line without the corresponding handler action is visual documentation only and must not be used as a contract target.

## Safe mutation loop

1. Read filtered graph context plus its bound-reference map and version token. Request only the object kinds, component types or references, and sections needed for this operation batch.
2. Apply the smallest coherent operation batch, echoing that map when using `ref:<name>`.
3. For a dependent graph batch, optionally chain from this response only when `committed: true`, `validation.valid: true`, and its returned binding map is untruncated; otherwise reread.
4. At a surface boundary or before final verification, reread focused context, discard the old map, and use the replacement map and references.
5. Run `validate_graph`.
6. Resolve errors before contract authoring; record non-blocking warnings for the final explanation.

Use `apply_graph_layout` only for layout. Do not encode semantic behavior through positioning or labels. Reread and validate again after applying layout because the layout call commits a new graph version.

The server never persists `reference_bindings`. `local:<name>` still means a same-batch identity and resolves after bound references. Use `get_graph_draft` only for complete-document replacement, graph-wide or cross-page rebasing, or explicit lossless debugging; it retains geometry that focused context omits. Use `reference_format: "expanded"` only to debug an opaque reference or prepare a whole-document operation that does not accept bindings.
