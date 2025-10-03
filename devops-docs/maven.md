# Maven - Build Automation Tool

## Overview

Apache Maven is a build automation tool used primarily for Java projects. Maven addresses two aspects of building software: how software is built, and its dependencies. Unlike earlier tools like Apache Ant, Maven uses conventions for directory structure and default build lifecycle.

### Key Features
- **Convention over Configuration**: Standard directory structure and build lifecycle
- **Dependency Management**: Automatic downloading of dependencies
- **Build Lifecycle**: Standardized build process
- **Plugin Architecture**: Extensible through plugins
- **Project Object Model (POM)**: XML file that defines project configuration
- **Repository Management**: Central and remote repositories for dependencies

## Core Concepts

### Maven Architecture
- **POM (Project Object Model)**: Core configuration file (pom.xml)
- **Build Lifecycle**: Phases that define the build process
- **Plugins**: Tools that perform specific build tasks
- **Dependencies**: External libraries required by the project
- **Repositories**: Storage locations for dependencies and plugins
- **Profiles**: Different build configurations

### Maven Directory Structure
```
my-project/
├── pom.xml                    # Project configuration
├── src/
│   ├── main/
│   │   ├── java/              # Java source files
│   │   ├── resources/         # Resource files
│   │   └── webapp/            # Web application files
│   └── test/
│       ├── java/              # Test source files
│       └── resources/         # Test resource files
├── target/                    # Build output directory
└── .mvn/                      # Maven configuration
```

### Maven Lifecycle
- **Clean Lifecycle**: Cleans up artifacts created by prior builds
- **Default Lifecycle**: Main build lifecycle (compile, test, package, install, deploy)
- **Site Lifecycle**: Generates project documentation

## Installation & Setup

### Prerequisites
- **Java**: Maven requires Java JDK 7 or later
- **Environment Variables**: JAVA_HOME must be set
- **Memory**: Minimum 256MB RAM (512MB recommended)

### Installation Methods

#### Using Package Manager (Ubuntu/Debian)
```bash
# Update package index
sudo apt update

# Install Maven
sudo apt install maven

# Verify installation
mvn -version
```

#### Using Package Manager (CentOS/RHEL)
```bash
# Install Maven
sudo yum install maven

# Verify installation
mvn -version
```

#### Using Homebrew (macOS)
```bash
# Install Maven
brew install maven

# Verify installation
mvn -version
```

#### Manual Installation
```bash
# Download Maven
wget https://archive.apache.org/dist/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.tar.gz

# Extract Maven
tar -xzf apache-maven-3.8.4-bin.tar.gz

# Move to /opt
sudo mv apache-maven-3.8.4 /opt/maven

# Set environment variables
echo 'export MAVEN_HOME=/opt/maven' >> ~/.bashrc
echo 'export PATH=$MAVEN_HOME/bin:$PATH' >> ~/.bashrc

# Apply changes
source ~/.bashrc

# Verify installation
mvn -version
```

## Maven POM (Project Object Model)

### Basic POM Structure
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Project Coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <!-- Project Information -->
    <name>My Project</name>
    <description>A sample Maven project</description>
    <url>https://example.com</url>
    
    <!-- Properties -->
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <junit.version>5.8.2</junit.version>
    </properties>
    
    <!-- Dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.6.3</version>
        </dependency>
    </dependencies>
    
    <!-- Build Configuration -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
            
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0-M5</version>
            </plugin>
        </plugins>
    </build>
    
</project>
```

### POM Elements
- **modelVersion**: POM model version
- **groupId**: Organization or group ID
- **artifactId**: Project name
- **version**: Project version
- **packaging**: Project packaging (jar, war, pom, etc.)
- **dependencies**: Project dependencies
- **build**: Build configuration
- **plugins**: Build plugins
- **properties**: Project properties
- **profiles**: Build profiles

## Maven Commands

### Basic Commands
```bash
# Create new Maven project
mvn archetype:generate -DgroupId=com.example -DartifactId=my-project -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

# Compile source code
mvn compile

# Compile test code
mvn test-compile

# Run tests
mvn test

# Package project
mvn package

# Install to local repository
mvn install

# Deploy to remote repository
mvn deploy

# Clean project
mvn clean

# Clean and compile
mvn clean compile

# Clean, compile, test, package
mvn clean package

# Skip tests
mvn package -DskipTests

# Skip test compilation
mvn package -Dmaven.test.skip=true

# Run with specific profile
mvn package -P production

# Run in offline mode
mvn package -o

# Show dependency tree
mvn dependency:tree

# Show effective POM
mvn help:effective-pom

# Show plugin information
mvn help:describe -Dplugin=compiler
```

### Advanced Commands
```bash
# Generate project site
mvn site

# Generate source JAR
mvn source:jar

# Generate Javadoc JAR
mvn javadoc:jar

# Sign artifacts
mvn gpg:sign

# Verify signatures
mvn gpg:verify

# Check for dependency updates
mvn versions:display-dependency-updates

# Update parent version
mvn versions:update-parent

# Update plugin versions
mvn versions:update-plugin-versions

# Update project version
mvn versions:set -DnewVersion=1.0.1

# Resolve dependencies
mvn dependency:resolve

# Copy dependencies
mvn dependency:copy-dependencies

# Analyze dependencies
mvn dependency:analyze

# Find unused dependencies
mvn dependency:analyze-duplicate

# Check for vulnerabilities
mvn org.owasp:dependency-check-maven:check
```

## Maven Dependencies

### Dependency Scopes
- **compile**: Available in all classpaths (default)
- **provided**: Available in compilation and test, but not runtime
- **runtime**: Available in runtime and test, but not compilation
- **test**: Available only in test compilation and execution
- **system**: Similar to provided but must be explicitly provided
- **import**: Only used in dependencyManagement of type pom

### Dependency Management
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.6.3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### Excluding Dependencies
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### Optional Dependencies
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
    <optional>true</optional>
</dependency>
```

## Maven Plugins

### Common Plugins

#### Maven Compiler Plugin
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <source>11</source>
        <target>11</target>
        <encoding>UTF-8</encoding>
        <compilerArgs>
            <arg>-Xlint:unchecked</arg>
            <arg>-parameters</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

#### Maven Surefire Plugin
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.0.0-M5</version>
    <configuration>
        <includes>
            <include>**/*Test.java</include>
        </includes>
        <excludes>
            <exclude>**/*IntegrationTest.java</exclude>
        </excludes>
        <parallel>methods</parallel>
        <threadCount>4</threadCount>
    </configuration>
</plugin>
```

#### Maven JAR Plugin
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>com.example.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

#### Maven WAR Plugin
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.3.2</version>
    <configuration>
        <webResources>
            <resource>
                <directory>src/main/webapp</directory>
                <filtering>true</filtering>
            </resource>
        </webResources>
    </configuration>
</plugin>
```

#### Maven Assembly Plugin
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
            <manifest>
                <mainClass>com.example.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## Maven Repositories

### Repository Types
- **Local Repository**: Local cache on your machine
- **Central Repository**: Maven's default repository
- **Remote Repository**: Third-party repositories
- **Mirror Repository**: Mirror of another repository

### Repository Configuration
```xml
<repositories>
    <repository>
        <id>central</id>
        <url>https://repo.maven.apache.org/maven2</url>
    </repository>
    
    <repository>
        <id>spring-milestone</id>
        <url>https://repo.spring.io/milestone</url>
    </repository>
    
    <repository>
        <id>jboss-public-repository</id>
        <url>https://repository.jboss.org/nexus/content/groups/public</url>
    </repository>
</repositories>

<pluginRepositories>
    <pluginRepository>
        <id>spring-milestone</id>
        <url>https://repo.spring.io/milestone</url>
    </pluginRepository>
</pluginRepositories>
```

### Mirror Configuration
```xml
<mirrors>
    <mirror>
        <id>aliyun-maven</id>
        <mirrorOf>central</mirrorOf>
        <url>https://maven.aliyun.com/repository/central</url>
    </mirror>
</mirrors>
```

## Maven Profiles

### Profile Configuration
```xml
<profiles>
    <profile>
        <id>development</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <environment>dev</environment>
            <database.url>jdbc:h2:mem:testdb</database.url>
        </properties>
    </profile>
    
    <profile>
        <id>production</id>
        <properties>
            <environment>prod</environment>
            <database.url>jdbc:mysql://prod-db:3306/mydb</database.url>
        </properties>
    </profile>
    
    <profile>
        <id>test</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <includes>
                            <include>**/*Test.java</include>
                            <include>**/*IntegrationTest.java</include>
                        </includes>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

## Maven Multi-Module Projects

### Parent POM
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>my-project-parent</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>core</module>
        <module>web</module>
        <module>api</module>
    </modules>
    
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.6.3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
    
</project>
```

### Child Module POM
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>com.example</groupId>
        <artifactId>my-project-parent</artifactId>
        <version>1.0.0</version>
    </parent>
    
    <artifactId>my-project-core</artifactId>
    <packaging>jar</packaging>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>
    
</project>
```

## Maven Settings

### settings.xml Configuration
```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
          http://maven.apache.org/xsd/settings-1.0.0.xsd">
    
    <localRepository>${user.home}/.m2/repository</localRepository>
    
    <interactiveMode>true</interactiveMode>
    <offline>false</offline>
    
    <mirrors>
        <mirror>
            <id>aliyun-maven</id>
            <mirrorOf>central</mirrorOf>
            <url>https://maven.aliyun.com/repository/central</url>
        </mirror>
    </mirrors>
    
    <proxies>
        <proxy>
            <id>my-proxy</id>
            <active>true</active>
            <protocol>http</protocol>
            <host>proxy.example.com</host>
            <port>8080</port>
            <username>proxyuser</username>
            <password>proxypass</password>
        </proxy>
    </proxies>
    
    <servers>
        <server>
            <id>my-server</id>
            <username>serveruser</username>
            <password>serverpass</password>
        </server>
    </servers>
    
    <profiles>
        <profile>
            <id>my-profile</id>
            <properties>
                <my.property>value</my.property>
            </properties>
        </profile>
    </profiles>
    
    <activeProfiles>
        <activeProfile>my-profile</activeProfile>
    </activeProfiles>
    
</settings>
```

## Maven Integration with IDEs

### IntelliJ IDEA
- **Import Maven Project**: File → Open → Select pom.xml
- **Maven Tool Window**: View → Tool Windows → Maven
- **Run Maven Goals**: Right-click on project → Maven → [Goal]

### Eclipse
- **Install m2e Plugin**: Help → Eclipse Marketplace → Install m2e
- **Import Maven Project**: File → Import → Maven → Existing Maven Projects
- **Run Maven Goals**: Right-click on project → Run As → Maven Build

### VS Code
- **Install Maven Extension**: Extensions → Search for "Maven for Java"
- **Open Maven Project**: File → Open Folder → Select project directory
- **Run Maven Goals**: Command Palette → Maven: Execute Command

## Interview Questions

### Beginner Level
1. **What is Maven?**
   - Maven is a build automation tool used primarily for Java projects that manages project builds, dependencies, and documentation

2. **What is a POM file?**
   - POM (Project Object Model) is an XML file that contains project configuration and dependencies

3. **What is Maven's default directory structure?**
   - src/main/java, src/main/resources, src/test/java, src/test/resources

4. **What is Maven's build lifecycle?**
   - Clean, Default, and Site lifecycles with phases like compile, test, package, install, deploy

### Intermediate Level
1. **What is the difference between compile and provided scope?**
   - Compile scope is available in all classpaths, provided scope is available in compilation and test but not runtime

2. **What is a Maven repository?**
   - A repository is a directory where Maven stores dependencies and plugins

3. **What are Maven profiles?**
   - Profiles allow different build configurations for different environments

4. **What is dependency management in Maven?**
   - Dependency management allows centralized control over dependency versions

### Advanced Level
1. **What is the difference between Maven and Gradle?**
   - Maven uses XML configuration and has a rigid structure, while Gradle uses Groovy/Kotlin DSL and is more flexible

2. **How does Maven resolve dependencies?**
   - Maven resolves dependencies by checking local repository first, then remote repositories

3. **What is a multi-module project in Maven?**
   - A multi-module project consists of a parent POM and multiple child modules

4. **How do you create a custom Maven plugin?**
   - Create a Maven project with packaging type 'maven-plugin' and extend AbstractMojo

## Best Practices

### POM Configuration
- **Use Properties**: Define version numbers as properties
- **Dependency Management**: Use dependencyManagement for version control
- **Plugin Management**: Use pluginManagement for plugin version control
- **Profiles**: Use profiles for environment-specific configurations
- **Version Control**: Keep POM files in version control

### Build Optimization
- **Parallel Builds**: Use parallel builds for multi-module projects
- **Incremental Builds**: Use incremental builds to save time
- **Dependency Optimization**: Remove unused dependencies
- **Build Profiles**: Use profiles for different build environments

### Security
- **Secure Repositories**: Use secure repositories for dependencies
- **Dependency Scanning**: Scan dependencies for vulnerabilities
- **Version Management**: Keep dependencies updated
- **Private Repositories**: Use private repositories for internal dependencies

## Troubleshooting

### Common Issues
```bash
# Check Maven version
mvn -version

# Check Java version
java -version

# Check JAVA_HOME
echo $JAVA_HOME

# Clean and rebuild
mvn clean install

# Check dependency tree
mvn dependency:tree

# Run in debug mode
mvn install -X

# Run with offline mode
mvn install -o

# Check effective POM
mvn help:effective-pom

# Check plugin information
mvn help:describe -Dplugin=compiler
```

### Dependency Issues
```bash
# Resolve dependency conflicts
mvn dependency:tree -Dverbose

# Find unused dependencies
mvn dependency:analyze

# Copy dependencies
mvn dependency:copy-dependencies

# Resolve dependencies
mvn dependency:resolve

# Check for dependency updates
mvn versions:display-dependency-updates
```

## Resources

### Official Documentation
- [Maven Documentation](https://maven.apache.org/guides/)
- [Maven POM Reference](https://maven.apache.org/pom.html)
- [Maven Plugins](https://maven.apache.org/plugins/)

### Learning Resources
- [Maven Tutorial](https://maven.apache.org/guides/getting-started/)
- [Maven Best Practices](https://maven.apache.org/guides/mini/guide-best-practices.html)
- [Maven Settings Reference](https://maven.apache.org/settings.html)

### Community
- [Maven Mailing Lists](https://maven.apache.org/mailing-lists.html)
- [Maven Stack Overflow](https://stackoverflow.com/questions/tagged/maven)
- [Maven GitHub](https://github.com/apache/maven)