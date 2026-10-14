# vProfile Application Repository

A multi-tier Java Spring MVC application built with Maven, containerized with Docker, and delivered via GitHub Actions CI/CD to Amazon ECR and a GitOps Helm repository.

## Tech Stack

- **Language**: Java 21 (Temurin JDK)
- **Framework**: Spring MVC
- **Build Tool**: Apache Maven
- **Web Server**: Apache Tomcat 10
- **Database**: MySQL 8
- **Caching**: Memcached
- **Messaging**: RabbitMQ
- **Code Quality**: SonarQube & Checkstyle

## Continuous Integration & Delivery Pipeline

The `.github/workflows/ci.yml` pipeline defines a two-phase delivery workflow:

### Phase 1: Pull Request to `main`
- Checks out code with full history for Sonar blame tracking
- Sets up JDK 21 and caches Maven dependencies
- Runs `mvn clean verify` (compilation, unit tests, Checkstyle linting, JaCoCo coverage)
- Executes SonarQube scanner against self-hosted SonarQube
- Evaluates Quality Gate (`-Dsonar.qualitygate.wait=true`), blocking PR merge if checks fail

### Phase 2: Push / Merge to `main`
- Derives an immutable image tag from the first 7 characters of the Git commit SHA
- Builds multi-stage Docker image (`Docker-files/app/multistage/Dockerfile`)
- Pushes both `:sha` and `:latest` tags to Amazon ECR
- Clones the `vprofile-helm` GitOps repository
- Uses `yq` to surgically update `app.image` and `app.tag` in `values.yaml`
- Commits and pushes the update to `vprofile-helm` with `[skip ci]`, triggering ArgoCD synchronization
