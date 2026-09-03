# Anti-Pattern and Scope Firewall

This is a guard reference, not a license to impose generic architecture over repository conventions.

## Reject unless justified

- speculative abstraction
- unrelated refactoring
- global utility dumping
- cross-feature persistence access
- domain/framework coupling
- infrastructure leakage across architectural boundaries
- duplicate models/enums without a boundary reason
- business logic in triggers/listeners/jobs
- silent exception swallowing
- tests that prove only existence rather than behavior
- generated/temp/debug artifacts
- unrelated formatting churn

## Exception rule

An unconventional implementation may proceed when:
1. repository-local architecture explicitly requires it,
2. the existing codebase consistently establishes the convention,
3. the user explicitly requests it,
4. or it is required for correctness.

When an exception is used, verify it against repository evidence.
