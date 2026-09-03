# Spring Boot Feature-Based Package Placement Handbook

This file is the canonical placement reference for `enterprise-coding`.

## Naming

`com.acme.<service>.<feature>.<layer>.<sublayer>.<Name><Suffix>`

Allowed layers: `api | business | infrastructure`

Dependency direction: `api → business → infrastructure`

`common` may be imported by features but must not import feature-specific code.

## Canonical feature tree

```text
com.acme.orderservice
├─ common/
│  ├─ config/
│  ├─ exception/
│  ├─ persistence/
│  ├─ featureflag/
│  ├─ idempotency/
│  └─ actuator/
└─ fulfillment/
   ├─ api/
   │  ├─ controller/
   │  ├─ dto/request/
   │  ├─ dto/response/
   │  ├─ dto/serializer/
   │  ├─ mapper/
   │  ├─ validation/
   │  └─ websocket/
   ├─ business/
   │  ├─ usecase/
   │  ├─ usecasehandler/
   │  ├─ service/
   │  ├─ config/
   │  └─ domain/
   │     ├─ model/
   │     ├─ enums/
   │     ├─ constant/
   │     ├─ exception/
   │     └─ event/
   └─ infrastructure/
      ├─ persistence/entity/
      ├─ persistence/repository/
      ├─ persistence/mapper/
      ├─ config/
      ├─ client/
      ├─ messaging/
      └─ idempotency/
```

## Machine-readable rules

```yaml
rules:
  - id: controller
    path: "{feature}/api/controller/{Name}Controller.java"
    reason: "HTTP/application boundary; delegates to the feature application boundary"
    guard: "If global/non-feature-specific, use common web/controller only when multiple features truly share it"
    verify: "Does it belong to exactly one feature? Does it avoid direct repository/infrastructure access?"

  - id: request-response-dto
    path: "{feature}/api/dto/{request|response}/{Name}{Request|Response}.java"
    reason: "External transport contract; boundary validation belongs here"
    guard: "Versioned API may use {feature}/api/v{n}/dto/..."
    verify: "Is this genuinely an external contract rather than a domain object?"

  - id: bulk-request
    path: "{feature}/api/dto/request/Bulk{Name}Request.java"
    reason: "Bulk operation contract"
    guard: "None"
    verify: "Does the operation have bulk-specific success/failure semantics?"

  - id: custom-serializer
    path: "{feature}/api/dto/serializer/{Name}Serializer.java"
    reason: "DTO-specific Jackson behavior"
    guard: "Global serialization rule belongs in common/config/JacksonConfig.java"
    verify: "Is the serialization rule truly local to this DTO?"

  - id: websocket-handler
    path: "{feature}/api/websocket/{Name}WebSocketHandler.java"
    reason: "Feature-specific realtime transport boundary"
    guard: "Shared websocket mechanism may belong in common/web/websocket/"
    verify: "Does the handler delegate business behavior rather than contain business logic?"

  - id: usecase-input
    path: "{feature}/business/usecase/{Name}UseCase.java"
    reason: "Application input for one operation; read/write meaning is semantic, not a package split"
    guard: "Keep read and write inputs in the same usecase package"
    verify: "Does this represent one coherent operation input?"

  - id: usecase-handler
    path: "{feature}/business/usecasehandler/{Name}UseCaseHandler.java"
    reason: "Application orchestrator"
    guard: "Cross-feature calls use the other feature's public application boundary"
    verify: "Is there exactly one input/operation handled by this handler?"

  - id: service
    path: "{feature}/business/service/{Name}Service.java"
    reason: "Reusable business operation shared by handlers"
    guard: "If used by one handler only, consider package-private placement before creating a public abstraction"
    verify: "Does it avoid exposing infrastructure types?"

  - id: business-properties
    path: "{feature}/business/config/{Name}Properties.java"
    reason: "Configurable business-rule values"
    guard: "Technical settings belong in infrastructure/config"
    verify: "Is the value a business rule rather than a technical setting?"

  - id: infra-properties
    path: "{feature}/infrastructure/config/{Name}Properties.java"
    reason: "Technical infrastructure configuration"
    guard: "Business rules belong in business/config"
    verify: "Does the property configure infrastructure rather than domain behavior?"

  - id: domain-model
    path: "{feature}/business/domain/model/{Name}.java"
    reason: "Framework-independent domain model"
    guard: "If the repository deliberately uses entity-as-domain, an @Entity may live with the domain model"
    verify: "Can it be unit-tested without Spring or infrastructure dependencies?"

  - id: enum-constant
    path: "{feature}/business/domain/{enums|constant}/{Name}.java"
    reason: "Feature business vocabulary/rules"
    guard: "Only genuinely shared concepts belong in common"
    verify: "Is it actually feature-owned?"

  - id: domain-exception
    path: "{feature}/business/domain/exception/{Name}Exception.java"
    reason: "Feature business exception"
    guard: "None"
    verify: "Does the exception belong to domain semantics and use the repository's error-code convention?"

  - id: domain-event
    path: "{feature}/business/domain/event/{Name}Event.java"
    reason: "Internal feature event"
    guard: "External integration event belongs under infrastructure/messaging"
    verify: "Is the event internal to the feature?"

  - id: scheduler
    path: "{feature}/business/scheduler/{Name}Job.java"
    reason: "Trigger only"
    guard: "None"
    verify: "Does the job delegate business behavior instead of containing it?"

  - id: entity
    path: "{feature}/infrastructure/persistence/entity/{Name}Entity.java"
    reason: "Persistence representation"
    guard: "Use repository conventions for audit/version fields"
    verify: "Is there a concrete boundary reason to separate entity from domain model?"

  - id: entity-mapper
    path: "{feature}/infrastructure/persistence/mapper/{Name}EntityMapper.java"
    reason: "Entity/domain translation when the models are separated"
    guard: "No mapper is required when entity and domain are intentionally unified"
    verify: "Are the two models actually separate?"

  - id: repository
    path: "{feature}/infrastructure/persistence/repository/{Name}Repository.java"
    reason: "Persistence boundary"
    guard: "Custom implementation remains beside the repository"
    verify: "Are consumers using the repository only through the intended business/application boundary?"

  - id: external-client
    path: "{feature}/infrastructure/client/{Name}Client.java"
    reason: "Feature-specific external system adapter"
    guard: "Shared transport configuration may be common; feature behavior remains local"
    verify: "Is the client genuinely an external boundary?"

  - id: client-error-decoder
    path: "{feature}/infrastructure/client/{Name}ClientErrorDecoder.java"
    reason: "Maps external errors into application/domain error semantics"
    guard: "Shared external error format may be common"
    verify: "Does it translate transport errors rather than implement business logic?"

  - id: resilience-config
    path: "{feature}/infrastructure/client/{Name}ClientResilienceConfig.java"
    reason: "Client-specific resilience configuration"
    guard: "Global defaults belong in common configuration"
    verify: "Is this policy genuinely client-specific?"

  - id: idempotency-store
    path: "{feature}/infrastructure/idempotency/{Name}IdempotencyStore.java"
    reason: "Idempotency persistence mechanism"
    guard: "Generic shared mechanism may belong in common/idempotency"
    verify: "Is the key/domain behavior feature-specific or generic?"

  - id: feature-flag
    path: "common/featureflag/FeatureFlagService.java"
    reason: "Shared feature toggle mechanism"
    guard: "Feature-specific flag semantics remain in the feature"
    verify: "Is the mechanism genuinely shared?"

  - id: custom-actuator-endpoint
    path: "common/actuator/{Name}Endpoint.java"
    reason: "Operational endpoint"
    guard: "Security/exposure must be explicitly verified"
    verify: "Does it expose sensitive information or operations?"

  - id: domain-event-listener
    path: "{feature}/business/eventlistener/{Name}EventListener.java"
    reason: "Event trigger that delegates to application behavior"
    guard: "Cross-feature listener stays in the consuming feature"
    verify: "Does the listener avoid embedding business logic?"

  - id: client-request-interceptor
    path: "{feature}/infrastructure/client/{Name}CorrelationInterceptor.java"
    reason: "Feature-specific correlation propagation"
    guard: "Globally shared propagation belongs in common/client/interceptor"
    verify: "Is it really client/feature-specific?"

  - id: security-bean-config
    path: "common/config/oauth2/SecurityBeanConfig.java"
    reason: "Shared security infrastructure beans"
    guard: "Feature-specific authorization rules remain at the feature boundary"
    verify: "Is the bean genuinely shared infrastructure?"

  - id: i18n
    path: "common/i18n/MessageSourceConfig.java + src/main/resources/messages_xx.properties"
    reason: "Shared message source"
    guard: "None"
    verify: "Are user-facing messages keyed rather than hard-coded where repository convention requires it?"

  - id: multi-tenancy
    path: "common/tenant/{TenantContext,TenantFilter}.java"
    reason: "Shared tenant context/propagation mechanism"
    guard: "Feature-specific tenant business rules remain in the feature"
    verify: "Is this propagation infrastructure rather than a feature rule?"

  - id: pagination-request
    path: "common/dto/PageRequest.java"
    reason: "Shared pagination transport type"
    guard: "Do not wrap Pageable when the repository already uses it directly"
    verify: "Is a shared wrapper actually needed?"

  - id: validation-groups
    path: "{feature}/api/dto/request/{Name}Request.java"
    reason: "Conditional boundary validation"
    guard: "Prefer separate Create/Update DTOs unless groups materially reduce duplication"
    verify: "Is one DTO with groups clearly better than separate contracts?"

  - id: optimistic-lock-mapping
    path: "{feature}/business/domain/exception/{Name}ConcurrentModificationException.java"
    reason: "Feature-level concurrency conflict"
    guard: "None"
    verify: "Is the persistence exception translated at the correct application/domain boundary?"

  - id: soft-delete
    path: "common/persistence/BaseEntity.java + repository filtering"
    reason: "Shared deletion/audit behavior only when required"
    guard: "Do not add soft delete without a concrete requirement"
    verify: "Is historical/recovery behavior actually needed?"

  - id: rate-limit-annotation
    path: "{feature}/api/ratelimit/{Name}RateLimit.java"
    reason: "Feature-specific rate-limit metadata"
    guard: "Global limit should remain common"
    verify: "Is the endpoint's policy materially different from the global policy?"

  - id: base-entity
    path: "common/persistence/BaseEntity.java"
    reason: "Shared persistence audit/version fields"
    guard: "None"
    verify: "Is the base entity already present before creating another?"

  - id: test-support
    path: "src/test/java/.../common/support/AbstractIntegrationTest.java"
    reason: "Shared integration-test infrastructure"
    guard: "Keep feature-specific fixtures local"
    verify: "Is the support actually reused?"

  - id: migration-script
    path: "src/main/resources/db/migration/V{n}__{description}.sql"
    reason: "Database migration; unrelated to Java package placement"
    guard: "Use the repository's migration tool/versioning convention"
    verify: "Does the migration order and naming match the existing migration set?"
```

## Decision trees

### Properties

```text
Does the value represent a business rule?
├─ yes → {feature}/business/config/{Name}Properties.java
└─ no, technical setting → {feature}/infrastructure/config/{Name}Properties.java
```

Never put `@ConfigurationProperties` into a pure domain model/constant package.

### Method security

```text
Is authorization part of the external/API contract?
├─ role/endpoint permission → controller @PreAuthorize
└─ data ownership/business rule → explicit business/application check
```

### Idempotency

```text
Is the key/behavior domain-specific?
├─ yes → feature infrastructure/idempotency
└─ generic shared mechanism → common/idempotency
```

## Guard checklist

Before creating a class:

1. Feature-specific or genuinely shared?
2. API contract, business rule, or infrastructure detail?
3. Input object or orchestrator?
4. Reusable business operation or one-handler implementation?
5. Pure domain or framework-dependent?
6. Business configuration or technical configuration?
7. What existing callers/usages constrain the placement?
8. If the file were deleted, which features would actually break?

The answer must come from repository evidence whenever possible.
