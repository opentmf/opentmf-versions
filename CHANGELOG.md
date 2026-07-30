# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.15] - 2026-07-30

### Fixed
- Updated `opentmf-http-clients` to 2.1.5 — fixes an `HttpClientRegistry` regression
  shipped in 2.1.4 that broke ApplicationContext startup on any classpath without
  reactor-netty (a synthetic registry method exposed a reactor-netty type, so Spring's
  bean introspection failed with `NoClassDefFoundError`). Sync-only apps that adopted
  2.1.14 should upgrade.

## [2.1.14] - 2026-07-30

### Added
- Added `opentmf-errors` 1.0.0 — shared error-code catalog with `opentmf-errors-core`
  (registry contract + seed catalog + generated OAS 3.1 components) and
  `opentmf-errors-spring` (ProblemDetail and TMF-style renderers).

### Updated
- Updated `tmf630-toolkit` to **3.0.0** (major): JPA positional `[N]` filter with
  `@OrderColumn`, multi-hop correlated sort with `min()`/`max()` aggregators, and
  entity versioning per TMF-630 Part 4 §2. Repo split adds new sub-modules
  `tmf630-toolkit-jpa-correlated-sort`, `tmf630-toolkit-jsonb`, and
  `tmf630-toolkit-mongo-split-collection`.
- Updated `openid-rbac-security` to 2.2.0 (pluggable 401/403 response handlers via
  bean-presence; fixes ineffective servlet management-port security shipped in 2.1.0).
- Updated `opentmf-http-clients` to 2.1.4 (new `HttpClientRegistry` for programmatic
  runtime lifecycle; optional resilience4j circuit-breaker / bulkhead / time-limiter
  integration; Apache pool micrometer gauges).
- Updated `opentmf-mockserver` to 2.1.10 (repo is now multi-module under
  `opentmf-mockserver-parent`; adds the new `opentmf-mockserver-test-support` module
  — fluent JUnit 5 harness with TMF/OIDC/stub builders and JVM-shared MockServer mode).
- Updated `camunda7-bpmn-sync-service` to 2.1.1 (single configurable
  `resource-location` for BPMN + DMN files; fixes 2.1.0 startup failure when
  `classpath:dmn/` is absent).

## [2.1.13] - 2026-07-24

### Updated
- Updated `tmf630-toolkit` to 2.1.5 (several further TMF630 compliancy improvements).
- Updated `opentmf-mockserver` to 2.1.8 (fixes: Rare concurrent cache update issues under heavy load).
- Updated `opentmf-camunda7` to 24.0.6 (fixes a GraalJS polyglot-context memory leak in
  JavaScript script tasks — contexts are now closed per evaluation; Spring Boot 3.5.16,
  GraalJS 25.1.3, Docker images on Eclipse Temurin JRE 25).
- Updated `camunda7-bpmn-sync-service` to 2.1.0 (DMN deployment support, and Multi tenancy deployment support)

## [2.1.12] - 2026-06-26

### Updated
- Updated `opentmf-db-lock-library` to 2.2.1 (dynamically removes stale locks).
- Updated `opentmf-mockserver` to 2.1.6 (external token validation enhancements, + idempotency-key handling)

## [2.1.11] - 2026-06-10

### Updated
- Updated `tmf630-toolkit` to 2.1.3 (JSONPath sort terms now accept coercion (num() / str() / date()) and aggregator (min() / max()) wrappers).

## [2.1.10] - 2026-06-03

### Updated
- Updated `tmf630-toolkit` to 2.1.2 (Fixes multi-term Mongo correlated sort failing with "parallel arrays").

## [2.1.9] - 2026-06-01

### Updated
- Updated `camunda7-incident-logger` to 2.0.1 (Migrates to cibseven 2.2.0).
- Updated `opentmf-mockserver` to 2.1.4 (Adds dynamic PUT callback).
- Updated `camunda7-test-framework` to 2.0.2 (Migrates to cibseven 2.2.0 and boot-4 starters, removing the previous bridge classes).
- Updated `opentmf-api-clients` to 2.0.9 (Adds PUT methods to TmfClient implementations).

## [2.1.8] - 2026-05-18

### Fixed
- Updated `opentmf-api-clients` to 2.0.8.
  - Fix double percent-encoding of resource path segments when UriBuilderUtil.withContext(...) or UriBuilderUtil.withPagination(...) is applied to a URI that already contains an encoded {id} (typically TMF composite keys like Spec:(version=1))

## [2.1.7] - 2026-05-16

### Fixed
- Updated `tmf630-toolkit` to 2.1.1.
  - Mongo correlated-sort executor now places rows with null / missing sort keys last regardless of direction.

## [2.1.6] - 2026-05-11

### Updated
- Updated `tmf630-toolkit` to 2.1.0.
  - Comma-separated value lists are now accepted for the multi-value attribute-filter operators .in, .nin, and .between.
  - JSON Path filter grammar now supports the unary negation form `[?(!@.field)]`
  - Simple-rich correlated sort now accepts the outer coercion wrapper form
  - New setting `opentmf.tmf630.attribute-filtering.on-unknown-json-path-field` (default IGNORE).

## [2.1.5] - 2026-05-11

### Updated
- Updated `opentmf-v4-api` to 4.1.1 (regeneration with the new version of openapi-multi-generator, and per-TMF api submodules from `*.10` to `*.11`).
- Updated `opentmf-v4-models` to 4.1.1 (regeneration with the new version of openapi-multi-generator, and per-TMF model submodules from `*.10` to `*.11`).
- Updated `dnext-v4-models` to 2.12.1 (Adds Snapshot API on product/service/resource inventories, `pointOfNoReturnIFOC`/`pointOfNoChange` on orders, `completionCallback` on cancel-orders, `extensions` on resource shapes, and with the release of the new openapi-multi-generator, fixes the datatypes that were set as String because the generator didn't follow `oneOf` previously).
- Updated `opentmf-api-clients` to 2.0.7 (supports multiple-add-objects through `patchCollections` methods).
- Updated `opentmf-commons` to 2.2.0 (starts accepting unicode alphabetical characters in `@SafeText`, makes `@SafeQuery` deprecated, introduces `@SafeUrl` and optimizations).
- Updated `opentmf-mockserver` to 2.1.3 (adds multi-jsonPatch add statements).

## [2.1.4] - 2026-05-06

### Fixed
- Updated tmf630-toolkit to 2.0.2 (Fixes filter coexistence with rich sort variations)

## [2.1.3] - 2026-05-05

### Added
- Added the new library tmf630-toolkit-mongo-aggregation version 2.0.1.

### Changed
- Updated tmf630-toolkit to 2.0.1 (Correlated sort for MongoDB + TMF630 syntax shorthand)

## [2.1.2] - 2026-04-27

### Added
- Added the new library auditor-aware version 3.0.0. It replaces now obsoleted auditor-aware-jpa 2.0.0 library.

## [2.1.1] - 2026-04-25

### Updated
- Updated openid-rbac-security version to 2.1.0. (Adds management server security configuration as well).

## [2.1.0] - 2026-04-23

### Updated
- Updated opentmf-db-lock-service version to 2.2.0. (Adds multi-db support).

## [2.0.9] - 2026-04-20

### Fixed
- Updated dnext-catalog-sync-service to 2.0.4. (Adds `Accept` headers).

### Updated
- Updated opentmf-db-lock-service to 2.1.0 (Enhances `@UsingClusterLock` so that the success condition can be specified)

## [2.0.8] - 2026-04-16

### Fixed
- Updated opentmf-http-clients to 2.1.3 (Fixes Content-Type header during token retrieval when Content-Type already exists in the fixed-headers).
- Updated opentmf-api-clients to 2.0.6 (Started setting `Accept: application/json` if the `Accept` header is not set via fixed-headers or in TmfRequestContext)

## [2.0.7] - 2026-04-14

### Fixed
- Updated opentmf-api-clients to 2.0.5 (re-adds the missed fixed-headers from the old library)

## [2.0.6] - 2026-04-14

### Fixed
- Updated camunda7-test-framework to 2.0.1 (fixes JerseyApplicationPath registration)

## [2.0.5] - 2026-04-06

### Fixed
- opentmf-http-clients updated to 2.1.2 (Fixes logbook configuration timing issue)

### Changed
- opentmf-api-clients updated to 2.0.3 (starts using opentmf-http-clients 2.1.2)

## [2.0.4] - 2026-04-06

### Fixed

- opentmf-api-clients updated to 2.0.3 (All four auto-configuration classes refactored from constructor-based bean registration to static BeanDefinitionRegistryPostProcessor pattern. This fixes the "Ghost Configuration" problem in Spring Boot 4 where auto-configuration classes that only register beans in their constructor are never instantiated if no @Bean method output is requested by the dependency graph.)

## [2.0.3] - 2026-04-06

### Changed

- opentmf-api-clients updated to 2.0.2 (Ensures autoconfiguration to run after opentmf-http-clients)

## [2.0.2] - 2026-03-30

### Changed

- opentmf-http-clients updated to 2.0.1 (Fixes scope:pom in the started artifacts).
- opentmf-api-clients updated to 2.0.1 (Starts using the newly released opentmf-http-clients)

## [2.0.1] - 2026-03-30

### Changed

- dnext-catalog-sync-service updated to 2.0.3 (Starts synchronizing product catalog endpoints too).

## [2.0.0] - 2026-03-28

**Backward Incompatible** — targets Spring Boot 4.x and Jackson 3.x.

### Added

- Added opentmf-json-patch for JSON Patch (RFC 6902) support
- Added opentmf-v4-api with generated model interfaces for 80+ TMF v4 specifications
- Added opentmf-http-clients with REST and reactive HTTP client support and cached bearer-token handling
- Added opentmf-api-clients with REST and reactive TMF-630 compliant API clients
- Expanded opentmf-v4-models from 25 to 85 per-TMF module versions
- Added dnext-v4-models, a complete set of models generated from DNext swaggers covering every available DNext microservice

### Changed

- All libraries upgraded to their 2.x versions for Spring Boot 4.x and Jackson 3.x compatibility

### Removed

- Removed opentmf-web-clients (replaced by opentmf-http-clients)
- Removed opentmf-clients-base (replaced by opentmf-api-clients)
- Removed opentmf-v4-clients (no direct replacement, but easy client bean generation via opentmf-api-clients)
- Removed dnext-opentmf-v4-models (replaced by dnext-v4-models)

## [1.4.9] - 2026-03-20

### Changed

- Updated `opentmf-clients-base` to 1.1.8 (Does not try to set Content-Type on post operations, if the header already exists).
- Updated `opentmf-v4-clients` and its submodules to 1.1.8 (Starts using the new opentmf-clients-base).

## [1.4.8] - 2026-03-20

### Changed

- Updated `dnext-catalog-sync-service` to 1.1.1 (Context-aware resourceSpecification endpoints)
- Updated `tmf630-toolkit` and its submodules to 1.0.7 (Better error messages for invalid dates and reduces noise in the logs).
- Updated `opentmf-clients-base` to 1.1.7 (Does not try to set Content-Type on patch operations, if the header already exists).
- Updated `opentmf-v4-clients` and its submodules to 1.1.7 (Starts using the new opentmf-clients-base).

## [1.4.7] - 2026-03-11

### Changed

- Updated tmf630-toolkit to 1.0.6 (new `@Tmf630Result` annotation)
- Updated opentmf-camunda7 to 24.0.3 (Separate docker image that supports AWS IAM)
- Updated opentmf-clients-base to 1.1.6 (Fix: Enforces jakarta validations for all client configurations)
- Updated opentmf-v4-clients to 1.1.6 (starts using 1.1.6 of opentmf-clients-base)

## [1.4.6] - 2026-03-03

### Fixed

- Updated tmf630-toolkit to 1.0.5. It adds offset, limit, and fields to the reserved parameter set to prevent 400 Bad Request when `onUnknownField=REJECT` and request contains offset, limit, and/or fields. 

## [1.4.5] - 2026-03-03

### Changed

- Updated tmf630-toolkit to 1.0.4 (enhances enum handling)
- Updated camunda7-test-framework to 1.0.8 (adds withCount/withExecutionConsumer builder methods)

## [1.4.4]

### Fixed

- Updated tmf630-toolkit to 1.0.3 (fixes a bug in the `sort=` parameter handling)

## [1.4.3]

### Fixed

- Updated tmf630-toolkit to 1.0.2 (fixes deep nested key parsing and enables default .eq match)

## [1.4.2]

### Added

- Updated tmf630-toolkit to 1.0.1 (adds `filter=` JsonPath support both for JPA and MongoDB)

## [1.4.1]

### Added

- Added tmf630-toolkit artifacts

## [1.4.0]

### Changed

- Updated camunda7-test-framework to 1.0.7 (enhances support for more task types that can receive a correlation message)

## [1.3.9]

### Changed

- Updated openid-rbac-security to 1.1.1 (includes fallback user claims support)
- Updated auditor-aware-jpa to 1.0.5 (fixes potential NPE in auditor-aware providers when authentication.getName() returns null)

## [1.3.8]

### Fixed

- Updated opentmf-commons to 1.0.7 (fixes reading from the classpath by using the current thread's classloader)

## [1.3.7]

### Changed

- Updated opentmf-mockserver to 1.1.1
- Updated opentmf-camunda7 to 24.0.2
- Updated opentmf-clients-base to 1.1.5
- Updated opentmf-v4-clients to 1.1.5

## [1.3.6]

### Changed

- Updated camunda-incident-logger to 1.0.4 (camunda 7.24 upgrade)
- Updated camunda7-test-framework to 1.0.6 (camunda 7.24 upgrade)
- Updated opentmf-camunda7 to 24.0.0 (camunda 7.24 upgrade)
- Updated opentmf-mockserver to 1.0.7 (fixes encoded path parsing)

## [1.3.5]

### Changed

- Updated opentmf-clients-base to 1.1.3 (fixes default error handling)
- Updated opentmf-v4-clients to 1.1.3 (uses the new opentmf-clients-base)
- Updated opentmf-db-lock-service to 1.0.9 (adds a new method isLocked(LockType))
- Updated auditor-aware-jpa to 1.0.4 (marks createdOn and createdBy fields as not updatable)

## [1.3.4]

### Fixed

- Updated opentmf-clients-base to 1.1.3 (fixes default error handling)
- Updated opentmf-v4-clients to 1.1.3 (uses the new opentmf-clients-base)

## [1.3.3]

### Fixed

- Updated camunda7-bpmn-sync-service from 1.1.1 to 1.1.3 (fixes resource name read issue)
- Updated opentmf-mockserver from 1.0.2 to 1.0.6 (fixes versioned entity handling)

### Changed

- Updated opentmf-camunda7 from 23.0.0 to 23.0.2 (single sign-on support added for OpenID auth)

## [1.3.2]

### Changed

- Updated opentmf-commons to 1.0.6 (numeric values allowed in OffsetDateTime deserialization)
- Updated opentmf-clients-base to 1.1.2 (better error handling for non-json content or no content)

## [1.3.1]

### Changed

- Updated dnext-catalog-sync-service to 1.1.0

## [1.3.0]

### Changed

- Initial open-source version, replacing pia with opentmf

## [1.2.7]

**Backward Incompatible**

### Changed

- Updated tmf-clients-base to 1.1.0
- Updated tmf-v4-clients to 1.1.1

## [1.2.6]

### Changed

- Updated tmf-clients-base to 1.0.5
- Updated tmf-v4-clients to 1.1.0

## [1.2.5]

### Added

- Updated dnext-tmf-v4-models to 1.0.9 (adds DNextProductOffering with the extended field "rules")

## [1.2.4]

### Added

- Updated camunda-7-test-framework to 1.0.3 (adds the ability to use set listeners for each and every task, not only receive tasks)

## [1.2.3]

### Added

- Updated pia-web-clients to 1.0.9 (adds the ability to use mutual TLS authentication)

## [1.2.2]

### Added

- Updated tmf-v4-utils to 1.0.4 (adds validateOrder method to ServiceOrderUtil)

## [1.2.1]

### Changed

- Updated dnext-tmf-v4-models to 1.0.8

## [1.2.0]

### Changed

- Updated dnext-tmf-v4-models to 1.0.7

## [1.1.9]

### Changed

- Updated auditor-aware-jpa to 1.0.2
- Updated pia-commons to 1.0.2

## [1.1.8]

### Changed

- Updated pia-security to 1.0.9

## [1.1.7]

### Changed

- Updated pia-commons to 1.0.1
- Updated dnext-tmf-v4-models to 1.0.6

## [1.1.6]

### Changed

- Updated pia-bpmn-sync-service to 1.1.0
- Updated pia-catalog-sync-service to 1.0.8

## [1.1.5]

### Changed

- Updated pia-security to 1.0.8

## [1.1.4]

### Changed

- Updated pia-web-clients to 1.0.8 (fewer dependencies for the reactive WebClient)
- Updated pia-bpmn-sync-service, pia-catalog-sync-service, pia-web-clients, tmf-clients-base, tmf-v4-clients
- Corrected the link from tmf-clients-base to pia-web-clients in pia-libraries.png

## [1.1.3]

### Changed

- Updated pia-bpmn-sync-service to 1.0.8

## [1.1.2]

### Changed

- Updated dynamic-mock-expectations to 1.0.1
- Updated pia-db-lock-service to 1.0.7
- Updated pia-bpmn-sync-service to 1.0.7
- Updated pia-catalog-sync-service to 1.0.6

## [1.1.1]

### Added

- Introduced auditor-aware-jpa
- Added dnext-tmf-633-model to dnext-tmf-v4-models

### Changed

- Updated pia-security to 1.0.7
- Updated pia-camunda-7 to 22.0.6

## [1.1.0]

### Changed

- Updated many library versions to their latest

### Fixed

- Fixed quoteClientProvider bean name

## [1.0.9]

### Added

- Added tmf-681 v4 model and tmf-client

### Changed

- Updated dnext tmf v4 models and tmf v4 utils

## [1.0.8]

### Changed

- Updated pia-security from 1.0.3 to 1.0.5
- Updated pia-db-lock-service from 1.0.4 to 1.0.5
- Updated pia-bpmn-sync-service from 1.0.4 to 1.0.5
- Updated pia-catalog-sync-service from 1.0.3 to 1.0.4

## [1.0.7]

### Changed

- Updated dnext-tmf-v4-models versions

## [1.0.6]

### Changed

- Updated pia-web-clients to 1.0.5
- Updated pia-db-lock-service to 1.0.3
- Updated pia-bpmn-sync-service to 1.0.3
- Updated pia-catalog-sync-service to 1.0.2
- Updated tmf-clients-base to 1.0.2
- Updated tmf-v4-clients to 1.0.4

## [1.0.5]

### Changed

- Updated pia-security to 1.0.3
- Updated pia-camunda-7 to 22.0.2 (to use pia-security 1.0.3)

## [1.0.0 - 1.0.4]

### Added

- Initial releases
