This study sheet focuses on the "Governance as Code" and "Configuration Management" aspects of the **DOP-C02** exam. **AWS Config** and **AWS Systems Manager (SSM)** are often tested together in scenarios involving compliance, automated remediation, and managing hybrid environments at scale.

## ---

**1\. AWS Config: The Compliance Auditor**

AWS Config provides a detailed inventory of your AWS resources and their configuration history. It is the "What changed and when?" service.

### **Points to Remember & Recall:**

* **Configuration Recorder & Delivery Channel:** Must be enabled to start tracking. It records changes and delivers them to an S3 bucket and SNS topic.  
* **Config Rules (Managed & Custom):** \* **Managed Rules:** Pre-built rules by AWS (e.g., s3-bucket-public-read-prohibited).  
  * **Custom Rules:** Use AWS Lambda to define proprietary compliance logic.  
* **Remediation (Manual & Automatic):** Uses **SSM Automation Documents** to fix non-compliant resources (e.g., if a bucket is public, trigger an SSM document to make it private).  
* **Multi-Account/Multi-Region Aggregation:** Use **Aggregators** to view compliance status across the entire AWS Organization in a single dashboard.  
* **Advanced Queries:** Use SQL-like syntax to query the current configuration state of your resources (e.g., SELECT resourceId WHERE resourceType \= 'AWS::EC2::Instance' AND configuration.instanceType \= 't2.micro').

## ---

**2\. AWS Systems Manager (SSM): The Operational Hub**

SSM is a collection of capabilities that allow you to manage your infrastructure at scale, including EC2 instances, on-premises servers, and edge devices.

### **Points to Remember & Recall:**

* **SSM Agent:** Must be installed on the target machine. Standard on Amazon Linux, Ubuntu, and Windows AMIs.  
* **Parameter Store:**  
  * Stores configuration data (strings) and secrets (SecureString).  
  * **Standard vs. Advanced:** Advanced allows larger parameters and parameter policies (TTL, NoChange).  
  * **Hierarchies:** Organize by path (e.g., /prod/db/password).  
* **State Manager:** Maintains a defined state (e.g., "This server must always have antivirus installed"). It runs on a schedule.  
* **Patch Manager:** Automates patching. Uses **Patch Baselines** (which patches to install) and **Patch Groups** (which instances to target via tags).  
* **Automation:** Orchestrates complex workflows. Used frequently as the **Remediation target** for AWS Config.  
* **Run Command:** Executes scripts or commands on a fleet of instances without SSH/RDP access.  
* **Session Manager:** Secure, audited shell access. Removes the need for **Bastion Hosts** or opening port 22\.  
* **Maintenance Windows:** Defines a schedule for when disruptive actions (patching, reboots) can occur.

## ---

**Practice Questions (DOP-C02)**

Format start\[

**Question 1**

A DevOps Engineer needs to ensure that all Amazon S3 buckets in an AWS Organization have server-side encryption enabled. If a bucket is created without encryption, it should be automatically remediated. Which combination of services provides the most automated solution?

**Options:**

A) AWS CloudTrail and a Lambda function that encrypts the bucket.

B) AWS Config managed rule s3-bucket-server-side-encryption-enabled with an SSM Automation remediation action.

C) Amazon S3 Event Notifications triggering an SNS topic.

D) An IAM Policy that denies s3:CreateBucket without encryption headers.

**Correct answer with explanation:** **B**. AWS Config is the standard service for compliance monitoring. It can detect the non-compliant state and natively trigger an SSM Automation document to "remediate" the resource (apply the encryption) without writing custom Lambda code.

**Explanation for why other options are incorrect:** A: Requires custom code and is reactive to an API call, not necessarily the state. C: S3 events can notify but don't inherently monitor state or provide remediation. D: This prevents the action but doesn't fix existing buckets or handle cases where encryption is removed later.

---

**Question 2**

An organization has several on-premises servers that need to be managed alongside their EC2 instances using AWS Systems Manager. What is the first step the Engineer must take to allow the SSM service to manage these on-premises servers?

**Options:**

A) Open port 22 in the on-premises firewall for AWS access.

B) Create a hybrid activation in SSM and install the SSM Agent on the on-premises servers.

C) Establish a Site-to-Site VPN between the data center and AWS.

D) Use AWS Direct Connect to map on-premises IP addresses to the VPC.

**Correct answer with explanation:** **B**. To manage non-AWS resources, you must create a "Hybrid Activation" in SSM, which provides an Activation ID and Code. These are used during the SSM Agent installation on the on-premises server to register it with AWS.

**Explanation for why other options are incorrect:** A: SSM uses outbound HTTPS (port 443\) and doesn't require inbound SSH. C and D: While useful for connectivity, they are not the mechanism that registers a server with the SSM service.

---

**Question 3**

A DevOps Engineer wants to store a database password in SSM Parameter Store. The password must be rotated every 30 days. Which SSM feature or integration should be used?

**Options:**

A) SSM State Manager

B) SSM Parameter Store with AWS Secrets Manager integration

C) SSM Parameter Store Advanced parameters with a TTL policy

D) SSM Inventory

**Correct answer with explanation:** **B**. While Parameter Store can store secrets, it does not have built-in rotation. Secrets Manager *does* have native rotation. You can reference Secrets Manager secrets directly from Parameter Store for a unified configuration interface.

**Explanation for why other options are incorrect:** A: State Manager is for OS-level configuration. C: Parameter policies can set expiration (TTL) but cannot trigger a rotation logic; they only notify or delete. D: Inventory is for metadata about the OS/software.

---

**Question 4**

A company requires a log of every command executed by developers on their EC2 instances for auditing purposes. They also want to eliminate the need for SSH keys. Which SSM capability should be implemented?

**Options:**

A) SSM Run Command with S3 logging enabled.

B) SSM Session Manager with logging to CloudWatch Logs or S3.

C) SSM Patch Manager.

D) SSM Automation.

**Correct answer with explanation:** **B**. Session Manager provides interactive shell access without SSH keys. It can be configured to record every keystroke and command to CloudWatch Logs or an S3 bucket for auditing.

**Explanation for why other options are incorrect:** A: Run Command is for non-interactive scripts, not a live developer session. C: For software updates. D: For workflow orchestration.

---

**Question 5**

During an audit, it is found that several EC2 instances are not running the latest approved security patches. How can the Engineer automate the identification and patching of these instances?

**Options:**

A) Use AWS Config to delete non-compliant instances.

B) Define a Patch Baseline in SSM Patch Manager and associate it with a Patch Group via tags.

C) Use SSM Run Command to manually execute yum update on all instances.

D) Use Amazon Inspector to patch the instances.

**Correct answer with explanation:** **B**. Patch Manager uses Patch Baselines to define which patches are approved and Patch Groups (based on tags) to target specific instances. This can be scheduled to run automatically.

**Explanation for why other options are incorrect:** A: Deleting instances is destructive and doesn't solve the patching requirement. C: This is manual and doesn't provide a systematic way to track "approved" patches. D: Inspector finds vulnerabilities but does not perform the patching itself.

---

**Question 6**

A DevOps Engineer is creating an AWS Config custom rule. Which AWS service is required to evaluate the resource configuration against the rule logic?

**Options:**

A) AWS Systems Manager

B) AWS Lambda

C) Amazon CloudWatch

D) AWS Step Functions

**Correct answer with explanation:** **B**. Custom AWS Config rules use AWS Lambda functions to host the logic that determines whether a resource is "COMPLIANT" or "NON\_COMPLIANT."

**Explanation for why other options are incorrect:** A, C, and D: While these services can integrate with Config, they are not the engine used to define custom rule logic.

---

**Question 7**

How can an Engineer prevent an SSM Parameter Store value from being changed unless a specific process is followed?

**Options:**

A) Use an IAM Policy with a Condition that requires a specific tag.

B) Enable "Write-Once" mode in Parameter Store.

C) Use AWS Config to roll back the change.

D) Store the parameter in a private S3 bucket instead.

**Correct answer with explanation:** **A**. IAM is the primary mechanism for controlling access. You can use conditions to restrict who can call PutParameter or require certain tags/Multi-Factor Authentication.

**Explanation for why other options are incorrect:** B: No such mode exists. C: Config can detect and remediate, but IAM is the way to *prevent* the change. D: S3 is not the SSM Parameter Store.

---

**Question 8**

Which SSM capability should be used to ensure that a specific security agent is always installed and running on every new EC2 instance as soon as it joins the cluster?

**Options:**

A) SSM Session Manager

B) SSM State Manager

C) SSM Automation

D) SSM Maintenance Windows

**Correct answer with explanation:** **B**. State Manager allows you to create an "Association" that ensures specific configurations (like installing an agent) are applied to instances and kept in that state over time.

**Explanation for why other options are incorrect:** A: For shell access. C: For one-time or triggered workflows. D: For scheduling when tasks happen, not for maintaining a persistent state.

---

**Question 9**

In AWS Config, what is the purpose of a "Configuration Aggregator"?

**Options:**

A) To combine multiple S3 buckets into one.

B) To collect configuration and compliance data from multiple AWS accounts and regions into a single account.

C) To increase the speed of resource recording.

D) To encrypt Config logs.

**Correct answer with explanation:** **B**. An Aggregator is the multi-account/multi-region visibility tool. It allows a centralized "Security" or "Audit" account to see the compliance status of the entire organization.

**Explanation for why other options are incorrect:** A, C, and D: These are not functions of the Configuration Aggregator.

---

**Question 10**

A developer needs to run a one-time cleanup script across a fleet of 500 EC2 instances. Which SSM feature is most suitable for this, providing the ability to control the rate of execution?

**Options:**

A) SSM Session Manager

B) SSM Run Command with Concurrency and Error Threshold controls.

C) SSM Patch Manager

D) SSM Inventory

**Correct answer with explanation:** **B**. Run Command allows you to execute a script across a fleet and provides "Rate Control" (concurrency) and "Error Thresholds" to stop the execution if too many instances fail.

**Explanation for why other options are incorrect:** A: Interactive/one-by-one. C: Specifically for patches. D: For data collection, not execution.

