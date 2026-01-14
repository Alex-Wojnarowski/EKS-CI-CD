EKS CI/CD Pipeline with Terraform and GitHub Actions
Overview

This project demonstrates a production-style workflow for deploying a containerized application to Amazon EKS using Terraform for infrastructure provisioning and GitHub Actions for CI/CD. The focus of the project is on secure authentication, controlled deployments, and operational correctness rather than feature breadth.

The repository shows how infrastructure, application delivery, and access control are intentionally separated, reflecting how these responsibilities are commonly handled in real-world environments.
________________________________________

Architecture

The solution is composed of three clearly separated layers.

The infrastructure layer is provisioned with Terraform and is responsible for networking and the Kubernetes control plane. The application layer runs on Kubernetes and defines how the workload is deployed and exposed. The CI/CD layer is implemented using GitHub Actions and controls how new application versions are built and released.

Terraform provisions an Amazon EKS cluster running in a VPC with public and private subnets. Managed node groups are used to simplify node lifecycle management. Terraform is intentionally limited to infrastructure concerns and does not manage application deployments or Kubernetes runtime configuration.

The application is a simple Nginx-based container deployed via a Kubernetes Deployment and exposed externally using a Service of type LoadBalancer, backed by an AWS Network Load Balancer. The deployment defines readiness and liveness probes as well as CPU and memory requests and limits to ensure predictable scheduling and safe rollouts.

GitHub Actions is used to implement a CI/CD pipeline that builds container images, pushes them to Amazon ECR, and deploys them to the cluster in a controlled manner.
________________________________________

CI/CD Flow

The pipeline is triggered on every push to the main branch.

The CI phase runs automatically and is responsible for building and publishing artifacts. It checks out the repository, authenticates to AWS using OIDC, builds a Docker image, and pushes it to Amazon ECR. Images are tagged immutably using the commit SHA to ensure reproducibility and avoid caching issues.

The CD phase runs only after a successful build and requires manual approval. Deployment is gated using GitHub Environments to prevent unintentional production releases. Once approved, the pipeline updates the Kubernetes Deployment image and waits for the rollout to complete.

This separation ensures that integration remains fast while production releases remain deliberate and auditable.
________________________________________

Authentication and Access Model

AWS authentication for CI/CD is implemented using GitHub Actions OIDC. No static AWS credentials are stored in the repository or in GitHub. GitHub issues a short-lived identity token at runtime, which is exchanged for AWS credentials by assuming a dedicated IAM role.

IAM roles required for cluster access and CI/CD are bootstrapped manually. This avoids circular dependencies where Terraform would need to manage the permissions required to run itself and mirrors how IAM foundations are often handled in real environments.

The CI/CD IAM role is restricted to this repository through its trust policy and is scoped to the minimum AWS permissions required to push images to ECR and interact with the EKS control plane.

Kubernetes access is configured via the aws-auth ConfigMap. The CI/CD IAM role is mapped to the system:masters group, granting cluster-admin permissions. This choice was made to simplify the project and focus on CI/CD and EKS integration rather than fine-grained RBAC configuration. In a production environment, this would be replaced with a namespace-scoped Role and RoleBinding following the principle of least privilege.
________________________________________

Kubernetes Configuration

The application is deployed using a Kubernetes Deployment with multiple replicas to ensure availability. Readiness and liveness probes are configured to ensure traffic is only sent to healthy pods and that unresponsive containers are restarted automatically.

CPU and memory requests and limits are defined to provide predictable resource usage and prevent a single workload from exhausting node capacity. Rolling updates are used so that new versions are deployed without downtime.

The Service is exposed using an AWS Network Load Balancer. No application-level caching is involved, ensuring that traffic always reaches the currently active pods once the rollout is complete.
________________________________________

Terraform Scope and Boundaries

Terraform is responsible for provisioning cloud infrastructure components, including the VPC, EKS cluster, node groups, and supporting AWS resources. Terraform intentionally does not manage Kubernetes manifests or application lifecycle operations.

This separation reflects real-world ownership boundaries, where infrastructure teams manage clusters and networking while application delivery is handled through CI/CD and Kubernetes tooling.
________________________________________

Deployment Process

A typical deployment follows this flow:

A change is pushed to the main branch

The CI job builds and pushes a new container image

The CD job pauses and awaits manual approval

Once approved, the deployment image is updated in Kubernetes

Kubernetes performs a rolling update and verifies pod readiness
________________________________________

Design Decisions

Several design decisions were made intentionally:

OIDC is used instead of static credentials to reduce long-lived secrets and improve security

Deployments are gated to prevent accidental production changes

Immutable image tags are used to ensure traceability and reproducibility

Infrastructure and application responsibilities are clearly separated
________________________________________

Future Improvements

Possible extensions to this project include adding observability with Prometheus and Grafana, introducing alerting rules, implementing canary or blue/green deployments, and restricting Kubernetes RBAC to namespace-scoped permissions.
________________________________________

Technology Stack

AWS (EKS, ECR, IAM, VPC)

Terraform

Kubernetes

Docker

GitHub Actions
