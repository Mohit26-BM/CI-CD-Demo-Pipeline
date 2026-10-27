# CI-CD-Demo-Pipeline

A small Spring Boot app used to demonstrate a CI/CD pipeline with Jenkins, Docker and Kubernetes.

## The app

Spring Boot 4.1.2, Java 17, Maven. It listens on port **8080**.

| Endpoint | Response |
| --- | --- |
| `/` | `Welcome to the CI/CD Demo Pipeline!` |
| `/hello` | `Hello, World!` |
| `/greet?name=Mohit` | `Hello, Mohit!` (defaults to `User`) |

## Run it locally

You need JDK 17+ and Maven.

```bash
mvn clean package
java -jar target/ci-cd-demo-pipeline-0.0.1-SNAPSHOT.jar
```

Then open <http://localhost:8080/hello>. Run the tests alone with `mvn test`.

## Docker

The `Dockerfile` runs the built jar on Java 17. Build the jar first, then:

```bash
docker build -t ci-cd-pipeline-image .
docker run -dit --name ci-cd-pipeline-demo -p 9091:8080 ci-cd-pipeline-image
```

Open <http://localhost:9091/hello>. Port 8081 is avoided because Jenkins uses it.

## Kubernetes

The `k8s` folder holds a Deployment (2 replicas of `ci-cd-demo-pipeline:latest`) and a NodePort Service on **30080**.

```bash
kubectl apply -f k8s/
kubectl get pods
```

Open <http://localhost:30080/hello>. The image must already exist locally (Docker Desktop's Kubernetes uses local images).

## Jenkins pipeline

The `Jenkinsfile` defines four stages:

| Stage | What it does |
| --- | --- |
| `checkout` | Checks out this repository |
| `build` | `mvn clean install` creates the jar |
| `docker-image` | `docker build` creates the image |
| `deploy` | `docker run` starts a container on port 9091 |

Each stage prints a confirmation message to the console.

To use it, create a Jenkins **Pipeline** job and set:

- Definition: **Pipeline script from SCM**
- SCM: **Git**, repository URL of this repo, branch `*/main`
- Script Path: `Jenkinsfile`

Jenkins needs Maven (`maven3`) and JDK 17 (`JDK17`) configured under **Manage Jenkins > Tools**, and Docker available on the machine it runs on.

## Project layout

```text
.
├── Dockerfile
├── Jenkinsfile
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── pom.xml
└── src/
    ├── main/java/com/example/cicddemo/
    └── test/java/com/example/cicddemo/
```
