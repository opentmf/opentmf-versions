# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
