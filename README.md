# enterprise-coding

Master Claude Code skill for enterprise repository changes.

It orchestrates execution and verification while delegating Java/Spring/SOLID architecture and placement to the existing `java-spring-solid` skill.

## Expected companion skill

```text
java-spring-solid
```

That skill remains the authority for `placement-rules.yaml`, package placement, feature boundaries, and Java/Spring/SOLID architecture.

## Design goal

```text
Filesystem truth
+ deterministic architecture delegation
+ minimal change
+ bidirectional verification
+ real test/build evidence
+ final diff reconciliation
```

The skill deliberately does not copy the Java/Spring placement table. This prevents two independent architecture authorities from drifting apart.