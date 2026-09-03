# Enterprise Coding Verification Contract

This reference defines the evidence model used by `enterprise-coding`.

## States

- `OBSERVED`: directly established from repository/tool output.
- `REQUIRED`: explicitly requested or required by repository rules.
- `INFERRED`: derived from observed evidence.
- `UNKNOWN`: not established.
- `PASS`: a verification predicate was executed and passed.
- `FAIL`: a verification predicate was executed and failed.
- `NOT RUN`: a relevant verification was not executed.
- `NOT APPLICABLE`: genuinely irrelevant to the change.

## Artifact lifecycle

```text
REQUESTED
→ PLANNED
→ CREATED/MODIFIED
→ RE-READ
→ STRUCTURALLY VALID
→ ARCHITECTURALLY VALID
→ BEHAVIORALLY VERIFIED
→ RECONCILED
→ COMPLETE
```

No state may be skipped merely by assertion.

## Bidirectional verification

### Forward

```text
request
→ expected artifact
→ filesystem artifact
→ actual behavior
```

### Reverse

```text
filesystem changes
→ architectural justification
→ request justification
```

The task is complete only when both directions reconcile.

## Evidence rules

Strong evidence:
- actual filesystem read,
- actual source inspection,
- actual test/build output,
- actual git diff/status.

Weak evidence:
- prior assistant statement,
- plan,
- generated patch acknowledgement,
- model memory.

Weak evidence must never be used as proof of completion.
