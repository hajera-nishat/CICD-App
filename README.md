# Automated CI/CD Pipeline for a Java Web Application

This project is an end-to-end DevOps CI/CD project for a Java web application.

The goal was to understand how source code moves from GitHub through the CI/CD pipeline and finally gets deployed to Kubernetes on AWS.

## Tools Used

- GitHub – Source code management
- Jenkins – CI/CD pipeline
- Maven – Build and testing
- SonarQube – Code quality analysis
- JFrog Artifactory – Maven artifact repository
- Docker – Containerization
- Docker Hub – Docker image registry
- Trivy – Container vulnerability scanning
- Terraform – Infrastructure setup
- Amazon EKS – Kubernetes cluster
- Kubernetes – Application deployment
- AWS LoadBalancer – Application access

## CI/CD Flow

```text
GitHub
   ↓
Jenkins
   ↓
Maven Build & Test
   ↓
SonarQube
   ↓
Quality Gate
   ↓
JFrog Artifactory
   ↓
Docker Build
   ↓
Docker Hub
   ↓
Trivy Scan
   ↓
Amazon EKS
   ↓
Kubernetes
   ↓
AWS LoadBalancer
   ↓
Java Web Application


What the Jenkins Pipeline Does

The Jenkins pipeline automates the following steps:

Checks out the application code from GitHub.
Builds and tests the application using Maven.
Runs SonarQube code analysis.
Checks the SonarQube quality gate.
Publishes the Maven artifact to JFrog Artifactory.
Builds the Docker image.
Pushes the image to Docker Hub.
Scans the Docker image using Trivy.
Connects Jenkins to the Amazon EKS cluster.
Deploys the application using Kubernetes.
Checks the Kubernetes deployment and running pods.
Exposes the application through an AWS LoadBalancer.

Project Structure
Automated-CICD-App/
│
├── Kubernete/
│   ├── regapp-deploy.yml
│   └── regapp-service.yml
│
├── server/
│   ├── src/
│   └── pom.xml
│
├── webapp/
│   ├── src/
│   └── pom.xml
│
├── Terraform/
│   ├── install.sh
│   ├── main.tf
│   ├── provider.tf
│   └── .terraform.lock.hcl
│
├── Dockerfile
├── Jenkinsfile
├── pom.xml
├── .gitignore
└── README.md

Kubernetes Deployment

The application is deployed on Amazon EKS using Kubernetes.

The deployment runs two replicas:

regapp-deployment
   ├── Pod 1
   └── Pod 2

The application is exposed using a Kubernetes LoadBalancer service, which creates an AWS LoadBalancer.

What I Worked On

This project helped me understand how the different parts of a DevOps pipeline work together.

During the project, I worked on:

Setting up and configuring Jenkins
Connecting Jenkins with GitHub
Building and testing the Java application with Maven
Setting up SonarQube and a quality gate
Configuring JFrog Artifactory
Building and pushing Docker images
Adding Trivy container scanning
Setting up an Amazon EKS cluster
Connecting Jenkins with AWS and EKS
Deploying the application using Kubernetes
Exposing the application through an AWS LoadBalancer
Troubleshooting Jenkins, Docker, JFrog Artifactory and Kubernetes issues
Security

Credentials are managed through Jenkins credentials instead of being stored directly in the Jenkinsfile.

Sensitive files such as Terraform state files, environment files, private keys and tokens are excluded using .gitignore.

Future Improvements
Add monitoring and logging
Add HTTPS
Add Kubernetes resource limits
Add automated rollback
Use AWS Secrets Manager or Kubernetes Secrets
Add separate development and production environments

Author

Hajera Nishat

GitHub: https://github.com/hajera-nishat