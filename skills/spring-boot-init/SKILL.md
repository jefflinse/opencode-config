---
name: spring-boot-init
description: Standard Spring Boot project scaffolding with Maven/Gradle setup, application configuration, security, JPA, CI pipeline, and Dockerfile
---

## What This Skill Does

Guides scaffolding a new Spring Boot project with a production-ready structure. Covers project structure, build system configuration, application properties, security setup, database configuration, testing setup, containerization, and CI pipeline.

## When To Use

Use this when starting a new Spring Boot application from scratch — a REST API, web application, or microservice. Do not use this to restructure an existing project.

## Workflow

### Phase 1: Gather Requirements
**Agent**: planner

Collect the information needed to scaffold correctly:
1. Java version (17, 21, etc.)
2. Spring Boot version (3.x)
3. Build system: Maven or Gradle (Kotlin DSL)
4. Project type: REST API, web app with templates, or microservice
5. Database: PostgreSQL, MySQL, or none
6. Required Spring Boot starters: web, data-jpa, security, actuator, validation, etc.
7. CI platform: GitHub Actions, GitLab CI, or other
8. Whether the organization has existing projects with conventions to match

**Gate**: Requirements must be confirmed before scaffolding begins.

### Phase 2: Scaffold Project Structure
**Agent**: java-builder

Create the project layout:

```
.
├── src/
│   ├── main/
│   │   ├── java/com/example/<project>/
│   │   │   ├── Application.java            # Main class
│   │   │   ├── config/                      # Configuration classes
│   │   │   ├── <domain>/                    # Feature packages
│   │   │   │   ├── <Domain>Controller.java
│   │   │   │   ├── <Domain>Service.java
│   │   │   │   ├── <Domain>Repository.java
│   │   │   │   ├── <Domain>.java            # Entity
│   │   │   │   └── <Domain>DTO.java
│   │   │   └── common/                      # Shared utilities
│   │   │       ├── exception/               # Global exception handling
│   │   │       └── security/                # Security configuration
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── db/migration/                # Flyway migrations
│   └── test/
│       └── java/com/example/<project>/
├── .gitignore
├── pom.xml (or build.gradle.kts)
├── Dockerfile
└── README.md
```

**Important**: Research current Spring Boot starter versions, dependency BOM versions, and Maven/Gradle plugin versions before writing files.

**Acceptance criteria**: `mvn clean compile` (or `./gradlew build`) succeeds.

### Phase 3: Application Configuration
**Agent**: java-builder

Create `application.yml` with:
1. Server configuration (port, context path)
2. Database connection (with environment variable placeholders)
3. JPA/Hibernate settings (ddl-auto: validate for production, create-drop for tests)
4. Logging configuration (structured JSON logging for production)
5. Actuator endpoints (health, info, metrics, prometheus)
6. Spring profiles for dev/prod separation

### Phase 4: Security Setup (if required)
**Agent**: java-builder

1. `SecurityFilterChain` bean with sensible defaults
2. CORS configuration
3. CSRF configuration appropriate for the API type
4. `PasswordEncoder` bean (BCrypt)
5. Basic authentication or JWT setup depending on requirements

### Phase 5: Build Tooling
**Agent**: ci-ops

1. **Dockerfile**: Multi-stage build (JDK for compilation, JRE for runtime)
2. **docker-compose.yml**: Application + database for local development
3. **Maven/Gradle wrapper**: Committed to the repository
4. **.editorconfig**: Consistent formatting across IDEs

**Important**: Research current JDK/JRE base image tags before writing Dockerfile.

### Phase 6: CI Pipeline
**Agent**: ci-ops

Create GitHub Actions workflow (`.github/workflows/ci.yml`):
1. Setup Java with the project's JDK version
2. Cache Maven/Gradle dependencies
3. Build: `mvn clean verify` or `./gradlew build`
4. Test results: publish JUnit test reports
5. Pin all action versions to specific SHAs

### Phase 7: Documentation
**Agent**: docs-writer

Create `README.md` with:
1. One-line project description
2. Prerequisites (Java version, Maven/Gradle)
3. Quick-start setup instructions
4. Running locally (with docker-compose for dependencies)
5. API documentation endpoint (Swagger UI if configured)
6. Configuration options (environment variables)

### Phase 8: Review
**Agent**: code-reviewer

1. Verify consistency across all generated files
2. Application compiles and tests pass
3. No hardcoded secrets or placeholder values
4. Security configuration follows Spring Security best practices
5. Database configuration is safe for production (no `ddl-auto: create`)

**Gate**: Present review findings to the user.

## Reference: Maven POM Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
    https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version><!-- RESEARCH CURRENT VERSION --></version>
    </parent>

    <groupId>com.example</groupId>
    <artifactId><!-- project name --></artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <java.version><!-- 17 or 21 --></java.version>
    </properties>

    <dependencies>
        <!-- Add starters based on requirements -->
    </dependencies>
</project>
```

## Principles

- **Security from day one**: Include Spring Security even for internal services. It's much harder to add later.
- **Profiles for environments**: Never put production configuration in `application.yml`. Use profiles.
- **Flyway for migrations**: Never rely on Hibernate's `ddl-auto` for schema management beyond local development.
- **Research current versions**: Spring Boot, starter dependencies, and Maven plugins change frequently. Always verify.
- **Constructor injection**: All beans use constructor injection, never `@Autowired` on fields.
