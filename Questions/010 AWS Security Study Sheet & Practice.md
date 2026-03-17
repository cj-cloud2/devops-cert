This study sheet covers the "Security, Identity, and Compliance" domain of the **DOP-C02** exam. In a DevOps Professional context, security isn't just about permissions; it’s about **automation**, **rotation**, and **least-privilege at scale**.

## ---

**1\. AWS IAM (Identity and Access Management)**

The core of AWS security. Focus on how identities and policies interact in a distributed system.

### **Points to Remember & Recall:**

* **Policy Types:**  
  * **Identity-based:** Attached to users, groups, or roles.  
  * **Resource-based:** Attached to resources (S3 buckets, KMS keys, SQS queues). These can allow cross-account access without requiring the user to "assume" a role.  
  * **Permission Boundaries:** Sets the maximum permissions an identity can have. Used often to delegate IAM creation to developers without letting them create admin roles.  
* **IAM Roles for Services:**  
  * **EC2 Instance Profiles:** Provides temporary credentials to applications on EC2.  
  * **Cross-Account Roles:** Requires a **Trust Policy** (who can assume the role) and a **Permissions Policy** (what they can do once they assume it).  
* **Conditions:** The "if" statement of a policy.  
  * aws:SourceIp: Restrict access to a specific network.  
  * aws:PrincipalOrgID: Restrict access to only accounts within your AWS Organization.  
  * aws:RequestedRegion: Restrict actions to specific regions.

## ---

**2\. AWS Secrets Manager**

Used to store, distribute, and rotate secrets (API keys, DB passwords).

### **Points to Remember & Recall:**

* **Rotation:** Natively supports rotation for RDS, Redshift, and DocumentDB using a **Lambda function**.  
  * **Strategy:** "Single User" (updates one user) or "Alternating User" (creates a new user, updates the app, deletes the old user—preferred for zero downtime).  
* **Cross-Account Access:** You can attach a **Resource-based policy** to a secret to allow another account to read it.  
* **Versioning:** Uses staging labels like AWSCURRENT, AWSPREVIOUS, and AWSPENDING. This allows applications to transition to new secrets smoothly.  
* **Secret vs. Parameter Store:** Use Secrets Manager if you need **automatic rotation** or **cross-account sharing**. Use SSM Parameter Store for simple configuration and lower costs.

## ---

**3\. AWS KMS (Key Management Service)**

The service for creating and managing cryptographic keys.

### **Points to Remember & Recall:**

* **Key Types:**  
  * **AWS Managed Keys:** (e.g., aws/s3). Free, but you can’t manage the policy or rotation.  
  * **Customer Managed Keys (CMK):** You control the policy, rotation, and deletion. **Required for cross-account access.**  
* **Key Policies:** The primary way to control access to a key.  
  * **Crucial:** If a CMK's policy doesn't explicitly allow the account root or a user access, even an Admin cannot use the key.  
* **Envelop Encryption:** KMS generates a **Data Key**. The Data Key encrypts the data; KMS encrypts the Data Key (using the CMK). Only the encrypted Data Key is stored with the data.  
* **Grants:** A mechanism to allow a service (like EBS or EC2) to use a key temporarily for a specific task.  
* **ViaService:** A condition key (kms:ViaService) that restricts key usage to a specific service (e.g., "This key can only be used if the request comes through S3").

## ---

**Practice Questions (DOP-C02)**

Format start\[

**Question 1**

A DevOps Engineer is setting up a cross-account CI/CD pipeline. The pipeline in Account A needs to pull an encrypted object from an S3 bucket in Account B. The object is encrypted using a Customer Managed Key (CMK) in Account B. What two things must be done to allow Account A to access the object?

**Options:**

A) Make the S3 bucket public and disable KMS.

B) Update the S3 bucket policy in Account B to allow Account A, and update the KMS Key Policy in Account B to allow Account A's root/role to use the key.

C) Use an AWS Managed Key instead of a CMK.

D) Attach an IAM policy to Account A's role and assume a role in Account B.

**Correct answer with explanation:** **B**. For cross-account S3 access with KMS encryption, the caller needs permission from both the S3 bucket policy (to get the object) and the KMS key policy (to decrypt it).

**Explanation for why other options are incorrect:** A: Security risk and doesn't solve the encryption requirement. C: AWS Managed Keys cannot be shared cross-account. D: While "Assuming a Role" works, the prompt asks what is required to allow the *access*; the key policy update is the specific requirement for CMK usage cross-account.

---

**Question 2**

An application running on an EC2 instance needs to access an RDS database. The database password is stored in AWS Secrets Manager. The Engineer wants to rotate the password every 30 days without causing downtime. Which rotation strategy should be used?

**Options:**

A) Single User rotation.

B) Alternating User rotation.

C) Manual rotation via the CLI.

D) Reboot the database every 30 days.

**Correct answer with explanation:** **B**. Alternating User rotation creates a new database user with a new password, updates the secret, and then switches the application. This ensures the old password remains valid until the app is updated, preventing downtime.

**Explanation for why other options are incorrect:** A: Single User rotation updates the existing user's password; there is a small window where the app's old cached password will fail. C: Not automated/DevOps. D: Irrelevant to password management.

---

**Question 3**

A security audit reveals that a developer's IAM user has "AdministratorAccess," but they should only be able to manage S3 and EC2. To prevent this user from escalating their own privileges or creating other admin users, which IAM feature should be applied?

**Options:**

A) IAM Group

B) Permission Boundary

C) Service Control Policy (SCP)

D) Access Analyzer

**Correct answer with explanation:** **B**. A Permission Boundary is an advanced feature used to set the maximum permissions that an identity-based policy can grant to an IAM entity. It prevents the user from gaining more power than the boundary allows, even if they have an "Administrator" policy.

**Explanation for why other options are incorrect:** A: Groups don't set maximum limits. C: SCPs apply at the account/OU level in Organizations, not to a single IAM user. D: This is a tool to identify shared resources, not a preventive control.

---

**Question 4**

A company wants to ensure that a specific KMS CMK is only used when a request comes from Amazon S3 within a specific VPC. Which condition key should be used in the KMS Key Policy?

**Options:**

A) aws:SourceVpc

B) kms:ViaService

C) aws:PrincipalArn

D) kms:EncryptionContext

**Correct answer with explanation:** **B**. The kms:ViaService condition key allows you to restrict the use of a CMK to requests made by specific AWS services on your behalf.

**Explanation for why other options are incorrect:** A: Restricts by the network location, but doesn't necessarily tie it to a specific service like S3. C: Restricts based on the user/role. D: This is for adding non-secret metadata to the encryption for integrity checks.

---

**Question 5**

You are storing a sensitive configuration file in SSM Parameter Store as a SecureString. When the application tries to retrieve the parameter, it receives an "Access Denied" error, even though it has the ssm:GetParameter permission. What is the most likely missing permission?

**Options:**

A) s3:GetObject

B) kms:Decrypt

C) sts:AssumeRole

D) ssm:DescribeParameters

**Correct answer with explanation:** **B**. Because the parameter is a SecureString, it is encrypted using KMS. To retrieve and read the value, the application needs kms:Decrypt permissions for the key used to encrypt the parameter.

**Explanation for why other options are incorrect:** A: Parameter store doesn't use S3 for standard retrieval. C: Not required if the identity already has the ssm permission. D: This lists parameters but doesn't fetch the values.

---

**Question 6**

A DevOps Engineer needs to delegate the ability to rotate a secret in Secrets Manager to a specific team. However, the team should not be able to view the secret's value. Which permission should be **denied**?

**Options:**

A) secretsmanager:RotateSecret

B) secretsmanager:GetSecretValue

C) secretsmanager:DescribeSecret

D) secretsmanager:ListSecrets

**Correct answer with explanation:** **B**. GetSecretValue is the permission that allows a user to see the actual plaintext secret. One can have the permission to trigger or configure rotation without needing to know the secret itself.

**Explanation for why other options are incorrect:** A: Required for the task. C and D: These provide metadata about the secret (like its name or when it was last rotated) but not the secret value itself.

---

**Question 7**

How can you ensure that an IAM Role can only be assumed by an EC2 instance that has a specific tag, Project: Alpha?

**Options:**

A) Add a condition to the IAM Role's Trust Policy.

B) Use a Permission Boundary.

C) Use an SCP on the account.

D) Use a Resource-based policy on the EC2 instance.

**Correct answer with explanation:** **A**. The Trust Policy (AssumeRolePolicyDocument) determines who can assume the role. You can add conditions here based on the tags of the principal attempting to assume the role.

**Explanation for why other options are incorrect:** B: Sets max permissions, doesn't control "who" assumes a role. C: For account-level guardrails. D: EC2 does not use resource-based policies for IAM role assumption.

---

**Question 8**

In KMS, what is the purpose of the "Grant" feature?

**Options:**

A) To permanently give a user access to a key.

B) To allow an AWS service to use a CMK for a specific, often long-running task without changing the key policy.

C) To allow cross-account access via a master account.

D) To encrypt data larger than 4KB.

**Correct answer with explanation:** **B**. Grants are used by services like EBS or EC2 to perform tasks like attaching an encrypted volume. They are more flexible than key policies for programmatic, temporary access.

**Explanation for why other options are incorrect:** A: Key policies are better for permanent user access. C: Key policies handle cross-account. D: Data keys are used for large data; grants are a permission mechanism.

---

**Question 9**

A company wants to enforce that all developers use a specific KMS key for encrypting their EBS volumes. If they try to use any other key, the creation should fail. Where should this be enforced?

**Options:**

A) S3 Bucket Policy

B) IAM Group Policy

C) Service Control Policy (SCP) or IAM Policy with a kms:KeyArn condition.

D) AWS Config

**Correct answer with explanation:** **C**. By adding a "Deny" rule in an IAM policy or SCP for ec2:CreateVolume where the kms:KeyArn does not match the approved key, you can strictly enforce key usage.

**Explanation for why other options are incorrect:** A: Irrelevant to EBS. B: Helpful, but SCP is stronger for organization-wide enforcement. D: Config is detective; it will find the error but won't *prevent* the creation.

---

**Question 10**

What is the "Envelope Encryption" pattern?

**Options:**

A) Encrypting a secret and then mailing it to the user.

B) Encrypting plaintext data with a data key, and then encrypting the data key with a master key.

C) Using two different KMS keys for the same file.

D) Encrypting data once with AES-256 and once with RSA.

**Correct answer with explanation:** **B**. This is the standard AWS encryption pattern. It allows for efficient encryption of large data (using the data key) while maintaining central control (via the master key).

**Explanation for why other options are incorrect:** A, C, and D are incorrect definitions of the envelope encryption technical process.

