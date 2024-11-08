# pia-commons-versions
A Super pom project that exposes the latest release versions of pia-commons artifacts.

## Usage
In order to easily obtain the compatible versions of the latest released pia-commons libraries in your projects, import this super pom dependencies:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.pia.commons</groupId>
      <artifactId>pia-commons-versions</artifactId>
      <type>pom</type>
      <scope>import</scope>
      <version>RELEASE</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

RELEASE is a reserved word for Maven. It means the latest released version, snapshots not included.

Instead of using the keyword RELEASE, it is also possible to explicitly specify a version for tmf-commons-versions to achieve predictable builds even in the future.

And then you can depend on any com.pia.commons project without specifying a version. For example:

```xml
<dependencies>
  <dependency>
    <groupId>com.pia.commons</groupId>
    <artifactId>pia-openid-webclient-provider</artifactId>
  </dependency>
</dependencies>
```
## Release Notes
### 1.0.0 - 1.0.4
- Initial Releases
### 1.0.5
- Updated pia-security to 1.0.3
- Updated pia-camunda-7 to 22.0.2 (to use pia-security 1.0.3)
### 1.0.6
- Updated pia-web-clients to 1.0.5
- Updated pia-db-lock-service to 1.0.3
- Updated pia-bpmn-sync-service to 1.0.3
- Updated pia-catalog-sync-service to 1.0.2
- Updated pia-web-clients to 1.0.5
- Updated tmf-clients-base to 1.0.2
- Updated tmf-v4-clients to 1.0.4
### 1.0.7
- Updated dnext-tmf-v4-models versions