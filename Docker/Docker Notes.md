
# What is Docker?

#### Definition

**Docker is a containerization platform used to package an application and its dependencies into a portable container image and run it consistently across environments.**

The problem Docker solves:

```text
Application
   +
Java
   +
Libraries
   +
Runtime configuration
   +
System dependencies
        ↓
   Docker Image
        ↓
   Container
```

Instead of saying:

> "It works on my machine."

We package the application environment so it can run consistently elsewhere.

#### Interview Answer

> Docker packages an application and its dependencies into a container image, providing a consistent runtime environment across development, testing, and production.

### What is a Docker container
A Docker container is a lightweight, isolated runtime environment that runs an application together with its dependencies. Containers are created from Docker images and share the host OS kernel, unlike virtual machines which include a separate guest operating system. Docker uses Linux kernel features such as namespaces and cgroups to provide isolation and resource control.

---

## 2. Docker vs Virtual Machine

### VM

```text
Hardware
   ↓
Host OS
   ↓
Hypervisor
   ↓
Guest OS
   ↓
Application
```

Each VM normally has its own guest operating system.

### Container

```text
Hardware
   ↓
Host OS
   ↓
Container Runtime
   ↓
Container
   ↓
Application
```

Containers share the host kernel rather than running a complete guest OS.

### Comparison

| VM | Container |
|---|---|
| Virtualizes a complete machine | Provides process-level isolation |
| Has a guest OS | Shares host kernel |
| Heavier | Lightweight |
| Slower startup | Faster startup |
| More resource usage | Generally more resource efficient |

#### Interview Answer

> VMs virtualize an entire machine including a guest operating system, whereas containers provide process-level isolation while sharing the host kernel. That's why containers are generally lighter and faster to start.

---

## 3. Hypervisor vs Docker Engine

### Hypervisor

A **hypervisor** creates and manages virtual machines.

```text
Hardware
   ↓
Host OS
   ↓
Hypervisor
   ↓
Guest OS
   ↓
Application
```

Examples:

- VMware
- VirtualBox
- Hyper-V

The hypervisor provides each VM with virtualized hardware.

### Docker Engine / Container Runtime

Docker provides the tooling and runtime needed to build and run containers.

```text
Host OS
   ↓
Docker Engine / container runtime
   ↓
Container
   ↓
Application
```

The container does not normally need its own complete guest OS.

#### Interview Point

> A hypervisor virtualizes machines; Docker/container runtimes isolate application processes using OS-level mechanisms.

---

## 4. Docker Image vs Container

### Image

#### Definition

> A Docker image is an immutable package containing an application, its dependencies, and the filesystem/runtime information needed to create a container.

Think:

```text
Image = Blueprint / Template
```

Example:

```text
my-spring-app:1.0
```

### Container

#### Definition

> A container is a running or stopped instance created from a Docker image.

Think:

```text
Image
  ↓
Container
```

One image can create multiple containers:

```text
my-app:1.0
    ↓
 ┌──┼──┐
 ↓  ↓  ↓
C1  C2  C3
```

---

## 5. Dockerfile

#### Definition

> A Dockerfile is a text file containing instructions used by Docker to build a container image.

Example:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Important Dockerfile Instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Selects the base image |
| `WORKDIR` | Sets working directory |
| `COPY` | Copies files from build context |
| `RUN` | Executes command during image build |
| `EXPOSE` | Documents intended container port |
| `ENV` | Defines environment variable |
| `ARG` | Defines build-time argument |
| `CMD` | Default command/arguments |
| `ENTRYPOINT` | Main executable for the container |
| `USER` | Specifies user to run as |

---

## 6. `FROM`

```dockerfile
FROM eclipse-temurin:21-jre
```

#### Definition

> `FROM` specifies the base image from which the new image is built.

For a Spring Boot application:

```text
Java 21 JRE base image
       +
Spring Boot JAR
       ↓
Application Image
```

---

## 7. `WORKDIR`

```dockerfile
WORKDIR /app
```

Sets the working directory for subsequent Dockerfile instructions and the container process.

Instead of:

```text
/
```

the application works from:

```text
/app
```

---

## 8. `COPY`

```dockerfile
COPY target/app.jar app.jar
```

Copies:

```text
Build context:
target/app.jar

        ↓

Container image:
/app/app.jar
```

because:

```dockerfile
WORKDIR /app
```

was specified.

---

## 9. `RUN`

#### Definition

> `RUN` executes a command while building the image.

Example:

```dockerfile
RUN mvn clean package
```

This happens during:

```bash
docker build
```

It does **not** happen every time the container starts if the resulting layer is already part of the image.

---

## 10. `EXPOSE`

```dockerfile
EXPOSE 8080
```

#### Definition

> `EXPOSE` documents the port on which the containerized application is expected to listen.

Important:

**`EXPOSE` does not publish the port to the host.**

You still need:

```bash
docker run -p 8080:8080 my-app
```

---

## 11. Port Mapping

```bash
docker run -p 8081:8080 my-app
```

Means:

```text
Host              Container
8081       ───→   8080
```

If Spring Boot listens on container port `8080`, access it through:

```text
http://localhost:8081
```

#### Interview Trap

`EXPOSE 8080` does not mean the application is automatically available on `localhost:8080`.

---

## 12. `ENTRYPOINT` vs `CMD`

Example:

```dockerfile
ENTRYPOINT ["java", "-jar"]
CMD ["app.jar"]
```

Conceptually:

```text
ENTRYPOINT = main executable
CMD        = default arguments
```

Another common Spring Boot example:

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### Interview Answer

> `ENTRYPOINT` defines the main executable of the container, while `CMD` provides default arguments or a default command that can be overridden.

---

## 13. Build Context

When you run:

```bash
docker build -t my-app .
```

the final `.` means:

> Use the current directory as the Docker build context.

Docker can only `COPY` files available in the build context.

Example:

```text
project/
├── Dockerfile
├── pom.xml
├── src/
└── target/
```

If context is:

```bash
docker build .
```

Docker can access those files for `COPY`.

---

## 14. `.dockerignore`

#### Definition

> `.dockerignore` excludes files/directories from the Docker build context.

Example:

```text
.git
.idea
target
node_modules
*.log
.env
*.pem
credentials.json
```

Benefits:

- Smaller build context
- Faster builds
- Avoid accidental inclusion of unnecessary/sensitive files

---

## 15. Building a Spring Boot Image

If the JAR already exists:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/app.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build:

```bash
docker build -t my-app:1.0 .
```

Run:

```bash
docker run -p 8080:8080 my-app:1.0
```

---

## 16. Maven Inside Docker

A Dockerfile can build the application itself.

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-21

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "target/app.jar"]
```

However, this image contains Maven/JDK/build tools and is not ideal as a final production runtime image.

For production, use a multi-stage build.

---

## 17. Multi-stage Builds

#### Definition

> A multi-stage Docker build uses multiple `FROM` stages so build tools can be kept out of the final runtime image.

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn clean package -DskipTests


FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Build stage

Contains:

- Maven
- JDK
- Source code
- Build dependencies

### Runtime stage

Contains:

- JRE
- Application JAR

#### Benefit

The final image is smaller and has a smaller attack surface.

---

## 18. Docker `ARG`

#### Definition

> `ARG` defines a build-time variable.

Example:

```dockerfile
ARG APP_VERSION
RUN echo "Building version $APP_VERSION"
```

Build:

```bash
docker build --build-arg APP_VERSION=1.2.0 .
```

The value is available during image building.

```text
--build-arg
      ↓
Docker build
      ↓
ARG
      ↓
Build process
```

Do not use `ARG` as a secure mechanism for secrets.

---

## 19. Docker `ENV`

#### Definition

> `ENV` defines an environment variable that is available to the image/container.

Example:

```dockerfile
ENV APP_MODE=prod
```

You can override/provide values at runtime:

```bash
docker run -e APP_MODE=dev my-app
```

### ARG vs ENV

| ARG | ENV |
|---|---|
| Build-time | Runtime |
| `--build-arg` | `-e` |
| Used while building | Available to application/container |

---

## 20. Docker Image Layers

#### Definition

> Docker images are built from layers representing filesystem changes produced during the image build.

Conceptually:

```text
┌────────────────────────────┐
│ Application JAR            │
├────────────────────────────┤
│ Runtime                    │
├────────────────────────────┤
│ Base image                 │
└────────────────────────────┘
```

Layers can be reused between builds/images.

---

## 21. Docker Layer Caching

#### Definition

> Docker can reuse previously built results when a Dockerfile instruction and its relevant inputs have not changed.

Example:

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn clean package
```

First build:

```text
COPY pom.xml          → build
Download dependencies → build
COPY src              → build
Maven package         → build
```

Change only Java source:

```text
COPY pom.xml          → CACHED
Dependencies          → CACHED
COPY src              → CACHE MISS
Maven package         → rebuild
```

### Why?

Docker evaluates the Dockerfile sequentially and checks whether it can reuse a cached result based on the instruction and relevant build inputs/state.

For `COPY`, the contents/metadata of the files being copied are relevant.

Once a cache miss occurs, subsequent dependent layers generally need to be rebuilt.

#### Optimization Rule

> Put instructions that change less frequently before instructions that change frequently.

Good:

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
```

Bad:

```dockerfile
COPY . .
RUN mvn dependency:go-offline
```

With the second version, changing a Java source file can invalidate the layer containing the whole project before dependency caching.

---

## 22. Where is Docker Build Cache Stored?

Docker manages build cache in its own Docker storage/build system.

On Docker Desktop, Docker manages this inside its Docker environment/VM rather than as a normal project folder.

Inspect:

```bash
docker builder du
```

Clean unused build cache:

```bash
docker builder prune
```

---

## 23. Docker Image Optimization

#### Main techniques

1. Multi-stage builds
2. Use an appropriate minimal runtime image
3. Use JRE instead of JDK in runtime stage
4. Optimize layer ordering for caching
5. Use `.dockerignore`
6. Avoid unnecessary packages
7. Use appropriate/versioned base images

#### Interview Answer

> I'd use a multi-stage build so Maven and the JDK remain in the build stage and only the application artifact is copied into a minimal runtime image. I'd place stable files such as `pom.xml` before frequently changing source code to maximize layer-cache reuse, and I'd use `.dockerignore` to exclude unnecessary files.

---

## 24. Docker Volumes

#### Definition

> A Docker volume is Docker-managed persistent storage that allows data to survive beyond the lifecycle of a container.

Without a volume:

```text
Container
   ↓
Data
   ↓
Container deleted
   ↓
Data may be lost
```

With a volume:

```text
Container
   ↓
Volume
   ↓
Persistent data
```

Create:

```bash
docker volume create postgres-data
```

Use:

```bash
docker run \
  -v postgres-data:/var/lib/postgresql/data \
  postgres
```

---

## 25. Bind Mount

#### Definition

> A bind mount maps a specific host file/directory directly into a container.

Example:

```bash
docker run \
  -v /host/config:/app/config \
  my-app
```

Conceptually:

```text
Host directory
      ↓
/host/config
      │
      ↓
Container
/app/config
```

### Volume vs Bind Mount

| Volume | Bind Mount |
|---|---|
| Managed by Docker | Host path is explicitly specified |
| Good for persistent application data | Good for local development/config/files |
| Less dependent on host path structure | More host-dependent |

---

## 26. Docker Compose

#### Definition

> Docker Compose is a tool for defining and running multi-container Docker applications using a YAML configuration file.

Example:

```yaml
services:

  app:
    image: my-spring-app:1.0
    ports:
      - "8080:8080"

  redis:
    image: redis:7
```

Start:

```bash
docker compose up
```

Background:

```bash
docker compose up -d
```

Build images:

```bash
docker compose build
```

Build and start:

```bash
docker compose up --build
```

Stop/remove containers:

```bash
docker compose down
```

---

## 27. Compose `build` vs `image`

```yaml
services:
  app:
    build: .
```

Means Compose should build the image using the specified build context.

```yaml
services:
  redis:
    image: redis:7
```

Means use an existing image.

So:

```bash
docker compose build
```

builds/rebuilds services that have `build:` configuration.

It does not build services that only specify `image:`.

---

## 28. Compose Service Names and DNS

Example:

```yaml
services:

  app:
    image: my-app

  postgres:
    image: postgres:17
```

Inside the Compose network, the application can connect to:

```text
postgres:5432
```

not:

```text
localhost:5432
```

Because:

```text
postgres
   ↓
Compose service name
   ↓
Docker DNS
   ↓
PostgreSQL container
```

Important:

> `localhost` inside a container refers to that container itself.

---

## 29. Compose Volumes

Example:

```yaml
services:

  postgres:
    image: postgres:17
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

The service-level entry:

```yaml
volumes:
  - postgres-data:/var/lib/postgresql/data
```

mounts the volume.

The top-level:

```yaml
volumes:
  postgres-data:
```

declares the named volume for Compose to manage.

---

## 30. `depends_on`

Example:

```yaml
services:

  app:
    image: my-app
    depends_on:
      - postgres

  postgres:
    image: postgres:17
```

It controls startup ordering.

Important:

> `depends_on` does not by itself guarantee that PostgreSQL is ready to accept connections.

Container started != application ready.

Health checks/readiness mechanisms are needed when readiness matters.

---

## 31. Docker Registry

#### Definition

> A Docker/container registry is a service used to store, manage, and distribute container images.

Examples:

- Docker Hub
- Amazon ECR
- GitHub Container Registry

Flow:

```text
Docker Image
    ↓
Registry
    ↓
Other machine
    ↓
docker pull
```

---

## 32. Amazon ECR

#### Definition

> Amazon Elastic Container Registry (ECR) is AWS's managed container registry for storing and distributing container images.

Typical AWS flow:

```text
Developer / CI
      ↓
docker build
      ↓
Docker Image
      ↓
ECR
      ↓
ECS / EKS
      ↓
Container
```

Typical image:

```text
<account>.dkr.ecr.<region>.amazonaws.com/order-service:1.0.0
```

---

## 33. Docker + CI/CD

Typical pipeline:

```text
Git Push
   ↓
CI/CD
   ↓
Maven Test
   ↓
Maven Package
   ↓
Docker Build
   ↓
Docker Image
   ↓
Tag
   ↓
Push to ECR
   ↓
Deploy
   ↓
ECS / EKS
```

#### Key Principle

> Build once, deploy the same image across environments.

Don't rebuild a different image separately for development, QA, and production if the goal is an immutable artifact.

Environment-specific configuration should be supplied separately through configuration/secrets.

---

## 34. Image Tags

Examples:

```text
order-service:latest
order-service:1.2.0
order-service:a81f92c
```

For production, version/commit-based tags are generally preferable to relying only on `latest`.

Benefits:

- Traceability
- Easier rollback
- Easier debugging
- Clear deployment version

---

## 35. ECS

#### Definition

> Amazon ECS (Elastic Container Service) is AWS's managed container orchestration service used to deploy and manage containerized applications.

Docker:

```text
Build/run containers
```

ECS:

```text
Deploy/manage containers
Scale
Replace failed tasks
Manage services
Integrate with AWS
```

---

## 36. ECS Core Components

```text
ECS Cluster
     ↓
ECS Service
     ↓
Task Definition
     ↓
Task
     ↓
Container
```

### Cluster

Logical grouping/environment where ECS workloads run.

### Task Definition

#### Definition

> A blueprint describing how ECS should run one or more containers.

Can contain:

- Image
- CPU
- Memory
- Container ports
- Environment variables
- Secrets
- IAM task role
- Logging
- Health checks

### Task

#### Definition

> A running instance of a task definition.

### Service

#### Definition

> An ECS service maintains the desired number of running tasks and manages their deployment/replacement.

Example:

```text
Desired count = 3

Task 1
Task 2
Task 3
```

If one crashes, ECS can launch a replacement to maintain the desired count.

---

## 37. ECS EC2 vs Fargate

### ECS on EC2

```text
AWS
 ↓
EC2 instances
 ↓
ECS
 ↓
Containers
```

You manage the underlying EC2 capacity.

### Fargate

```text
AWS
 ↓
Fargate
 ↓
ECS Task
 ↓
Container
```

AWS manages the underlying compute infrastructure.

#### Interview Answer

> With ECS on EC2, we manage the underlying EC2 capacity. With Fargate, AWS manages the underlying compute infrastructure and we specify task-level resources such as CPU and memory.

---

## 38. ECS + ALB

Typical architecture:

```text
Client
  ↓
ALB
  ↓
ECS Service
  ↓
┌──────┬──────┬──────┐
Task 1 Task 2 Task 3
```

The ALB distributes traffic to healthy tasks.

Possible architecture:

```text
Client
  ↓
API Gateway
  ↓
ALB
  ↓
ECS Service
  ↓
Spring Boot Tasks
```

The exact architecture depends on the application.

---

## 39. Docker Health Checks

#### Definition

> A health check determines whether the application inside a running container is actually functioning, rather than only checking whether its process is alive.

Running:

```text
Container = running
```

does not necessarily mean:

```text
Application = healthy
```

Example Dockerfile:

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
```

Important practical point:

> The runtime image must contain `curl` if the health check uses `curl`.

---

## 40. Spring Boot Health Checks

Spring Boot Actuator commonly exposes:

```text
/actuator/health
```

A health check can use this endpoint:

```text
Docker/ECS/ALB
       ↓
/actuator/health
       ↓
Spring Boot
```

---

## 41. Container Health vs ALB Health

These are separate mechanisms.

### Container health

Configured through the container/task configuration and checks application/container health.

### ALB health

The Application Load Balancer checks whether a target should receive traffic.

```text
ALB
 ↓
GET /actuator/health
 ↓
Container
```

A target can be removed from traffic when the ALB considers it unhealthy.

#### Important

> Docker `HEALTHCHECK` and ALB health checks are not the same mechanism.

---

## 42. Running vs Healthy

```text
RUNNING
→ Main process is alive

HEALTHY
→ Configured health check is passing

ALB HEALTHY
→ Load balancer considers target eligible for traffic
```

A Docker health check reporting `unhealthy` does not by itself mean Docker automatically restarts the container. Restart/replacement behavior depends on the runtime/orchestrator configuration.

---

## 43. Docker Security

### 1. Don't run as root

Use:

```dockerfile
USER appuser
```

Example:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY app.jar app.jar

RUN useradd -r appuser

USER appuser

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 2. Don't bake secrets into images

Avoid:

```dockerfile
ENV DB_PASSWORD=secret
```

Prefer runtime secret injection through:

- AWS Secrets Manager
- ECS secrets
- Kubernetes Secrets
- Other secret management systems

### 3. Use trusted/minimal base images

Fewer unnecessary packages generally means:

```text
Smaller image
+
Smaller attack surface
+
Fewer potential vulnerabilities
```

### 4. Scan images

Common tools/services:

- Trivy
- Grype
- ECR image scanning

### 5. Prefer versioned image tags

Avoid relying only on:

```text
latest
```

Prefer:

```text
1.2.3
```

or a commit identifier.

---

## 44. Docker Troubleshooting

### First Principle

Troubleshoot by layers:

```text
Image
 ↓
Container
 ↓
Process
 ↓
Application
 ↓
Network
 ↓
Dependency
```

---

### Scenario: Container exits immediately

Check:

```bash
docker ps -a
docker logs <container>
```

Possible causes:

- Application startup failure
- Missing JAR
- Wrong entrypoint
- Invalid configuration
- Missing environment variable

---

### Scenario: Container running but API inaccessible

Check:

```bash
docker ps
```

Verify:

```text
Host port → Container port
```

Example:

```bash
docker run -p 8081:8080 my-app
```

means:

```text
localhost:8081 → container:8080
```

---

### Scenario: Spring Boot cannot connect to PostgreSQL

Check:

- Is PostgreSQL running?
- Are containers on the same network?
- Is hostname correct?
- Is port correct?
- Are credentials correct?
- Is database ready?

In Compose, use:

```text
postgres:5432
```

not:

```text
localhost:5432
```

---

### Scenario: Container keeps restarting

Check:

```bash
docker logs <container>
docker inspect <container>
```

Common causes:

- Startup exception
- Bad configuration
- Missing environment variable
- Database connection failure
- Wrong command
- Missing application artifact

---

### Scenario: Port already in use

Error:

```text
port is already allocated
```

Use another host port:

```bash
docker run -p 8081:8080 my-app
```

or identify the process using the host port.

---

### Scenario: Need to inspect a running container

```bash
docker exec -it <container> /bin/sh
```

Then inspect:

```bash
env
ls
```

depending on what tools exist in the image.

---

### Scenario: Need detailed container configuration

```bash
docker inspect <container>
```

Useful for:

- Environment variables
- Network
- Mounts
- Ports
- Image
- Entrypoint

---

### Scenario: Need resource usage

```bash
docker stats
```

Shows:

- CPU
- Memory
- Network I/O
- Block I/O

---

## 45. High-Value Docker Commands

```bash
# Images
docker images
docker build -t my-app:1.0 .
docker pull image:tag
docker push image:tag

# Containers
docker run -p 8080:8080 my-app
docker ps
docker ps -a
docker start <container>
docker stop <container>
docker restart <container>
docker rm <container>

# Logs / inspection
docker logs <container>
docker logs -f <container>
docker exec -it <container> /bin/sh
docker inspect <container>
docker stats

# Networks
docker network ls
docker network inspect <network>

# Volumes
docker volume ls
docker volume create <name>
docker volume inspect <name>
docker volume rm <name>

# Compose
docker compose up
docker compose up -d
docker compose up --build
docker compose build
docker compose ps
docker compose logs
docker compose down
docker compose down -v

# Build cache
docker builder du
docker builder prune
```

---

## 46. Docker + Kubernetes/EKS

Kubernetes is a container orchestration platform.

Docker:

```text
Build/package/run containers
```

Kubernetes:

```text
Deploy
Scale
Network
Self-heal
Roll out
Manage containers
```

#### Important modern detail

Kubernetes does not require Docker Engine as its container runtime.

Modern Kubernetes commonly uses a CRI-compatible runtime such as:

```text
containerd
```

Docker-built OCI-compatible images can still be used.

---

## 47. Docker Image → EKS

Flow:

```text
Dockerfile
    ↓
Docker Image
    ↓
ECR
    ↓
Kubernetes Deployment
    ↓
Pod
    ↓
Container
    ↓
Spring Boot
```

A Deployment might reference:

```yaml
image: <account>.dkr.ecr.<region>.amazonaws.com/order-service:1.2.0
```

The Kubernetes node's container runtime pulls the image from ECR and starts the container.

---

## 48. Kubernetes Pod

#### Definition

> A Pod is Kubernetes' smallest deployable unit and can contain one or more containers that share networking and storage resources.

Typical Spring Boot case:

```text
Pod
└── Spring Boot Container
```

Do not say:

```text
Pod = Container
```

A Pod contains one or more containers.

---

## 49. Docker Compose vs ECS vs Kubernetes

| Technology | Main Purpose |
|---|---|
| Docker | Build/package/run containers |
| Docker Compose | Run multi-container applications, especially locally |
| ECR | Store container images |
| ECS | AWS container orchestration |
| Kubernetes | Container orchestration |
| EKS | AWS managed Kubernetes |

#### Key distinction

Compose is not normally your production ECS/Kubernetes orchestration model.

Typical setup:

```text
LOCAL DEVELOPMENT
Docker Compose
├── Spring Boot
├── PostgreSQL
├── Redis
└── Kafka

PRODUCTION
ECR
 ↓
ECS / EKS
 ↓
Spring Boot containers
```

---

## 50. Kubernetes YAML vs Helm

Kubernetes has its own configuration model using YAML manifests.

Example:

```yaml
kind: Deployment
```

and:

```yaml
kind: Service
```

These can be applied directly:

```bash
kubectl apply -f deployment.yaml
```

### Helm

#### Definition

> Helm is a package manager for Kubernetes that uses templates and values to simplify managing Kubernetes resources.

Example:

```text
my-spring-app/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    └── secret.yaml
```

Conceptually:

```text
values.yaml
     ↓
Helm templates
     ↓
Rendered Kubernetes YAML
     ↓
Kubernetes
```

Helm does not replace Kubernetes.

---

## 51. Complete AWS Deployment Flow

### ECS

```text
Developer
   ↓
Git
   ↓
CI/CD
   ↓
Maven Test
   ↓
Docker Build
   ↓
Docker Image
   ↓
ECR
   ↓
ECS Task Definition
   ↓
ECS Service
   ↓
Tasks
   ↓
Containers
   ↓
Spring Boot
```

### EKS

```text
Developer
   ↓
Git
   ↓
CI/CD
   ↓
Docker Build
   ↓
Docker Image
   ↓
ECR
   ↓
Helm / Kubernetes manifests
   ↓
EKS
   ↓
Deployment
   ↓
Pods
   ↓
Containers
   ↓
Spring Boot
```

---

## 52. One-Minute Docker Interview Answer

> Docker is a containerization platform that packages an application and its dependencies into an image. A container is a running instance of that image. We define images using Dockerfiles and build them with `docker build`. Images are composed of layers, and Docker can reuse unchanged layers through build caching. For multi-container local development we can use Docker Compose. In AWS, we can push images to ECR and deploy them through ECS or EKS. For optimization, we use multi-stage builds, appropriate runtime images, layer caching, and `.dockerignore`. For security, we avoid running as root, don't bake secrets into images, and scan images for vulnerabilities.

---

## 53. High-Value Interview Questions

### Fundamentals

1. What is Docker?
2. Why do we use Docker?
3. Docker vs VM?
4. Hypervisor vs Docker Engine?
5. Image vs Container?

### Dockerfile

6. What is a Dockerfile?
7. Explain `FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`.
8. `CMD` vs `ENTRYPOINT`?
9. `ARG` vs `ENV`?
10. `COPY` vs `ADD`?
11. What is a multi-stage build?

### Build & Optimization

12. What is Docker build context?
13. What is `.dockerignore`?
14. What are Docker image layers?
15. How does Docker layer caching work?
16. What invalidates a cache layer?
17. How would you reduce image size?
18. Why use JRE instead of JDK in the runtime image?

### Runtime / Networking

19. What does `docker run -p 8081:8080` mean?
20. Why can't one container use `localhost` to reach another?
21. How do containers communicate?
22. What are Docker volumes?
23. Volume vs bind mount?

### Compose

24. What is Docker Compose?
25. `docker compose build` vs `docker compose up --build`?
26. How do Compose services communicate?
27. What does `depends_on` do?
28. Why doesn't `depends_on` guarantee readiness?

### AWS

29. What is a Docker registry?
30. What is ECR?
31. Explain Docker → ECR → ECS.
32. Explain ECS Cluster vs Service vs Task vs Task Definition.
33. ECS EC2 vs Fargate?
34. How does an ECS service handle a failed task?

### Health / Security

35. Running vs healthy container?
36. Docker health check vs ALB health check?
37. How would you secure a Docker image?
38. Why shouldn't secrets be stored in a Dockerfile?
39. Why avoid running as root?
40. Why scan container images?

### Troubleshooting

41. Container exits immediately. What do you check?
42. Container is running but API is inaccessible.
43. Spring Boot cannot connect to PostgreSQL.
44. Container keeps restarting.
45. Port is already in use.
46. Container-to-container communication fails.

---

## 54. Must-Remember Concepts

If you are short on revision time, prioritize these:

```text
1. Image vs Container

2. Dockerfile
   FROM / COPY / RUN / EXPOSE / ENTRYPOINT

3. CMD vs ENTRYPOINT

4. ARG vs ENV

5. Port mapping
   Host:Container

6. localhost inside container
   = that container itself

7. Docker networking
   service/container name → DNS

8. Volumes
   persistent data

9. Compose
   multi-container local environment

10. Image layers + caching
    stable → frequently changing

11. Multi-stage build
    build tools out of final image

12. ECR
    stores Docker images

13. ECS
    Cluster → Service → Task Definition → Task → Container

14. Health checks
    running ≠ healthy

15. Security
    non-root + no secrets in image + scanning

16. Troubleshooting
    ps → logs → inspect → exec → network → stats
```

---

## 55. Quick Mental Model

```text
                 Dockerfile
                     ↓
                docker build
                     ↓
                Docker Image
                     ↓
              ┌──────┴──────┐
              ↓             ↓
        Docker Compose    Registry
          (local)           ↓
                           ECR
                            ↓
                    ┌───────┴───────┐
                    ↓               ↓
                   ECS             EKS
                    ↓               ↓
                  Task             Pod
                    ↓               ↓
                Container       Container
                    ↓               ↓
                 Spring Boot     Spring Boot
```

---

## 56. Final Interview Perspective

For a 3+ year Spring Boot developer, you do **not** need to memorize every Docker command or understand Docker internals at operating-system level.

You should be able to confidently explain:

```text
Why Docker?
     ↓
How is an image built?
     ↓
How does caching work?
     ↓
How does a container run?
     ↓
How do containers communicate?
     ↓
How is persistent data handled?
     ↓
How do we run multiple containers locally?
     ↓
How does CI/CD build and push the image?
     ↓
How does ECR store it?
     ↓
How does ECS/EKS deploy it?
     ↓
How do we troubleshoot it?
```

If you can explain that flow clearly with your Spring Boot examples, your Docker knowledge is interview-ready.
