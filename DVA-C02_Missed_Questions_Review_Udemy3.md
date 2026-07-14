# DVA-C02 Practice Exam — Missed Questions Review (Set 4)
*Compiled from a Digital Cloud Training practice exam — 18 questions you got wrong, with explanations.*

---

### Q1 — AWS Networking & Content Delivery
A legacy service has an XML-based SOAP interface. A developer wants to expose its functionality to external clients via API Gateway.

Which technique accomplishes this?
- A. Create a RESTful API; transform incoming JSON into valid XML for the SOAP interface using mapping templates.
- B. Create a RESTful API; pass incoming JSON to the SOAP interface through an ALB.
- C. Create a RESTful API; transform incoming XML into a valid message for the SOAP interface using mapping templates.
- D. Create a RESTful API; pass incoming XML to the SOAP interface through an ALB.

**Answer: A.** External clients calling a modernized REST API send JSON, so API Gateway needs a mapping template to transform that incoming JSON into the XML/SOAP format the legacy backend expects — this is exactly what mapping templates are built for, and it avoids the added cost/complexity of routing through an ALB. Since clients are sending JSON (not XML) to the new REST interface, both XML-input options misread the direction of the transformation, and using an ALB as a pass-through skips the necessary format conversion entirely.

---

### Q2 — AWS Application Integration
A company needs to ingest several terabytes of data every hour from many distributed sources, delivered continuously 24/7, with real-time delivery for security analysis and live dashboards.

Which approach meets these requirements?
- A. Send messages to an SQS queue, processed by a fleet of EC2 instances.
- B. Use Amazon Kinesis Data Streams with the Kinesis Client Library.
- C. Use AWS Data Pipeline to automate data movement/transformation.
- D. Write messages to S3 via the S3 API, then process with Amazon Redshift.

**Answer: B.** Kinesis Data Streams is purpose-built for ingesting and processing massive, continuous real-time data streams, with the Kinesis Client Library making it straightforward to build consumer applications that feed dashboards and alerts. SQS is meant for decoupling components, not raw streaming ingestion, and using EC2 fleets to process it adds cost and latency. Redshift is a data warehouse for analytics queries, not a message/stream processor, and Data Pipeline is about scheduled data movement/transformation jobs, not real-time streaming ingestion.

---

### Q3 — AWS Compute (Select TWO)
A developer uses AWS SAM to create a template deploying a Lambda function and a DynamoDB table.

Which resource types should be specified?
- A. `AWS::Serverless::SimpleTable`
- B. `AWS::Serverless::API`
- C. `AWS::Serverless::Function`
- D. `AWS::Serverless::LayerVersion`
- E. `AWS::Serverless::Application`

**Answer: A & C.** `AWS::Serverless::Function` creates the Lambda function (plus its execution role and event source mappings), and `AWS::Serverless::SimpleTable` creates a DynamoDB table with a single-attribute primary key — together covering exactly what's needed. `AWS::Serverless::API` is for API Gateway resources, `LayerVersion` packages shared library/runtime code for functions, and `AWS::Serverless::Application` embeds a nested app from the Serverless Application Repository or S3 — none of which are what this scenario calls for.

---

### Q4 — AWS Compute (Select THREE)
A developer needs to create an instance profile for an EC2 instance using the AWS CLI.

How can this be achieved?
- A. Run `aws ec2 associate-instance-profile`.
- B. Run `aws iam add-role-to-instance-profile`.
- C. Run the `CreateInstanceProfile` API.
- D. Run the `AssignInstanceProfile` API.
- E. Run the `AddRoleToInstanceProfile` API.
- F. Run `aws iam create-instance-profile`.

**Answer: A, B, F.** Attaching a role to an EC2 instance via CLI takes three sequential steps: create the instance profile (`aws iam create-instance-profile`), attach a role to it (`aws iam add-role-to-instance-profile`), and then associate that profile with the instance (`aws ec2 associate-iam-instance-profile`) — all CLI commands, matching the three correct answers. The distractors are the same underlying operations but named as raw API actions rather than their actual CLI command syntax, which is what the question specifically asked for.

---

### Q5 — AWS Database
A developer is designing a fault-tolerant environment where client sessions are saved, and needs to ensure no sessions are lost if an EC2 instance fails.

How can this be ensured?
- A. Use ELB connection draining to stop sending requests to failing instances.
- B. Use DynamoDB for scalable session handling.
- C. Use SQS to save session data.
- D. Use sticky sessions with an ELB target group.

**Answer: B.** Session data must live in a durable external store to survive an instance failure, and DynamoDB (with its ready-made session-handler libraries) is built for exactly this. Sticky sessions keep session state tied to a specific instance's local storage, so a failure there still loses the data. SQS is for decoupling application components, not for storing session state, and connection draining just stops routing new requests to a failing instance — it does nothing to preserve data that was already stored there.

---

### Q6 — AWS Database
An app uses API Gateway, Lambda, and a DynamoDB table. The developer needs another Lambda function triggered when an item lifecycle activity occurs in the table.

How can this be achieved?
- A. Enable a DynamoDB stream and trigger the Lambda function synchronously from the stream.
- B. Enable a DynamoDB stream and trigger the Lambda function asynchronously from the stream.
- C. Configure a CloudTrail API alarm that sends a message to SQS; have Lambda poll and invoke synchronously.
- D. Configure a CloudWatch alarm that sends an SNS notification; trigger Lambda asynchronously.

**Answer: A.** DynamoDB Streams integrates directly with Lambda — once enabled, Lambda polls the stream and invokes your function synchronously whenever an item changes, which is the native, built-in mechanism for this exact use case. There's no CloudWatch alarm or CloudTrail event type that fires specifically on DynamoDB item lifecycle changes, so both of those routes don't actually exist as described, and the correct invocation model here is synchronous (not asynchronous) since Lambda polls the stream directly.

---

### Q7 — AWS Database
A developer is creating a DynamoDB table with 5 WCUs and needs to configure RCUs efficiently.

Which configuration is the most efficient use of throughput?
- A. Strongly consistent reads of 15 RCUs reading 1KB items.
- B. Eventually consistent reads of 5 RCUs reading 4KB items.
- C. Strongly consistent reads of 5 RCUs reading 4KB items.
- D. Eventually consistent reads of 15 RCUs reading 1KB items.

**Answer: D.** Eventually consistent reads consume half the RCU of strongly consistent reads for the same item size, and smaller items (1KB vs 4KB) require fewer RCUs per read — combining both factors makes this option capable of the highest number of actual read operations per RCU spent. The strongly-consistent options pay a premium per read without that efficiency, and the 4KB-item options need more RCU per read regardless of consistency mode.

---

### Q8 — AWS Compute (Select TWO)
A web app runs on EC2 behind an ELB. The company wants to secure it with SSL certificates without any performance impact on the EC2 instances.

Which two steps should be taken?
- A. Configure the ELB with SSL passthrough.
- B. Configure Server-Side Encryption with KMS managed keys.
- C. Add an SSL certificate to the ELB.
- D. Configure the ELB for SSL termination.
- E. Install SSL certificates on the EC2 instances.

**Answer: C & D.** Since encryption/decryption overhead can't touch the EC2 instances, SSL must be terminated at the load balancer — that means adding a certificate to the ELB and configuring SSL termination there, offloading all the crypto work away from the instances. SSL passthrough would forward encrypted traffic straight to the instances for them to decrypt, adding exactly the CPU burden the requirement rules out — same problem with installing certificates directly on the instances. Server-side encryption with KMS is an S3 feature, unrelated to ELB traffic encryption.

---

### Q9 — AWS Compute (Select THREE)
A developer is designing a cloud-native application using several Lambda functions that process items read from an event source.

Which AWS services are supported for Lambda event source mappings?
- A. Amazon DynamoDB
- B. Another Lambda function
- C. Amazon SQS
- D. Amazon S3
- E. Amazon SNS
- F. Amazon Kinesis

**Answer: A, C, F.** Event source mappings are Lambda's mechanism for polling a stream or queue that doesn't invoke Lambda directly — and Kinesis, DynamoDB Streams, and SQS are the services this polling model supports. S3 instead pushes notifications directly to Lambda via its own event notification configuration (not a poll-based event source mapping). SNS and another Lambda function both invoke Lambda directly/asynchronously rather than being polled as an event source.

---

### Q10 — AWS Security, Identity, & Compliance (Select TWO)
A financial-transaction application uses `GenerateDataKey` for end-to-end encryption and hits `ThrottlingException` errors from KMS under high volume.

Which actions are best practices to resolve this?
- A. Create a KMS custom key store and generate data keys through CloudHSM.
- B. Call the KMS `Encrypt` operation directly instead.
- C. Create a local cache using the AWS Encryption SDK's `LocalCryptoMaterialsCache`.
- D. Create an AWS Support case to increase the account's KMS quota.
- E. Use SQS to queue the requests and configure KMS to poll the queue.

**Answer: C & D.** Data key caching reuses previously generated keys instead of calling KMS for every single operation, directly cutting down on API call volume — exactly the kind of best practice AWS recommends before anything else. Requesting a quota increase is the appropriate next step if caching alone isn't sufficient. Calling `Encrypt` directly doesn't reduce call volume and also caps out at 4096 bytes per operation, a custom key store with CloudHSM adds unnecessary cost without addressing the throttling itself, and KMS has no capability to poll an SQS queue.

---

### Q11 — AWS Database
A high-traffic e-commerce site has real-time dynamic pricing, and simultaneous updates sometimes overwrite an original editor's changes.

How can developers prevent overwriting?
- A. Use atomic counters.
- B. Use conditional writes.
- C. Use concurrent writes.
- D. Use batch operations.

**Answer: B.** Conditional writes let an update succeed only if the item still matches an expected condition (e.g., the price is still what you last read) — so if someone else changed it in between, your write fails instead of silently overwriting theirs. Atomic counters are for unconditional numeric increments (like view counters), not for protecting against conflicting updates. Concurrent writes are literally the problem being described, not a solution, and batch operations just reduce network round trips — they don't add any conflict protection.

---

### Q12 — AWS Compute
An engineer wants to log a unique identifier in a Lambda function to correlate events with a specific invocation.

Which approach is correct?
- A. Use `context.invocationId`.
- B. Use `event.requestId`.
- C. Use `context.awsRequestId`.
- D. Use `context.lambdaId`.

**Answer: C.** `context.awsRequestId` is the actual property Lambda's context object exposes for this exact purpose — a unique ID tied to each individual invocation, ready to log alongside your custom events. `invocationId`, `lambdaId`, and `event.requestId` are all invented property names that don't exist on Lambda's context or event objects.

---

### Q13 — AWS Security, Identity, & Compliance (Select TWO)
A programmer needs to finalize a Signature Version 4 signed request for calling other AWS services.

Which strategies can be used?
- A. Append the signature to a query parameter called "X-Amz-Credentials."
- B. Insert the signature into a query parameter called "X-Amz-Signature."
- C. Add the signature to a query parameter called "Signature-Token."
- D. Embed the signature in an HTTP header called "Authorization-Key."
- E. Incorporate the signature into an HTTP header called "Authorization."

**Answer: B & E.** SigV4 supports exactly two valid places for the signature: the `Authorization` HTTP header, or the `X-Amz-Signature` query string parameter — both are the actual, documented mechanisms. "X-Amz-Credentials" is a real SigV4 parameter, but it's meant for the access key ID and scope info, not the signature itself, and neither "Authorization-Key" nor "Signature-Token" are recognized SigV4 fields at all.

---

### Q14 — AWS Networking & Content Delivery (Select TWO)
An app uses an ASG of EC2 instances, an ALB, SQS, and a CloudFront distribution caching content for global users. A developer needs end-to-end SSL between the CloudFront origin and end users.

How can this be achieved?
- A. Create an "encrypted distribution."
- B. Configure the Origin Protocol Policy.
- C. Create an Origin Access Identity (OAI).
- D. Add a certificate to the Auto Scaling Group.
- E. Configure the Viewer Protocol Policy.

**Answer: B & E.** True end-to-end SSL requires encrypting both legs of the CloudFront path: the Viewer Protocol Policy secures the connection between end users and CloudFront, while the Origin Protocol Policy secures the connection between CloudFront and the origin (here, the ALB) — together covering the full path. OAI is an S3-specific access-control mechanism, unrelated to SSL between CloudFront and a non-S3 origin. There's no such thing as an "encrypted distribution" setting, and certificates for the origin side belong on the ALB listener, not on the Auto Scaling Group itself.

---

### Q15 — AWS Security, Identity, & Compliance
A company needs users to access AWS services and be able to reset their own passwords.

Which combination allows managing users/authorization while letting users self-reset passwords?
- A. Cognito identity pools and AWS STS.
- B. Cognito user pools and AWS KMS.
- C. Cognito identity pools and IAM.
- D. Cognito user pools and identity pools.

**Answer: D.** A Cognito user pool provides the user directory with sign-up/sign-in and self-service password reset, while a Cognito identity pool takes those authenticated users and exchanges their identity for temporary AWS credentials to access AWS services — you need both together to satisfy both stated requirements. Identity pools alone (paired with STS or IAM) have no user directory or self-reset capability at all, and KMS is purely an encryption key service, unrelated to user authentication.

---

### Q16 — AWS Developer Tools
An app instrumented with the X-Ray SDK sets the `user` field on segments to identify the requesting user.

How can the developer search for segments associated with specific users?
- A. Use the `GetTraceGraph` API with a filter expression.
- B. Search for the `user` field in segment metadata via a filter expression.
- C. Search for the `user` field in segment annotations via a filter expression.
- D. Use the `GetTraceSummaries` API with a filter expression.

**Answer: D.** The `user` field is one of the specific segment fields X-Ray indexes for filter-expression searches, and `GetTraceSummaries` (or the equivalent console search) is how you query against it. `GetTraceGraph` retrieves a service graph for known trace IDs — it doesn't perform this kind of field-based search. The `user` field is neither part of segment metadata (which isn't indexed for search at all) nor part of annotations — it's its own distinct, indexed segment field.

---

### Q17 — AWS Compute
A SAM template includes several Lambda functions, an S3 bucket, and a CloudFront distribution, with one Lambda@Edge function integrated into CloudFront and the S3 bucket as an origin. Deploying the stack in us-west-1 fails.

What is the likely reason?
- A. Lambda@Edge functions can only be deployed in us-east-1.
- B. S3 origin buckets must be in a different region from the CloudFront distribution.
- C. SAM templates aren't supported in us-west-1.
- D. Lambda functions integrated with CloudFront can't be deployed via SAM.

**Answer: A.** Lambda@Edge has a hard regional requirement — functions must be created in us-east-1 regardless of which region the rest of the stack (or the CloudFront distribution's edge locations) lives in, so deploying from us-west-1 fails. There's no restriction preventing SAM templates from being used in us-west-1 generally, S3 origin buckets are commonly and validly located in the same region as the stack, and Lambda-CloudFront integration is fully supported through SAM — none of those are real constraints.

---

### Q18 — AWS Compute
CodeBuild builds an app, creates a Docker image, pushes it to ECR, and tags it with a unique identifier. Developers already have the AWS CLI configured locally.

How can the Docker images be pulled to their workstations?
- A. Run `aws ecr get-download-url-for-layer`, then run `docker pull REPOSITORY_URI:TAG`.
- B. Run `docker pull REPOSITORY_URI:TAG` directly.
- C. Run `aws ecr get-login-password`, then run `docker pull REPOSITORY_URI:TAG`.
- D. Run the *output* of `aws ecr get-login-password`, then run `docker pull REPOSITORY_URI:TAG`.

**Answer: D.** Since Docker CLI doesn't understand native AWS authentication, you first need to obtain an auth token by running `aws ecr get-login-password` and actually use its *output* to log in via `docker login` before you can pull — simply running the command isn't enough, you need to act on what it returns. Pulling directly without authenticating first will fail since ECR requires this login step. `get-download-url-for-layer` is for retrieving a specific image layer's pre-signed S3 URL, not for authenticating a general docker pull.

---

## Quick Answer Key

| # | Answer |
|---|--------|
| 1 | A |
| 2 | B |
| 3 | A, C |
| 4 | A, B, F |
| 5 | B |
| 6 | A |
| 7 | D |
| 8 | C, D |
| 9 | A, C, F |
| 10 | C, D |
| 11 | B |
| 12 | C |
| 13 | B, E |
| 14 | B, E |
| 15 | D |
| 16 | D |
| 17 | A |
| 18 | D |
