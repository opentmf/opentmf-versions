# opentmf-versions

A Maven BOM (Bill of Materials) that provides compatible release versions for all OpenTMF artifacts.

See [CHANGELOG.md](CHANGELOG.md) for release notes.

## Version Compatibility

| BOM version | Spring Boot | Jackson | Status |
|---|---|---|---|
| **2.x** | 4.x | 3.x | Active development |
| **1.x** | 3.5.x | 2.x | Maintenance only |

## Overview

The diagram below shows the OpenTMF libraries and their compile-time dependencies. Many of these are multi-module projects that provide several additional artifacts.

![](module-dependencies.png)

## Usage

Import the BOM in your `<dependencyManagement>` section:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.opentmf</groupId>
      <artifactId>opentmf-versions</artifactId>
      <type>pom</type>
      <scope>import</scope>
      <version>RELEASE</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

> **Heads Up** &mdash; `RELEASE` is a Maven reserved word that resolves to the latest released (non-snapshot) version. While convenient, Maven discourages its use because builds become non-reproducible when a new version is published. It is **recommended** to pin a specific version of `opentmf-versions` for predictable builds.

Then depend on any OpenTMF artifact without specifying its version:

```xml
<dependencies>
  <dependency>
    <groupId>org.opentmf.client</groupId>
    <artifactId>opentmf-http-clients-starter</artifactId>
  </dependency>
</dependencies>
```

## Artifacts

| groupId | artifactId | type | description                                                                                                               |
|---|---|---|---------------------------------------------------------------------------------------------------------------------------|
| org.opentmf.commons | [opentmf-commons](https://github.com/opentmf/opentmf-commons) | jar | Shared utilities, annotations, and base classes used across all OpenTMF projects.                                         |
| org.opentmf.commons | [opentmf-json-patch](https://github.com/opentmf/opentmf-commons) | jar | JSON Patch (RFC 6902) implementation for partial resource updates.                                                        |
| org.opentmf.model | [opentmf-v4-models](https://github.com/opentmf/opentmf-v4-models) | pom | Generated Java model classes for 80+ TMF v4 API specifications.                                                           |
| org.opentmf.model | [opentmf-v4-api](https://github.com/opentmf/opentmf-v4-models) | pom | Generated Spring controller interfaces for 80+ TMF v4 API specifications.                                                 |
| org.opentmf.util | [opentmf-v4-utils](https://github.com/opentmf/opentmf-v4-utils) | pom | Convenience utilities for working with TMF v4 model objects.                                                              |
| org.opentmf.model | [dnext-v4-models](https://github.com/opentmf/dnext-opentmf-v4-models) | pom | Extended model classes covering all available DNext TMF implementations and their custom extensions.                      |
| org.opentmf.client | [opentmf-http-clients](https://github.com/opentmf/opentmf-http-clients) | pom | HTTP client foundation for REST and Reactive clients with unified configuration and cached bearer-token and mTLS support. |
| org.opentmf.client | [opentmf-api-clients](https://github.com/opentmf/opentmf-api-clients) | pom | Ready-to-use TMF-630 compliant API clients for TMF v4 backends (REST and reactive).                                       |
| org.opentmf.query | [tmf630-toolkit](https://github.com/opentmf/tmf630-toolkit) | pom | TMF-630 compliant filtering, paging, and sorting for Spring Web MVC and MongoDB.                                          |
| org.opentmf.mockserver | [opentmf-mockserver](https://github.com/opentmf/opentmf-mockserver) | jar | [MockServer](https://mock-server.com/)-based test double that emulates a TMF-630 compliant backend.                       |
| org.opentmf.security | [openid-rbac-security](https://github.com/opentmf/openid-rbac-security) | jar | OpenID Connect authentication with role-based access control (RBAC) for Spring Boot.                                      |
| org.opentmf.util | [auditor-aware-jpa](https://github.com/opentmf/auditor-aware-jpa) | jar | JPA mapped superclasses that automatically track created/modified-by audit fields.                                        |
| org.opentmf.util | [opentmf-db-lock-service](https://github.com/opentmf/opentmf-db-lock-service) | jar | Database-backed distributed lock service for cluster-level coordination.                                                  |
| org.opentmf.camunda | [camunda7-incident-logger](https://github.com/opentmf/camunda7-incident-logger) | jar | Produces structured, easy-to-trace log entries when a Camunda 7 incident occurs.                                          |
| org.opentmf.camunda | [camunda7-test-framework](https://github.com/opentmf/camunda7-test-framework) | jar | Test harness for verifying Camunda 7 process definitions and task flows.                                                  |
| org.opentmf.camunda | [camunda7-bpmn-sync-service](https://github.com/opentmf/camunda7-bpmn-sync-service) | jar | Deploys BPMN definitions to Camunda 7 on startup with optional process instance migration.                                |
| org.opentmf.dnext | [dnext-catalog-sync-service](https://github.com/opentmf/dnext-catalog-sync-service) | jar | Keeps versioned resource, service, and product catalogs in sync with DNext catalog backends.                              |
| org.opentmf.camunda | [opentmf-camunda7](https://github.com/opentmf/opentmf-camunda7) | jar | Spring Boot microservice embedding Camunda 7 Community Edition with OpenID SSO and RBAC.                                  |
| N/A | [http-endpoint-kicker](https://github.com/opentmf/http-endpoint-kicker) | docker | Minimal Docker sidecar that optionally obtains an access token, fires a single HTTP request, and exits.                   |
