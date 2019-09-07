# Public Maven Repository

This repository contains my own public artifacts for Maven, Gradle, Ivy etc.

### Usage

The first step is configure GitHub repository authentication information.
For this purpose, you need generate GitHub personal access token (`GitHub` -> `Settings` -> `Developer settings` -> `Personal access tokens` -> `Generate new token`)
and then create Maven settings file located at `$HOME/.m2/settings.xml`.

```xml
<settings xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <servers>
    <server>
      <id>github</id>
      <password>[GITHUB_TOKEN]</password>
    </server>
  </servers>
</settings>
```

Next step is configure [site-maven-plugin](https://github.com/github/maven-plugins#site-plugin) in your `pom.xml`.

```xml
<properties>
  <github.global.server>github</github.global.server>
</properties>

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-deploy-plugin</artifactId>
      <version>2.8.2</version>
      <configuration>
        <altDeploymentRepository>
          internal.repo::default::file://${project.build.directory}/mvn-repo
        </altDeploymentRepository>
      </configuration>
    </plugin>
    <plugin>
      <groupId>com.github.github</groupId>
      <artifactId>site-maven-plugin</artifactId>
      <version>0.12</version>
      <configuration>
        <message>${project.groupId}:${project.artifactId}:${project.version}</message>
        <merge>true</merge>
        <outputDirectory>${project.build.directory}/mvn-repo</outputDirectory>
        <branch>refs/heads/release</branch>
        <includes>
          <include>**/*</include>
        </includes>
        <repositoryName>public-maven-repository</repositoryName>
        <repositoryOwner>vitalibo</repositoryOwner>
      </configuration>
      <executions>
        <execution>
          <goals>
            <goal>site</goal>
          </goals>
          <phase>deploy</phase>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>

<distributionManagement>
  <repository>
    <id>internal.repo</id>
    <url>file://${project.build.directory}/mvn-repo</url>
  </repository>
</distributionManagement>
```

Now you can put artifacts in your repository

```bash
mvn clean deploy
```

To use artifacts as dependency in your project, you need add [public-maven-repository](https://github.com/vitalibo/public-maven-repository) as remote Maven repository.

```xml
<dependencies>
  <dependency>
      <groupId>com.github.vitalibo</groupId>
      <artifactId>[ARTIFACT-ID]</artifactId>
      <version>[VERSION]</version>
  </dependency>
</dependencies>

<repositories>
  <repository>
    <id>vitalibo-public-maven-repository</id>
    <url>https://raw.github.com/vitalibo/public-maven-repository/release/</url>
    <snapshots>
      <enabled>true</enabled>
      <updatePolicy>always</updatePolicy>
    </snapshots>
  </repository>
</repositories>
```

#### Integration with Travis CI

Securely pass `GITHUB_TOKEN` in Maven, you can create environment variable and tells Maven to use it.
Add `.travis.settings.xml` file to your Maven project.

```xml
<servers>
  <server>
    <id>github</id>
    <password>${env.GITHUB_TOKEN}</password>
  </server>
</servers>
```

Create `.travis.yml` file for integration with [Travis CI](https://travis-ci.org/).

```yaml
sudo: false
language: java
jdk:
  - openjdk8
cache:
  directories:
    - "$HOME/.cache"

deploy:
  provider: script
  script: "cp .travis.settings.xml $HOME/.m2/settings.xml && mvn deploy"
  skip_cleanup: true
  on:
    tags: true
```

**Note**: properties `on: tags: true` enable triggering deployment only when a tagged commit is pushed.

### Links

- [Как сделать свой Maven Repository на Github?](https://devcolibri.com/как-сделать-свой-maven-repository-на-github/)
- [How to Deploy Maven Artifacts with Travis CI and packagecloud](https://blog.travis-ci.com/2017-03-30-deploy-maven-travis-ci-packagecloud/)
