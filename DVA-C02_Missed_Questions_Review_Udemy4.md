# DVA-C02 Practice Exam — Missed Questions Review (Set 5)
*Compiled from a Digital Cloud Training practice exam — 12 questions you got wrong, with explanations.*

---

### Q1 — AWS Compute
A developer needs to deploy code updates to an Elastic Beanstalk environment. Due to the app's criticality, the ability to quickly roll back must be prioritized above all other considerations.

Which deployment policy should be chosen?
- A. Immutable
- B. Rolling with additional batch
- C. Rolling
- D. All at once

**Answer: A.** Immutable deployments launch a brand-new set of instances in a new ASG, deploy the update there, and only swap traffic once healthy — leaving the original, unmodified instances fully intact the whole time. That means rollback is just a matter of discarding the new instances and keeping the old ones, the fastest and safest rollback path available, even though it costs more and takes longer to deploy. Rolling, rolling-with-additional-batch, and all-at-once all update the *existing* instances directly, so if something goes wrong, recovering means a manual redeployment rather than an instant swap-back.

---

### Q2 — AWS Security, Identity, & Compliance
A developer gets an `UnauthorizedOperation` error when launching an EC2 instance via CLI, including a long encoded authorization failure message.

What action makes this error human-readable?
- A. Call AWS KMS to decode the message.
- B. Use the IAM `decode-authorization-message` API.
- C. Use an open-source decoding library.
- D. Use the AWS STS `decode-authorization-message` API.

**Answer: D.** This specific encoded-message format is decoded exclusively through STS's `DecodeAuthorizationMessage` API — it's not encryption (so KMS is irrelevant), it's a proprietary AWS encoding, so no external library can interpret it, and the decode API lives under STS, not IAM, despite IAM being where permissions themselves are defined.

---

### Q3 — AWS Security, Identity, & Compliance
A mobile app has hundreds of users, each potentially using multiple devices. The developer wants unique identifiers per user regardless of device.

Which method should be used?
- A. Implement developer-authenticated identities via Amazon Cognito and get credentials for those identities.
- B. Use IAM-generated access key IDs as unique identifiers, without storing secret keys.
- C. Create a DynamoDB user table as key-value pairs of users and devices, using those keys as identifiers.
- D. Assign IAM users and roles to users, using the IAM resource ID as the identifier.

**Answer: A.** Cognito's developer-authenticated identities let you plug in your own existing authentication system while Cognito still issues a single consistent identity per user (not per device), which is exactly the "same user, multiple devices" requirement. Building your own DynamoDB-based user/device mapping duplicates functionality Cognito already provides, and adds unnecessary application complexity. Handing out IAM users, roles, or access keys to individual end users of a mobile app is both an anti-pattern and operationally unmanageable at scale — Cognito exists specifically to avoid that.

---

### Q4 — AWS Application Integration (Select TWO)
A company is refactoring a monolithic app into microservices that need to communicate asynchronously in a decoupled manner.

Which AWS services support asynchronous message passing?
- A. Amazon Kinesis
- B. Amazon ECS
- C. AWS Lambda
- D. Amazon SNS
- E. Amazon SQS

**Answer: D & E.** SQS provides a durable message queue (pull-based) and SNS provides pub/sub notifications (push-based) — together they're the two core AWS building blocks for decoupled, asynchronous inter-service messaging. Kinesis is designed for high-throughput streaming/analytics use cases, not general decoupled messaging between services. ECS just runs containers, and Lambda is a compute service that executes in response to triggers — neither is itself a messaging service.

---

### Q5 — AWS Compute
A developer needs code that connects to and pulls data from several hundred websites, on a daily schedule, running in under 60 seconds.

Which service is most suitable and cost-effective?
- A. Amazon API Gateway
- B. AWS Lambda
- C. Amazon EC2
- D. Amazon ECS Fargate

**Answer: B.** A short-duration (well under Lambda's 900-second max), daily-scheduled task is a textbook Lambda use case — pay only for the ~60 seconds of actual execution, with CloudWatch Events handling the schedule. Running EC2 continuously (or even on a schedule) for a task this brief wastes most of the paid-for time, and ECS Fargate is better suited to longer-running containerized microservices than a short daily job. API Gateway is for exposing APIs, not for running arbitrary scheduled code.

---

### Q6 — AWS Compute
A Lambda function needs several secret-value environment variables that must stay obscured in the console and API output, even to users with permission to use the encryption key. Minimizing complexity and latency is required.

What's the best way to achieve this?
- A. Encrypt the secret values client-side using Lambda's encryption helpers.
- B. Encrypt the secret values with a customer-managed CMK.
- C. Store encrypted values in an encrypted S3 bucket and reference them from code.
- D. Use an external encryption infrastructure and add the encrypted values as environment variables.

**Answer: A.** Lambda's built-in encryption helpers let you encrypt values client-side (in the console, before they're ever sent to Lambda) so they never appear in plaintext in the console or API responses — even to users who otherwise have decrypt permission on the key — while still using standard KMS for the actual keys, keeping this low-complexity. Just switching to a customer-managed CMK on its own doesn't hide the values from users with access to that key. Routing through S3 or a separate external encryption system both add real complexity and latency that this specific approach avoids.

---

### Q7 — AWS Application Integration
A web app uses Kinesis Data Streams for IoT data storage (up to 24 hours before processing).

How can the developer implement encryption at rest for data in Kinesis Data Streams?
- A. Add a certificate and enable SSL/TLS connections to Kinesis Data Streams.
- B. Encrypt the data once at rest with a Lambda function.
- C. Use the Kinesis Consumer Library (KCL) to encrypt the data.
- D. Enable server-side encryption on Kinesis Data Streams with a KMS CMK.

**Answer: D.** Kinesis Data Streams has a native server-side encryption feature that automatically encrypts data with a specified KMS CMK before it's written to storage and decrypts it after retrieval — producers/consumers don't need to manage any of this themselves. SSL/TLS is already used by Kinesis by default and only covers data in transit, not at rest, so adding a certificate doesn't address the actual requirement. The KCL is a library for building stream-consuming applications, not an encryption tool, and bolting on a separate Lambda function to encrypt data after the fact is unnecessary when native SSE already handles this.

---

### Q8 — AWS Compute
A developer needs each ECS task to be placed on a different container instance.

How can this be achieved?
- A. Create a service on Fargate.
- B. Use a task placement strategy.
- C. Use a task placement constraint.
- D. Create a cluster with multiple container instances.

**Answer: C.** The `distinctInstance` task placement constraint is the specific rule guaranteeing each task lands on a separate container instance — exactly what's asked for. Task placement *strategies* (binpack, random, spread) influence which instances are preferred, but don't guarantee distinctness on their own. Fargate spreads tasks across Availability Zones, not necessarily distinct underlying instances (and Fargate abstracts away the instance layer entirely), and simply having multiple instances in a cluster doesn't by itself prevent two tasks from landing on the same one.

---

### Q9 — AWS Security, Identity, & Compliance (Select TWO)
A new AWS account is being set up with IAM users and policies, following best practices.

Which two strategies should be followed?
- A. Use groups to assign permissions to users.
- B. Create standalone policies instead of inline policies.
- C. Create shared user accounts for efficiency.
- D. Always use customer managed policies instead of AWS managed policies.
- E. Use user accounts to delegate permissions.

**Answer: A & B.** Assigning permissions via groups (rather than directly to individual users) and creating reusable standalone/customer managed policies (rather than one-off inline policies) are both explicitly documented IAM best practices for maintainability and consistency. Sharing user accounts across multiple people is an anti-pattern — every person should have their own individual account for accountability. Delegating permissions should be done through roles, not by sharing or reusing user accounts, and AWS actually recommends starting with AWS managed policies rather than mandating customer managed ones in all cases.

---

### Q10 — AWS Application Integration
A company uses an SQS Standard queue, and applications are picking up messages still being processed by another consumer, causing duplication.

What can a developer do to resolve this?
- A. Increase the `DelaySeconds` setting on the queue.
- B. Increase `ReceiveMessageWaitTimeSeconds` on the queue.
- C. Increase the `VisibilityTimeout` on the queue.
- D. Create a RedrivePolicy for the queue.

**Answer: C.** The visibility timeout is the window during which a message, once received by one consumer, is hidden from other consumers — if that window is too short relative to actual processing time, other consumers can pick up the same message before the first one finishes, causing duplication. `DelaySeconds` controls an initial delay before a message becomes available at all, unrelated to reprocessing during consumption. `ReceiveMessageWaitTimeSeconds` configures long polling (how long a receive call waits for a message to arrive), and a RedrivePolicy routes messages to a dead-letter queue after repeated failures — neither addresses the visibility/duplication issue directly.

---

### Q11 — AWS Storage
A developer revising multiple Lambda functions notices they share the same custom libraries, and wants to centralize them, update them easily, and keep them version-controlled — with minimal development effort.

Which solution fits?
- A. Create a CodeCommit repository to store the custom libraries.
- B. Create a custom AMI that includes the custom libraries.
- C. Create a Lambda layer including all the custom libraries.
- D. Create an S3 bucket to store all the custom libraries.

**Answer: C.** Lambda layers are purpose-built for sharing common code/libraries across multiple functions, with built-in versioning — exactly matching "centralize, easily update, version control" with minimal extra engineering. CodeCommit is a source-control repository, not something Lambda functions load code from directly at runtime. A custom AMI is irrelevant since Lambda is serverless and doesn't run on instances you manage, and an S3 bucket would require you to build your own logic for fetching, caching, and versioning the libraries — layers already do all of that natively.

---

### Q12 — AWS Networking & Content Delivery
A website delivered via CloudFront had some images updated, but testing shows the new versions aren't displaying.

What should the developer do to force the new images to display?
- A. Invalidate the old versions of the images on the origin.
- B. Invalidate the old versions of the images on the edge caches.
- C. Delete the images from the origin and save the new version there.
- D. Force an update of the cache.

**Answer: B.** The stale images are being served from CloudFront's edge caches, not the origin — so invalidating them at the edge is what forces CloudFront to go back to the origin and fetch the new version on the next request. There's no "invalidate at the origin" concept since the origin isn't a cache in this sense. Simply replacing the images at the origin doesn't clear what's already cached at the edges, and there's no direct "force cache update" action — invalidation is the actual mechanism that achieves this.

---

## Quick Answer Key

| # | Answer |
|---|--------|
| 1 | A |
| 2 | D |
| 3 | A |
| 4 | D, E |
| 5 | B |
| 6 | A |
| 7 | D |
| 8 | C |
| 9 | A, B |
| 10 | C |
| 11 | C |
| 12 | B |
