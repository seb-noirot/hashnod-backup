---
title: "Migrating from Javax to Jakarta"
seoTitle: "Migrating from Javax to Jakarta: A Maven Plugin Approach | Seb's Tech"
seoDescription: "Migrate a javax library to a jakarta library using Maven's transformer-maven-plugin. A clear, step-by-step guide for a seamless migration process."
datePublished: Wed Oct 04 2023 08:29:58 GMT+0000 (Coordinated Universal Time)
cuid: clnbhn3gs000k09k04k2jdwa6
slug: migrating-from-javax-to-jakarta
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/KgLtFCgfC28/upload/67a352501180a15765875cfa04a07e92.jpeg
tags: plugins, maven, migration, jakarta

---

### Introduction

Transitioning from `javax` to `jakarta` namespaces is pivotal in the Java EE ecosystem, especially when maintaining or lacking control over `javax` libraries. The `transformer-maven-plugin` facilitates this migration. In this article, we'll generate a `javax` library from an XSD file, build a jar, and then create its `jakarta` version within a Maven project comprising two modules: one for `javax`, the other for `jakarta`.

### Prerequisites

In this article, we'll be utilizing Java and Apache Maven. It's assumed that you have basic familiarity with these technologies. Additionally, we will employ the `transformer-maven-plugin` to aid in the migration process. Now, with the basics covered, let's proceed to set up our Maven project for migrating a `javax` library to a `jakarta` one.

### **Creating the Maven Project and Javax Module**

1. **Create a New Maven Project**
    
    ```bash
    mvn archetype:generate -DgroupId=com.example -DartifactId=javax-to-jakarta -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```
    
2. **Convert to a Multi-module Project**
    
    Edit the `pom.xml` in the `javax-to-jakarta` directory to include the `pom` packaging type and Java 8 source/target:
    
    ```xml
    <packaging>pom</packaging>
    <properties>
       <maven.compiler.source>8</maven.compiler.source>
       <maven.compiler.target>8</maven.compiler.target>
    </properties>
    ```
    
3. **Create a Module for Javax**
    
    ```bash
    mvn archetype:generate -DgroupId=com.example -DartifactId=javax-module -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```
    
4. **Generate Javax Library from XSD**
    
    Create a directory `src/main/xsd` within `javax-module`, and place the following `person.xsd` file in it:
    
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
      <xs:element name="Person" type="PersonType"/>
      <xs:complexType name="PersonType">
        <xs:sequence>
          <xs:element name="FirstName" type="xs:string"/>
          <xs:element name="LastName" type="xs:string"/>
          <xs:element name="Age" type="xs:integer"/>
        </xs:sequence>
      </xs:complexType>
    </xs:schema>
    ```
    
    Update `javax-module/pom.xml` to include the `jaxb2-maven-plugin` and `javax` dependencies:
    
    ```xml
    <dependencies>
        <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
            <version>2.3.1</version>
        </dependency>
    </dependencies>
    <build>
      <plugins>
        <plugin>
          <groupId>org.codehaus.mojo</groupId>
          <artifactId>jaxb2-maven-plugin</artifactId>
          <version>2.5.0</version>
          <executions>
            <execution>
              <goals>
                <goal>xjc</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
      </plugins>
    </build>
    ```
    
5. **Build and Package the Javax Library**
    
    ```bash
    mvn package
    ```
    

### **Setting up the Jakarta Module**

1. **Create a Module for Jakarta**
    
    ```bash
    mvn archetype:generate -DgroupId=com.example -DartifactId=jakarta-module -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```
    
2. **Update Parent POM**
    
    Add `jakarta-module` to the parent `pom.xml`:
    
    ```xml
    <modules>
        <module>javax-module</module>
        <module>jakarta-module</module>
    </modules>
    ```
    
3. **Configure Transformer-Maven-Plugin in Jakarta Module**
    
    Navigate to the `jakarta-module` directory and edit its `pom.xml` to include the `transformer-maven-plugin` and specify the dependency on the `javax` module:
    
    ```xml
    <dependencies>
        <dependency>
            <groupId>jakarta.xml.bind</groupId>
            <artifactId>jakarta.xml.bind-api</artifactId>
            <version>3.0.1</version>
        </dependency>
    </dependencies>
    <build>
            <plugins>
                <plugin>
                    <groupId>org.eclipse.transformer</groupId>
                    <artifactId>transformer-maven-plugin</artifactId>
                    <version>0.5.0</version>
                    <extensions>true</extensions>
                    <configuration>
                        <rules>
                            <jakartaDefaults>true</jakartaDefaults>
                        </rules>
                    </configuration>
                    <executions>
                        <execution>
                            <id>default-jar</id>
                            <goals>
                                <goal>jar</goal>
                            </goals>
                            <configuration>
                                <artifact>
                                    <groupId>com.example</groupId>
                                    <artifactId>javax-module</artifactId>
                                    <version>${project.version}</version>
                                </artifact>
                            </configuration>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    ```
    
4. **Execute the Migration**
    
    From the root project directory (`javax-to-jakarta`), execute the following command to perform the migration:
    
    ```bash
    mvn clean package
    ```
    

### **Conclusion**

In this article, we walked through a structured approach to migrate a `javax` library to a `jakarta` library using the `transformer-maven-plugin` within a Maven multi-module project. This methodology proves to be beneficial especially when there's a need to maintain a `javax` library or when control over the `javax` library is not available. Our setup delineated clear, easy-to-follow steps for migration, which is not only reproducible but also sets a foundation for testing and validation, ensuring the integrity of the libraries across the transition.

The `transformer-maven-plugin` offers a range of features beyond what was explored in this guide, making it a robust tool worth exploring for similar migration projects.

The complete code for the project discussed is available on [GitHub](https://github.com/seb-noirot/javax-to-jakarta), providing a practical reference to jump-start your migration endeavors from `javax` to `jakarta`.