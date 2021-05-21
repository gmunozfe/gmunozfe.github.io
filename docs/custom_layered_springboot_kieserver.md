---
title: 'Custom Layered Immutable Springboot Kie Server'
permalink: /custom-layered-springboot-kieserver/
categories:
  - kie-server
  - springboot
---

# Custom Layered Immutable Springboot Kie Server

When creating an image for immutable Springboot Kie Server, it is possible to split up the files and folders belonging to the fat-jar into different layers.

Main advantages of this approach are:

1. Layers are downloaded once (saving bandwith), and can be reused for other images.
2. Libraries, code and resources are grouped into layers based on the likelihood to change between builds. This reduces the image generation time.
3. Execution over the unzipped classes is a little bit faster than launching the fat-jar (_java -jar app.jar_)

Considering all these points, it has sense to layer the immutable Springboot Kie Server -isolating the KJARs into a new custom layer- because business assets in KJARs are more likely to change.

## Packaging with layers



```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>${springboot.version}</version>
    <configuration>
        <layers>
            <enabled>true</enabled>
            <configuration>${project.basedir}/src/layers.xml</configuration>
        </layers>
        <image>
            <name>${spring-boot.build-image.name}</name>
        </image>
    </configuration>
    <executions>
      <execution>
          <goals>
              <goal>repackage</goal>
          </goals>
      </execution>
    </executions>
</plugin>
```


```xml
<layers xmlns="http://www.springframework.org/schema/boot/layers"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/boot/layers
                          https://www.springframework.org/schema/boot/layers/layers-2.5.xsd">
    <application>
        <into layer="spring-boot-loader">
            <include>org/springframework/boot/loader/**</include>
        </into>
        <into layer="kjars">
            <include>BOOT-INF/classes/KIE-INF/**</include>
        </into>
        <into layer="application" />
    </application>
    <dependencies>
        <into layer="snapshot-dependencies">
            <include>*:*:*SNAPSHOT</include>
        </into>
        <into layer="dependencies">
            <includeModuleDependencies />
        </into>
    </dependencies>
    <layerOrder>
        <layer>dependencies</layer>
        <layer>spring-boot-loader</layer>
        <layer>snapshot-dependencies</layer>
        <layer>application</layer>
        <layer>kjars</layer>
    </layerOrder>
</layers>
```


```console
foo@bar:~$ mvn clean package -DskipTests
```

```console
foo@bar:~$ java -Djarmode=layertools -jar target/immutable-springboot-kie-server-1.0.0.jar list
dependencies
spring-boot-loader
snapshot-dependencies
application
kjars
```

```console
foo@bar:~$ java -Djarmode=layertools -jar target/immutable-springboot-kie-server-1.0.0.jar extract
foo@bar:~$ tree kjars 
kjars
└── BOOT-INF
    └── classes
        └── KIE-INF
            └── lib
                ├── kjar-sample-1.0.0.jar
                ├── kjar-sample-1.1.0.jar
                └── other-kjar-1.0.0.jar

4 directories, 3 files

```

```console
foo@bar:~$ cat application/BOOT-INF/layers.idx 
- "dependencies":
  - "BOOT-INF/lib/"
- "spring-boot-loader":
  - "org/"
- "snapshot-dependencies":
- "application":
  - "BOOT-INF/classes/application.properties"
  - "BOOT-INF/classes/org/"
  - "BOOT-INF/classes/quartz-db.properties"
  - "BOOT-INF/classpath.idx"
  - "BOOT-INF/layers.idx"
  - "META-INF/"
- "kjars":
  - "BOOT-INF/classes/KIE-INF/"
```



Custom tags may be also added to the process or case by means of Business Central, introducing a non-predefined value for the target variable. 
