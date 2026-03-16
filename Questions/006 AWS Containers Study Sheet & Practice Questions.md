This study sheet focuses on the **Containerization** domain of the **DOP-C02** exam. At the Professional level, the exam emphasizes operational excellence: how to manage secrets, perform complex deployments (Canary/Blue-Green), optimize scaling, and handle cross-account image sharing.

## ---

**1\. Amazon ECR (Elastic Container Registry)**

ECR is your managed Docker registry. The exam focuses on security and lifecycle management.

* **Repository Policies:** Control access at the repository level (IAM for ECR). Essential for **Cross-Account access**.  
* **Image Scanning:** Scans for software vulnerabilities. Can be "On Push" or "Continuous."  
* **Lifecycle Policies:** Automatically clean up old or untagged images to save costs.  
* **Replication:** Supports both **Cross-Region** and **Cross-Account** replication (useful for disaster recovery and multi-region pipelines).  
* **Authorization:** The aws ecr get-login-password command retrieves a token valid for **12 hours**.

## ---

**2\. Amazon ECS (Elastic Container Service)**

ECS is a highly scalable, high-performance container orchestration service.

* **Task Definitions:** The "Blueprint" (JSON). Defines CPU, Memory, IAM roles, and Docker images.  
  * **Task Role:** Permissions for the application *inside* the container (e.g., access to S3).  
  * **Task Execution Role:** Permissions for the ECS *agent* (e.g., pulling images from ECR, sending logs to CloudWatch).  
* **Service & Scheduling:** \* **REPLICA:** Standard scaling.  
  * **DAEMON:** One task per EC2 instance (useful for log collectors/monitoring agents).  
* **Deployment Strategies:** \* **Rolling Update:** Uses minimumHealthyPercent and maximumPercent to control availability.  
  * **Blue/Green:** Managed by **AWS CodeDeploy**. Swaps traffic using an ALB listener.  
* **Networking:** \* awsvpc mode is recommended; it gives each task its own Elastic Network Interface (ENI) and Security Group.

## ---

**3\. Amazon EKS (Elastic Kubernetes Service)**

EKS provides a managed Kubernetes control plane.

* **Control Plane:** AWS manages the API server and etcd; you pay $0.10/hour per cluster.  
* **Data Plane (Worker Nodes):**  
  * **Managed Node Groups:** AWS automates the provisioning and lifecycle of EC2 instances.  
  * **Fargate:** No nodes to manage; pay per pod.  
* **Authentication:** Uses **IAM** for cluster access via the aws-auth ConfigMap (standard) or EKS Access Entries (modern).  
* **IRSA (IAM Roles for Service Accounts):** Provides fine-grained IAM permissions to specific Pods using OIDC. This is the EKS equivalent of the ECS Task Role.  
* **VPC CNI:** Assigns real VPC IP addresses to Pods, allowing them to be first-class citizens in your network.

## ---

**4\. AWS Fargate**

Fargate is the "serverless" engine for both ECS and EKS.

* **No Infrastructure Management:** You do not manage EC2 instances or patching.  
* **Isolation:** Each task/pod runs in its own kernel-isolated environment.  
* **Storage:** Supports ephemeral storage and persistent storage via **Amazon EFS**.  
* **Scaling:** Scales based on the number of tasks. Faster than scaling EC2-backed clusters because you don't wait for "instance warm-up."

## ---

**Practice Questions (DOP-C02)**

Format start\[

**Question 1**

A DevOps Engineer is setting up an ECS cluster using Fargate. The application needs to access a database password stored in AWS Secrets Manager. What is the most secure and efficient way to provide this password to the container?

**Options:**

A) Hardcode the password in the Task Definition.

B) Use the secrets section in the Task Definition to reference the Secrets Manager ARN.

C) Run a script in the container that calls the AWS CLI to fetch the secret.

D) Pass the secret as a plaintext environment variable in the Task Definition.

**Correct answer with explanation:** **B**. ECS has native integration with Secrets Manager. By referencing the ARN in the secrets section, the ECS agent fetches the secret and injects it as an environment variable at runtime, ensuring it's never stored in code or the console.

**Explanation for why other options are incorrect:** A and D: Severe security risks. C: Requires installing the AWS CLI and providing IAM permissions to the container, increasing complexity and the attack surface.

---

**Question 2**

A company has a multi-account setup. The CI/CD pipeline in the "Shared Services" account builds a Docker image and pushes it to an ECR repository. The "Production" account needs to pull this image to run on ECS. What must be configured?

**Options:**

A) Make the ECR repository public.

B) Create an IAM user in the Production account and share its keys.

C) Update the ECR Repository Policy in the Shared Services account to allow ecr:BatchGetImage and ecr:GetDownloadUrlForLayer for the Production account ID.

D) Use S3 to store the image instead of ECR.

**Correct answer with explanation:** **C**. ECR Repository Policies are resource-based policies that explicitly allow cross-account access. The destination account needs these permissions to pull the image layers.

**Explanation for why other options are incorrect:** A: Security risk. B: Sharing long-lived keys is against best practices. D: S3 is not a container registry and lacks the native integration required for ECS/EKS.

---

**Question 3**

An EKS application needs to write logs to an S3 bucket. The DevOps Engineer wants to follow the principle of least privilege by ensuring only the specific "Logging Pod" has access, rather than the entire worker node. How should this be implemented?

**Options:**

A) Attach an IAM role to the EC2 worker node.

B) Use IAM Roles for Service Accounts (IRSA) to associate an IAM role with the Kubernetes Service Account used by the Pod.

C) Hardcode AWS credentials into the Pod's environment variables.

D) Use a NodeSelector to run the pod on a specific instance with a specific role.

**Correct answer with explanation:** **B**. IRSA uses an OIDC provider to allow Kubernetes pods to assume IAM roles directly. This provides fine-grained permissions at the Pod level.

**Explanation for why other options are incorrect:** A: This would give every pod on that node access to the S3 bucket. C: Security risk. D: Inefficient and violates the principle of least privilege if other pods are also on that node.

---

**Question 4**

A DevOps Engineer is implementing a Blue/Green deployment for an ECS service using CodeDeploy. During the test phase, they want to verify the "Green" environment using a separate test port before shifting production traffic. Which component handles this?

**Options:**

A) The Task Definition.

B) The Application Load Balancer (ALB) Test Listener.

C) The Target Group.

D) The ECS Service Auto Scaling policy.

**Correct answer with explanation:** **B**. In an ECS Blue/Green deployment, the ALB usually has a production listener (e.g., port 80\) and a test listener (e.g., port 8080). CodeDeploy uses the test listener to route traffic to the Green set for validation.

**Explanation for why other options are incorrect:** A: Defines the container, not the traffic flow. C: Target groups hold the tasks, but the listener manages the routing logic. D: Auto Scaling manages the number of tasks, not the deployment lifecycle.

---

**Question 5**

An ECS service on Fargate is experiencing intermittent performance issues. The Engineer wants to collect detailed per-process metrics from inside the container. What should they enable?

**Options:**

A) CloudWatch Container Insights.

B) Standard CloudWatch Metrics.

C) ECS Event Bridge integration.

D) Flow Logs.

**Correct answer with explanation:** **A**. Container Insights provides highly granular metrics (CPU, memory, network, and disk) at the cluster, service, and task levels, including pod-level metrics for EKS.

**Explanation for why other options are incorrect:** B: Standard metrics are too high-level (cluster/service only). C: Useful for state changes, not performance monitoring. D: Flow logs are for network traffic metadata, not container internals.

---

**Question 6**

A company wants to run a containerized background job on EKS that should only run once and then terminate. Which Kubernetes resource should be used?

**Options:**

A) Deployment

B) DaemonSet

C) Job

D) ReplicaSet

**Correct answer with explanation:** **C**. The Job resource is designed for short-lived tasks that run to completion.

**Explanation for why other options are incorrect:** A and D: These are designed for long-running web services that stay active. B: Runs one pod on every node in the cluster.

---

**Question 7**

During an ECS Rolling Update, the Engineer notices that the deployment is taking too long because only one task is being replaced at a time. The cluster has plenty of spare capacity. Which parameters should be adjusted to speed up the rollout?

**Options:**

A) minimumHealthyPercent to 50% and maximumPercent to 200%.

B) minimumHealthyPercent to 100% and maximumPercent to 100%.

C) Increase the Task CPU.

D) Change the deployment type to Blue/Green.

**Correct answer with explanation:** **A**. Setting maximumPercent to 200% allows ECS to double the number of tasks during the update, while minimumHealthyPercent at 50% allows it to kill more old tasks simultaneously.

**Explanation for why other options are incorrect:** B: This would prevent the update as no new tasks could start and no old ones could stop. C: CPU affects performance, not deployment speed. D: This is a different deployment model entirely.

---

**Question 8**

What is the primary difference between the **Task Execution Role** and the **Task Role** in ECS?

**Options:**

A) There is no difference; they are the same.

B) Task Role is for the ECS agent; Task Execution Role is for the application.

C) Task Execution Role is for the ECS agent/Fargate infrastructure; Task Role is for the application code.

D) Task Execution Role is only for EC2; Task Role is only for Fargate.

**Correct answer with explanation:** **C**. The **Task Execution Role** allows the container platform (Fargate/ECS Agent) to pull images and push logs. The **Task Role** is assigned to the application running inside the container to access AWS services like S3 or DynamoDB.

**Explanation for why other options are incorrect:** B: It is exactly the reverse. D: Both are used in both launch types.

---

**Question 9**

An ECR repository has hundreds of images, causing high storage costs. The company only needs to keep the 10 most recent images. What should be implemented?

**Options:**

A) A Lambda function that deletes images every night.

B) An ECR Lifecycle Policy.

C) An S3 Lifecycle Policy.

D) Manual deletion via the CLI.

**Correct answer with explanation:** **B**. ECR Lifecycle Policies are built-in and allow you to define rules like "Keep only the last 10 images" or "Delete images older than 30 days."

**Explanation for why other options are incorrect:** A: Overly complex for a native feature. C: ECR does not expose its underlying S3 storage for lifecycle management. D: Not an automated solution.

---

**Question 10**

In EKS, what is the purpose of the **Horizontal Pod Autoscaler (HPA)**?

**Options:**

A) To add more EC2 worker nodes to the cluster.

B) To increase the number of Pod replicas based on CPU or memory usage.

C) To increase the CPU allocated to a single Pod.

D) To move Pods between regions.

**Correct answer with explanation:** **B**. HPA automatically scales the number of Pods in a deployment or replica set based on observed resource utilization.

**Explanation for why other options are incorrect:** A: This is the job of the Cluster Autoscaler (CA) or Karpenter. C: This is the job of the Vertical Pod Autoscaler (VPA). D: EKS clusters are region-specific.

---

**Question 11**

A DevOps Engineer is using ECS with EC2 launch type. One specific task requires 8 vCPUs, but the current EC2 instances only have 4 vCPUs. The task fails to start. How should the Engineer resolve this?

**Options:**

A) Use Fargate instead.

B) Enable Auto Scaling for the ECS Service.

C) Add a new ASG to the Capacity Provider with larger instance types (e.g., c5.4xlarge).

D) Increase the maximumPercent in the deployment configuration.

**Correct answer with explanation:** **C**. Capacity Providers allow ECS to scale the underlying EC2 infrastructure. If the existing instances are too small for the task's requirements, you must provide larger instances.

**Explanation for why other options are incorrect:** A: Possible, but the question implies using EC2. B: Scaling the *service* adds more tasks, but if one task can't fit, adding more won't help. D: This affects the deployment rollout, not resource allocation.

---

**Question 12**

Which ECS networking mode provides the best performance and simplified networking by giving each task its own IP address from the VPC subnet?

**Options:**

A) bridge

B) host

C) awsvpc

D) none

**Correct answer with explanation:** **C**. awsvpc is the modern standard. It assigns an ENI to every task, allowing for full Security Group support and VPC visibility.

**Explanation for why other options are incorrect:** A: Uses Docker's internal virtual network (slower). B: Shares the host's IP, which can cause port conflicts. D: No networking.

---

**Question 13**

How does **AWS App Mesh** enhance a microservices application running on ECS or EKS?

**Options:**

A) By providing a managed registry for Docker images.

B) By providing application-level networking, observability, and traffic control (Service Mesh).

C) By automatically patching worker nodes.

D) By reducing the cost of S3 storage.

**Correct answer with explanation:** **B**. App Mesh is a service mesh that uses the Envoy proxy to manage communication between services, providing features like circuit breaking and fine-grained traffic shifting.

**Explanation for why other options are incorrect:** A: That's ECR. C: That's Managed Node Groups. D: Irrelevant.

---

**Question 14**

A DevOps Engineer needs to run a task on ECS every night at 2:00 AM. What is the most integrated way to trigger this?

**Options:**

A) A cron job on a separate EC2 instance.

B) An Amazon EventBridge (CloudWatch Events) Scheduled Rule.

C) A manual trigger from a developer.

D) An SQS queue with a very long delay.

**Correct answer with explanation:** **B**. EventBridge can be configured with a cron expression to trigger an ECS RunTask action automatically.

**Explanation for why other options are incorrect:** A: Creates a single point of failure and unneeded infrastructure. C: Not automated. D: Not what SQS is designed for.

---

**Question 15**

An EKS cluster is running on Fargate. The Engineer wants to use a Load Balancer to expose the service to the internet. Which controller must be installed in the cluster?

**Options:**

A) The EBS CSI Driver.

B) The AWS Load Balancer Controller.

C) The CoreDNS Controller.

D) The Metrics Server.

**Correct answer with explanation:** **B**. The AWS Load Balancer Controller is required to manage ALBs and NLBs for Kubernetes services and ingresses in an AWS environment.

**Explanation for why other options are incorrect:** A: For persistent storage. C: For service discovery (DNS). D: For resource monitoring.