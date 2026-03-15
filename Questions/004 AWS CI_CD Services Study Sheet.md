## **1\. AWS CodeBuild: The Build & Test Engine**

CodeBuild is a fully managed build service that compiles source code, runs tests, and produces software packages.

### **Points to Remember & Recall:**

* **Buildspec.yml:** The heart of CodeBuild. Must be in the root directory (unless overridden).  
  * **Phases:** install (runtime versions), pre\_build (login to ECR), build (commands), post\_build (push to ECR, package).  
  * **Artifacts:** Defines what files to upload to S3.  
  * **Cache:** Use S3 or local caching for node\_modules or maven dependencies to speed up builds.  
* **Environment:**  
  * **VPC Access:** To access internal resources (like an RDS or a private GitHub Enterprise), CodeBuild needs a VPC configuration (Subnets/Security Groups).  
  * **Compute Types:** Choose based on CPU/Memory needs. GPU-optimized environments are available.  
  * **Privileged Mode:** **Crucial.** Must be enabled to build **Docker images** (allows the daemon to run).  
* **Security:**  
  * **Service Role:** The IAM role CodeBuild assumes. Needs permissions for S3 (artifacts), CloudWatch (logs), and ECR (pushing images).  
  * **Secrets:** Never hardcode. Reference **AWS Secrets Manager** or **SSM Parameter Store** directly in the buildspec.yml.  
* **Logging:** Logs are sent to CloudWatch Logs and/or S3.

## ---

**2\. AWS CodeDeploy: The Deployment Orchestrator**

CodeDeploy automates code deployments to EC2, On-premises, Lambda, or ECS.

### **Points to Remember & Recall:**

* **AppSpec.yml:** \* **EC2/On-premises:** Uses files and hooks (e.g., BeforeInstall, AfterInstall, ApplicationStart).  
  * **Lambda/ECS:** Uses resources and hooks (e.g., BeforeAllowTraffic, AfterAllowTraffic).  
* **Deployment Configurations:**  
  * **AllAtOnce:** Fast but risky (downtime).  
  * **HalfAtATime / OneAtATime:** Maintains availability.  
  * **Linear/Canary (Lambda/ECS):** Shifting traffic gradually (e.g., LambdaCanary10Percent5Minutes).  
* **Deployment Groups:** Define *where* to deploy (tags, ASG names, or specific instances).  
* **Rollbacks:**  
  * **Automatic Rollback:** Can be triggered on deployment failure or when a **CloudWatch Alarm** (e.g., 5XX errors) is breached.  
* **Traffic Re-routing:**  
  * **In-place (EC2):** Updates existing instances.  
  * **Blue/Green (EC2/ECS/Lambda):** Provisions new resources and swaps traffic (via Load Balancer or DNS).

## ---

**3\. AWS CodePipeline: The Workflow Manager**

CodePipeline is the "glue" that connects source, build, and deploy stages.

### **Points to Remember & Recall:**

* **Structure:** Composed of **Stages** (Source, Build, Test, Staging, Production) which contain **Actions**.  
* **Artifacts:**  
  * **Input/Output Artifacts:** Passed between stages via an S3 **Artifact Store**.  
  * **Encryption:** Artifacts are encrypted by default with an AWS Managed Key (KMS), but **Cross-Account** pipelines require a **Customer Managed Key (CMK)**.  
* **Action Types:** Source (GitHub, S3, CodeCommit), Build (CodeBuild, Jenkins), Deploy (CodeDeploy, CloudFormation, S3, ECS, Elastic Beanstalk).  
* **Triggers:**  
  * **Webhooks (V2):** For GitHub/Bitbucket.  
  * **EventBridge:** The standard way to trigger pipelines on S3 or CodeCommit changes.  
* **Advanced Patterns:**  
  * **Manual Approvals:** Stops the pipeline until a human approves (via SNS notification).  
  * **Cross-Account Deployments:** Requires cross-account IAM roles and a CMK in the S3 Artifact Store bucket.

## ---

**Practice Questions (DOP-C02)**

Format start\[

**Question 1**

A DevOps Engineer is building a CI/CD pipeline. The build stage in CodeBuild needs to build a Docker image and push it to Amazon ECR. The build keeps failing during the docker build command with a "cannot connect to the Docker daemon" error. What is the most likely fix?

**Options:**

A) Add the CodeBuild service role to the ECR repository policy.

B) Enable "Privileged Mode" in the CodeBuild project environment settings.

C) Increase the compute size of the CodeBuild environment.

D) Use a custom Docker image as the build environment.

**Correct answer with explanation:** **B**. To build Docker images in CodeBuild, you must enable the "Privileged" flag. This allows the build container to interact with the Docker daemon required for build commands.

**Explanation for why other options are incorrect:** A: This handles permissions, but the error is about the daemon running. C: Compute size doesn't affect daemon permissions. D: Even a custom image requires privileged mode to run Docker-in-Docker.

---

**Question 2**

An application is deployed to an Auto Scaling Group (ASG) using CodeDeploy with an "In-place" deployment. The Engineer wants to ensure that the application is fully warmed up before traffic is sent to it. Which AppSpec hook should be used to run a warmup script?

**Options:**

A) BeforeInstall

B) AfterInstall

C) ValidateService

D) ApplicationStart

**Correct answer with explanation:** **C**. ValidateService is the final hook in the deployment lifecycle. It is used to verify that the service is running correctly and is ready to serve traffic before the instance is marked as "Healthy" in the Load Balancer.

**Explanation for why other options are incorrect:** A and B: These occur before the application is actually started. D: This starts the service, but doesn't guarantee it's ready/warmed up for users.

---

**Question 3**

A company has a CodePipeline in Account A that needs to deploy code to Account B using CodeDeploy. The artifacts are stored in an S3 bucket in Account A. What is a critical requirement for this cross-account setup?

**Options:**

A) The S3 bucket must be public.

B) Use a Customer Managed Key (CMK) for the S3 bucket and give Account B's role access to that key.

C) Use an AWS Managed KMS Key for the S3 bucket.

D) Both accounts must be in the same AWS Organization.

**Correct answer with explanation:** **B**. AWS Managed Keys cannot be shared across accounts. For Account B to decrypt the artifacts stored in Account A, you must use a CMK and update the KMS Key Policy to allow Account B's cross-account role to use it.

**Explanation for why other options are incorrect:** A: Security risk and doesn't solve IAM/KMS issues. C: Managed keys cannot be shared cross-account. D: Helpful for billing/policy, but technically not required for cross-account IAM.

---

**Question 4**

A DevOps Engineer wants to implement a Blue/Green deployment for an application on ECS. They want to shift 10% of traffic to the new version, wait for 10 minutes, and then shift the remaining 90%. Which CodeDeploy configuration should be used?

**Options:**

A) Canary10Percent10Minutes

B) Linear10PercentEvery10Minutes

C) AllAtOnce

D) HalfAtATime

**Correct answer with explanation:** **A**. A "Canary" deployment shifts a small percentage initially and the rest after a specified interval.

**Explanation for why other options are incorrect:** B: This would shift 10% *every* 10 minutes (taking 100 minutes total). C: Shifts 100% immediately. D: Not available for ECS Blue/Green.

---

**Question 5**

A CodePipeline fails at the Source stage because it cannot detect changes in a private GitHub repository. The pipeline uses a GitHub personal access token. What is the AWS-recommended way to resolve this and improve security?

**Options:**

A) Store the token in a public S3 bucket.

B) Use AWS CodeStar Connections to connect to GitHub.

C) Switch the repository to AWS CodeCommit.

D) Hardcode the token in the pipeline JSON definition.

**Correct answer with explanation:** **B**. CodeStar Connections is the modern, secure way to connect to external providers like GitHub. it uses OAuth and does not require managing long-lived tokens in your code or environment.

**Explanation for why other options are incorrect:** A and D: Severe security risks. C: While a solution, it requires migrating the source code, which may not be feasible.

---

**Question 6**

You are using CodeDeploy to update a Lambda function. You want to automatically roll back the deployment if the number of 5XX errors on the new version exceeds a certain threshold. How do you implement this?

**Options:**

A) Add a script in the BeforeAllowTraffic hook to check errors.

B) Configure CloudWatch Alarms and add them to the CodeDeploy Deployment Group's "Rollback" configuration.

C) Manually stop the deployment in the console if you see errors.

D) Use a CodeBuild stage after deployment to test for errors.

**Correct answer with explanation:** **B**. CodeDeploy has a native feature to monitor CloudWatch Alarms during a deployment. If the alarm is triggered, CodeDeploy immediately initiates a rollback to the previous version.

**Explanation for why other options are incorrect:** A: This hook runs before any traffic is even allowed, so there would be no 5XX errors to check yet. C: Not an automated "DevOps" solution. D: Too late; the traffic is already shifted.

---

**Question 7**

A build in CodeBuild is taking 15 minutes, mostly due to downloading large dependencies from the internet. How can you optimize the build time?

**Options:**

A) Switch to a larger compute type.

B) Enable "S3 Cache" or "Local Cache" in the CodeBuild project settings.

C) Move the build to a Lambda function.

D) Split the build into five smaller CodeBuild projects.

**Correct answer with explanation:** **B**. Caching dependencies (like node\_modules or .m2 folders) in S3 or locally on the build host allows subsequent builds to skip the download phase, significantly reducing build time.

**Explanation for why other options are incorrect:** A: A faster CPU won't speed up a network-bottlenecked download significantly. C: Lambda has limited disk space and execution time. D: Increases complexity without solving the underlying download issue.

---

**Question 8**

Which file is used to define the lifecycle hooks and file mapping for a CodeDeploy deployment to an EC2 instance?

**Options:**

A) buildspec.yml

B) appspec.yml

C) template.yaml

D) config.json

**Correct answer with explanation:** **B**. The appspec.yml file is mandatory for CodeDeploy and defines how the deployment should be handled on the target instances.

**Explanation for why other options are incorrect:** A: Used by CodeBuild. C: Used by SAM/CloudFormation. D: General configuration file.

---

**Question 9**

In CodePipeline, you want to ensure that a senior manager approves the deployment to production. Which action type should you add between the Staging and Production stages?

**Options:**

A) Manual Approval

B) Lambda Invoke

C) SNS Publish

D) CodeBuild Test

**Correct answer with explanation:** **A**. The "Manual Approval" action type pauses the pipeline and can send a notification via SNS. The pipeline only resumes once an authorized user clicks "Approve."

**Explanation for why other options are incorrect:** B: This automates a task, but doesn't provide a UI for a manager's "Approval." C: SNS can notify, but it doesn't pause the pipeline flow. D: This is for automated testing.

---

**Question 10**

A CodeBuild project needs to access a database in a private subnet in your VPC. What must be configured for the build to succeed?

**Options:**

A) Enable "Public IP" on the CodeBuild project.

B) Provide the VPC ID, Subnets, and Security Groups in the CodeBuild project configuration.

C) Use an Internet Gateway in the private subnet.

D) Use a NAT Gateway and route all traffic to the database through it.

**Correct answer with explanation:** **B**. To allow CodeBuild to "reach" into your VPC, it must be configured with specific Subnet and Security Group IDs so it can create the necessary ENIs.

**Explanation for why other options are incorrect:** A: Public IPs don't help access private VPC resources. C: Private subnets, by definition, don't have IGWs. D: A NAT Gateway helps a private resource reach the *internet*, not a build service reach a private database.

---

**Question 11**

A developer has checked in a buildspec.yml file that contains SECRET\_KEY: "my-super-secret-password". What is the best practice to fix this?

**Options:**

A) Encrypt the buildspec.yml file.

B) Move the secret to AWS Secrets Manager and reference it using secrets-manager:name.

C) Use an S3 bucket with strict permissions to store the buildspec.

D) Change the password and hope no one saw it.

**Correct answer with explanation:** **B**. CodeBuild has direct integration with Secrets Manager and SSM Parameter Store. You should reference the secret ARN or name, and CodeBuild will fetch it at runtime without exposing it in the logs or source code.

**Explanation for why other options are incorrect:** A: The file still needs to be decrypted for CodeBuild to read it. C: Doesn't prevent the secret from being visible in the CodeBuild console/logs. D: Not a technical fix for the security hole.

---

**Question 12**

You are deploying a web application via CodeDeploy to an ASG. You want to ensure that the deployment fails if even a single instance fails to update. What should you set minimumHealthyHosts to?

**Options:**

A) 100%

B) 0%

C) 50%

D) 1

**Correct answer with explanation:** **A**. Setting minimumHealthyHosts to 100% means that all instances must remain healthy. If any instance fails the update, the requirement is breached and the deployment fails.

**Explanation for why other options are incorrect:** B: The deployment would continue even if all instances fail. C: Allows half the instances to fail. D: Only requires one instance to be alive.

---

**Question 13**

CodePipeline is not starting when a new file is uploaded to the source S3 bucket. What is the most likely reason?

**Options:**

A) S3 Versioning is not enabled on the bucket.

B) The bucket is in a different region.

C) The file name is too long.

D) The pipeline is already running.

**Correct answer with explanation:** **A**. CodePipeline's S3 source action requires **S3 Versioning** to be enabled on the bucket so it can track specific changes and trigger the pipeline correctly.

**Explanation for why other options are incorrect:** B: Cross-region is supported but requires extra configuration; however, versioning is a foundational requirement. C: Irrelevant. D: Pipelines can queue or run concurrent executions depending on settings.

---

**Question 14**

Which environment variable in CodeBuild can be used to identify the unique ID of the build being executed?

**Options:**

A) $CODEBUILD\_BUILD\_ID

B) $BUILD\_NUMBER

C) $UNIQUE\_ID

D) $AWS\_ACCOUNT\_ID

**Correct answer with explanation:** **A**. CODEBUILD\_BUILD\_ID is a standard environment variable provided by the service to identify the specific build execution.

**Explanation for why other options are incorrect:** B: Common in Jenkins, but not CodeBuild. C: Not a standard variable. D: Identifies the account, not the build.

---

**Question 15**

A company wants to reduce the time it takes to deploy a small change. Currently, the "Test" stage takes 20 minutes because it runs the entire suite. What should they implement?

**Options:**

A) Skip the Test stage for small changes.

B) Use CodeBuild's "Batch Build" to run tests in parallel.

C) Increase the CPU of the build server.

D) Run the tests only once a week.

**Correct answer with explanation:** **B**. Batch builds allow you to run multiple builds (or different test suites) in parallel, significantly reducing the total time required for the testing phase.

**Explanation for why other options are incorrect:** A and D: Compromises software quality. C: Parallelization is generally more effective than raw CPU for large test suites.