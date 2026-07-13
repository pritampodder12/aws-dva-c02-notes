# DVA-C02 Practice Exam — Missed Questions Review (Set 2)
*Compiled from a Digital Cloud Training practice exam — 23 questions you got wrong, with explanations.*

---

### Q1 — AWS Developer Tools
A developer is updating an Amazon ECS app that uses an ALB with two target groups and a single listener. There's an AppSpec file in S3 and a CodeDeploy deployment group tied to the ALB and AppSpec file. The developer needs a Lambda function to validate the update before deployment.

Which solution meets these requirements?
- A. Attach a listener to the deployment group. Link the Lambda function to the `AfterAllowTraffic` hook.
- B. Attach a listener to the deployment group. Link the Lambda function to the `BeforeAllowTraffic` hook.
- C. Add a listener to the ALB. Link the Lambda function to the `AfterAllowTraffic` hook.
- D. Add a listener to the ALB. Link the Lambda function to the `BeforeAllowTraffic` hook.

**Answer: D.** Listeners are configured on the ALB itself, not on the deployment group — so any option attaching a listener "to the deployment group" is wrong on that basis alone. For validation logic that must run before production traffic hits the new version, `BeforeAllowTraffic` is the right hook, since it fires before the updated task set starts receiving live traffic; `AfterAllowTraffic` fires only after traffic has already shifted, which is too late for pre-deployment validation.

---

### Q2 — AWS Database
A developer needs to scan a full 50GB DynamoDB table within non-peak hours. About half of the strongly consistent RCUs are used during non-peak hours, and scan duration must be minimized without affecting production workloads.

How can the developer optimize scan execution time?
- A. Use parallel scans while limiting the rate.
- B. Change to eventually consistent RCUs during the scan.
- C. Increase the RCUs during the scan operation.
- D. Use sequential scans.

**Answer: A.** Sequential scans are the default and read one partition at a time, making them slow for large tables. Parallel scans split the table across segments read concurrently, taking advantage of the RCU headroom available during non-peak hours — but must be rate-limited so they don't consume every available RCU and impact concurrent workloads. Simply increasing RCUs would work but costs more, and switching read consistency doesn't address speed or the impact on production capacity.

---

### Q3 — AWS Developer Tools
A developer uses AWS CodeBuild with a buildspec file to build an application into a Docker image, and needs to push that image to Amazon ECR only after each build succeeds.

What should the developer do?
- A. Add a `post_build` phase using the `commands` block to push the image.
- B. Add a `post_build` phase using the `finally` block to push the image.
- C. Add a `post_build` phase using the `artifacts` sequence to find and push the image.
- D. Add an `install` phase using the `commands` block to push the image.

**Answer: A.** `post_build` is meant for exactly this — actions to take only after the main build succeeds, like pushing a completed image to ECR. A `finally` block runs regardless of whether earlier commands failed, which risks pushing a broken image. The `install` phase runs before the build even starts, so pushing there makes no sense, and the `artifacts` sequence is for packaging build output files, not pushing Docker images.

---

### Q4 — AWS Management & Governance
A company uses AWS CodePipeline and is writing a Lambda function to send notifications on state changes of pipeline actions.

Which steps associate the Lambda function with the event source?
- A. Create a trigger from the Lambda console, selecting CodePipeline as the event source.
- B. Create a CloudWatch alarm that monitors CodePipeline status changes and triggers the Lambda function.
- C. Create a CloudWatch Events rule using CodePipeline as an event source.
- D. Create an event trigger and specify the Lambda function from the CodePipeline console.

**Answer: C.** CodePipeline integrates with CloudWatch Events as an event source — state changes stream into CloudWatch Events, and a rule there can route matching events to a Lambda function. CodePipeline can't be configured directly as a Lambda trigger from either the Lambda console or the CodePipeline console, and CloudWatch alarms are for metric thresholds, not discrete state-change events.

---

### Q5 — AWS Database
A developer is designing a fault-tolerant app using EC2 instances behind an ELB, and needs to ensure session data isn't lost if an instance fails.

How can this be achieved?
- A. Enable Sticky Sessions on the load balancer.
- B. Use DynamoDB to perform scalable session handling.
- C. Use SQS to save session data.
- D. Use an Auto Scaling group to automatically launch new instances.

**Answer: B.** Session data needs to live in a shared, durable store outside individual instances so it survives instance failure — DynamoDB is built for this and has ready-made session-handler libraries for several languages. Sticky sessions just route a returning user back to the same instance; it does nothing to preserve data if that instance goes down. SQS is a message queue, not a session store, and Auto Scaling replacing failed instances doesn't recover any data that lived only on the failed one.

---

### Q6 — AWS Security, Identity, & Compliance
An organization sells memorabilia that's illegal in certain countries and needs to restrict website access from those countries.

How can this be achieved?
- A. Create a Web ACL in AWS Shield matching the countries, blocking access.
- B. Create a Web ACL in AWS WAF matching the countries, blocking access.
- C. Create a Web ACL in AWS WAF matching the countries, triggering an SNS notification.
- D. Create a Web ACL in AWS Shield matching the countries, triggering an SNS notification.

**Answer: B.** AWS WAF supports geographic-match rules that can block requests from specified countries — the right tool for this exact use case. AWS Shield is a DDoS protection service, not a geo-blocking tool, so both Shield-based options are off target, and merely sending an SNS notification (without blocking) doesn't actually restrict access at all.

---

### Q7 — AWS Security, Identity, & Compliance
An organization is launching a new IoT service and needs secure communication protocols over the internet for the IoT devices during launch.

How can this be achieved?
- A. Use IoT Core to provide TLS-secured communications by issuing X.509 certificates.
- B. Use IoT Greengrass to enable TLS-secured communications by issuing X.509 certificates.
- C. Use AWS Certificate Manager (ACM) to provide TLS-secured communications and deploy X.509 certificates.
- D. Use AWS Private Certificate Authority (CA) to provide TLS-secured communications and deploy X.509 certificates.

**Answer: C.** ACM provisions X.509 certificates for TLS/SSL and is compatible with IoT services like IoT Core and Greengrass — a direct fit for securing communications over the public internet. AWS Private CA is meant for private/internal PKI rather than internet-facing TLS in this context, and neither IoT Core nor IoT Greengrass is itself a certificate authority — they consume certificates, they don't issue them.

---

### Q8 — AWS Compute
A developer is deploying an AWS Lambda update using CodeDeploy. Which is a valid order of hooks in the `appspec.yaml` file?
- A. `BeforeInstall > AfterInstall > ApplicationStart > ValidateService`
- B. `BeforeAllowTraffic > AfterAllowTraffic`
- C. `BeforeBlockTraffic > AfterBlockTraffic > BeforeAllowTraffic > AfterAllowTraffic`
- D. `BeforeInstall > AfterInstall > AfterAllowTestTraffic > BeforeAllowTraffic > AfterAllowTraffic`

**Answer: B.** For a Lambda (or ECS) deployment, the valid hook set is limited to traffic-shifting validation hooks like `BeforeAllowTraffic` and `AfterAllowTraffic`. Option A's hook sequence (`BeforeInstall`, `AfterInstall`, `ApplicationStart`, `ValidateService`) is the EC2/on-premises pattern, not Lambda. Option D mixes in ECS-specific hooks (`AfterAllowTestTraffic`), and option C's `BeforeBlockTraffic`/`AfterBlockTraffic` hooks are EC2-only and this sequence is also incomplete for that platform.

---

### Q9 — AWS Analytics (Select TWO)
A developer partitions data using Athena to improve query performance. Which two things would counter the benefit of partitioning?
- A. Segmenting data too finely.
- B. Skewing data heavily to one partition value.
- C. Using a Hive-style partition format.
- D. Creating partitions directly from the data source.
- E. Storing the data in S3.

**Answer: A & B.** Over-partitioning (too many small partitions) adds metadata retrieval and processing overhead that can outweigh the benefits, and if most queries hit one heavily skewed partition, that overhead again cancels out the gains. Storing data in S3 is simply a requirement for Athena to work at all, Athena reads directly from an S3 source without issue, and Hive-style partition formats are natively compatible with Athena — none of these three are problems.

---

### Q10 — AWS Management & Governance
A developer writes code to run in a cron job on an EC2 instance to send application status information to CloudWatch.

Which method should be used?
- A. Use the unified CloudWatch agent to publish custom metrics.
- B. Use the AWS CLI `put-metric-data` command.
- C. Use the CloudWatch console with detailed monitoring.
- D. Use the AWS CLI `put-metric-alarm` command.

**Answer: B.** `put-metric-data` is the direct CLI command for publishing custom data points to CloudWatch — simple enough to call from a cron job without extra infrastructure. `put-metric-alarm` creates/updates alarms, not data points. The unified CloudWatch agent is a heavier install that isn't necessary when the CLI alone can do the job from a cron job, and detailed monitoring in the console only changes metric granularity (1-min vs 5-min) — it doesn't let you push custom data at all.

---

### Q11 — AWS Database
An app uses Amazon RDS and needs a caching layer in front of it to improve read performance. The cached data must be encrypted, and the solution must be highly available.

Which solution meets these requirements?
- A. Amazon DynamoDB Accelerator (DAX).
- B. Amazon ElastiCache for Redis in cluster mode.
- C. Amazon ElastiCache for Memcached.
- D. Amazon CloudFront with multiple origins.

**Answer: B.** Redis in cluster mode is the ElastiCache engine that supports both encryption and high availability, making it the only option that checks both required boxes. Memcached doesn't support encryption or HA. DAX is a cache specifically for DynamoDB, not RDS, and CloudFront is a CDN — it can't sit "in front of" a database as a query cache, and there's no second origin to speak of with a single database.

---

### Q12 — AWS Analytics
A developer runs queries on Hive-compatible partitions in Athena using DDL but hits timeout issues.

What is the most effective way to prevent this?
- A. Use `ALTER TABLE ADD PARTITION` to update column names.
- B. Export data to a JSON document, clean errors, and re-upload to S3.
- C. Export data to DynamoDB for more flexible schema querying.
- D. Use `MSCK REPAIR TABLE` to update the metadata catalog.

**Answer: D.** `MSCK REPAIR TABLE` scans S3 for new Hive-compatible partitions and syncs them into the table's metadata in one operation, which scales better than manual DDL when there are large numbers of partitions and avoids the timeout issue. `ALTER TABLE ADD PARTITION` is for adding partition columns, not for bulk-syncing existing S3 partitions. Exporting to JSON or DynamoDB doesn't address the root cause (partition metadata sync) and adds unnecessary data-migration overhead.

---

### Q13 — AWS Compute
A team wants to run containers on Amazon ECS where each application container shares data with another container for logs and metrics.

What should the team do?
- A. Create two task definitions, one per container, and mount a shared volume between the two tasks.
- B. Create two pod specifications, linking the pods together.
- C. Create a single pod specification including both containers, with a persistent volume.
- D. Create one task definition specifying both containers, with a shared volume mounted between them.

**Answer: D.** ECS lets you define multiple containers plus shared Docker volumes within a single task definition — that's the natural mechanism for sharing data between containers that run together. "Pods" are a Kubernetes/EKS concept and don't apply to ECS at all, and splitting the containers across two separate task definitions defeats the purpose since a shared volume is meant to connect containers within the same task.

---

### Q14 — AWS Compute
An app on Elastic Beanstalk sees increased error rates during deployments due to reduced capacity during deployment steps. The team wants to maintain full capacity during deployment while reusing existing instances.

Which deployment policy meets this requirement?
- A. Rolling with additional batch.
- B. Rolling.
- C. Immutable.
- D. All at once.

**Answer: A.** "Rolling with additional batch" launches an extra batch of instances first, so capacity never dips below normal while the existing instances are updated in batches. Plain "Rolling" updates existing instances directly, which does reduce capacity for the batch currently being updated. "All at once" updates every instance simultaneously, causing a full outage during deployment, and "Immutable" deploys onto brand-new instances in a separate Auto Scaling group rather than the existing ones — which doesn't match the "using existing instances" requirement.

---

### Q15 — AWS Compute
A company deploys a custom AMI-based app via CloudFormation, currently running in us-east-1, and wants to extend it to us-west-1. The stack creation fails there with an "AMI ID does not exist" error.

What should the developer do to fix this with minimal complexity?
- A. Copy the AMI from us-east-1 to us-west-1 and use the new AMI ID in the template.
- B. Use Lambda to create an AMI in us-west-1 during stack creation.
- C. Modify the template to refer to the AMI in us-east-1.
- D. Create a new AMI from scratch in us-west-1 and update the template.

**Answer: A.** AMIs are region-specific resources, so the us-east-1 AMI simply doesn't exist in us-west-1 — copying it across regions preserves the exact same image with minimal effort. Referencing the us-east-1 AMI directly from a us-west-1 template won't work since cross-region AMI references aren't supported. Building a brand-new AMI from scratch (whether manually or via a Lambda-driven process during stack creation) is more work and risks producing an image that isn't identical to the original.

---

### Q16 — AWS Compute
A developer is setting up a code update to Amazon ECS using CodeDeploy and needs the update to complete quickly.

Which deployment type should be used?
- A. Canary.
- B. Linear.
- C. Blue/green.
- D. In-place.

**Answer: C.** ECS deployments through CodeDeploy only support the Blue/Green deployment type — in-place isn't available for Lambda or ECS at all. "Canary" and "Linear" are traffic-shifting configurations you choose within a blue/green deployment (how traffic ramps between the old and new task sets), not deployment types themselves.

---

### Q17 — AWS Security, Identity, & Compliance
A company migrating to AWS needs a managed Public Key Infrastructure (PKI) supporting IAM integration, CloudTrail auditing, private certificates, and subordinate CAs.

Which solution meets these requirements?
- A. AWS Secrets Manager.
- B. AWS Key Management Service.
- C. AWS Certificate Manager.
- D. AWS Private Certificate Authority.

**Answer: D.** AWS Private CA is purpose-built for standing up a private PKI hierarchy — supporting root and subordinate CAs, integrating directly with IAM for access control, and logging activity through CloudTrail. Regular ACM issues public/private certificates but isn't itself a CA hierarchy management tool with subordinate CA support. KMS handles encryption keys, not certificate authorities, and Secrets Manager is for storing secrets like credentials, unrelated to PKI.

---

### Q18 — AWS Management & Governance (Select TWO)
CloudWatch metrics show a high number of reads on a primary Aurora MySQL database.

What can a developer do to improve read scaling?
- A. Create Aurora Replicas in a global S3 bucket as the primary read source.
- B. Create a separate Aurora MySQL cluster and configure binlog replication.
- C. Create Aurora Replicas in the same cluster as the primary database instance.
- D. Create a duplicate Aurora primary database to process read requests.
- E. Create a duplicate Aurora database cluster to process read requests.

**Answer: B & C.** Aurora Replicas within the same cluster synchronously stay near up-to-date with the primary (typically within ~100ms) and are the built-in mechanism for read scaling. For cross-cluster (even cross-region) read scaling, Aurora MySQL also supports binlog replication to a separate cluster. A duplicate cluster or duplicate primary would just be separate read/write databases, not a read-scaling mechanism, and S3 is object storage — it can't host a database or serve as a read replica target.

---

### Q19 — AWS Application Integration
A company's Step Functions state machine hits errors in a task state. To troubleshoot, the developer needs the original state input preserved alongside the error message in the state output.

Which coding practice achieves this?
- A. Use `ResultPath` in a `Catch` statement to include the original input with the error.
- B. Use `ErrorEquals` in a `Retry` statement to include the original input with the error.
- C. Use `InputPath` in a `Catch` statement to include the original input with the error.
- D. Use `OutputPath` in a `Retry` statement to include the original input with the error.

**Answer: A.** `ResultPath` controls how a state's result is merged with its input rather than replacing it — used inside a `Catch` block, it lets the error get combined with (not overwrite) the original input in the output. `InputPath` only selects a portion of the input, not merges error data in. `ErrorEquals` belongs to the retry policy matching logic, not to preserving data, and `OutputPath` just filters what's passed to the next state — none of these combine input with error the way `ResultPath` does.

---

### Q20 — AWS Security, Identity, & Compliance
A business gives clients read-only access to their own files in an S3 bucket via IAM permissions, and regulatory compliance requires in-transit encryption for all S3 communication.

What solution fulfills these requirements?
- A. Enable S3 server-side encryption to enforce in-transit encryption.
- B. Update the bucket policy to require `aws:SecureTransport` for all actions.
- C. Assign IAM roles enforcing SSL/TLS encryption to each customer.
- D. Enable S3 Transfer Acceleration to ensure encryption in transit.

**Answer: B.** A bucket policy condition requiring `aws:SecureTransport` explicitly denies any request that isn't made over SSL/TLS, directly enforcing in-transit encryption for every interaction with the bucket. Server-side encryption only protects data at rest, not in transit. IAM roles control access permissions but don't inherently mandate an encrypted transport protocol, and Transfer Acceleration speeds up long-distance transfers using TLS as a side effect — it isn't designed to enforce encryption as a compliance control.

---

### Q21 — AWS Database
An ecommerce storefront uses API Gateway + Lambda + RDS for MySQL, with transaction volume that spikes sporadically during marketing campaigns and drops near zero otherwise.

How can a developer increase elasticity most cost-effectively?
- A. Migrate to Aurora MySQL; scale read replicas based on average CPU utilization.
- B. Create an SQS queue to invoke Lambda, with reserved concurrency equal to max DB connections.
- C. Create an SNS topic with an SQS queue destination, and have Lambda process from the queue.
- D. Migrate to Aurora MySQL; scale read replicas based on average connections of Aurora Replicas.

**Answer: D.** Average connections to Aurora Replicas correlates most directly with actual transaction load, making it the most accurate signal for an Auto Scaling policy that adds/removes replicas as real demand changes. CPU utilization is a weaker proxy for this specific bursty-transaction pattern. Reserved concurrency on Lambda would mean paying for that capacity constantly, even during quiet periods — the opposite of cost-effective elasticity — and adding an SNS topic on top of SQS adds unnecessary complexity with no real benefit here.

---

### Q22 — AWS Developer Tools
A developer built an app using Lambda, DynamoDB, and SNS notifications, and needs to analyze what's happening across all components since the app isn't working as expected.

What is the best way to analyze the issue?
- A. Create a CloudWatch Events rule.
- B. Assess the application with Amazon Inspector.
- C. Monitor the application with AWS Trusted Advisor.
- D. Enable X-Ray tracing for the Lambda function.

**Answer: D.** X-Ray provides end-to-end, cross-service tracing — aggregating data from each component into a single trace and a visual service map, which is exactly what's needed to pinpoint where a multi-service issue is occurring. CloudWatch Events triggers actions on state changes, not deep tracing. Amazon Inspector is a security/vulnerability assessment tool, and Trusted Advisor gives best-practice guidance — neither is built for tracing a request's path across services.

---

### Q23 — AWS Database
A critical app uses API Gateway + Lambda + RDS for PostgreSQL (2 vCPUs, 16 GB RAM). Customers intermittently see HTTP 500 errors during unpredictable peak times, with CloudWatch Logs showing "connection limit exceeded" errors. The company wants resilience with no unscheduled database downtime.

Which solution best fits these requirements?
- A. Double the RAM and vCPUs of the RDS instance.
- B. Use Lambda to create a connection pool for the RDS instance.
- C. Implement auto-scaling for the RDS instance based on connection count.
- D. Use Amazon RDS Proxy and update the Lambda function to connect to the proxy.

**Answer: D.** RDS Proxy pools and shares database connections, absorbing bursts in Lambda-driven connection spikes without exhausting the database's own connection limit or adding load to the DB itself — with no downtime required to fix it. Bigger vCPU/RAM doesn't directly solve a too-many-connections problem. RDS doesn't offer native auto-scaling based on connection count, and rolling your own connection pool inside Lambda is both costlier and constrained by Lambda's execution-time limits compared to using the purpose-built RDS Proxy.

---

## Quick Answer Key

| # | Answer |
|---|--------|
| 1 | D |
| 2 | A |
| 3 | A |
| 4 | C |
| 5 | B |
| 6 | B |
| 7 | C |
| 8 | B |
| 9 | A, B |
| 10 | B |
| 11 | B |
| 12 | D |
| 13 | D |
| 14 | A |
| 15 | A |
| 16 | C |
| 17 | D |
| 18 | B, C |
| 19 | A |
| 20 | B |
| 21 | D |
| 22 | D |
| 23 | D |
