## **AWS Monitoring & Logging: Deep Dive Study Sheet (DOP-C02)**

For the **DOP-C02** exam, monitoring is about more than just seeing if a server is "up." You must understand **Observability**—correlating metrics, logs, and traces to find the "why" behind failures, and automating the response to those failures.

### ---

**1\. Amazon CloudWatch: The Foundation**

CloudWatch is the primary monitoring and observability service for AWS.

* **CloudWatch Metrics:**  
  * **Standard Resolution:** 1-minute intervals.  
  * **High Resolution:** Up to 1-second intervals (use StorageResolution: 1 in PutMetricData).  
  * **Statistics:** Average, Sum, Min, Max, and SampleCount. Use p99 (99th percentile) to identify outliers that affect the "tail" of your user base.  
* **CloudWatch Logs:**  
  * **Log Groups & Streams:** Organize by application and instance.  
  * **Metric Filters:** Search logs for patterns (e.g., "ERROR") and turn them into numeric metrics for alarming.  
  * **Logs Insights:** A powerful query language to analyze gigabytes of logs in seconds.  
* **CloudWatch Alarms:**  
  * **Metric Alarms:** Watch a single metric.  
  * **Composite Alarms:** Watch multiple alarms (e.g., "CPU is high" **AND** "IOPS are high"). Reduces "alarm noise."  
* **CloudWatch Events / EventBridge:**  
  * The "system bus" of AWS. Triggers actions (Lambda, SSM, SNS) based on state changes (e.g., "EC2 instance terminated").

### **2\. AWS X-Ray: Distributed Tracing**

X-Ray helps you debug and analyze distributed microservices.

* **Segments & Subsegments:** A segment contains information about the work done by a single service; subsegments provide more granular detail (e.g., a specific database call).  
* **Service Map:** A visual representation of how your microservices interact.  
* **Sampling:** To control costs and performance, X-Ray doesn't trace every request. You define **Sampling Rules** (e.g., "10% of all requests, but 100% of errors").  
* **Instrumentation:** Requires the X-Ray SDK in your code or the X-Ray Daemon (on EC2/ECS).

### **3\. Monitoring & Logging Patterns**

* **Centralized Logging:** Use **Kinesis Data Firehose** to stream logs from multiple accounts/regions into a single S3 bucket or an OpenSearch cluster.  
* **Real-time Analysis:** Use **Kinesis Data Analytics** to perform SQL queries on streaming log data for immediate anomaly detection.  
* **VPC Flow Logs:** Capture IP traffic information to/from network interfaces. Use this to debug Security Group or Network ACL issues.

## ---

**Practice Questions (DOP-C02)**

Format start\[

**Question 1**

A microservices application is experiencing high latency. The DevOps Engineer can see that the overall response time is slow but cannot identify which specific internal service call is the bottleneck. Which service should be used to visualize the end-to-end request flow?

**Options:**

A) CloudWatch Logs Insights

B) Amazon CloudTrail

C) AWS X-Ray

D) VPC Flow Logs

**Correct answer with explanation:** **C**. AWS X-Ray provides distributed tracing, allowing you to see the path of a request as it travels through multiple services and identifying the latency at each hop via a Service Map.

**Explanation for why other options are incorrect:** A: Logs show what happened in a single service but don't correlate requests across multiple services natively. B: Logs API calls, not application-level request latency. D: Logs network traffic at the interface level, not application-level logic.

---

**Question 2**

A DevOps Engineer wants to be alerted if the average CPU utilization of an Auto Scaling Group (ASG) exceeds 80% for more than 5 minutes. However, the alarm should only trigger if the "SystemErrors" metric on the attached Load Balancer is also greater than 5\. Which CloudWatch feature should be used?

**Options:**

A) Metric Math

B) Composite Alarms

C) Metric Filters

D) CloudWatch Logs Insights

**Correct answer with explanation:** **B**. Composite Alarms allow you to combine multiple existing alarms using logical operators (AND, OR). This ensures that the alert only fires when a specific combination of conditions is met.

**Explanation for why other options are incorrect:** A: Metric math can create a new metric from others, but a composite alarm is the specific tool for joining alarm *states*. C: Used to turn log data into metrics. D: A query tool, not an alarming mechanism.

---

**Question 3**

A company requires high-resolution monitoring for a critical payment processing application, with metrics collected every 10 seconds. How should the DevOps Engineer implement this?

**Options:**

A) Enable "Detailed Monitoring" in the EC2 instance settings.

B) Use PutMetricData with a StorageResolution of 10\.

C) Use a CloudWatch Logs Metric Filter.

D) High-resolution monitoring is not supported in CloudWatch.

**Correct answer with explanation:** **B**. By setting the StorageResolution parameter to a value less than 60 (minimum 1\) in the PutMetricData API call, you enable high-resolution metrics.

**Explanation for why other options are incorrect:** A: Detailed monitoring increases resolution to 1 minute, not 10 seconds. C: Metric filters depend on log ingestion speed and aren't designed for high-resolution timing. D: High-resolution metrics are supported up to 1-second resolution.

---

**Question 4**

An application logs error messages in a specific format: \[ERROR\] Code: \<Code\>. The Engineer wants to create a dashboard that shows the total number of errors per hour. What is the most efficient way to do this?

**Options:**

A) Write a Lambda function to parse logs and send metrics to CloudWatch.

B) Create a CloudWatch Logs Metric Filter with the pattern \[ERROR\] and use the resulting metric in a dashboard.

C) Export the logs to S3 and use Amazon Athena to query them.

D) Use VPC Flow Logs to track error packets.

**Correct answer with explanation:** **B**. Metric Filters are a built-in, low-cost feature of CloudWatch Logs that scan log data for patterns and automatically increment a metric when a match is found.

**Explanation for why other options are incorrect:** A: Adds unnecessary cost and complexity for a native feature. C: Good for historical analysis, but not for real-time dashboarding. D: Flow logs do not see application-level log text.

---

**Question 5**

A DevOps Engineer needs to capture every API call made in an AWS account, including the identity of the caller and the source IP address, and store these logs for 7 years to meet compliance requirements. Which service is primarily responsible for this?

**Options:**

A) Amazon CloudWatch

B) AWS CloudTrail

C) AWS Config

D) Amazon Inspector

**Correct answer with explanation:** **B**. CloudTrail is the service that records AWS API calls for an account. These logs can be delivered to an S3 bucket, where S3 Lifecycle policies can be used to retain them for 7 years.

**Explanation for why other options are incorrect:** A: Focuses on performance and application logs. C: Focuses on resource configuration history (the "state"), not the specific API calls (the "action"). D: A vulnerability scanner.

---

**Question 6**

How can a DevOps Engineer identify which IAM user deleted an S3 bucket?

**Options:**

A) Check the S3 Access Logs.

B) Search AWS CloudTrail logs for the DeleteBucket event.

C) Use CloudWatch Metrics for S3.

D) Look at the AWS Config timeline for the S3 bucket.

**Correct answer with explanation:** **B**. CloudTrail captures the "Who, What, When" of API calls. The DeleteBucket event will contain the identity of the IAM user or role that performed the action.

**Explanation for why other options are incorrect:** A: Access logs show who accessed the *data* inside the bucket, not who deleted the bucket itself. C: Shows volume/latency metrics, not identity info. D: Config shows that the bucket was deleted, but doesn't always provide the identity of the actor in the same detail as CloudTrail.

---

**Question 7**

A company wants to stream application logs from 500 EC2 instances into an Amazon OpenSearch Service (formerly Elasticsearch) cluster for real-time searching. What is the most scalable architectural choice?

**Options:**

A) Install the OpenSearch agent on every EC2 instance.

B) Use the CloudWatch Logs agent to send logs to CloudWatch, then use a Lambda function to subscribe and push to OpenSearch.

C) Use Kinesis Data Firehose to collect logs and deliver them directly to the OpenSearch cluster.

D) Send logs to S3 and use a scheduled job to import them into OpenSearch.

**Correct answer with explanation:** **C**. Kinesis Data Firehose is a fully managed service that can ingest massive streams of data and has a native "destination" for Amazon OpenSearch Service, making it highly scalable and simple to manage.

**Explanation for why other options are incorrect:** A: High management overhead. B: Works, but has more moving parts and potential bottlenecks than Firehose. D: Not real-time.

---

**Question 8**

Which X-Ray feature allows an Engineer to see the relationship between a frontend web service, an SNS topic, and a downstream Lambda function?

**Options:**

A) Segments

B) Annotations

C) Service Map

D) Metadata

**Correct answer with explanation:** **C**. The Service Map provides a visual topology of your application's components and their connections, including health and latency for each link.

**Explanation for why other options are incorrect:** A: Refers to the data for a single service. B: Key-value pairs used for searching traces. D: Key-value pairs for additional info, but not indexed for searching.

---

**Question 9**

An application is running on ECS Fargate. The DevOps Engineer wants to see CPU and Memory utilization for each individual container. What should be enabled?

**Options:**

A) CloudWatch Detailed Monitoring

B) CloudWatch Container Insights

C) AWS X-Ray

D) VPC Flow Logs

**Correct answer with explanation:** **B**. Container Insights is specifically designed to provide granular metrics for ECS, EKS, and Fargate at the task and container level.

**Explanation for why other options are incorrect:** A: For EC2 instances, not Fargate. C: For tracing requests. D: For network metadata.

---

**Question 10**

A DevOps Engineer wants to automatically reboot an EC2 instance if its "StatusCheckFailed\_System" metric is 1 for three consecutive 1-minute periods. What is the best way to do this?

**Options:**

A) Create an EventBridge rule that triggers a Lambda function.

B) Create a CloudWatch Alarm and add an EC2 Action to "Reboot this instance."

C) Write a script on the EC2 instance that checks its own status.

D) Use AWS Config to remediate the instance.

**Correct answer with explanation:** **B**. CloudWatch Alarms have a built-in feature called "EC2 Actions" that can natively reboot, terminate, or recover an instance without needing any code or external services.

**Explanation for why other options are incorrect:** A: Possible, but more complex than the native EC2 Action. C: If the system is failing, the script likely won't run. D: Config is for compliance/state, not real-time health-based reboots.

