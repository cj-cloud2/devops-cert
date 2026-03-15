## **AWS CLI & CDK: Deep Dive Study Sheet (DOP-C02)**

For the **AWS Certified DevOps Engineer – Professional (DOP-C02)** exam, the AWS CLI and CDK are not just tools—they are the primary interfaces for automating complex multi-account and multi-region environments. You must understand their internal mechanics, configuration precedence, and deployment lifecycles.

### ---

**1\. AWS CLI (Command Line Interface)**

The CLI is the tactical tool for scripting and quick infrastructure manipulation.

* **Configuration Precedence (The Order of Truth):**  
  1. **Command Line Options:** Highest priority (e.g., \--region, \--profile).  
  2. **Environment Variables:** (e.g., AWS\_ACCESS\_KEY\_ID, AWS\_REGION).  
  3. **Config/Credentials Files:** (Located in \~/.aws/).  
  4. **Instance Metadata:** (IAM Role attached to EC2/Lambda).  
* **Key Environment Variables:**  
  * AWS\_PROFILE: Selects a specific named profile.  
  * AWS\_DEFAULT\_REGION: Sets the target region.  
* **Output Control & Filtering:**  
  * \--output: json, text, table, or yaml.  
  * \--query: Uses **JMESPath** to filter results on the client side (e.g., Reservations\[\*\].Instances\[\*\].InstanceId).  
  * \--filter: Server-side filtering (more efficient, reduces data transfer).  
* **Complex CLI Patterns:**  
  * **Assuming Roles:** Configured in \~/.aws/config using role\_arn and source\_profile.  
  * **Shorthand Syntax:** Used for JSON parameters (e.g., Key=value,Key2=value2).  
  * **Wait Commands:** aws ec2 wait instance-running—essential for bash-based automation.

### **2\. AWS CDK (Cloud Development Kit)**

CDK allows you to define IaC using familiar programming languages (TypeScript, Python, Java, etc.), which is then synthesized into CloudFormation templates.

* **The CDK App Lifecycle:**  
  1. **Construction:** The code is executed, and the "tree" of constructs is built.  
  2. **Preparation:** Constructs adjust themselves based on the final state.  
  3. **Synthesis:** Produces the **Cloud Assembly** (CloudFormation templates, assets).  
  4. **Deployment:** Assets are uploaded to S3/ECR, and templates are submitted to CloudFormation.  
* **Construct Levels:**  
  * **L1 (Cfn Resources):** 1:1 mapping to CloudFormation resources (prefixed with Cfn).  
  * **L2:** Higher-level abstractions with "sensible defaults" and helper methods (e.g., vpc.addInterfaceEndpoint).  
  * **L3 (Patterns):** Curated solutions for specific tasks (e.g., ApplicationLoadBalancedFargateService).  
* **Bootstrapping:**  
  * Required once per **Account/Region** combination.  
  * Creates an S3 bucket (for templates/files), ECR repo (for Docker images), and IAM roles (for deployment).  
  * Use cdk bootstrap \--trust \<AccountID\> for cross-account deployments.  
* **Key CLI Commands:**  
  * cdk synth: Generates the template (check for logic errors).  
  * cdk diff: Compares local code with the deployed stack.  
  * cdk deploy: Deploys changes (implies synth).  
  * cdk watch: Continuously monitors for changes and hot-swaps (Lambda/ECS).

## ---

**Practice Questions (DOP-C02 Format)**

Format start\[

**Question 1**

A DevOps Engineer is running a bash script on an EC2 instance that uses the AWS CLI to describe RDS instances. The script is failing with "Access Denied." The instance has an IAM Role with full RDS permissions, but the Engineer also has a legacy access key configured in \~/.aws/credentials with no permissions. What is the most efficient way to ensure the script uses the IAM Role?

**Options:**

A) Run aws configure and delete the access keys.

B) Set the AWS\_ACCESS\_KEY\_ID environment variable to an empty string.

C) Use the \--profile default flag in every command.

D) Delete the \~/.aws/credentials file or unset the credentials environment variables.

**Correct Answer with explanation:** **D**. Because the CLI follows a strict precedence (Credentials file \> Instance Metadata), the local credentials file is overriding the IAM Role. Deleting the file or unsetting the variables forces the CLI to fall back to the Instance Metadata Service (IMDS).

**Explanation for why other options are incorrect:** A: Re-configuring takes time and isn't a "clean" programmatic solution. B: Setting it to an empty string will cause the CLI to error out searching for a valid key rather than falling back. C: If the default profile exists in the file, it will still point to the unauthorized keys.

---

**Question 2**

A company is using AWS CDK to manage a multi-stack application. Stack A creates a VPC, and Stack B creates an RDS instance within that VPC. When attempting to deploy, the Engineer receives an error stating that Stack B cannot find the VPC ID. How should the Engineer link these stacks?

**Options:**

A) Use Fn::ImportValue inside the CDK code to manually pull the VPC ID.

B) Pass the VPC object from Stack A as a property to the constructor of Stack B.

C) Use the AWS CLI to describe the VPC and pass it as a CDK Context variable.

D) Hardcode the VPC ID in the Stack B configuration file.

**Correct Answer with explanation:** **B**. CDK handles cross-stack references automatically. By passing the resource object (the VPC) from one stack to another, CDK creates the necessary Export in Stack A and Fn::ImportValue in Stack B's synthesized templates.

**Explanation for why other options are incorrect:** A: While functional, it defeats the purpose of CDK's object-oriented nature and is prone to typos. C: This adds unnecessary complexity and external dependencies. D: Hardcoding prevents environment reusability (Dev/Test/Prod).

---

**Question 3**

A DevOps Engineer needs to deploy a CDK application to a "Production" account from a "Shared Services" CI/CD account. The Production account has never been used with CDK before. Which command must be run first?

**Options:**

A) cdk deploy \--profile production

B) cdk bootstrap aws://ProductionAccountID/Region \--trust SharedServicesAccountID

C) aws cloudformation create-stack \--stack-name CDKToolkit

D) cdk synth \--context env=prod

**Correct Answer with explanation:** **B**. Before any deployment, the environment must be bootstrapped. To allow a different account (Shared Services) to deploy into the Production account, the \--trust flag must be used during bootstrapping to establish the cross-account IAM relationship.

**Explanation for why other options are incorrect:** A: Deployment will fail if the bootstrap resources (S3/ECR) don't exist. C: While cdk bootstrap creates a stack named CDKToolkit, you should use the CDK CLI to ensure the correct modern bootstrap template is used. D: Synthesis only checks the template locally; it doesn't prepare the remote account.

---

**Question 4**

You want to view only the InstanceId and PublicIpAddress of all running EC2 instances using the AWS CLI, formatted as a table. Which command is correct?

**Options:**

A) aws ec2 describe-instances \--filters Name=instance-state-name,Values=running \--output table

B) aws ec2 describe-instances \--query 'Reservations\[\*\].Instances\[\*\].{ID:InstanceId,IP:PublicIpAddress}' \--output table

C) aws ec2 describe-instances \--filter "running" \--query "InstanceId, PublicIpAddress"

D) aws ec2 list-instances \--output table \--query InstanceId

**Correct Answer with explanation:** **B**. The \--query parameter uses JMESPath to select specific fields and rename them for the output. The {Key:Value} syntax is required to map these into a table format correctly.

**Explanation for why other options are incorrect:** A: This filters for running instances but returns every single attribute of the instance, not just the two requested. C: The syntax for \--filter is incorrect, and the query is missing the array mapping. D: list-instances is not a valid AWS CLI command for EC2.

---

**Question 5**

An Engineer is refactoring a CDK application. They have renamed a high-level L3 Construct (e.g., StaticWebsite) which contains an S3 bucket. What will happen during the next cdk deploy?

**Options:**

A) CDK will rename the S3 bucket in place.

B) CloudFormation will detect the logical ID change and replace (Delete and Recreate) the S3 bucket.

C) CDK will throw an error stating the resource already exists.

D) Nothing; CDK tracks resources by their physical IDs, not logical IDs.

**Correct Answer with explanation:** **B**. CDK generates logical IDs based on the construct's path in the tree. Renaming a construct changes its path, which changes the logical ID in the synthesized template. CloudFormation treats a new logical ID as a new resource and will attempt to delete the old one.

**Explanation for why other options are incorrect:** A: S3 buckets cannot be renamed; they must be replaced. C: It won't error; it will proceed with a replacement unless a DeletionPolicy prevents it. D: CloudFormation relies heavily on logical IDs to track stack state.

---

**Question 6**

Which CDK CLI command is most useful for a "Sanity Check" to ensure that your code doesn't have syntax errors and will produce a valid CloudFormation template without actually touching your AWS environment?

**Options:**

A) cdk synth

B) cdk diff

C) cdk bootstrap

D) cdk metadata

**Correct Answer with explanation:** **A**. cdk synth executes the code and generates the CloudFormation template in the cdk.out directory. If there are code errors or logical impossibilities (like a VPC with no subnets), synth will fail.

**Explanation for why other options are incorrect:** B: diff requires access to the current state in the AWS account. C: bootstrap creates resources in the account. D: metadata provides info about the constructs but doesn't validate the full template synthesis.

---

**Question 7**

A DevOps team is using the AWS CLI to automate the rotation of IAM access keys. They need to get the current user's Account ID to construct a resource ARN. Which command is the standard way to retrieve this?

**Options:**

A) aws iam get-user

B) aws sts get-caller-identity \--query Account \--output text

C) aws organizations describe-account

D) aws ec2 describe-instances \--query AccountId

**Correct Answer with explanation:** **B**. sts get-caller-identity is the universal way to find the Account, User ID, and ARN currently being used by the CLI/SDK, regardless of whether you are using a user or a role.

**Explanation for why other options are incorrect:** A: This only works if you are an IAM user; it fails if you are using an assumed role. C: This requires Organizations permissions and doesn't tell you "who" you currently are. D: EC2 metadata doesn't provide account details through this specific command.

---

**Question 8**

In a CDK application, you need to store a database password that changes frequently. What is the best practice for accessing this value in your CDK code?

**Options:**

A) Hardcode the password in cdk.json.

B) Use Secret.fromSecretNameV2 to reference an existing AWS Secrets Manager secret.

C) Pass the password as a CLI parameter using \-c password=xxxx.

D) Store the password in an environment variable on the developer's machine.

**Correct Answer with explanation:** **B**. This is the most secure and scalable method. It creates a dynamic reference in the synthesized template so the actual password is only retrieved by the resource (like RDS) at deployment/runtime, not stored in the code.

**Explanation for why other options are incorrect:** A: Massive security risk. C: Passwords will be visible in the shell history and CI/CD logs. D: This doesn't work for automated pipelines where the environment variables aren't present.

---

**Question 9**

You are trying to deploy a CDK stack, but it fails with an error: "This stack uses assets, so the toolkit stack must be deployed to the environment." What does this mean?

**Options:**

A) Your IAM role is missing the S3:PutObject permission.

B) You must run cdk bootstrap in the target account/region.

C) You need to manually create an S3 bucket named cdk-assets.

D) Your CDK version is incompatible with the template.

**Correct Answer with explanation:** **B**. Assets (like Lambda code or Docker files) require the bootstrap resources (specifically the S3 bucket and ECR repo) to be staged before CloudFormation can use them.

**Explanation for why other options are incorrect:** A: While true, the specific error message mentioned is the standard CDK response when the CDKToolkit stack is missing. C: cdk bootstrap creates this bucket with the correct naming convention and permissions; manual creation is not recommended. D: This error is specifically about missing infrastructure, not software versions.

---

**Question 10**

A DevOps Engineer wants to ensure that a CDK stack update does not proceed if it would result in the deletion of an RDS instance. Which feature provides a "safety check" *before* the deployment starts?

**Options:**

A) cdk diff

B) TerminationProtection

C) Stack Policies

D) cdk synth

**Correct Answer with explanation:** **A**. cdk diff compares the local code against the deployed stack and explicitly highlights resources that will be "destroyed" or "replaced." This allows the engineer to stop the deployment manually if the impact is unexpected.

**Explanation for why other options are incorrect:** B: This prevents the *entire stack* from being deleted, but doesn't stop individual resource replacement during an update. C: These are applied at the CloudFormation level and will stop the update *during* execution, but cdk diff is the tool for checking *before* starting. D: Synthesis only shows what the template *will* look like, not how it compares to the current state.

---

