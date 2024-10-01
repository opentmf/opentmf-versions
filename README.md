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

And then you can depend on any com.pia.commons project without specifying a version. For example:

```xml
<dependencies>
  <dependency>
    <groupId>com.pia.commons</groupId>
    <artifactId>pia-openid-webclient-provider</artifactId>
  </dependency>
</dependencies>
```