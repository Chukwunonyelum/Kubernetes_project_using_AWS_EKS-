

# Hosting a Dynamic Application with Kubernetes and AWS EKS

This project demonstrates a comprehensive DevOps pipeline for deploying and managing a dynamic, containerized application on AWS using Kubernetes and AWS EKS (Elastic Kubernetes Service). This setup enables a secure, scalable infrastructure optimized for hosting high-performance applications.

---

## Business Problem

### Problem Overview
Many businesses today face challenges in deploying applications that are:
1. **Scalable** – Handling increasing or variable loads without sacrificing performance.
2. **Secure** – Ensuring protection from external threats with robust network and data security.
3. **Cost-Efficient and Agile** – Quickly adapting to new feature requirements and optimizing resource usage.

For growing applications, these challenges can lead to:
- **Performance Bottlenecks**: Traditional infrastructure struggles to handle sudden spikes in traffic.
- **Security Risks**: Vulnerabilities in unsegregated environments that lack robust access management.
- **High Operational Overheads**: Manual updates and scaling are time-consuming and costly.

### Solution
This project solves these issues by implementing a **cloud-native DevOps pipeline** on AWS using containerization, Kubernetes, and automation tools. Specifically, the project leverages:
- **Elastic Kubernetes Service (EKS)**: Provides automatic scaling and flexible deployment, allowing for optimized resource usage and handling of high loads.
- **Docker Containers**: Ensures consistency across different environments, from development to production, minimizing deployment issues.
- **Secure Networking with VPC**: Configures private and public subnets, security groups, and NAT gateways to control access and secure sensitive data.
- **AWS Secret Management**: Manages environment variables securely, minimizing the risk of exposure.
- **Automated CI/CD**: Using Docker and Kubernetes for continuous integration and continuous deployment, significantly reducing operational overhead.

This approach provides a robust infrastructure that is **cost-effective**, **scalable**, and **highly secure**—ideal for dynamic, high-performance applications.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Steps](#steps)
  - [1. VPC and Network Setup](#1-vpc-and-network-setup)
  - [2. Security Groups Configuration](#2-security-groups-configuration)
  - [3. EC2 Instance and Endpoint Setup](#3-ec2-instance-and-endpoint-setup)
  - [4. Database Setup with Amazon RDS](#4-database-setup-with-amazon-rds)
  - [5. Domain and SSL Configuration](#5-domain-and-ssl-configuration)
  - [6. Application Code Repository Setup](#6-application-code-repository-setup)
  - [7. Docker and Image Management](#7-docker-and-image-management)
  - [8. Data Migration to RDS](#8-data-migration-to-rds)
  - [9. Install Kubernetes Tools](#9-install-kubernetes-tools)
  - [10. AWS Secret Management](#10-aws-secret-management)
  - [11. EKS Cluster Setup](#11-eks-cluster-setup)
  - [12. Kubernetes Manifest Files and Deployment](#12-kubernetes-manifest-files-and-deployment)
  - [13. Load Balancer Configuration](#13-load-balancer-configuration)
  - [14. Route 53 DNS Setup](#14-route-53-dns-setup)
- [Additional Resources](#additional-resources)


---

## Project Overview

In this project, we build a secure, scalable infrastructure for hosting a dynamic application. The application will be deployed in Docker containers on AWS and managed using Kubernetes with AWS EKS, using a combination of VPC, RDS, S3, IAM, and EKS services.

---

## Prerequisites

1. AWS account with necessary permissions for VPC, EKS, IAM, RDS, and Route 53.
2. Installed:
   - Docker
   - AWS CLI
   - kubectl
   - eksctl
   - Helm
3. GitHub and Docker Hub accounts for managing application code and Docker images.
4. Visual Studio Code (VS Code) or any code editor.

---

## Steps

### 1. VPC and Network Setup

- **Create VPC and Enable DNS Host Name**: Start by creating a VPC and enabling DNS host names.
- **Internet Gateway**: Create an internet gateway and attach it to the VPC.
- **Subnets Creation**:
  - Public Subnet AZ1: `10.0.0.0/24`
  - Private App Subnet AZ1: `10.0.2.0/24`
  - Private Data Subnet AZ1: `10.0.4.0/24`
  - Public Subnet AZ2: `10.0.1.0/24`
  - Private App Subnet AZ2: `10.0.3.0/24`
  - Private Data Subnet AZ2: `10.0.5.0/24`
- **Auto-Assign IP**: Enable auto-assign public IPs for the public subnets.
- **Route Table Configuration**:
  - Public Route Table: Create a route table for the public subnets and add a route to the internet gateway.
  - Associate the public subnets with the public route table.
  - NAT Gateway: Create NAT gateways for private subnets to access the internet.
  - Private Route Table: Set up a private route table for private subnets and add a route through the NAT gateway.

### 2. Security Groups Configuration

- **EICE SG**: Allows outbound SSH to the IP of the VPC.
- **Data Migration Server SG**: Allows inbound SSH access from the EICE SG.
- **RDS SG**: Allows inbound MySQL/Aurora (port 3306) traffic from the Data Migration Server SG.

### 3. EC2 Instance and Endpoint Setup

- **EC2 Instance Connect Endpoint**: Set up an EC2 instance connect endpoint.
- **Launch EC2 Instance**: Create an EC2 instance for data migration and application setup tasks.

### 4. Database Setup with Amazon RDS

- **RDS Instance**: Set up an Amazon RDS instance with a MySQL or Aurora database.
- **Database Configuration**: Set database credentials and configure the necessary parameters.

### 5. Domain and SSL Configuration

- **SSL Certificate in AWS Certificate Manager**: Request an SSL certificate for your application domain.
- **Domain Registration in Route 53**: Register a domain name in Route 53.
- **DNS Record Creation**: Set up a DNS record in Route 53 to map the domain to your application.

### 6. Application Code Repository Setup

- **GitHub Repository Setup**: Create a repository on GitHub for the application code and Dockerfile.
- **Clone the Repository**:
  ```bash
  git clone <repository-ssh-url>
  ```
- **Upload Application Code**: Copy application code into the cloned GitHub directory, open in Visual Studio Code, and commit changes using Git.

### 7. Docker and Image Management

- **Dockerfile Creation**:
  - Create a `Dockerfile` in the project directory and configure it for your web application.
  - Configure build arguments and environment variables for secure builds.
- **Build Docker Image**:
  ```bash
  docker build -t your-dockerhub-username/your-app-name .
  ```
- **Push Docker Image to ECR**:
  - Create an Amazon ECR repository.
  - Tag and push your Docker image to ECR:
    ```bash
    docker tag your-dockerhub-username/your-app-name aws-account-id.dkr.ecr.region.amazonaws.com/your-ecr-repo-name
    docker push aws-account-id.dkr.ecr.region.amazonaws.com/your-ecr-repo-name
    ```

### 8. Data Migration to RDS

- **Create S3 Bucket**: Create an S3 bucket and upload your SQL migration scripts.
- **IAM Role with S3 Access**: Create an IAM role with AmazonS3FullAccess for accessing the SQL scripts.
- **Data Migration Using Flyway**: Update Flyway environment variables and connect to the EC2 instance to run the migration scripts on the RDS database.

### 9. Install Kubernetes Tools

- **Install Chocolatey**:
  - Open PowerShell as Administrator and install Chocolatey:
    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    ```
- **Install kubectl and eksctl**:
  - Install `kubectl` using Chocolatey:
    ```powershell
    choco install kubernetes-cli
    ```
  - Download and install `eksctl` for EKS management.
- **Install Helm**:
  ```powershell
  choco install kubernetes-helm
  ```

### 10. AWS Secret Management

- **Secrets in AWS Secret Manager**: Use AWS Secret Manager to securely store sensitive environment variables.
- **Secret Manager Policy**: Create a policy that grants access to the stored secrets, and assign it to necessary roles.

### 11. EKS Cluster Setup

- **EKS IAM Role Creation**: Set up IAM roles for EKS cluster and worker nodes.
- **EKS Cluster Creation**: Create an EKS cluster with the latest add-ons, configuring nodes for the Kubernetes environment.

### 12. Kubernetes Manifest Files and Deployment

- **Creating Kubernetes Manifest Files**:
  - **

SecretProviderClass**: Define secrets in AWS Secret Manager and map them to Kubernetes secrets.
  - **Deployment File**: Create a deployment file specifying your Docker image and replica settings.
  - **Service File**: Define a service file to expose the application within the Kubernetes cluster.
- **Namespace and CSI Driver**:
  - Create a namespace for your application.
  - Install the Secrets Store CSI Driver to securely access secrets.

### 13. Load Balancer Configuration

- **TLS Listener Addition**: Add a TLS listener to the ALB to securely route traffic.
- **Modify Security Groups**: Update RDS SG inbound rules to allow traffic from the EKS cluster.

### 14. Route 53 DNS Setup

- **Route 53 Record Set**: Create a Route 53 record to route traffic to the EKS cluster.

---

## Additional Resources

- [AWS Documentation](https://docs.aws.amazon.com/)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

