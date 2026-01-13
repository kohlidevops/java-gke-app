# Complete Java Spring Boot CI/CD to GKE - End to End Guide

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


## Project Structure

```
java-gke-app/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── k8s/
│   └── deployment.yaml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── example/
│   │   │           └── demo/
│   │   │               ├── DemoApplication.java
│   │   │               └── controller/
│   │   │                   └── HealthController.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/
│           └── com/
│               └── example/
│                   └── demo/
│                       └── DemoApplicationTests.java
├── pom.xml
├── Dockerfile
├── .dockerignore
├── .gitignore
└── README.md
```

## FILE 1: pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0.0</version>
    <name>demo</name>
    <description>Demo Spring Boot application for GKE</description>
    
    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Spring Boot Actuator (Health checks) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        
        <!-- Test Dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <finalName>app</finalName>
    </build>
</project>
```

## FILE 2: src/main/java/com/example/demo/DemoApplication.java

```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

## FILE 3: src/main/java/com/example/demo/controller/HealthController.java

```
package com.example.demo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestController
public class HealthController {

    @GetMapping("/")
    public Map<String, Object> home() {
        Map<String, Object> response = new HashMap<>();
        response.put("message", "Hello from Java Spring Boot on GKE!");
        response.put("version", "1.0.0");
        response.put("timestamp", LocalDateTime.now().toString());
        response.put("status", "running");
        return response;
    }

    @GetMapping("/health")
    public Map<String, String> health() {
        Map<String, String> response = new HashMap<>();
        response.put("status", "UP");
        return response;
    }

    @GetMapping("/api/info")
    public Map<String, Object> info() {
        Map<String, Object> response = new HashMap<>();
        response.put("application", "Java GKE Demo");
        response.put("version", "1.0.0");
        response.put("java_version", System.getProperty("java.version"));
        response.put("spring_boot_version", "3.2.0");
        return response;
    }
}
```

## FILE 4: src/main/resources/application.properties

```
# Server Configuration
server.port=8080
spring.application.name=java-gke-app

# Actuator Configuration
management.endpoints.web.exposure.include=health,info
management.endpoint.health.show-details=always
management.health.livenessState.enabled=true
management.health.readinessState.enabled=true

# Logging
logging.level.root=INFO
logging.level.com.example.demo=DEBUG
```

## FILE 5: src/test/java/com/example/demo/DemoApplicationTests.java

```
package com.example.demo;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class DemoApplicationTests {

    @Test
    void contextLoads() {
        // Test that Spring context loads successfully
    }
}
```

## FILE 6: Dockerfile

```
# Multi-stage build for smaller image size

# Stage 1: Build
FROM maven:3.9.5-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Create non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Copy JAR from build stage
COPY --from=build /app/target/app.jar app.jar

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:8080/health || exit 1

# Run application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## FILE 7: .dockerignore

```
target/
.mvn/
mvnw
mvnw.cmd
.git
.gitignore
README.md
*.md
.env
.DS_Store
.idea/
*.iml
.vscode/
```

## FILE 8: .gitignore

```
# Maven
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties
.mvn/wrapper/maven-wrapper.jar

# Eclipse
.classpath
.project
.settings/

# IntelliJ IDEA
.idea/
*.iws
*.iml
*.ipr
out/

# VS Code
.vscode/

# OS
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Logs
*.log

# GCP
key.json
service-account.json

# Spring Boot
application-local.properties
```

## FILE 9: k8s/deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-app
  labels:
    app: java-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: java-app
  template:
    metadata:
      labels:
        app: java-app
    spec:
      containers:
      - name: java-app
        image: gcr.io/PROJECT_ID/java-app:IMAGE_TAG
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: JAVA_OPTS
          value: "-Xmx512m -Xms256m"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
---
apiVersion: v1
kind: Service
metadata:
  name: java-app-service
spec:
  type: LoadBalancer
  selector:
    app: java-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

## FILE 10: .github/workflows/deploy.yml

```
name: Build and Deploy Java App to GKE

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

env:
  PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
  GKE_CLUSTER: java-gke-cluster
  GKE_ZONE: us-central1-a
  DEPLOYMENT_NAME: java-app
  IMAGE: java-app

jobs:
  setup-build-publish-deploy:
    name: Setup, Build, Publish, and Deploy
    runs-on: ubuntu-latest
    
    permissions:
      contents: read
      id-token: write

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Run tests
      run: mvn test

    - name: Build with Maven
      run: mvn clean package -DskipTests

    - name: Authenticate to Google Cloud
      uses: google-github-actions/auth@v2
      with:
        credentials_json: ${{ secrets.GCP_SA_KEY }}

    - name: Set up Cloud SDK
      uses: google-github-actions/setup-gcloud@v2
      with:
        install_components: 'gke-gcloud-auth-plugin'

    - name: Configure Docker and kubectl
      run: |
        gcloud auth configure-docker
        echo "USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> $GITHUB_ENV

    - name: Get GKE credentials
      run: |
        gcloud container clusters get-credentials $GKE_CLUSTER \
          --zone $GKE_ZONE \
          --project $PROJECT_ID

    - name: Build Docker image
      run: |
        docker build -t gcr.io/$PROJECT_ID/$IMAGE:$GITHUB_SHA \
          -t gcr.io/$PROJECT_ID/$IMAGE:latest .

    - name: Publish Docker image to GCR
      run: |
        docker push gcr.io/$PROJECT_ID/$IMAGE:$GITHUB_SHA
        docker push gcr.io/$PROJECT_ID/$IMAGE:latest

    - name: Deploy to GKE
      run: |
        sed -i "s|gcr.io/PROJECT_ID/java-app:IMAGE_TAG|gcr.io/$PROJECT_ID/$IMAGE:$GITHUB_SHA|g" k8s/deployment.yaml
        kubectl apply -f k8s/deployment.yaml
        kubectl rollout status deployment/$DEPLOYMENT_NAME
        kubectl get services -o wide
```

## STEP-BY-STEP SETUP GUIDE

### PART 1: GCP Setup

#### Step 1: Create GCP Project

```
# Login to GCP
gcloud auth login

# Create project
gcloud projects create YOUR_PROJECT_ID --name="Java GKE Project"

# Set project
gcloud config set project YOUR_PROJECT_ID

# Enable billing (do this in GCP Console)
# Go to: https://console.cloud.google.com/billing
```

#### Step 2: Enable Required APIs

```
# Enable APIs
gcloud services enable container.googleapis.com
gcloud services enable containerregistry.googleapis.com
gcloud services enable cloudbuild.googleapis.com
gcloud services enable artifactregistry.googleapis.com
```

#### Step 3: Create GKE Cluster

```
# Create GKE cluster (takes 5-10 minutes)
gcloud container clusters create java-gke-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type e2-standard-2 \
  --enable-autoscaling \
  --min-nodes 1 \
  --max-nodes 5 \
  --disk-size 50 \
  --disk-type pd-standard

# Get credentials
gcloud container clusters get-credentials java-gke-cluster --zone us-central1-a
```

#### Step 4: Create Service Account

```
# Replace with your actual values
PROJECT_ID="YOUR_PROJECT_ID"
SA_EMAIL=$(gcloud iam service-accounts list \
  --filter="displayName:GitHub Actions Java SA" \
  --format='value(email)')

echo "Service Account Email: $SA_EMAIL"

# 1. Kubernetes Engine Developer
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SA_EMAIL" --role="roles/container.developer"

# 2. Storage Admin (GCR uses Cloud Storage)
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SA_EMAIL" --role="roles/storage.admin"

# 3. Artifact Registry Admin (create repos on push)
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SA_EMAIL" --role="roles/artifactregistry.admin"

# 4. Service Account User
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SA_EMAIL" --role="roles/iam.serviceAccountUser"

# 5. Container Registry Service Agent
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$SA_EMAIL" --role="roles/containerregistry.ServiceAgent"

gcloud projects get-iam-policy $PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:$SA_EMAIL" \
  --format="table(bindings.role)"
```

#### Initialize GCR Manually

```
# Set your project ID
PROJECT_ID="YOUR_PROJECT_ID"
gcloud config set project $PROJECT_ID

# Enable Container Registry API
gcloud services enable containerregistry.googleapis.com

# Authenticate Docker with GCR
gcloud auth configure-docker

# Create a minimal image to initialize GCR for your project
docker pull busybox:latest
docker tag busybox:latest gcr.io/$PROJECT_ID/java-app:init
docker push gcr.io/$PROJECT_ID/java-app:init
```

### PART 2: Local Project Setup

#### Step 5: Create Project Structure

```
# Create project directory
mkdir -p java-gke-app
cd java-gke-app

# Create directory structure
mkdir -p .github/workflows
mkdir -p k8s
mkdir -p src/main/java/com/example/demo/controller
mkdir -p src/main/resources
mkdir -p src/test/java/com/example/demo
```

#### Step 6: Create All Files

Create all the files listed above (FILES 1-11) in their respective directories.

#### Step 7: Initialize Git

```
# Initialize git
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit: Java Spring Boot GKE app"
```

### PART 3: GitHub Setup

#### Step 8: Create GitHub Repository

- Go to GitHub and create new repository: java-gke-app
- Don't initialize with README

#### Step 9: Configure GitHub Secrets

Go to: Settings → Secrets and variables → Actions → New repository secret

Add these secrets:

```
GCP_PROJECT_ID
Value: Your GCP project ID

GCP_SA_KEY
Value: Complete content of key.json file
```

#### Step 10: Push to GitHub

```
# Add remote
git remote add origin https://github.com/YOUR_USERNAME/java-gke-app.git

# Push code
git branch -M main
git push -u origin main
```

### PART 4: Verify Deployment

#### Step 11: Monitor GitHub Actions

1. Go to your repository
2. Click Actions tab
3. Watch the workflow execute
4. Check for any errors


<img width="1820" height="892" alt="image" src="https://github.com/user-attachments/assets/6d4efcfb-54e7-4767-aede-85e70f9049db" />


#### Step 12: Check GKE Deployment

```
# Get cluster credentials
gcloud container clusters get-credentials java-gke-cluster --zone us-central1-a

# Check pods
kubectl get pods

# Check deployments
kubectl get deployments

# Check services
kubectl get services

# Get external IP
kubectl get service java-app-service

# Wait for external IP (may take 2-3 minutes)
watch kubectl get service java-app-service
```

<img width="1122" height="500" alt="image" src="https://github.com/user-attachments/assets/1bfc176f-1d23-4034-82d7-cdd13d66f071" />


#### Step 13: Test Application

```
# Get external IP
EXTERNAL_IP=$(kubectl get service java-app-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

echo "Application URL: http://$EXTERNAL_IP"

# Test endpoints
curl http://$EXTERNAL_IP
curl http://$EXTERNAL_IP/health
curl http://$EXTERNAL_IP/api/info
```

<img width="1227" height="295" alt="image" src="https://github.com/user-attachments/assets/0eb44572-e532-46cb-810d-c22d2e6c0929" />


### PART 5: Useful Commands

#### View Logs

```
# View all pod logs
kubectl logs -l app=java-app

# Follow logs
kubectl logs -l app=java-app -f

# View specific pod logs
kubectl logs POD_NAME
```

#### Scale Application

```
# Scale to 5 replicas
kubectl scale deployment java-app --replicas=5

# Check status
kubectl get deployment java-app
```

#### Update Application

```
# Make code changes
# Commit and push
git add .
git commit -m "Update application"
git push origin main

# GitHub Actions will automatically deploy
```

#### Rollback Deployment

```
# Rollback to previous version
kubectl rollout undo deployment/java-app

# Check rollback status
kubectl rollout status deployment/java-app
```

#### Debug Issues

```
# Describe pod
kubectl describe pod POD_NAME

# Get events
kubectl get events --sort-by=.metadata.creationTimestamp

# Execute into pod
kubectl exec -it POD_NAME -- /bin/sh

# Check deployment status
kubectl describe deployment java-app
```

### PART 6: Cleanup

```
# Delete Kubernetes resources
kubectl delete -f k8s/deployment.yaml

# Delete GKE cluster
gcloud container clusters delete java-gke-cluster --zone us-central1-a

# Delete container images
gcloud container images delete gcr.io/YOUR_PROJECT_ID/java-app:latest

# Delete service account
gcloud iam service-accounts delete $SA_EMAIL

# Delete key file
rm key.json
```

### Troubleshooting

**Issue: Maven build fails**

```
#Clean and rebuild
mvn clean install -U
```

**Issue: Docker build fails**

```
# Build with verbose output
docker build --no-cache -t java-app .
```

**Issue: Pod CrashLoopBackOff**

```
#Check logs
kubectl logs POD_NAME

#Check pod description
kubectl describe pod POD_NAME
```

**Issue: Service external IP pending**

```
# Wait 2-3 minutes
# Check service status
kubectl describe service java-app-service
```
