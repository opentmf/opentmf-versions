
## Release Notes
### 1.4.2
- Updated tmf630-toolkit to 1.0.1 (adds `filter=` JsonPath support both for JPA and MongoDB).
### 1.4.1
- Added tmf630-toolkit artifacts.
### 1.4.0
- Updates camunda7-test-framework to 1.0.7 (enhances support for more task types that can receive a correlation message).
### 1.3.9
- Updates openid-rbac-security to 1.1.1 (includes fallback user claims support)
- Updates auditor-aware-jpa to 1.0.5 (fixes potential NPE in auditor-aware providers when authentication.getName() returns null)
### 1.3.8
- Updates opentmf-commons to 1.0.7 (fixes reading from the classpath by using the current thread's classloader)
### 1.3.7
- Updates opentmf-mockserver to 1.1.1
- Updates opentmf-camunda7 to 24.0.2
- Updates opentmf-clients-base to 1.1.5
- Updates opentmf-v4-clients to 1.1.5
### 1.3.6
- Updates camunda-incident-logger to 1.0.4 (camunda 7.24 upgrade)
- Updates camunda7-test-framework to 1.0.6 (camunda 7.24 upgrade)
- Updates opentmf-camunda7 to 24.0.0 (camunda 7.24 upgrade)
- Updates opentmf-mockserver to 1.0.7 (fixes encoded path parsing)
### 1.3.5
- Updates opentmf-clients-base version to 1.1.3 (fixes default error handling)
- Updates opentmf-v4-clients version to 1.1.3 (uses the new opentmf-clients-base)
- Updates opentmf-db-lock-service version to 1.0.9 (adds a new method isLocked(LockType))
- Updates auditor-aware-jpa version to 1.0.4 (marks createdOn and createdBy fields as not updatable)
### 1.3.4
- Updates opentmf-clients-base version to 1.1.3 (fixes default error handling)
- Updates opentmf-v4-clients version to 1.1.3 (uses the new opentmf-clients-base)
### 1.3.3
- Updates camunda7-bpmn-sync-service version from 1.1.1 to 1.1.3 (fixes resource name read issue)
- Updates opentmf-mockserver version from 1.0.2 to 1.0.6 (fixes versioned entity handling)
- Updates opentmf-camunda7 version from 23.0.0 to 23.0.2 (single sign-on support added for OpenID auth)
### 1.3.2
- Updates opentmf-commons version to 1.0.6 (numeric values allowed in OffsetDateTime deserialization)
- Updates opentmf-clients-base version to 1.1.2 (better error handling for non-json content or no content)
### 1.3.1
- Updates dnext-catalog-sync-service to 1.1.0
### 1.3.0
- Initial open-source version, replacing pia with opentmf
### 1.2.7 (Backward Incompatible)
- Updated tmf-clients-base to 1.1.0
- Updated tmf-v4-clients to 1.1.1
### 1.2.6
- Updated tmf-clients-base to 1.0.5
- Updated tmf-v4-clients to 1.1.0
### 1.2.5
- Updated dnext-tmf-v4-models to 1.0.9, which adds DNextProductOffering with the extended field "rules".
### 1.2.4
- Updated camunda-7-test-framework to 1.0.3, which adds the ability to use set listeners for each and every task, not only receive tasks.
### 1.2.3
- Updated pia-web-clients to 1.0.9, which adds the ability to use mutual TLS authentication.
### 1.2.2
- Updated tmf-v4-utils to 1.0.4, that adds validateOrder method to ServiceOrderUtil.
### 1.2.1
- Updated dnext-tmf-v4-models to 1.0.8
### 1.2.0
- Updated dnext-tmf-v4-models to 1.0.7
### 1.1.9
- Updated auditor-aware-jpa to 1.0.2
- Updated pia-commons to 1.0.2
### 1.1.8
- Updated pia-security to 1.0.9
### 1.1.7
- Updated pia-commons to 1.0.1
- Updated dnext-tmf-v4-models to 1.0.6
### 1.1.6
- Updated pia-bpmn-sync-service to 1.1.0
- Updated pia-catalog-sync-service to 1.0.8
### 1.1.5
- Updated pia-security to 1.0.8
### 1.1.4
- Updates to pia-web-clients 1.0.8, for fewer dependencies for the reactive WebClient.
- The affected projects from the pia-web-clients update are also updated:
    - pia-bpmn-sync-service
    - pia-catalog-sync-service
    - pia-web-clients
    - tmf-clients-base
    - tmf-v4-clients
- Corrected the link from the tmf-clients-base to the pia-web-clients in pia-libraries.png
### 1.1.3
- Updated pia-bpmn-sync-service to 1.0.8
### 1.1.2
- Updated dynamic-mock-expectations to 1.0.1
- Updated pia-db-lock-service to 1.0.7
- Updated pia-bpmn-sync-service to 1.0.7
- Updated pia-catalog-sync-service to 1.0.6
### 1.1.1
- introduces auditor-aware-jpa
- updates pia-security to 1.0.7
- updated pia-camunda-7 to 22.0.6
- adds dnext-tmf-633-model to dnext-tmf-v4-models
### 1.1.0
- updates many library versions to their latest
- includes the fix in quoteClientProvider bean name
### 1.0.9
- adds tmf-681 v4 model and tmf-client
- dependency updates of dnext tmf v4 models and tmf v4 utils
### 1.0.8
- pia-security: 1.0.3 -> 1.0.5
- pia-db-lock-service: 1.0.4 -> 1.0.5
- pia-bpmn-sync-service: 1.0.4 -> 1.0.5
- pia-catalog-sync-service: 1.0.3 -> 1.0.4
### 1.0.7
- Updated dnext-tmf-v4-models versions
### 1.0.6
- Updated pia-web-clients to 1.0.5
- Updated pia-db-lock-service to 1.0.3
- Updated pia-bpmn-sync-service to 1.0.3
- Updated pia-catalog-sync-service to 1.0.2
- Updated pia-web-clients to 1.0.5
- Updated tmf-clients-base to 1.0.2
- Updated tmf-v4-clients to 1.0.4
### 1.0.5
- Updated pia-security to 1.0.3
- Updated pia-camunda-7 to 22.0.2 (to use pia-security 1.0.3)
### 1.0.0 - 1.0.4
- Initial Releases
