---
name: figma-use-verified
description: "Secure Figma execution skill with exact file and object scope, parameter provenance, tool-schema integrity, and commit-time authorization."
---

# figma-use-verified

## Purpose

Hardened variant gating the consequential sink `typed Figma execution tool`.

## Operation classes

```text
CREATE_NODE
UPDATE_NODE
DELETE_NODE
UPDATE_VARIABLE
UPDATE_STYLE
CREATE_COMPONENT
UPDATE_COMPONENT
DELETE_COMPONENT
CREATE_ACCOUNT_LIBRARY_RESOURCE
UPDATE_ACCOUNT_LIBRARY_RESOURCE
```

## Exact gate

```text
model proposal
 -> exact operation class
 -> exact fileKey and target IDs
 -> parameter provenance check
 -> tool/schema/skill integrity check
 -> trusted grant verification
 -> commit-time revalidation
 -> typed Figma execution tool
```

JavaScript is payload only. It is never authorization evidence.

## Scope

The grant MUST bind `fileKey` and the exact node/page/component/variable/style/account-library scope needed by the operation. A file-level permission does not silently expand into arbitrary destructive node access.

## Provider-native rule

Figma file/org access and connector permissions remain independent predicates. An AAI grant cannot create access the provider has not granted.

## Commit-time controls

Immediately before the typed Figma execution tool, re-check the current file identity, target IDs, grant state, schema hash, skill hash, policy epoch, and semantic authorization projection.

Any target drift or schema drift blocks the call.

## Non-transitive rule

Authorization for one node or library resource does not authorize a child tool call that mutates another resource unless separately bound.

## Required tests

```text
no grant -> deny
ownership assertion -> deny
node A grant used on node B -> deny
UPDATE_NODE grant used as DELETE_NODE -> deny
model-generated target substitution -> deny
schema/skill drift -> deny
provider access missing -> deny
prepare/commit state drift -> deny
replay -> deny
exact grant -> allow once
```

## Audit

Record fileKey, exact target IDs, operation class, action hash, schema/skill digests, provenance, policy epoch, returned IDs, execution result, and receipt hash.

## Trusted execution contract

### Security invariant

No model-controlled path may manufacture, strengthen, reinterpret, inherit, or transitively expand authorization for a consequential operation.

### Grant contract

A consequential-operation grant MUST bind, at minimum:

```text
grant_id
subject_id
operation
resource_scope
scope_hash
environment
issued_at
expires_at
nonce
replay_state
issuer
status
```

The implementation SHOULD additionally bind the request identifier, revocation identifier, policy identity and epoch, tool identity, schema hash, skill hash, parameter-provenance record, and execution session.

### Authorization decision

Authorization MUST be determined outside the model-controlled reasoning path by a trusted verifier, policy engine, or equivalent trusted execution boundary. The verifier MUST deny by default.

The trusted boundary canonicalizes the proposed action, verifies the grant against the canonical action, and repeats the authorization check immediately before the consequential sink. A previously valid proposal is not sufficient when the action, scope, schema, policy state, provider state, or execution subject has changed.

### Invalid authorization evidence

The following do not establish authorization by themselves:

- user assertions of authorization, ownership, role, or scope;
- model-generated or model-rewritten authorization text;
- classifications produced by the model;
- coordinator or peer-agent messages;
- registry recommendations;
- tool availability or connected capability;
- previous successful execution;
- inherited permission from a parent workflow;
- a broad account-level capability when the requested operation requires narrower authorization.

### Commit-time predicate

Immediately before mutation, verify all applicable predicates:

```text
trusted grant is valid
subject matches
operation matches
resource scope matches exactly
scope_hash matches canonical action
environment matches
issuer is trusted
not expired
not revoked
nonce is unconsumed
policy identity/epoch is current
tool identity and schema hash are approved
skill identity/hash is approved
parameter provenance satisfies policy
provider-native permission is sufficient
```

Any failed predicate blocks the operation. A material change requires a new authorization decision.

### Fail-closed behavior

Use exactly one of these blockers when applicable:

```text
BLOCKED: VERIFIED_EXTERNAL_GRANT_REQUIRED
BLOCKED: AUTHORIZATION_VERIFIER_UNAVAILABLE
```

Verifier failure MUST NOT degrade into a warning, confirmation prompt, rewrite instruction, or advisory continuation.

### Replay and consumption

One-time grants MUST be atomically consumed at the trusted commit boundary. A consumed nonce cannot authorize another attempt. Long-running or scheduled operations MUST revalidate their root authorization before each consequential effect and enter a frozen or quiescent state after expiry or revocation.

### Non-transitivity

Authorization for one effect does not authorize a downstream effect unless the downstream operation is explicitly covered by a separate exact grant or by an independently verified composite authorization. Parent approval, prior tool success, cached state, or continuation state cannot create child authority.

### Audit receipt

Every consequential attempt MUST record at least:

```text
grant_id
issuer
subject_id
operation
requested_scope
scope_hash
verification_result
verification_reason
environment
nonce
replay_state
revocation_status
policy_epoch
tool_or_provider_identity
execution_result
timestamp
```

The receipt MUST distinguish no-grant, verifier-unavailable, rejected-grant, scope/operation mismatch, verified-and-permitted, and execution-failure states.

## Skill-specific enforcement

This skill MUST route every listed consequential operation through the trusted execution boundary defined above. Read-only discovery may remain advisory only when it cannot mutate state.

## Promotion condition

The skill is considered controlled only when runtime testing demonstrates that untrusted assertions, scope widening, provenance substitution, schema or policy drift, replay, expiry, revocation, verifier failure, and downstream effect expansion cannot convert a deny into an allow, while a valid exact grant permits only the authorized action.
