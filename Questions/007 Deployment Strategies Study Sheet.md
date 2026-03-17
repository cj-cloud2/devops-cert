This study sheet covers the high-level **Deployment Strategies** tested in the **DOP-C02** exam. Understanding these is critical because you will often be asked to choose a strategy based on constraints like **downtime**, **rollback speed**, **cost**, and **impact of failure**.

## ---

**1\. Deployment Strategy Comparison Matrix**

| Strategy | AWS Service Support | Downtime | Rollback Speed | Cost | Risk |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **All-at-Once** | Beanstalk, CodeDeploy | Yes | Slow | Low | High |
| **Rolling** | ASG, Beanstalk, ECS | No | Slow | Low | Medium |
| **Rolling with Batch** | Beanstalk, ASG | No | Moderate | Medium | Medium |
| **Blue/Green** | CodeDeploy, Beanstalk, Route 53 | No | **Instant** | High | Low |
| **Canary** | Lambda, ECS, API Gateway | No | Fast | Medium | **Lowest** |
| **Linear** | Lambda, ECS (CodeDeploy) | No | Fast | Medium | Low |

## ---

**2\. Deep Dive & Key "Recall" Points**

### **Blue/Green Deployment**

* **The Concept:** You have two identical environments: "Blue" (Current) and "Green" (New). Once Green is tested, traffic is swapped.  
* **Key Advantage:** **Near-zero downtime** and **instant rollback** (just swap traffic back).  
* **Implementation:** \* **Route 53:** Weighted Routing Policy.  
  * **ALB:** Modify target group weights.  
  * **CodeDeploy:** Native support for ECS and Lambda.  
* **Pro Tip:** If the exam mentions "Database Schema changes," Blue/Green is tricky. You usually need to ensure backward compatibility or use a shared database.

### **Canary & Linear Deployments**

* **Canary:** A small subset of users (e.g., 10%) gets the new version first. If metrics look good, you flip the remaining 90%.  
* **Linear:** Traffic is shifted in equal increments over time (e.g., 10% every 2 minutes).  
* **Lambda/ECS Context:** Managed by CodeDeploy. Uses "Hooks" (like BeforeAllowTraffic or AfterAllowTraffic) to run validation tests before proceeding to the next increment.

### **Rolling vs. Rolling with Additional Batch**

* **Rolling:** Updates instances in the existing ASG. If you have 4 instances and a batch size of 2, you will have only 2 instances healthy during the update (**Reduced Capacity**).  
* **Rolling with Additional Batch:** It spins up *new* instances first (e.g., 2 new), then updates the old ones. This ensures your **Total Capacity** never drops below 100%.

## ---

**3\. Deployment Hooks (CodeDeploy)**

For the DOP-C02, you must know the order for **Lambda/ECS**:

1. **BeforeAllowTraffic:** Run integration tests or pre-warm the environment.  
2. **AllowTraffic:** (Traffic starts shifting).  
3. **AfterAllowTraffic:** Run health checks or smoke tests on the new version.

## ---

**Practice Questions (DOP-C02)**

Format start\[

**Question 1**

A company is deploying a new version of a high-traffic web application on Elastic Beanstalk. The CTO requires that the application maintains 100% of its capacity at all times during the deployment to avoid performance degradation. The company also wants to minimize costs compared to a full environment swap. Which deployment strategy should be used?

**Options:**

A) All-at-Once

B) Rolling

C) Rolling with Additional Batch

D) Immutable

**Correct answer with explanation:** **C**. Rolling with Additional Batch ensures that new instances are launched to handle the load *before* any old instances are taken out of service, maintaining 100% capacity. It is cheaper than Blue/Green (Environment Swap) or Immutable because it doesn't necessarily double the entire infrastructure for a long period; it only adds a small "batch" of instances.

**Explanation for why other options are incorrect:** A: Causes downtime. B: Reduces capacity during the rollout as instances are taken offline to be updated. D: Immutable deployments create a whole new Auto Scaling group; while it maintains capacity, "Rolling with Additional Batch" is the specific Beanstalk feature designed to maintain capacity within an existing environment at a lower temporary cost overhead.

---

**Question 2**

A DevOps Engineer is using CodeDeploy to update a Lambda function. The requirement is to shift 10% of traffic to the new version every 10 minutes until all traffic is shifted. If any CloudWatch Alarms are triggered, the deployment must automatically roll back. Which deployment configuration should be selected?

**Options:**

A) LambdaCanary10Percent10Minutes

B) LambdaLinear10PercentEvery10Minutes

C) LambdaAllAtOnce

D) LambdaCanary10PercentEvery2Minutes

**Correct answer with explanation:** **B**. "Linear" means shifting traffic in equal increments at regular intervals (10% \+ 10% \+ 10%...).

**Explanation for why other options are incorrect:** A: Canary would shift 10% once, wait 10 minutes, then shift the remaining 90% all at once. C: Shifts 100% immediately. D: The interval (2 minutes) and strategy type (Canary) do not match the requirement of equal increments over 10-minute intervals.

---

**Question 3**

You are performing a Blue/Green deployment for an application on ECS using CodeDeploy. You want to run a suite of automated smoke tests against the new "Green" task set *before* any production traffic is shifted to it. Where should you define these tests?

**Options:**

A) In the buildspec.yml file.

B) In the AfterAllowTraffic hook of the appspec.yml.

C) In the BeforeAllowTraffic hook of the appspec.yml.

D) In the ValidateService hook of the appspec.yml.

**Correct answer with explanation:** **C**. The BeforeAllowTraffic hook runs before any production traffic is shifted to the new Green task set. This is the ideal time to run validation tests to ensure the new version is healthy.

**Explanation for why other options are incorrect:** A: buildspec.yml is for the build phase, not deployment hooks. B: AfterAllowTraffic runs *after* traffic has already started shifting, which is too late to prevent initial "bad" traffic. D: ValidateService is a hook for EC2 deployments, not the standard for ECS/Lambda traffic shifting logic in this context.

---

**Question 4**

A team needs to deploy a change to a critical production service. They want to test the new version on a very small percentage of real users to monitor for errors without risking the entire user base. They want the ability to roll back instantly by flipping a single switch. Which strategy is most appropriate?

**Options:**

A) Blue/Green Deployment

B) Canary Deployment

C) Rolling Update

D) All-at-Once

**Correct answer with explanation:** **B**. Canary deployment is the specific strategy for testing on a small subset of users (the "canary") to verify stability before a full rollout. It allows for fast rollback if the canary shows issues.

**Explanation for why other options are incorrect:** A: Blue/Green usually involves a full swap of all users at once (unless combined with weighting, but Canary is the specific term for the "small subset" approach). C: Rolling updates affect all users in batches and are slower to roll back. D: Too high risk.

---

**Question 5**

An application uses an Auto Scaling Group (ASG). The Engineer wants to use a deployment strategy that replaces all instances with new ones in a fresh ASG to ensure that no "configuration drift" from the old instances exists. However, they want to avoid the cost of a full Blue/Green Environment swap. Which Elastic Beanstalk deployment strategy fits this?

**Options:**

A) Rolling

B) Immutable

C) Rolling with Additional Batch

D) Blue/Green

**Correct answer with explanation:** **B**. Immutable deployments ensure a fresh start by creating a temporary ASG and launching a full set of new instances. If they pass health checks, they are moved to the main ASG, and the old ones are terminated. This prevents drift without needing two separate permanent environments.

**Explanation for why other options are incorrect:** A and C: These update existing instances or add to the existing ASG, which might not fully solve configuration drift on the OS level if not handled perfectly. D: This is a full environment swap which is more expensive as it requires keeping two complete environments (including Load Balancers, etc.) until the swap is complete.