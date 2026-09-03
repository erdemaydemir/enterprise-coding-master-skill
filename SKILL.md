---
name: enterprise-coding
description: >
  Master coding workflow for enterprise Java/Spring Boot repositories. Use whenever
  implementing, modifying, refactoring, moving, deleting, or verifying code/files in
  a Java/Spring project. This skill is the execution and verification authority:
  filesystem-first inspection, minimal-change discipline, architecture guards,
  deterministic placement through the java-spring-solid skill, implementation,
  tests/build verification, bidirectional reconciliation, and completion gating.
  Always use with the existing java-spring-solid skill for Java/Spring/SOLID
  architecture and placement decisions.
---

# Enterprise Coding — Master Execution Protocol

## 0. Role

This is the master execution skill for repository changes.

It answers:

> **How must a coding task be executed so that the result is real, correct, minimal, consistent, and verified?**

The existing `java-spring-solid` skill answers:

> **What Java/Spring/SOLID architecture and placement rules apply?**

Do not duplicate or replace `java-spring-solid` rules here. Delegate Java/Spring architectural decisions to it.

### Authority model

```text
enterprise-coding
    ├── execution protocol
    ├── filesystem truth
    ├── change scope
    ├── anti-pattern firewall
    ├── verification
    ├── test/build evidence
    ├── diff/reconciliation
    └── completion gate
             │
             ▼
      java-spring-solid
    ├── Java/Spring/SOLID
    ├── feature boundaries
    ├── package placement
    ├── placement-rules.yaml
    └── architecture conventions
```

If the two skills appear to conflict:
- `enterprise-coding` controls **workflow and verification**.
- `java-spring-solid` controls **Java/Spring/SOLID architecture and placement**.
- Repository-local rules (`CLAUDE.md`, contribution rules, build conventions, etc.) must also be inspected and respected.
- Never silently invent a resolution. State the conflict and choose the narrowest evidence-backed interpretation.

---

# 1. Non-negotiable principles

### 1.1 Filesystem is the source of truth

Never claim that a file, class, package, import, test, build result, or configuration exists because:
- it was planned,
- it was mentioned earlier,
- it was generated in a response,
- it is visible in model context,
- it "should" exist.

Only filesystem/repository evidence establishes existence.

### 1.2 Created != correct != verified != complete

These are separate states:

```text
CREATED
  ≠
STRUCTURALLY CORRECT
  ≠
ARCHITECTURALLY CORRECT
  ≠
BEHAVIORALLY VERIFIED
  ≠
COMPLETE
```

Never collapse them into one statement.

### 1.3 No hallucinated repository state

Do not invent:
- paths,
- packages,
- classes,
- methods,
- tests,
- dependencies,
- configuration,
- command results,
- compiler/build results,
- git changes.

If evidence is unavailable, mark the state `UNKNOWN` and inspect.

### 1.4 Minimal change

Implement the smallest change that satisfies the request and repository conventions.

Do not:
- refactor unrelated code,
- rename unrelated classes,
- create abstractions "for future use",
- add utilities merely because they look reusable,
- move files without architectural need,
- introduce patterns without a concrete requirement,
- create tests or files that do not correspond to the requested behavior.

### 1.5 No silent scope expansion

If implementation reveals a necessary adjacent change, classify it:

```text
REQUIRED     -> must change
SUPPORTING   -> directly required for requested behavior
OPTIONAL     -> useful but not required
UNRELATED    -> do not change
```

Only `REQUIRED` and `SUPPORTING` changes belong in the implementation unless the user explicitly asks otherwise.

### 1.6 Do not expose hidden reasoning

Verification must be evidence-based, but internal chain-of-thought must not be exposed.

Report concise evidence/status such as:

```text
Placement: PASS
Filesystem existence: PASS
Architecture: PASS
Tests: PASS
Diff scope: PASS
```

Do not dump private reasoning.

---

# 2. Mandatory workflow

Every repository coding task follows this state machine:

```text
ORIENT
  ↓
CONTRACT
  ↓
DISCOVER
  ↓
ARCHITECTURE GUARD
  ↓
PLAN
  ↓
IMPLEMENT
  ↓
RE-READ
  ↓
STRUCTURAL VERIFY
  ↓
ARCHITECTURAL VERIFY
  ↓
BEHAVIOR VERIFY
  ↓
TEST / BUILD
  ↓
BIDIRECTIONAL RECONCILIATION
  ↓
COMPLETION GATE
  ↓
REPORT
```

Do not skip a stage merely because the change looks small.

For trivial edits, stages may be executed compactly, but they still exist.

---

# 3. ORIENT — establish repository reality

Before modifying files:

1. Inspect repository root.
2. Identify the build system:
   - Maven
   - Gradle
   - multi-module Maven/Gradle
   - other repository-specific build mechanism.
3. Locate relevant source/test roots.
4. Inspect repository-local instructions:
   - `CLAUDE.md`
   - `AGENTS.md`
   - contribution/development docs
   - module-specific instructions.
5. Identify the actual Java base package from the repository.
6. Inspect the existing package tree around the requested feature.
7. Inspect existing analogous classes before creating a new pattern.

Never infer repository conventions solely from the user's example if the repository itself provides evidence.

### Filesystem rule

If a path is named by the user, verify it exists before relying on it.

If a file is said to have been created previously, verify it from disk before modifying or reporting it.

---

# 4. CONTRACT — define the requested change

Extract the task into a compact contract:

```text
Goal:
Required behavior:
Artifacts allowed to change:
Artifacts expected to exist after completion:
Constraints:
Verification requirements:
```

Separate explicit requirements from assumptions.

Use:

```text
OBSERVED  = directly established from repository/user input
REQUIRED  = explicitly required
INFERRED  = derived from evidence
UNKNOWN   = not established
```

Never turn `INFERRED` or `UNKNOWN` into `OBSERVED`.

If a missing fact materially changes the implementation, stop and ask.

If it does not materially change the implementation, use the narrowest evidence-backed assumption and state it briefly.

---

# 5. DISCOVER — inspect before designing

Before implementation, inspect:

- target file if it exists,
- neighboring package/classes,
- analogous feature implementation,
- relevant interfaces,
- relevant tests,
- configuration,
- build/module boundaries,
- callers/importers when modifying existing code.

For a modification, read the actual current file.

For a new class, inspect at least one existing analogous class when available.

For a move/rename, inspect both:
- the source artifact,
- all relevant references/imports.

For deletion, inspect references before deleting.

Do not perform broad repository rewrites when local inspection is sufficient.

---

# 6. ARCHITECTURE GUARD — invoke java-spring-solid

Whenever Java/Spring code is created, moved, renamed, or its architecture is changed:

**Use the existing `java-spring-solid` skill.**

It is the authoritative source for:
- feature-based package placement,
- artifact-to-rule mapping,
- `placement-rules.yaml`,
- Java/Spring/SOLID architecture,
- feature boundaries,
- package conventions.

### Required delegation sequence

```text
requested artifact
      ↓
java-spring-solid
      ↓
artifact rule id
      ↓
guard
      ↓
path template
      ↓
verify
      ↓
enterprise-coding implementation
```

Do not reproduce placement paths from memory.

Do not invent a rule id.

Do not override a guard because a path "looks cleaner".

If `java-spring-solid` has no rule for the requested artifact:
- do not fabricate a canonical rule;
- inspect repository precedent;
- report that the canonical rule is missing;
- use the narrowest repository-supported placement only if implementation can proceed safely.

---

# 7. ARCHITECTURE FIREWALL

Before implementation, reject or pause on these patterns unless repository evidence explicitly requires them:

### Forbidden without evidence

- Controller directly accessing persistence infrastructure when the architecture requires an application/use-case boundary.
- Domain model depending on Spring/framework infrastructure.
- Business/application code leaking persistence entities into public contracts.
- Cross-feature direct access to another feature's persistence internals.
- Infrastructure details leaking into domain APIs.
- Feature-specific utility placed in global `common`.
- New abstraction with no current consumer or responsibility.
- Duplicate enum/model/mapper without a concrete boundary reason.
- Job/listener/consumer containing orchestration/business logic when the architecture delegates to handlers/use cases.
- Technical configuration embedded in domain objects.
- Business rules hidden inside infrastructure adapters.
- Catch-and-ignore exception handling.
- Tests that only assert object existence without behavior.
- "Fixing" unrelated pre-existing problems as part of the requested change.

This is a guard, not an automatic prohibition against every unconventional design. Repository evidence can justify an exception.

---

# 8. PLAN — make the smallest valid change

Before editing, identify:

```text
Files to create:
Files to modify:
Files to move/delete:
Tests to add/modify:
Why each file is required:
```

Every planned artifact must have a reason.

If a planned file cannot be tied to the requested behavior or architecture, remove it from the plan.

### No speculative artifacts

Do not create:
- empty interfaces,
- generic base classes,
- helper utilities,
- factories,
- strategy abstractions,
- wrappers,
- configuration classes,
- DTOs,
- mappers,
- tests,

unless the requirement or existing architecture makes them necessary.

---

# 9. IMPLEMENT — change only the contract

Implement only what the contract requires.

Preserve:
- existing conventions,
- public API compatibility unless change is requested,
- existing user changes,
- formatting style,
- package naming,
- test conventions.

Do not overwrite unrelated user modifications.

If the working tree already contains changes, treat them as protected unless the task explicitly targets them.

---

# 10. RE-READ — never verify from memory

After every meaningful file creation or modification:

1. Read the actual file from disk.
2. Confirm the expected package declaration.
3. Confirm class/interface/record name.
4. Confirm imports.
5. Confirm the implementation is non-empty and coherent.
6. Confirm no accidental content was inserted.
7. Confirm referenced files/classes actually exist.

A successful edit operation is not evidence that the resulting file is correct.

---

# 11. STRUCTURAL VERIFY

Verify filesystem facts independently.

For every expected artifact:

```text
EXPECTED PATH
    ↓
EXISTS?
    ↓
READABLE?
    ↓
EXPECTED TYPE/NAME?
    ↓
EXPECTED PACKAGE?
    ↓
NON-BROKEN CONTENT?
```

A file passing `exists` does not pass structural verification.

For moves/renames:

```text
old path absent or intentionally retained
AND
new path exists
AND
references point to the new location
```

For deletions:

```text
target absent
AND
no required reference still points to it
```

---

# 12. ARCHITECTURAL VERIFY

Verify both directions.

## Direction A — artifact → rule

For each created/moved Java artifact:

```text
actual file
  ↓
artifact type
  ↓
java-spring-solid rule id
  ↓
rule path template
  ↓
guard
  ↓
actual path
```

Must match.

## Direction B — rule/contract → artifact

For each requested artifact:

```text
requested artifact
  ↓
expected rule/path
  ↓
filesystem
  ↓
actual artifact
```

Must exist and match.

### Completion requires both directions

```text
artifact → rule = PASS
rule → artifact = PASS
```

If either fails:

```text
COMPLETION = FAIL
```

Do not report "created correctly".

---

# 13. DEPENDENCY VERIFY

For changed Java code, inspect imports and relevant call relationships.

Verify:

- dependency direction follows the repository architecture,
- no forbidden layer dependency was introduced,
- no feature boundary was crossed through an internal implementation,
- no infrastructure type leaks into an architectural boundary where it should not,
- no new cyclic dependency was introduced.

If the repository's `java-spring-solid` rules define a more specific architecture, follow those rules.

Do not blindly impose generic Clean Architecture terminology over repository-specific rules.

---

# 14. BEHAVIOR VERIFY

A structurally correct file can still implement the wrong behavior.

Verify the actual requested behavior:

```text
Requirement
    ↓
implementation
    ↓
observable behavior
```

Use the strongest practical evidence available:

1. existing tests,
2. new focused tests,
3. compile/type checking,
4. targeted execution,
5. integration tests,
6. broader test suite.

Do not claim behavior is verified merely because the file compiles.

If behavior cannot be executed or tested, report:

```text
Behavior verification: NOT EXECUTED
Reason: <concrete reason>
```

Do not convert this to PASS.

---

# 15. TEST / BUILD

Choose the narrowest command that provides meaningful evidence first.

Typical progression:

```text
focused test
  ↓
module test
  ↓
compile/package
  ↓
broader suite
```

Use the repository's actual build system and documented commands.

Never claim:
- tests passed,
- build passed,
- compilation succeeded,

unless the command actually ran and the result supports the claim.

If tests fail:
- distinguish implementation failure from pre-existing/environment failure when evidence allows,
- do not hide failures,
- do not modify unrelated code simply to make the command green.

---

# 16. BIDIRECTIONAL RECONCILIATION

Before completion, compare three realities:

```text
A. USER REQUEST
B. PLAN
C. ACTUAL FILESYSTEM / GIT DIFF
```

Perform both reconciliations.

### Forward reconciliation

```text
request → expected artifacts/behavior → actual result
```

Question:

> Did we actually deliver everything requested?

### Reverse reconciliation

```text
actual changed artifacts → request
```

Question:

> Did we change/create anything that was not required?

Anything not justified is a candidate for removal.

### Git reconciliation

If Git is available:

```text
git status
git diff
git diff --stat
```

Inspect the actual diff.

Check:
- only intended files changed,
- no accidental generated files,
- no unrelated formatting churn,
- no debug code,
- no temporary files,
- no credentials/secrets,
- no accidental deletion,
- no user changes overwritten.

---

# 17. COMPLETION GATE

The task may be reported as complete only if all applicable gates pass.

```text
[ ] Request understood
[ ] Repository inspected
[ ] Relevant local instructions inspected
[ ] java-spring-solid consulted for Java/Spring architecture
[ ] Guard passed
[ ] Planned scope established
[ ] Implementation completed
[ ] Actual files re-read
[ ] Structural verification passed
[ ] Artifact → rule verification passed
[ ] Rule → artifact verification passed
[ ] Dependency/architecture verification passed
[ ] Behavior verification passed OR explicitly marked not executable
[ ] Relevant tests/build executed
[ ] Results accurately reported
[ ] Git diff reconciled
[ ] No unexplained scope expansion
[ ] No accidental files/changes
```

### Hard rule

If an applicable mandatory gate is `FAIL` or `UNKNOWN`:

```text
COMPLETION = NOT VERIFIED
```

Do not say "done", "created successfully", or equivalent.

---

# 18. Verification status vocabulary

Use only evidence-backed statuses:

```text
PASS
FAIL
NOT RUN
UNKNOWN
NOT APPLICABLE
```

Do not use vague statuses such as:
- probably,
- should be fine,
- looks correct,
- seems created,
- likely passes.

### Evidence hierarchy

Prefer:

```text
filesystem read
    >
tool/edit acknowledgement
    >
model memory
```

and:

```text
actual test/build output
    >
static inspection
    >
assumption
```

---

# 19. Existing user changes are protected

If the repository contains pre-existing modifications:

1. Detect them before editing where practical.
2. Do not overwrite them.
3. Do not "clean them up".
4. Do not include them in the completion claim.
5. Reconcile final diff against the pre-task state when possible.

If the target file already contains user changes, preserve them unless the requested task explicitly modifies that content.

---

# 20. Refactoring firewall

A requested feature is not permission for a general refactor.

Refactor only when one of these is true:

```text
A. required for correctness
B. required by the architecture guard
C. required to remove a direct violation introduced by the requested change
D. explicitly requested by the user
```

Otherwise leave the code alone.

If an existing anti-pattern is discovered but is unrelated:

```text
Observed, not changed.
```

Mention it only if it materially affects the requested implementation.

---

# 21. Failure handling

When blocked:

```text
BLOCKED
Reason:
Evidence:
What is missing:
Smallest next action:
```

Do not invent a workaround that changes architectural intent.

When a guard fails:

```text
GUARD FAILED
Artifact:
Rule:
Condition:
Required alternative:
```

When verification fails:

```text
VERIFICATION FAILED
Check:
Expected:
Actual:
Next action:
```

---

# 22. Final report

Keep the final response concise and evidence-based.

Preferred format:

```text
## Result
PASS / NOT VERIFIED / BLOCKED

## Changes
- <actual changed artifact>
- <actual changed artifact>

## Verification
- Placement: PASS
- Filesystem: PASS
- Architecture: PASS
- Behavior: PASS / NOT RUN
- Tests/Build: PASS / FAIL / NOT RUN
- Diff scope: PASS

## Notes
- <only material assumptions, limitations, or failures>
```

Never report an artifact as created unless filesystem verification passed.

Never report tests/build as passing unless executed.

Never hide a failed or unknown mandatory verification.

---

# 23. Compact operating rule

When under time/token pressure, preserve this order:

```text
INSPECT
→ DELEGATE ARCHITECTURE
→ GUARD
→ MINIMAL CHANGE
→ RE-READ
→ VERIFY
→ TEST
→ DIFF
→ RECONCILE
→ REPORT
```

Do not optimize by removing verification. Optimize by reducing unnecessary implementation and narration.
