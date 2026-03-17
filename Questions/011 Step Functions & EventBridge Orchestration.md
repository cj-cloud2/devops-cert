This study sheet targets the "SDLC Automation" and "Monitoring and Logging" domains of the **DOP-C02** exam. At the Professional level, the focus is on **orchestration over choreography**—knowing when to use a state machine versus an event bus to manage complex, distributed workflows.

## ---

**1\. AWS Step Functions**

Step Functions is a serverless orchestrator that allows you to stitch multiple AWS services into business-critical workflows using Amazon States Language (ASL).

### **Points to Remember & Recall:**

* **Standard vs. Express Workflows:**  
  * **Standard:** Long-running (up to 1 year), exactly-once execution, visual history, durable. Use for order processing or human-in-the-loop steps.  
  * **Express:** High-volume (up to 100,000 events/sec), short-duration (up to 5 mins), at-least-once execution. Use for IoT data ingestion or streaming.  
* **Service Integrations:**  
  * **Optimized:** Direct integration with services like Lambda, Batch, SNS, SQS, and SageMaker.  
  * **SDK Integrations:** Ability to call 200+ AWS services directly from ASL without writing Lambda "glue" code.  
* **Error Handling:**  
  * **Retries:** Define ErrorEquals, IntervalSeconds, and MaxAttempts.  
  * **Catch:** Transition to a specific "fallback" state if an error persists.  
* **Wait for Callback (.waitForTaskToken):** Pauses the state machine until an external process (like a human approval via SNS/Lambda) returns a task token.

## ---

**2\. Amazon EventBridge**

EventBridge is a serverless event bus that makes it easy to connect applications using data from your own apps, SaaS apps, and AWS services.

### **Points to Remember & Recall:**

* **Buses:**  
  * **Default:** Receives events from AWS services.  
  * **Custom:** Receives events from your own applications.  
  * **SaaS:** Receives events from partners like Datadog, Zendesk, or Shopify.  
* **Rules & Patterns:**  
  * Rules match incoming events based on a **JSON pattern** and route them to targets (Lambda, SQS, Step Functions).  
* **Pipe:** A point-to-point integration tool (Source $\\rightarrow$ Optional Filtering/Enrichment $\\rightarrow$ Target). Best for SQS/Kinesis to Lambda/Step Functions.  
* **Schema Registry:** Automatically discovers and stores event structures, allowing developers to generate code bindings.  
* **Global Endpoints:** Improves availability by automatically failing over event ingestion to a secondary region.

## ---

**3\. Lambda & API Gateway (The "Front Door")**

API Gateway (APIGW) and Lambda are the core of most serverless APIs.

### **Points to Remember & Recall:**

* **API Gateway Types:**  
  * **REST:** Feature-rich (Request validation, caching, usage plans, transformations).  
  * **HTTP:** Low-latency, cost-effective (up to 70% cheaper than REST), supports OIDC/OAuth2.  
* **Integration Types:**  
  * **Lambda Proxy:** APIGW passes the entire request to Lambda; Lambda must return a specific JSON format.  
  * **Lambda Custom (Integration):** You map specific parts of the request to Lambda inputs and transform the response.  
* **Throttling & Quotas:**  
  * **Account Level:** Default 10,000 RPS across the account.  
  * **Usage Plans:** Limit specific API Keys to prevent "noisy neighbor" issues.  
* **Lambda Concurrency:**  
  * **Reserved:** Limits a function to $X$ concurrent executions (acts as a cost/capacity guardrail).  
  * **Provisioned:** Pre-warms environments to eliminate cold starts.

## ---

**Practice Questions (DOP-C02)**

Format start\[

**Question 1**

A company has a complex order fulfillment process that involves manual inventory checks which can take up to two days to complete. Which AWS service and pattern should be used to manage this workflow?

**Options:**

A) EventBridge using a custom event bus.

B) Step Functions Standard Workflow using .waitForTaskToken.

C) Step Functions Express Workflow with a Wait state.

D) API Gateway with a long timeout.

**Correct answer with explanation:** **B**. Standard Workflows can run for up to a year and the .waitForTaskToken pattern is specifically designed to pause a workflow until an external (human or manual) process sends back a success or failure signal.

**Explanation for why other options are incorrect:** A: EventBridge is for "fire and forget" event choreography, not long-running state management. C: Express workflows are limited to 5 minutes. D: API Gateway has a maximum integration timeout of 29 seconds.

---

**Question 2**

A DevOps Engineer wants to implement a solution where every time a file is uploaded to an S3 bucket, it is validated. If validation fails, an entry is made in a DynamoDB table. The solution must be highly decoupled and require no custom "glue" code to move data between S3 and the validation logic. What is the best approach?

**Options:**

A) S3 Event Notification to SNS.

B) EventBridge Rule matching S3 events, triggering a Step Functions state machine.

C) Lambda function triggered directly by S3.

D) Kinesis Data Firehose streaming S3 logs.

**Correct answer with explanation:** **B**. EventBridge is the AWS-recommended way to handle S3 events (via CloudTrail or S3 Event Notifications). Triggering Step Functions allows you to use SDK integrations to write to DynamoDB directly without needing a Lambda function to act as the "glue."

**Explanation for why other options are incorrect:** A: SNS doesn't perform validation logic. C: Requires writing custom Lambda code to handle the logic and the DynamoDB write. D: Firehose is for streaming data to storage, not for event-driven logic.

---

**Question 3**

An application behind an API Gateway is experiencing sudden spikes in traffic. The backend Lambda function is hitting its concurrency limit, causing 429 "Too Many Requests" errors for users. The Engineer wants to protect the backend while ensuring that no requests are lost, even if they take longer to process. What should be implemented?

**Options:**

A) Increase the Lambda Reserved Concurrency.

B) Use API Gateway caching.

C) Integrate API Gateway with an SQS queue as a buffer, with Lambda processing the queue.

D) Enable Provisioned Concurrency for the Lambda function.

**Correct answer with explanation:** **C**. This is the "Queue-based Load Leveling" pattern. By placing an SQS queue between APIGW and Lambda, you buffer the spikes. Lambda can then process the messages at a steady rate without dropping requests.

**Explanation for why other options are incorrect:** A: This only raises the ceiling but doesn't solve the issue if the spike exceeds the new limit. B: Caching helps if the requests are identical, but doesn't help with unique write operations. D: Prevents cold starts but doesn't handle volume exceeding the concurrency limit.

---

**Question 4**

A developer is using a Step Functions state machine to orchestrate a series of Lambda functions. One of the Lambda functions intermittently fails due to a third-party API being rate-limited. How should the state machine be configured to handle this gracefully?

**Options:**

A) Increase the Lambda timeout.

B) Use a Retry block in the ASL definition with ErrorEquals: \["Lambda.TooManyRequestsException"\] and an exponential backoff.

C) Use a Parallel state to run the Lambda multiple times.

D) Configure the Lambda to use a Dead Letter Queue (DLQ).

**Correct answer with explanation:** **B**. Step Functions native Retry logic is the best way to handle transient errors. Exponential backoff ensures the system doesn't overwhelm the third-party API further.

**Explanation for why other options are incorrect:** A: Timeout handles duration, not errors/retries. C: Running in parallel would worsen a rate-limiting issue. D: DLQs capture the failed event but don't provide the automatic retry logic within the orchestration.

---

**Question 5**

A DevOps Engineer needs to trigger a Lambda function whenever a specific SaaS partner (e.g., Zendesk) creates a new ticket. Which EventBridge feature is used to receive these events?

**Options:**

A) Custom Event Bus

B) Default Event Bus

C) Partner Event Bus

D) EventBridge Pipes

**Correct answer with explanation:** **C**. Partner Event Buses are specifically designed to receive events from supported SaaS providers using an event source managed by the partner.

**Explanation for why other options are incorrect:** A: For your own applications. B: For AWS service events. D: For point-to-point integrations between AWS services.

---

**Question 6**

When using API Gateway with Lambda, what is the difference between a "Proxy Integration" and a "Non-Proxy (Custom) Integration"?

**Options:**

A) Proxy is for REST; Custom is for HTTP APIs.

B) Proxy passes the raw request to Lambda; Custom allows you to transform the request using VTL mapping templates.

C) Proxy is faster; Custom is more secure.

D) Proxy requires an IAM role; Custom does not.

**Correct answer with explanation:** **B**. Proxy integration is simpler as it passes the entire event to Lambda. Custom integration gives the Engineer control to modify headers, parameters, or body before it reaches the backend.

**Explanation for why other options are incorrect:** A, C, and D are factually incorrect regarding the technical definition of these integration types.

---

**Question 7**

How can an Engineer ensure that a Step Functions Express Workflow is triggered only when an S3 object is created AND its size is greater than 10MB?

**Options:**

A) Use an EventBridge Rule with a content-based filter pattern.

B) Check the file size inside the first Lambda function of the state machine.

C) Use S3 Lifecycle policies.

D) Use API Gateway to filter the S3 events.

**Correct answer with explanation:** **A**. EventBridge supports advanced content filtering. You can write a rule pattern that checks specific fields in the event JSON (like object-size) before triggering the target.

**Explanation for why other options are incorrect:** B: This incurs costs for triggering the state machine and Lambda for every file, even those that should be ignored. C and D are not relevant to event-driven orchestration filtering.

---

**Question 8**

In Step Functions, what is the purpose of the Pass state?

**Options:**

A) To skip the next state.

B) To transform input data or inject a fixed JSON object into the workflow without calling an external service.

C) To terminate the state machine successfully.

D) To handle errors from the previous state.

**Correct answer with explanation:** **B**. The Pass state is used to manipulate data, rename keys, or provide "dummy" data for testing without the overhead of calling a Lambda function.

**Explanation for why other options are incorrect:** A: It's a transition, not a "skip" tool. C: That's the Succeed state. D: That's the Catch block functionality.

---

**Question 9**

What is the primary benefit of using **EventBridge Pipes**?

**Options:**

A) To replace Step Functions for complex logic.

B) To provide a simple way to create point-to-point integrations with built-in filtering and enrichment (e.g., SQS to Lambda).

C) To encrypt event data.

D) To increase the throughput of the default event bus.

**Correct answer with explanation:** **B**. Pipes simplify the "glue" code needed to move data from a source (like a queue or stream) to a target, including the ability to filter out unwanted events or enrich data via a Lambda call in the middle.

**Explanation for why other options are incorrect:** A: Pipes are for point-to-point, not complex branching/state management. C and D are not the primary functions of Pipes.

---

**Question 10**

A team wants to deploy a new version of their API using API Gateway. They want to shift 10% of the traffic to the new version and monitor it for errors before shifting the rest. Which APIGW feature should be used?

**Options:**

A) Usage Plans

B) Canary Release Deployment

C) Stages

D) Resource Policies

**Correct answer with explanation:** **B**. API Gateway supports "Canary Release" where you can specify a percentage of traffic to go to a new deployment while keeping the rest on the old one.

**Explanation for why other options are incorrect:** A: For rate limiting. C: Stages (Dev/Prod) are for environment separation, but a "Canary" is a specific deployment sub-feature. D: For access control.