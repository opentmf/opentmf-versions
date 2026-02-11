# opentmf-versions
A Super pom project that exposes the latest release versions of opentmf-commons artifacts.

See [CHANGELOG.md](CHANGELOG.md) for release notes.

The following image is a conceptual view of the artifacts whose compatible release versions are provided within this super pom project.

![](opentmf-libraries.png)

Note that many of those libraries are super pom projects themselves, providing several more artifacts. 

## Usage
In order to easily obtain the compatible versions of the latest released opentmf-commons libraries in your projects, import this super pom dependencies:

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
**Heads Up**
> RELEASE is a reserved word for Maven. It means the latest released version of an artifact, excluding snapshots.
> 
> Despite providing the comfort of always using the latest released version of an artifact, the use of the RELEASE keyword is discouraged by Maven, because the build will produce a different result on subsequent builds when a new version of the dependent artifact is released.
> 
> Therefore, instead of using the keyword RELEASE, it is **_recommended_** to explicitly specify a version for tmf-commons-versions to achieve predictable builds even in the future.

And then you can depend on any OpenTMF project without specifying its version. For example:

```xml
<dependencies>
  <dependency>
    <groupId>org.opentmf.client</groupId>
    <artifactId>opentmf-openid-webclient-provider</artifactId>
  </dependency>
</dependencies>
```

## Links to the Artifacts

| groupId                | artifactId                                                                          | type   | description                                                                                                                                                                                                                                                                     |
|------------------------|-------------------------------------------------------------------------------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| org.opentmf.commons    | [opentmf-commons](https://github.com/opentmf/opentmf-commons)                       | jar    | General purpose utility classes and annotations for any Java project.                                                                                                                                                                                                           |
| org.opentmf.model      | [opentmf-v4-models](https://github.com/opentmf/opentmf-v4-models)                   | pom    | Unified model classes generated for various TMF v4 specifications                                                                                                                                                                                                               |
| org.opentmf.util       | [opentmf-v4-utils](https://github.com/opentmf/opentmf-v4-utils)                     | pom    | Several utility classes for TMF v4 models.                                                                                                                                                                                                                                      |
| org.opentmf.dnext      | [dnext-tmf-v4-models](https://github.com/opentmf/dnext-opentmf-v4-models)           | pom    | DNext Model Extensions for TMF v4.                                                                                                                                                                                                                                              |
| org.opentmf.client     | [opentmf-web-clients](https://github.com/opentmf/opentmf-web-clients)               | pom    | General purpose WebClient libraries with cached getToken support                                                                                                                                                                                                                |
| org.opentmf.mockserver | [opentmf-mockserver](https://github.com/opentmf/opentmf-mockserver)                 | jar    | An extended [mock-server](https://mock-server.com/) with cached requests emulating a TMF-630 compliant backend. Very useful for integration and functional tests.                                                                                                               |
| org.opentmf.client     | [opentmf-clients-base](https://github.com/opentmf/opentmf-clients-base)             | jar    | TMF Clients Base Classes and Common Implementation                                                                                                                                                                                                                              |
| org.opentmf.client     | [opentmf-v4-clients](https://github.com/opentmf/opentmf-v4-clients)                 | pom    | TMF-630 compliant clients for several TMF v4 backends                                                                                                                                                                                                                           |
| org.opentmf.util       | [opentmf-db-lock-service](https://github.com/opentmf/opentmf-db-lock-service)       | jar    | Useful service to obtain cluster level persistent locks and memorize the latest lock versions.                                                                                                                                                                                  |
| org.opentmf.camunda    | [camunda7-bpmn-sync-service](https://github.com/opentmf/camunda7-bpmn-sync-service) | jar    | Synchronizes the BPMNs to Camunda 7 on app startup and if requested, performs process instance migration                                                                                                                                                                        |
| org.opentmf.dnext      | [dnext-catalog-sync-service](https://github.com/opentmf/dnext-catalog-sync-service) | jar    | Syncronizes versioned resource, service and product catalogs with the DNext catalog backends on app startup.                                                                                                                                                                    |
| org.opentmf.camunda    | [camunda7-incident-logger](https://github.com/opentmf/camunda7-incident-logger)     | jar    | Writes easy-to-track log statements when an incident occurs in Camunda7                                                                                                                                                                                                         |
| org.opentmf.camunda    | [camunda7-test-framework](https://github.com/opentmf/camunda7-test-framework)       | jar    | Writes easy-to-track log statements when an incident occurs in Camunda 7                                                                                                                                                                                                        |
| org.opentmf.security   | [openid-rbac-security](https://github.com/opentmf/openid-rbac-security)             | jar    | OpenID Role based Access Control (RBAC) Security Library                                                                                                                                                                                                                        |
| org.opentmf.util       | [auditor-aware-jpa](https://github.com/opentmf/auditor-aware-jpa)                   | jar    | Auditor aware base JPA mapped super classes for either servlet or reactive web applications                                                                                                                                                                                     |
| org.opentmf.camunda    | [opentmf-camunda7](https://github.com/opentmf/opentmf-camunda7)                     | jar    | An OpenTMF produced Spring Boot microservice that embeds the latest Camunda7 community edition with the public Spin, and OpenID auth for Keycloak plugins, as well as using OpenTMF's Camunda7 Incident Logger, and openid-rbac-security framework to secure the API endpoints. |
| org.opentmf.query      | [tmf630-toolkit](https://github.com/opentmf/tmf630-toolkit)                         | pom    | An adaptation of TMF-630 REST API Design Guidelines for Spring Web MVC, in a multi module project.                                                                                                                                                                              |
| N/A                    | [http-endpoint-kicker](https://github.com/opentmf/http-endpoint-kicker)             | docker | A tiny, opinionated Docker image that (optionally) gets an access token, makes exactly one configurable HTTP request, and then exits with 0 on success (2xx), non-zero otherwise.                                                                                               |
|
