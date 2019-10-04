# Using GRPC in Java Maven project

Text below explains how to create `pom.xml` to build Java GRPC project using
Maven. It can be done using [`protobuf-maven-plugin`](https://www.xolstice.org/protobuf-maven-plugin/)

Add `.proto` files into project file tree under `main/proto` and `test/proto`
directories.

```
src
+--main
|  +--proto
|     +--RealService.proto
+--test
   +--proto
      +--TestService.proto
```

Edit `pom.xml` file. Add properties to set Protobuf and GRPC versions.

```xml
  <properties>
    ...
    <protobuf.version>3.5.1</protobuf.version>
    <grpc.version>1.20.0</grpc.version>
  </properties>
```

Add necessary dependencies. At least Protobuf, GRPC and `javax.annotation` are
required.

```xml
  <dependencies>
    ...
    <dependency>
      <groupId>com.google.protobuf</groupId>
      <artifactId>protobuf-java</artifactId>
      <version>${protobuf.version}</version>
    </dependency>
    <dependency>
      <groupId>io.grpc</groupId>
      <artifactId>grpc-netty-shaded</artifactId>
    </dependency>
    <dependency>
      <groupId>io.grpc</groupId>
      <artifactId>grpc-protobuf</artifactId>
    </dependency>
    <dependency>
      <groupId>io.grpc</groupId>
      <artifactId>grpc-stub</artifactId>
    </dependency>
    <dependency>
      <groupId>javax.annotation</groupId>
      <artifactId>javax.annotation-api</artifactId>
      <version>1.3.2</version>
    </dependency>
    ...
  </dependencies>
```

Ensure that GPRC components are compartible.

```xml
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-bom</artifactId>
        <version>${grpc.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
```

Add `os-maven-plugin` to determine OS version and download
correct Protobuf compiler version automatically.

```xml
  <build>
  ...
    <extensions>
      <extension>
        <groupId>kr.motd.maven</groupId>
        <artifactId>os-maven-plugin</artifactId>
        <version>1.6.2</version>
      </extension>
    </extensions>
  ...
  </build>
```

Add `protobuf-maven-plugin` which finds and compiles `.proto` files under
`proto` directory.

```xml
  <build>
    ...
    <plugins>
	  ...
      <plugin>
        <groupId>org.xolstice.maven.plugins</groupId>
        <artifactId>protobuf-maven-plugin</artifactId>
        <version>0.6.1</version>
        <configuration>
          <!-- artifact to download binary protobuf compiler -->
          <protocArtifact>com.google.protobuf:protoc:${protobuf.version}:exe:${os.detected.classifier}</protocArtifact>
          <!-- make maven using GRPC plugin for compile-custom and
          test-compile-custom goals -->
          <pluginId>grpc-java</pluginId>
          <!-- artifact to download GRPC protobuf compiler plugin -->
          <pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}</pluginArtifact>
        </configuration>
        <executions>
          <execution>
            <goals>
              <!-- compile .proto files located under main directory -->
              <goal>compile</goal>
              <goal>compile-custom</goal>
              <!-- compile .proto files located under test directory -->
              <goal>test-compile</goal>
              <goal>test-compile-custom</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
	  ...
    </plugins>
  ...
  </build>
```

