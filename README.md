EKS CI/CD Pipeline with Terraform & GitHub Actions

Overview

This project demonstrates a production-style CI/CD workflow for deploying a containerized application to Amazon EKS, with infrastructure provisioned using Terraform and deployments controlled through GitHub Actions using OIDC authentication.

The focus of this project is not just deployment, but operability, security, and controlled releases.

________________________________________
Architecture Overview
Infrastructure layer (Terraform):
-	VPC with public and private subnets
-	Amazon EKS cluster with managed node groups
-	IAM roles and permissions required for cluster and CI/CD access
  
Application layer (Kubernetes):
-	Kubernetes Deployment running an Nginx-based application
-	Kubernetes Service of type LoadBalancer (AWS NLB)
-	Resource requests & limits
-	Readiness and liveness probes
  
CI/CD layer (GitHub Actions):
-	Automated image build & push to Amazon ECR
-	Manual approval gate for production deployment
-	Secure AWS authentication using OIDC (no static credentials)
  
________________________________________
CI/CD Flow
Continuous Integration (CI)
Triggered on every push to main:
1.	Checkout source code
2.	Authenticate to AWS using GitHub OIDC
3.	Build Docker image
4.	Push image to Amazon ECR with immutable SHA tag
   
CI runs automatically and does not require approval.
________________________________________
Continuous Deployment (CD)
Triggered after successful CI:
1.	Requires manual approval via GitHub Environments
2.	Assumes AWS role using OIDC
3.	Updates Kubernetes Deployment image
4.	Verifies rollout status
   
This approach ensures controlled production releases while keeping CI fully automated.
________________________________________
Security Considerations
-	No AWS access keys stored in GitHub
-	GitHub Actions authenticates via OIDC
-	IAM role trust restricted to this repository
-	Kubernetes RBAC handled via aws-auth mapping
-	Immutable image tags prevent accidental rollbacks
  
________________________________________
Kubernetes Configuration Highlights
-	Readiness & liveness probes ensure safe rollouts
-	CPU & memory requests/limits enforce fair scheduling
-	Rolling updates prevent downtime
-	NLB provides external access without pod exposure
  
________________________________________
Terraform Scope
Terraform is responsible for:
-	Networking (VPC, subnets)
-	EKS cluster & node groups
-	IAM roles and permissions
  
Terraform intentionally does not manage:
-	Application deployments
-	Kubernetes runtime configuration
  
This separation reflects real-world ownership boundaries.
________________________________________
How to Deploy
1.	Push changes to main
2.	CI builds and pushes image automatically
3.	Approve deployment in GitHub Actions
4.	Application rolls out to EKS
   
________________________________________
Design Decisions
-	OIDC over static credentials → improved security
-	Manual production gate → controlled releases
-	Immutable image tags → reproducibility
-	Separation of infra & app lifecycle → clarity and safety
  
________________________________________
Possible Extensions
-	Add observability (Prometheus & Grafana)
-	Introduce alerting rules
-	Blue/Green or Canary deployments
-	Cost monitoring dashboards
  
________________________________________
 Tech Stack
-	AWS (EKS, ECR, IAM, VPC)
-	Terraform
-	Kubernetes
-	Docker
-	GitHub Actions
  

