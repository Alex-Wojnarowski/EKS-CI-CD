# K8
This project demonstrates deploying a simple containerized "Hello World" website to AWS using Elastic Kubernetes Service (EKS). It covers the complete workflow from container creation, repository management, networking, cluster provisioning, to exposing the application via an internet-facing Network Load Balancer.

   ![Project3](https://github.com/user-attachments/assets/bf30e8c5-9c9e-4ac3-a56e-c1e9fdeed0b8)

Key Objectives:

1. Build and containerize a simple Nginx-based website with a custom index.html.

2. Push the Docker image to Amazon ECR.

3. Provision a custom VPC with public and private subnets using Terraform.

4. Deploy an EKS cluster with managed node groups.

5. Deploy the containerized website via a Kubernetes Deployment.

6. Expose the application externally using a Kubernetes Service of type LoadBalancer (NLB).

7. Ensure high availability by running multiple replicas across availability zones.

Technical Steps:

1. Build the Docker Image

 - Create an Nginx Docker image containing the index.html.

 - Test locally to verify functionality.

2. Create an Amazon ECR Repository

 - Follow AWS ECR instructions to push the Docker image:

    * Authenticate Docker with ECR.

    * Tag the image.

    * Push the image to the repository.

3. Provision Networking with Terraform

 - Create a custom VPC (VPC.tf) with:

    * Two public subnets for LoadBalancer access.

    * Two private subnets for EKS worker nodes.

    * NAT Gateway for private subnet internet access.

 - Configure subnet tags to integrate with EKS.

4. Deploy EKS Cluster

 - Use Terraform terraform-aws-modules/eks/aws module (EKS.tf) to provision:

    * Control plane in private subnets.

    * Managed node groups in private subnets.

    * Core AWS add-ons: VPC-CNI, CoreDNS, kube-proxy.

    * Internet-accessible endpoint.

 - IAM roles for node groups are automatically managed.

5. Configure Kubernetes Access

 - Retrieve kubeconfig for local kubectl access.

6. Deploy Application

 - Apply deployment.yaml for the Nginx pods.

 - Apply service.yaml to expose the deployment via an NLB.

7. Access Application

 - Retrieve the external NLB DNS from kubectl get svc and open it in a browser. 

Biggest Problems Encountered:

1. Certificate Authority Delay

 - Issue: Initially, the cluster creation seemed to hang, and the certificate authority took longer than expected to become available.

 - Solution: Patience. The cluster eventually became ready once AWS finished provisioning all control plane resources.

2. Node Group Configuration Issues

 - Issue: The managed node group initially showed:

    * No Auto Scaling Group name

    * No Node IAM Role ARN

    * Nodes stuck in CREATING

 - Solution: Tweaked the EKS.tf configuration:

    * Ensured proper IAM role attachments for the nodes by ensuring correct addons usage.

3. Pod Networking Failures

 - Issue: Pods were stuck in ContainerCreating due to CNI plugin.

 - Solution:

    * Correcting addon configuration resulting in proper creation priority of resources.

4. Load Balancer Type Not Updating

 - Issue: The Kubernetes Service initially provisioned a Classic Load Balancer instead of the desired NLB.

 - Solution:

    * Updated service annotations to specify NLB.

    * Re-applied the service YAML and waited for AWS to provision the NLB.

5. General EKS Provisioning Delays

 - Observation: EKS cluster and node group provisioning can take significantly longer than other AWS infrastructure.

 - Takeaway: Plan for up to 15–20 minutes for full node readiness, especially when deploying private subnets or multiple availability zones.
