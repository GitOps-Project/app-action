# Overview
This repository contains the **application source code** and its **CI/CD workflow** for building, testing, analyzing, containerizing, and deploying the VProfile application to **Amazon EKS**.  
The pipeline is fully automated using **GitHub Actions**, integrating **Maven**, **SonarCloud**, **Docker**, **Amazon ECR**, and **Helm**.

---

# GitHub Actions Workflow
The workflow automates the full application lifecycle:

## Trigger Conditions
- Runs on push or pull requests to the repository.  
- Can also be triggered manually via `workflow_dispatch`.  

## Jobs
### 1. Testing
- Checkout source code  
- Run `mvn test` for unit testing  
- Run `mvn checkstyle:checkstyle` for code style checks  
- Setup Java 11 for Sonar analysis  
- Run **SonarScanner CLI** to analyze code quality and upload reports (unit tests, coverage, checkstyle)  
- Validate results against **SonarCloud Quality Gates**  

### 2. Build & Publish
- Checkout source code  
- Build Docker image with the application  
- Tag image with `latest` and GitHub run number  
- Push image to **Amazon ECR**  

### 3. Deploy to EKS
- Configure AWS credentials  
- Generate kubeconfig for the EKS cluster  
- Create Kubernetes secret (`regcred`) for authenticating against Amazon ECR  
- Deploy Helm charts with dynamic values:  
  - `appimage` → Amazon ECR repository URI  
  - `apptag` → GitHub run number  

---

# Secrets Required
The following GitHub Secrets must be configured:

- `AWS_ACCESS_KEY_ID`  
- `AWS_SECRET_ACCESS_KEY`  
- `AWS_REGION`  
- `REGISTRY` (Amazon ECR registry URI)  
- `SONAR_TOKEN` (SonarCloud authentication token)  
- `SONAR_ORGANIZATION` (SonarCloud organization name)  
- `SONAR_PROJECT_KEY` (SonarCloud project key)  
- `SONAR_URL` (https://sonarcloud.io)  

---

# Helm Charts
The repository includes **Helm charts** under `/helm/vprofilecharts` to manage Kubernetes manifests for:  
- Application Deployment  
- Service (ClusterIP)  
- Ingress resource (exposed through NGINX Ingress Controller)  

Variables injected at deployment time:
- `appimage`: container image URI  
- `apptag`: container image tag  

---

# Branching Strategy
- **Feature Branches**  
  Run full workflow (tests, analysis, build). Used for development.  

- **Main Branch**  
  Runs workflow including deployment to EKS. Used for production-ready code.  

---

# Outputs
After a successful deployment:
- Application Docker image stored in Amazon ECR  
- Application pods running in Amazon EKS  
- Service accessible inside the cluster via ClusterIP  
- Ingress routing external traffic to the application through Load Balancer and domain mapping  