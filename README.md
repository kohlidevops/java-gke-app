# Java Spring Boot GKE Application

A Spring Boot application with automated CI/CD deployment to Google Kubernetes Engine (GKE) using GitHub Actions.

## Features

- Spring Boot 3.2.0
- Java 17
- REST API endpoints
- Health check endpoints
- Docker multi-stage build
- Kubernetes deployment
- Automated CI/CD with GitHub Actions
- Maven build system

## Prerequisites

- Java 17+
- Maven 3.6+
- Docker
- GCP account with billing enabled
- gcloud CLI
- kubectl
- GitHub account

## Local Development

### Build the application
```bash
mvn clean package
```

### Run locally
```bash
mvn spring-boot:run
```

### Run tests
```bash
mvn test
```

Application will be available at `http://localhost:8080`

### Test endpoints
```bash
curl http://localhost:8080
curl http://localhost:8080/health
curl http://localhost:8080/api/info
```

## Docker

### Build image
```bash
docker build -t java-gke-app .
```

### Run container
```bash
docker run -p 8080:8080 java-gke-app
```

## API Endpoints

- `GET /` - Welcome message
- `GET /health` - Health check
- `GET /api/info` - Application information
