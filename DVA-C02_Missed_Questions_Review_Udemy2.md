# DVA-C02 Practice Exam — Missed Questions Review (Set 3)
*Compiled from a Digital Cloud Training practice exam — 26 questions you got wrong, with explanations.*

---

### Q1 — AWS Management & Governance
A company manages multiple AWS accounts across multiple regions. The operations team wants a single operational dashboard showing key performance metrics from all these accounts and regions.

What is the simplest solution?
- A. Create a CloudWatch dashboard in one account/region and import data from the others.
- B. Create a Lambda function that collects metrics from each account/region and pushes them to the dashboard's account.
- C. Create a CloudWatch cross-account cross-Region dashboard.
- D. Create a CloudTrail trail covering all regions, deliver logs to one S3 bucket, and build a dashboard from that data.

**Answer: C.** CloudWatch natively supports cross-account, cross-Region dashboards that summarize data from multiple accounts and regions into a single view — no custom data pipeline needed. Manually importing data or writing a Lambda collector duplicates functionality CloudWatch already provides. CloudTrail logs API activity, not performance metrics, so it can't power a performance dashboard at all.

---

### Q2 — AWS Networking & Content Delivery
An organization wants minimal latency for customers on both the east and west coasts of the US.

What can they do to provide optimal performance with minimal latency?
- A. Use CloudWatch to monitor traffic spikes and scale horizontally.
- B. Use Route 53 with latency-based routing, creating latency records in multiple AWS Regions.
- C. Use CloudTrail to monitor traffic origin and deploy the application to those regions.
- D. Use Route 53 to direct traffic based on health checks.

**Answer: B.** Latency-based routing directs each user to the AWS Region that gives them the lowest latency, which directly matches the "minimal latency" goal — and requires deploying to multiple regions with latency records configured. Health checks only confirm an endpoint is up; they don't pick the fastest one. CloudTrail tracks API calls, not user traffic origins, and CloudWatch monitors usage spikes for scaling, not end-user latency.

---

### Q3 — AWS Networking & Content Delivery
A company runs a website behind CloudFront with an ALB origin. A developer wants custom HTTP responses for 404 errors (content removed from origin) that redirect users to another page, using a Lambda@Edge function, with minimal resources.

Which CloudFront event type should invoke the function?
- A. Origin response
- B. Viewer request
- C. Viewer response
- D. Origin request

**Answer: A.** The origin-response event fires when CloudFront receives the HTTP response from the origin — exactly where you can intercept a 404 and rewrite it into a redirect (e.g., a 301/302) before it reaches the viewer. Origin request and viewer request both fire before the origin even responds, so there's no error status yet to react to, and viewer response would work in principle but modifying the response as it comes back from the origin (origin response) is the more targeted, minimal-resource point to catch this specific error.

---

### Q4 — AWS Management & Governance
A serverless app's Lambda function calls an external HTTP payments API that occasionally times out. The company wants to notify support via an existing SNS topic when the API's error rate exceeds 10% per hour.

How can this be achieved?
- A. Use X-Ray to monitor the error rate and send alerts through the SNS topic.
- B. Implement CloudWatch Metrics with a custom threshold, linked to the existing SNS topic.
- C. Enable CloudTrail to monitor the Lambda function and alert via SNS when error rate exceeds 10%.
- D. Configure the Lambda function to directly send alerts to SNS on timeout.

**Answer: B.** CloudWatch Metrics and alarms are the purpose-built tool for tracking rates like "errors over total calls per hour" and triggering an existing SNS topic when a threshold is crossed. X-Ray is meant for tracing individual request paths for debugging, not real-time rate-based alerting. CloudTrail only records account API activity, not custom application-level error rates, and hardcoding alert logic directly into the Lambda function adds fragility and complexity compared to using a dedicated monitoring service.

---

### Q5 — AWS Security, Identity, & Compliance
An organization has separate accounts for Production, Testing, and Development. A developer with an IAM user in Development needs to launch resources in Production and Testing.

What is the most efficient way to provide access?
- A. Create a separate IAM user in each account and log in separately.
- B. Create an IAM group in Production/Testing and add the Development account's user to the groups.
- C. Create an IAM permissions policy in Production/Testing referencing the IAM user in Development.
- D. Create a role with the required permissions in Production/Testing and have the developer assume it.

**Answer: D.** Cross-account roles let a user in one account temporarily assume permissions in another, which is the standard, efficient AWS pattern for this exact scenario — no duplicate accounts or credentials needed. You can't add an IAM user from a different account into a group in another account, nor can you reference an external account's IAM user directly inside a permissions policy — both of those constructs don't cross account boundaries that way. Multiple separate logins is technically possible but clearly less efficient than a single assumable role.

---

### Q6 — AWS Networking & Content Delivery
A developer configures Route 53 health checks and needs to know what parameter Route 53 uses to decide if an endpoint is unhealthy.

Which parameter is it?
- A. Latency.
- B. Failure threshold.
- C. Network response.
- D. Fault tolerance.

**Answer: B.** The failure threshold — a customer-set number of consecutive failed checks — is the actual criterion Route 53 uses to flag an endpoint unhealthy. Latency, network response, and fault tolerance aren't parameters Route 53 evaluates for basic health-check status at all.

---

### Q7 — AWS Developer Tools
A developer uses AWS AppSync for a HIPAA-regulated healthcare app requiring data encryption, and wants to optimize performance and minimize latency.

How can the developer meet both requirements?
- A. Set API Cache to "None" and adjust encryption settings.
- B. Set API Cache to full request caching, leaving default encryption settings.
- C. Set API Cache to per-resolver caching, leaving default encryption settings.
- D. Set API Cache to full request caching and adjust encryption settings for at-rest and in-transit.

**Answer: D.** Full request caching gives the best latency/performance improvement of AppSync's cache options, and HIPAA compliance requires explicitly turning on encryption at rest and in transit rather than relying on AppSync's unencrypted defaults. "None" defeats the performance goal entirely since nothing gets cached. Per-resolver caching only covers specific calls, giving a smaller performance boost than full request caching, and leaving default encryption settings in either caching option fails the HIPAA requirement since defaults aren't encrypted.

---

### Q8 — AWS Database
A company stores sensitive data in DynamoDB and its security policy requires that data be encrypted *before* it's submitted to DynamoDB.

How can a developer meet this requirement?
- A. Use `UpdateTable` to switch to a customer managed CMK.
- B. Use ACM to create one certificate per DynamoDB table.
- C. Use `UpdateTable` to switch to an AWS managed CMK.
- D. Use the DynamoDB Encryption Client for end-to-end client-side encryption.

**Answer: D.** Server-side encryption (whether via AWS managed or customer managed CMKs) only protects data once it reaches AWS — it's still sent unencrypted over HTTPS and briefly decrypted server-side before storage. Since the requirement is encryption *before* submission, only client-side encryption via the DynamoDB Encryption Client provides true end-to-end protection. ACM issues SSL/TLS certificates and has no role in encrypting table data.

---

### Q9 — AWS Networking & Content Delivery
Customers using a REST API report performance issues. A developer needs to measure the time between when API Gateway receives a request and when it returns a response.

Which metric should be monitored?
- A. IntegrationLatency
- B. 5XXError
- C. CacheHitCount
- D. Latency

**Answer: D.** `Latency` covers the full request-response round trip as seen by the client, including both the backend integration time and API Gateway's own overhead — exactly what's being asked for. `IntegrationLatency` only measures the backend call portion, not the full client-facing time. `CacheHitCount` tracks cache usage, and `5XXError` just counts server-side errors — neither measures timing.

---

### Q10 — AWS Networking & Content Delivery
A REST API on API Gateway uses native API key validation. A new registration page creates API keys via `CreateApiKey` and sends them to users, but new users get 403 Forbidden errors when calling the API, while existing users are unaffected.

What code update grants the new users access?
- A. Call `createDeployment` to redeploy the API with the new key.
- B. Call `updateAuthorizer` to include the new API key.
- C. Call `createUsagePlanKey` to associate the new key with the correct usage plan.
- D. Call `importApiKeys` to import all new keys into the current stage.

**Answer: C.** API keys only grant access once they're linked to a usage plan — `createUsagePlanKey` is the specific call that associates an existing key with a plan, which is exactly the missing step here (new keys are created but never attached to a plan). Redeploying the API doesn't affect key-to-plan associations. `updateAuthorizer` manages a completely different resource (Lambda/Cognito authorizers), and `importApiKeys` is for bulk-importing keys from external sources like a CSV, not for linking one newly created key to a plan.

---

### Q11 — AWS Storage
A static website hosted in S3 calls a web service in API Gateway + Lambda, and shows the error: *"No 'Access-Control-Allow-Origin' header is present... Origin 'null' is therefore not allowed access."*

What should the developer do?
- A. Add the `Access-Control-Request-Headers` header to the request.
- B. Enable CORS on the S3 bucket.
- C. Add the `Access-Control-Request-Method` header to the request.
- D. Enable CORS for the method in API Gateway.

**Answer: D.** CORS must be enabled on the resource being requested — here, that's the API Gateway method, not the S3 bucket (which is the requester in this scenario, not the target). The `Access-Control-Request-*` headers are things a browser automatically sends as part of a CORS preflight request; they aren't something a developer manually adds to fix a CORS misconfiguration, and enabling CORS on S3 wouldn't affect calls made to API Gateway.

---

### Q12 — AWS Storage
A developer needs to be notified by email for all new object creation events in an S3 bucket, using SNS for the messages.

How can the developer enable these notifications?
- A. Create an event notification for `s3:ObjectCreated:Put`.
- B. Create an event notification for `s3:ObjectCreated:*`.
- C. Create an event notification for `s3:ObjectRemoved:Delete`.
- D. Create an event notification for `s3:ObjectRestore:Post`.

**Answer: B.** The wildcard `s3:ObjectCreated:*` catches object creation through any API (PUT, POST, COPY, multipart upload complete, etc.), which is needed for "all new object creation events." Limiting to `:Put` only would miss creations via other APIs like POST or COPY. `ObjectRemoved:Delete` is for deletions, and `ObjectRestore:Post` is for Glacier restore requests — neither relates to new object creation.

---

### Q13 — AWS Developer Tools
A developer uses AWS AppConfig and needs application data encrypted at rest.

Which statement correctly describes how this requirement is met?
- A. Data is auto-encrypted with AWS owned keys + KMS; the developer can disable this and use their own customer managed key instead.
- B. Data is auto-encrypted with AWS owned keys, which cannot be disabled; the developer can add an additional layer using customer managed keys.
- C. Data is stored unencrypted in S3 by default; the developer can apply KMS keys themselves.
- D. Data is unencrypted by default; the developer chooses AWS managed or customer managed keys to encrypt it.

**Answer: B.** AppConfig always encrypts data at rest by default using AWS owned keys — this baseline layer is mandatory and can't be turned off — but a customer can layer additional protection on top using their own customer managed keys if they want more control. The other options are all wrong on the basic premise that the default encryption is either disable-able or absent, when it's actually always on and non-optional.

---

### Q14 — AWS Storage
A Lambda function in a VPC is triggered when a file lands in S3, processes it, and writes result/log files that must be accessible by other AWS services and on-premises resources.

What should the developer use?
- A. Keep the result/log files in Amazon EFS, accessible by Lambda functions.
- B. Store the files in S3 and append new log entries to existing objects.
- C. Use AWS Glue to consolidate/catalog all files and append log entries.
- D. Use DynamoDB to store the files and DynamoDB Streams for change notifications.

**Answer: A.** EFS is a shared, scalable NFS file system that both Lambda (via native EFS mounting) and on-premises resources can access concurrently — a natural fit for shared read/write file access across a hybrid environment. S3 doesn't support appending to existing objects (each write creates/replaces the whole object). DynamoDB isn't designed for storing arbitrary files, and Glue is an ETL/cataloging service, not a file storage/sharing mechanism.

---

### Q15 — AWS Database
A DynamoDB table uses provisioned capacity. A manager wants to know how much of that provisioned capacity is actually being used to assess cost-effectiveness.

How can this be queried?
- A. Monitor `ReadThrottleEvents` and `WriteThrottleEvents`.
- B. Use X-Ray to instrument the table and monitor subsegments.
- C. Use CloudTrail to monitor the `DescribeLimits` API action.
- D. Monitor `ConsumedReadCapacityUnits` and `ConsumedWriteCapacityUnits` over time.

**Answer: D.** These two CloudWatch metrics directly show how much of the provisioned read/write capacity is actually being consumed over a given period — exactly what's needed to judge cost-effectiveness. Throttle-event metrics only show requests that exceeded the limit, not overall utilization. CloudTrail records API calls, not performance/utilization data, and X-Ray subsegments help trace downstream call behavior, not capacity utilization.

---

### Q16 — AWS Security, Identity, & Compliance (Select TWO)
A company's app provides S3 object access differentiated by user type (registered vs. guest), with 30,000 users total.

Which two approaches provide access to both user types most efficiently?
- A. Create a new IAM user per user and grant S3 access.
- B. Use IAM and have the application assume different roles depending on user type.
- C. Use Amazon Cognito with authenticated and unauthenticated roles.
- D. Use S3 bucket policies to restrict read access to specific IAM users.
- E. Store separate access keys in the application code for registered vs. guest users.

**Answer: B & C.** Cognito identity pools natively distinguish authenticated (registered) users from unauthenticated (guest) users, each mapped to a different IAM role with appropriate permissions — a purpose-built fit at this scale. Assuming different IAM roles based on user type is the complementary, secure IAM-side mechanism. Creating 30,000 individual IAM users, embedding hardcoded access keys in app code, or managing S3 bucket policies per-user would all be unmanageable and/or insecure at this scale.

---

### Q17 — AWS Management & Governance
A developer calling the CloudWatch API intermittently gets HTTP 400 `ThrottlingException` errors, with no data retrieved on failure.

What best practice should be tried first?
- A. Retry the call with exponential backoff.
- B. Contact AWS Support for a limit increase.
- C. Analyze the application and remove the API call.
- D. Use the AWS CLI to get the metrics.

**Answer: A.** AWS explicitly recommends retrying with exponential backoff (plus jitter) as the first response to throttling, alongside batching metrics into fewer calls and spreading requests over time — only requesting a limit increase after those steps. Removing the monitoring call sacrifices important visibility rather than fixing the throttling, and switching to the CLI still makes the same underlying API calls, so it doesn't help at all.

---

### Q18 — AWS Application Integration
A developer needs a serverless application using an event-driven architecture that automatically receives and processes events.

How should the application be configured?
- A. Create an SNS topic and an EC2 instance; subscribe the instance and submit events to the topic.
- B. Create an SNS topic and a Lambda function with an HTTP endpoint; subscribe the HTTP endpoint to the topic.
- C. Create an SQS queue; publish events and have an EC2 instance poll and consume them.
- D. Create an SNS topic and a Lambda function; subscribe the Lambda function directly to the topic.

**Answer: D.** Lambda functions can be subscribed directly as SNS topic targets — publishing an event to the topic automatically invokes the function, which is the simplest fully serverless event-driven pattern. EC2-based options (A, C) aren't serverless at all, since EC2 requires managing instances. Routing through an HTTP endpoint on Lambda (B) is unnecessary extra complexity — direct SNS-to-Lambda subscription already exists and needs no HTTP layer.

---

### Q19 — AWS Security, Identity, & Compliance
A web app uses Cognito for authentication. The company wants to allow sign-in from any source but automatically block sign-ins when the risk level is elevated.

Which Cognito feature meets this requirement?
- A. Case sensitive user pools.
- B. Adaptive authentication.
- C. Multi-factor authentication (MFA).
- D. Advanced security metrics.

**Answer: B.** Adaptive authentication scores each sign-in attempt for risk and lets you configure automatic responses per risk level, including outright blocking sign-ins at a chosen threshold — a direct match for "automatically block" behavior. Advanced security metrics only surfaces risk-level data to CloudWatch for analysis; it doesn't take blocking action on its own. MFA challenges a user for a second factor but doesn't block sign-ins based on risk, and case sensitivity is an unrelated username/email-matching setting.

---

### Q20 — AWS Developer Tools
A development team using AWS Amplify wants to test new features on current production code without impacting the last published application version.

How can the team do this?
- A. Connect the production environment to Cloud9 and test after publishing.
- B. Connect the production environment to Cloud9 and test after committing.
- C. Connect a new branch deploying to `https://main.applicationId.amplifyapp.com`.
- D. Connect a new branch deploying to `https://dev.applicationId.amplifyapp.com`.

**Answer: D.** Amplify lets you connect additional branches (like a "dev" branch) that deploy to their own separate URL, letting you build/test/roll back features without touching the live production version. The "main" branch URL (C) is the production environment itself, so deploying there would risk impacting the published version — the opposite of what's needed. Cloud9 is just an IDE option for writing code; using it doesn't by itself create an isolated testing environment separate from production.

---

### Q21 — AWS Management & Governance
A CloudFormation deployment fails with `DELETE_FAILED (The following resource(s) failed to delete: (sg-11223344))`.

What action should the developer take?
- A. Update the security group resource's logical ID to its ARN, then delete the stack.
- B. Manually delete the security group, then execute a change set to force stack deletion.
- C. Add a `DependsOn` attribute to the resource, then delete the stack.
- D. Modify the template to retain the security group resource, then manually delete it after deployment.

**Answer: D.** When a resource can't be deleted (e.g., a security group still attached to an ENI outside the stack), CloudFormation lets you mark it to be retained during stack deletion, letting the rest of the stack delete cleanly — the resource can then be cleaned up manually afterward. `DependsOn` only affects creation-order dependencies, not deletion behavior. Manually deleting the security group and then using a change set doesn't make sense — you'd just proceed with normal stack deletion at that point, not a change set. Updating the logical ID to an ARN has no bearing on why the resource fails to delete.

---

### Q22 — AWS Security, Identity, & Compliance (Select TWO)
Twelve months ago, an organization requested a public ACM certificate validated via DNS. What will ACM do prior to expiration?
- A. If the certificate was issued by a third party, ACM sends a request to the third party to verify the domain owner.
- B. If the domain is not validated, ACM sends Health or EventBridge events to notify the owner before expiration.
- C. If the certificate is used by an AWS service and ACM-provided CNAME records are accessible via public DNS, ACM considers the domain validated and auto-renews the certificate.
- D. If the certificate is used by an AWS service with accessible CNAME records, ACM considers it validated and sends Health/EventBridge events asking the owner to renew.
- E. If validated via DNS with a valid ACM CNAME and used by an AWS service, ACM uses SNS to notify of pending expiration.

**Answer: B & C.** When the CNAME is still valid and publicly resolvable and the certificate is actively used by an AWS service, ACM auto-renews without any action needed. If those conditions aren't met (domain not validated), ACM instead sends notifications (Health/EventBridge events) ahead of expiration so the owner can act. Options D and E both wrongly claim a validated, actively-used certificate still requires manual renewal via notification — but that scenario actually triggers automatic renewal, not just a notice. Third-party issued certificates aren't something ACM can verify or renew on your behalf at all.

---

### Q23 — AWS Security, Identity, & Compliance
A serverless application uses an IAM role for DynamoDB access. A developer troubleshooting access issues has access to that IAM role.

Which AWS CLI command helps test the role's permissions?
- A. `aws sts get-session-token`
- B. `aws dynamodb describe-endpoints`
- C. `aws iam get-role-policy`
- D. `aws sts assume-role`

**Answer: D.** `assume-role` actually obtains temporary credentials under that specific role, letting the developer run test calls exactly as the application would — the most direct way to reproduce and debug the access issue. `get-session-token` just gets temporary credentials for an existing IAM user/account, not for assuming a different role's permissions. `get-role-policy` only shows the policy document text (not whether it behaves as expected in practice), and `describe-endpoints` just returns regional endpoint info, unrelated to permissions testing.

---

### Q24 — AWS Application Integration
A serverless app uses different Lambda functions to look up separate pieces of customer information (address, phone number, etc.), each in its own Step Functions branch.

How can the developer optimize performance so lookups complete faster?
- A. Use a Map state to iterate over all items.
- B. Use a Wait state to reduce execution wait time.
- C. Use a Choice state to look up specific information.
- D. Use a Parallel state to run all branches concurrently.

**Answer: D.** The Parallel state runs multiple branches of a state machine concurrently — exactly what's needed to have several independent lookups execute at the same time instead of one after another. Map iterates the same set of steps over array items, which doesn't fit distinct lookup functions for different fields. Choice only adds conditional branching logic, and Wait just adds a delay — neither speeds up execution.

---

### Q25 — AWS Networking & Content Delivery (Select TWO)
A company runs a REST API on API Gateway with a Lambda authorizer, and needs to log who accessed the API and how, plus error/execution traces for the authorizer.

Which combination of actions meets these requirements?
- A. Enable API Gateway execution logging.
- B. Enable detailed logging in CloudWatch.
- C. Enable server access logging.
- D. Create an API Gateway usage plan.
- E. Enable API Gateway access logs.

**Answer: A & E.** Execution logging captures errors, execution traces, and Lambda authorizer details managed by API Gateway itself, while access logging is what records who called the API and how they accessed it — together they cover both stated requirements. "Detailed logging" isn't an actual CloudWatch feature that provides this data, server access logging is an S3-specific feature, and usage plans control quotas/throttling, not logging.

---

### Q26 — AWS Security, Identity, & Compliance
An EC2 application generates many small files (1KB each) with PII that must be encrypted, then stored on a proprietary network-attached file system (not natively AWS-KMS-integrated).

What is the safest way to encrypt the data using KMS?
- A. Encrypt the data directly with a customer managed CMK.
- B. Create a data encryption key from a CMK and encrypt the data with the CMK.
- C. Create a data encryption key from a CMK and encrypt the data with the data encryption key.
- D. Encrypt the data directly with an AWS managed CMK.

**Answer: A.** A CMK can directly encrypt payloads up to 4KB, which comfortably covers these 1KB files, and since CMKs never leave KMS, this is the safest approach — no external key material to manage or protect yourself. AWS managed CMKs are tied to specific AWS-service integrations and won't work with a proprietary, non-AWS-integrated file system. Generating a data encryption key adds unnecessary complexity here and requires you to separately secure that key outside of KMS, which is inherently less safe than just using the CMK directly for small payloads.

---

## Quick Answer Key

| # | Answer |
|---|--------|
| 1 | C |
| 2 | B |
| 3 | A |
| 4 | B |
| 5 | D |
| 6 | B |
| 7 | D |
| 8 | D |
| 9 | D |
| 10 | C |
| 11 | D |
| 12 | B |
| 13 | B |
| 14 | A |
| 15 | D |
| 16 | B, C |
| 17 | A |
| 18 | D |
| 19 | B |
| 20 | D |
| 21 | D |
| 22 | B, C |
| 23 | D |
| 24 | D |
| 25 | A, E |
| 26 | A |
