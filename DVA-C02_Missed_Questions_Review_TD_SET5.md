# DVA-C02 Practice Exam — Missed Questions Review
*Compiled from Tutorials Dojo practice exam — 32 questions you got wrong, with explanations.*

---

### Q1 — Development with AWS Services
A Lambda function has multiple sub-functions chained together to process large data synchronously. The function tends to exceed its maximum timeout limit, prompting the developer to break it into manageable coordinated states using Step Functions, so each sub-function runs in a separate process.

Which state type should the developer use to run processes?
- A. Pass State
- B. Parallel State
- C. Wait State
- D. Task State

**Answer: D — Task State.** Only Task and Parallel states can actually perform work in a state machine. Parallel is for branches that run concurrently/independently, but this workflow is synchronous — each function's output feeds the next — so Task State fits. Pass State only forwards or injects data without doing work, and Wait State just adds a delay.

---

### Q2 — Deployment
A company uses CodeDeploy for in-place deployments on EC2. A new version with a code regression fails, and the deployment group has automatic rollback configured.

What happens when the deployment fails?
- A. CodeDeploy redeploys the last known good version with a new deployment ID.
- B. CodeDeploy reroutes traffic back to the blue environment and terminates the green environment.
- C. CodeDeploy reverts the EC2 instance to a previous AMI snapshot from the last successful deployment.
- D. CodeDeploy pauses the deployment, restores the last stable version from S3, and reuses the deployment ID of the last SUCCEEDED deployment.

**Answer: A.** With automatic rollback, CodeDeploy doesn't "restore" or "pause" — it kicks off a brand-new deployment of the last known good revision, which gets its own new deployment ID. Blue/green traffic rerouting only applies to blue/green deployments, not in-place ones, and CodeDeploy never touches AMI snapshots.

---

### Q3 — Troubleshooting and Optimization
A Lambda function created via CloudFormation executes without errors but generates no logs, and no log group/stream exists. The execution role trusts the Lambda service but has no permissions attached, and there's no resource-based policy.

What must be done to fix this?
- A. Attach a resource-based policy with `logs:PutLogEvents`.
- B. Add the `CloudWatchLambdaInsightsExecutionRolePolicy` managed policy to the execution role.
- C. Add the `AWSLambdaBasicExecutionRole` managed policy to the execution role.
- D. Attach a resource-based policy with `logs:CreateLogGroup`, `logs:CreateLogStream`, and `logs:PutLogEvents`.

**Answer: C.** Logging permissions belong on the *execution role*, not a resource-based policy (resource-based policies control who can invoke the function, not what it can do). The Console auto-attaches `AWSLambdaBasicExecutionRole` when you create a function there, but IaC tools like CloudFormation require you to attach it explicitly. Lambda Insights' managed policy is a different, narrower purpose.

---

### Q4 — Security (Select TWO)
An application uses AWS KMS to encrypt data before storing it in DynamoDB.

Which features are supported by KMS?
- A. Re-enabling disabled keys
- B. Creation of symmetric encryption and asymmetric KMS keys
- C. Using AWS Certificate Manager as a custom key store
- D. Automatic key rotation for KMS keys in custom key stores
- E. Importing custom key material to an asymmetric KMS key

**Answer: A & B.** KMS lets you disable and re-enable keys, and create both symmetric and asymmetric keys. Custom key stores use AWS CloudHSM, not Certificate Manager. Automatic rotation only applies to symmetric keys — not custom-key-store or imported-material keys. And you can only import custom key material into symmetric keys, not asymmetric ones.

---

### Q5 — Security
An application uploads hundreds of thousands of objects per second to S3 in parallel using `PutObject`, encrypting with SSE-KMS. After enabling SSE-KMS, there is noticeable performance degradation.

What is the most likely cause?
- A. S3 throttles `PutObject` for SSE-KMS-encrypted objects.
- B. The API request rate has exceeded the quota for AWS KMS API operations.
- C. The KMS key uses AES-256, which is slower than AES-128.
- D. The KMS key lacks an alias, which SSE-KMS requires.

**Answer: B.** Every SSE-KMS upload/download triggers a `GenerateDataKey` or `Decrypt` call to KMS behind the scenes, and those calls count against KMS's own per-second API quota (5,500–30,000 depending on region). At very high throughput, that quota — not S3 itself — becomes the bottleneck. S3 scales fine on its own, AES-256 vs AES-128 speed difference is negligible, and key aliases aren't required for SSE-KMS.

---

### Q6 — Deployment
A team uses Step Functions to orchestrate a serverless workflow with Lambda and needs to extract custom metrics (e.g., processing times) directly from Lambda logs, with real-time alarms for anomalies.

Which approach should be used?
- A. Use CloudWatch Lambda Insights to extract and monitor custom metrics, with alarms/dashboards in Lambda Insights.
- B. Send custom metric data directly to EventBridge via `PutMetricData`, and create EventBridge rules.
- C. Use AWS's open-source libraries to format logs in CloudWatch embedded metric format (EMF), then monitor/alarm via CloudWatch.
- D. Stream logs via Data Firehose to Redshift and use SQL queries for anomaly alerts.

**Answer: C.** CloudWatch Embedded Metric Format (EMF) is a JSON log format that CloudWatch automatically parses to extract custom metrics — no separate instrumentation needed. AWS's open-source libraries make it easy to emit logs in this format from Lambda. `PutMetricData` isn't an EventBridge API (that's a CloudWatch API), Lambda Insights doesn't extract custom metrics from arbitrary log content, and Firehose→Redshift is way more infrastructure than this needs.

---

### Q7 — Troubleshooting and Optimization
An Elastic Beanstalk app has a lifecycle policy limiting versions to 15. The developer wants to keep old source code in S3, but it keeps getting deleted.

What should be changed?
- A. Set "application versions limit by age" to zero.
- B. Set "application versions limit by total count" to zero.
- C. Trigger a Lambda function to copy source code to another S3 bucket.
- D. Configure the Retention setting to retain the source bundle in S3.

**Answer: D.** Elastic Beanstalk's version lifecycle policy has a separate Retention setting that controls whether the S3 source bundle is deleted along with the old application version — set it to "Retain source bundle in S3" and the code stays even after the version record is cleaned up. You can't set either limit option to zero (minimum is 1), and building a Lambda copy job is unnecessary extra work.

---

### Q8 — Development with AWS Services
A startup needs a publicly accessible HTTPS endpoint for a third-party webhook, processed by Lambda. The platform signs each request with a secret key in the headers; the Lambda function must only execute logic for valid requests.

Which approach requires the least development effort?
- A. Create a Lambda function URL with a resource policy requiring `lambda:FunctionUrlAuthType: AWS_IAM`.
- B. Create a Lambda function URL with a resource policy requiring `lambda:CodeSigningConfigArn`.
- C. Use API Gateway with Lambda proxy integration and a Lambda authorizer validating the header signature.
- D. Create a Lambda function URL with `lambda:FunctionUrlAuthType: NONE`, and write custom authorization logic based on the header signature.

**Answer: D.** Lambda function URLs give you a public HTTPS endpoint without standing up API Gateway. `AUTH_TYPE=NONE` makes it publicly invokable, and then your own code validates the webhook's signature — exactly matching "least effort" since there's no IAM-based caller to require (the platform isn't an AWS principal). `AWS_IAM` auth would block the third party entirely, code signing verifies deployed code integrity (unrelated to request auth), and API Gateway + a custom authorizer works but is more setup than necessary for a single function.

---

### Q9 — Development with AWS Services
A Ruby developer wants to offload processing to AWS without managing servers. Submodules must be written in Ruby, mainly calling an external web API and storing results in MongoDB.

What should he do to develop the Lambda function in Ruby?
- A. Create a Lambda function with a supported runtime version for Ruby.
- B. Create a Lambda function using the AWS SDK for Ruby.
- C. Create a Lambda function with a custom runtime for Ruby, include the runtime in the deployment package, then migrate it to a separately managed layer.
- D. Create a Lambda function on Ruby with a custom runtime plus the AWS SDK for Ruby.

**Answer: A.** Lambda natively supports Ruby, so you just pick the Ruby runtime — no custom runtime needed. Custom runtimes are only for languages Lambda doesn't support out of the box. The AWS SDK for Ruby is just a library for calling AWS services, not a runtime environment, and it's irrelevant here since the function calls an external API, not AWS resources.

---

### Q10 — Security (Select THREE)
Static assets in an S3 bucket in the production account need to be accessed by a user in the development account. Full credential sharing between accounts is prohibited.

Which steps delegate access across the two accounts?
- A. In the production account, create a policy using STS to assume the dev account's IAM role, and attach it to IAM users.
- B. On the production account, create an IAM role specifying the development account as a trusted entity.
- C. On the development account, create an IAM role specifying the production account as a trusted entity.
- D. Set the policy granting S3 access for the IAM role created in the development account.
- E. Set the policy granting S3 access for the IAM role created in the production account.
- F. In the development account, create a policy using STS to assume the production account's IAM role, and attach it to IAM users.

**Answer: B, E, F.** Since the bucket lives in production, the role and its S3 permissions must be created there (trusting the dev account as the principal that can assume it). Then, in the development account, IAM users get a policy allowing them to call `sts:AssumeRole` on that production role. Creating the role or its S3 policy on the development side (C, D) is backwards, and building the assume-role policy in production (A) is also the wrong account.

---

### Q11 — Development with AWS Services
A developer debugging a Lambda-based application wants the function to return the log location of an invocation request with the least effort.

Which approach should be taken?
- A. Extract the request ID from the `Event` object, then call `FilterLogEvents` with the request ID.
- B. Extract the request ID from the `Context` object, then call `FilterLogEvents` with the request ID.
- C. Extract the log stream name from the `Event` object.
- D. Extract the log stream name from the `Context` object.

**Answer: D.** The `Context` object Lambda passes to your handler already exposes `log_stream_name` directly — no extra API calls needed. The `Event` object doesn't contain log stream or request-id info at all, and calling `FilterLogEvents` afterward is unnecessary extra work when the log stream name is already available.

---

### Q12 — Troubleshooting and Optimization
A developer wants to reduce the execution time of a DynamoDB table scan during low-demand periods without interfering with typical workloads. The scan consumes half of the strongly consistent read capacity during regular hours.

How can the scan operation be improved?
- A. Use a parallel scan operation.
- B. Use eventually consistent reads instead of strongly consistent reads.
- C. Perform a rate-limited parallel scan operation.
- D. Perform a rate-limited sequential scan operation.

**Answer: C.** DynamoDB Scan is sequential by default and can only read one partition at a time, which caps its throughput. A parallel scan splits the table into segments read concurrently by multiple workers — much faster — but without rate limiting it can consume all your provisioned throughput and throttle your normal workload. Combining both (rate-limited parallel scan) speeds things up while protecting other traffic. Switching consistency models only changes cost, not scan speed, and a "rate-limited sequential" scan is still sequential, so it won't get faster.

---

### Q13 — Deployment
A developer uploads code to S3 and uses CodeDeploy to deploy a new EC2 application version in N. Virginia. The deployment fails during the `DownloadBundle` lifecycle event with `UnknownError: not opened for reading`.

What is the possible cause?
- A. Versioning is not enabled on the S3 bucket holding the code.
- B. `DownloadBundle` is not supported in N. Virginia.
- C. Wrong configuration of `DownloadBundle` in the AppSpec file.
- D. The EC2 instance's IAM profile lacks permission to access the code in S3.

**Answer: D.** `DownloadBundle` errors typically stem from the instance's IAM role lacking S3 read permissions, an S3-side error, or a region mismatch between the instance and the bucket. `DownloadBundle` isn't something you configure manually in the AppSpec file — it's managed entirely by the CodeDeploy agent — and CodeDeploy is fully supported in N. Virginia. Bucket versioning is unrelated; it only affects object version history, not read access.

---

### Q14 — Development with AWS Services
An application makes GET calls to various AWS services, traced with AWS X-Ray. A developer maintaining a specific code block wants to record data tied to that code to group traces in the console.

Which X-Ray feature should be used?
- A. Subsegment
- B. Sampling
- C. Metadata
- D. Annotations

**Answer: D — Annotations.** Annotations are indexed key-value pairs specifically meant for filtering/grouping traces in the console or via `GetTraceSummaries`. Metadata is also key-value, but it isn't indexed, so it can't be used to search or group. Sampling just controls how much tracing data gets collected, and subsegments only add granular timing detail — neither helps with grouping.

---

### Q15 — Development with AWS Services
A developer uses Amazon ECS to orchestrate two Docker containers and needs them to share log data.

Which configuration should be used?
- A. Two pod specifications for each container, with an EFS volume mounted between pods.
- B. A single pod specification with EFS as its volume type.
- C. Two task definitions for each container, with an EFS volume mounted between tasks.
- D. A single task definition with EFS configured as its volume type.

**Answer: D.** ECS task definitions can include multiple containers and shared data volumes — so a single task definition with an EFS volume lets both containers read/write the same log data. "Pods" are a Kubernetes concept, not ECS, so those options don't apply here, and splitting into two task definitions is unnecessary since one definition already supports multiple containers.

---

### Q16 — Security
A developer needs to send used-memory-percentage and TCP-connection-count metrics from Auto Scaling Group instances to CloudWatch, using the most secure authentication method.

Which approach should be used?
- A. Create an IAM user with programmatic access, attach `cloudwatch:PutMetricData`, and insert the access/secret key into the launch template's user data.
- B. Modify the existing launch template to use an IAM role with `cloudwatch:PutMetricData`.
- C. Create an IAM role with `cloudwatch:PutMetricData` for a new launch template used to launch the instances.
- D. Create an IAM user with programmatic access, attach `cloudwatch:PutMetricData`, and store the keys in the instance's config file.

**Answer: C.** IAM roles attached to instances are always more secure than long-lived access keys, since roles issue short-lived, automatically rotated credentials. Launch templates, however, are immutable once created — you can't modify one after the fact, so a *new* launch template with the role must be created. Both IAM-user-with-hardcoded-keys options are less secure regardless of where the keys are stored.

---

### Q17 — Security (Select TWO)
A serverless API (API Gateway + Lambda) is defined entirely with CDK L2 constructs. The developer wants to test some Lambda functions locally, with SAM and CDK already installed.

Which combination of actions is needed?
- A. Run `sam local start-lambda`, pointing to the synthesized CloudFormation template plus function identifiers.
- B. Run `sam local invoke`, pointing to the synthesized CloudFormation template and each function's identifier.
- C. Run `cdk bootstrap` to prepare CDK asset staging.
- D. Run `cdk synth`, indicating the stack name of the Lambda functions to test.
- E. Run `sam package`, specifying the S3 bucket for uploading Lambda code.

**Answer: B & D.** SAM needs a CloudFormation template to work with, so `cdk synth` compiles the CDK app down into one first. From there, `sam local invoke` runs a specific function locally against that template. `cdk bootstrap` just prepares deployment infrastructure in the account (irrelevant to local testing), `sam package` prepares a template for actual AWS deployment (not local testing), and `sam local start-lambda` spins up a persistent emulated endpoint — better suited to testing how other services invoke Lambda, not for directly invoking one function at a time.

---

### Q18 — Troubleshooting and Optimization
A team builds a website with CodeBuild pulling source from GitHub. CodeBuild must run through a proxy server for privacy/security. A `RequestError` timeout appears in CloudWatch whenever CodeBuild is accessed.

What is a possible solution?
- A. Modify the `proxy` element of `buildspec.yml` in the source root.
- B. Modify the `proxy` element of `AppSpec.yml` in the source root.
- C. Modify the `artifacts` element of `buildspec.yml` in the source root.
- D. Modify the `phases` element of `AppSpec.yml` in the source root.

**Answer: A.** When CodeBuild runs behind an explicit proxy server, `buildspec.yml` needs a `proxy` element specifying settings like `upload-artifacts` and `logs`. AppSpec files belong to CodeDeploy, not CodeBuild, so both AppSpec options are out of scope, and the `artifacts` element only controls where build output goes — it has nothing to do with proxy configuration.

---

### Q19 — Troubleshooting and Optimization
A monolithic app is tested on Amazon ECS using the EC2 launch type. After terminating the container instance following testing, it still appears as a resource in the ECS cluster.

What is the possible cause?
- A. After terminating in the STOPPED state, the instance must be manually deregistered in the EC2 console (since it used the EC2 launch type).
- B. Terminating in the RUNNING state does not automatically deregister the instance.
- C. After terminating in the RUNNING state, the instance must be manually deregistered in the ECS console.
- D. Terminating in the STOPPED state does not automatically deregister the instance from the cluster.

**Answer: D.** Terminating a container instance while it's RUNNING triggers automatic deregistration from the ECS cluster. But if it's terminated while already STOPPED, ECS doesn't clean it up automatically — you have to manually deregister it, and that has to be done through the ECS console/CLI, not the EC2 console (EC2 launch type doesn't change this).

---

### Q20 — Security
A company runs AI software for automotive clients and wants to expose an API via API Gateway with different access levels (students, professionals, hobbyists), regulating and monetizing access.

What should the company do?
- A. Create three Usage Plans; set quota and throttling per access level.
- B. Create three stages with CloudWatch metrics and alarms per `ApiName`/`Method`/`Resource`/`Stage`.
- C. Create three stages; set quota and throttling per access level.
- D. Create three Authorizers to control API access.

**Answer: A.** Usage Plans are API Gateway's purpose-built mechanism for tiered access — they combine API keys with quota and throttling limits per client tier, which is exactly how AWS documents monetizing and segmenting API consumers. Stages don't support quotas directly, Authorizers only handle allow/deny access control (not usage-based tiers), and CloudWatch alarms only monitor — they don't enforce or monetize access.

---

### Q21 — Troubleshooting and Optimization
A serverless workflow built with Step Functions needs to handle and recover from state exception errors.

Which action should the developer take?
- A. Use try/catch in the application code to capture the error, then use a `Retry` field in the state definition.
- B. Use `Catch` and `Retry` fields in the state machine definition.
- C. Use a `Catch` field in the application code, then a `Retry` field in the state definition.
- D. Use `Catch` and `Retry` fields inside the application code.

**Answer: B.** `Catch` and `Retry` are native fields of the Amazon States Language, defined directly in the state machine's own definition (on Task/Parallel states) — no custom try/catch logic in your application code is needed at all. Any option putting `Catch` or `Retry` inside the application code misunderstands where these constructs live.

---

### Q22 — Development with AWS Services
A university is migrating historical alumnus records to S3, needing secure, durable object storage at the lowest possible cost.

Which S3 storage class should be recommended?
- A. S3 Glacier Deep Archive
- B. S3 One Zone-IA
- C. S3 Glacier
- D. S3 Infrequent Access

**Answer: A.** S3 Glacier Deep Archive is explicitly the lowest-cost S3 storage class, built for long-term retention of data accessed once or twice a year — a perfect match for historical archival records. Regular Glacier is also archival but noticeably pricier per GB, and One Zone-IA / Infrequent Access are meant for data accessed more regularly, not deep long-term archiving.

---

### Q23 — Security
A developer running `stop-instance` via the AWS CLI gets an `UnauthorizedOperation` error along with an additional ciphertext failure message.

How can the developer decode the message?
- A. Call the AWS KMS `decrypt` command.
- B. Call the AWS IAM `decode-authorization-message` command.
- C. Call the AWS STS `decode-authorization-message` command.
- D. Decode the message using an external cryptography library.

**Answer: C.** AWS encodes detailed authorization-failure reasons (to avoid leaking sensitive policy info to unauthorized callers), and only AWS STS's `DecodeAuthorizationMessage` API can decode that specific message — requiring the `sts:DecodeAuthorizationMessage` permission. It isn't KMS-encrypted, so KMS `decrypt` won't work, IAM has no such API, and no external library has access to AWS's internal encoding scheme.

---

### Q24 — Troubleshooting and Optimization
A serverless app uses Lambda and API Gateway. A version of `AccountService:Prod` has been published with the alias `AccountService:Beta`. The internal team wants to test the updates without impacting live users.

Which configuration should the company use?
- A. Create a 'Beta' stage; add a condition to `AccountService:Prod` checking an environment variable to invoke `AccountService:Beta`.
- B. Create a new 'Beta' stage in API Gateway and use stage variables to reference the Prod and Beta Lambda functions.
- C. Update the integration request settings in the 'Prod' stage to select the Lambda alias based on a query parameter.
- D. Modify the existing 'Prod' stage to add a stage variable referencing both Prod and Beta functions.

**Answer: B.** Stage variables let each API Gateway stage point to a different backend (in this case, a different Lambda alias) without touching the production stage or its function code at all. A dedicated Beta stage isolates testers completely from live traffic. Modifying the Prod stage (C, D) risks disrupting real users, and baking beta-routing logic into the production function itself (A) requires a production code change — exactly what you're trying to avoid.

---

### Q25 — Troubleshooting and Optimization
A developer automates EC2 snapshot creation via a CLI script and gets an `InvalidInstanceID.NotFound` error on every run, even though the instance exists.

What is the most likely cause?
- A. The AWS Region used to configure the CLI doesn't match the instance's region.
- B. The AWS Access Key ID used to configure the CLI is invalid.
- C. The AWS Region where CLI programmatic access was created doesn't match the instance's region.
- D. The Image ID used for the snapshot command is incorrect.

**Answer: A.** `InvalidInstanceID.NotFound` means the CLI is looking for the instance in the wrong region — since instances live in a specific region and the CLI defaults to whatever region is configured. IAM users/access keys are global (not tied to a region), so option C's premise doesn't hold. An invalid access key would produce a distinct `InvalidAccessKeyId` error, not this one, and the error is about the instance ID, not an image ID.

---

### Q26 — Development with AWS Services
A developer manages an Application Load Balancer targeting a Lambda function and needs to obtain all values for identical query-parameter keys in a request.

How can this be implemented?
- A. Set a custom HTTP response header in the Lambda function.
- B. Replace the ALB with a Network Load Balancer and enable multi-value headers.
- C. Enable multi-value headers on the Application Load Balancer.
- D. Decode the URL-encoded query string values in the Lambda function.

**Answer: C.** ALB supports an optional multi-value headers mode for Lambda targets — once enabled, duplicate query params (or headers) are delivered to Lambda as arrays instead of the ALB silently keeping only the last value. Custom response headers are an API Gateway feature, not ALB. NLB doesn't support Lambda as a target type at all, and URL-decoding wouldn't change which values the ALB forwards.

---

### Q27 — Development with AWS Services
A mobile game uses DynamoDB with Web Identity Federation. Each item stores a user's game data, with user ID as the partition key. Access must be restricted so users can only retrieve their own items.

Which solution should be implemented?
- A. Set the `dynamodb:Select` condition key to the user IDs in the IAM policy.
- B. Set the `dynamodb:Attributes` condition key to the user IDs in the IAM policy.
- C. Set the `dynamodb:LeadingKeys` condition key to the user IDs in the IAM policy.
- D. Set the `dynamodb:ReturnValues` condition key to the user IDs in the IAM policy.

**Answer: C.** `dynamodb:LeadingKeys` is specifically the DynamoDB fine-grained access control condition key used to restrict access to items based on their partition key value — a perfect fit since user ID is the partition key here. `dynamodb:Attributes` restricts visibility to specific *attributes* within items (not whole items), `dynamodb:Select` only limits which attributes come back from a Query/Scan, and `dynamodb:ReturnValues` just controls what's returned on updates — none of these restrict item-level ownership.

---

### Q28 — Security
A developer is building a web app allowing registered/logged-in users to save and retrieve images in S3.

Which combination of services should handle the user authentication module?
- A. Amazon Cognito User Pools and AWS KMS
- B. Amazon User Pools and AWS STS
- C. Amazon Cognito Identity Pools and User Pools
- D. Amazon Cognito Identity Pools and an IAM Role

**Answer: C.** User Pools handle actual user sign-up/sign-in (including via social/SAML providers), while Identity Pools take those authenticated users and federate them into temporary AWS credentials for accessing services like S3. You need both together — Identity Pools alone (D) has no sign-in mechanism, KMS (A) is just for encryption, and STS (B) is already abstracted away by Identity Pools, so you configure an Identity Pool rather than calling STS directly.

---

### Q29 — Development with AWS Services
An online survey is processed by a Step Functions workflow with four states managing logic and error handling. All data passing through the nodes must be aggregated if the process fails.

What should be done to meet this requirement?
- A. Include a `Parameters` field to capture errors, then use `ResultPath` to combine each node's input with its output.
- B. Include a `Catch` field to capture errors, then use `ItemsPath` to combine each node's input with its output.
- C. Include a `Catch` field to capture errors, then use `ResultPath` to combine each node's input with its output.
- D. Include a `Parameters` field to capture errors, then use `ItemsPath` to combine each node's input with its output.

**Answer: C.** `Catch` is the correct field for capturing errors on Task/Parallel states, and `ResultPath` controls how a state's result is merged with (rather than replacing) its input — letting you aggregate input, error, and output together. `Parameters` is for passing static or path-derived key-value data, not for error capture, and `ItemsPath` is exclusively a Map-state construct.

---

### Q30 — Development with AWS Services
A developer needs a queueing mechanism to consume SQS messages larger than 256 KB, up to 1 GB in size.

How should the messages be managed?
- A. Use S3 and the Amazon SQS Extended Client Library for Java.
- B. Use S3 and the Amazon SQS Console.
- C. Use S3 and the Amazon SQS HTTP API.
- D. Use S3 and the Amazon SQS CLI.

**Answer: A.** SQS messages are capped at 256 KB natively, so oversized payloads (up to 2 GB) are stored in S3, with only a reference/pointer sent through the actual SQS message — and the Extended Client Library for Java is the specific tool that automates that store/retrieve/delete pattern. This capability is a Java-SDK-only feature; the SQS Console, CLI, and HTTP API have no built-in support for it.

---

### Q31 — Development with AWS Services (Select TWO)
A Python script relies on the low-level `BatchGetItem` API to fetch large amounts of DynamoDB data, and often gets partial results with data listed under `UnprocessedKeys`.

Which approaches handle data retrieval most reliably?
- A. Create a Global Secondary Index (GSI) with its own read capacity settings.
- B. Implement an exponential backoff algorithm with randomized delay between retries.
- C. Increase RCUs for the table and enable Auto Scaling.
- D. Use the AWS SDK to send batch requests.
- E. Implement logic that immediately retries the batch request.

**Answer: B & D.** `BatchGetItem` can return partial results for several reasons beyond just throughput (response size limits, per-partition limits, etc.), so the recommended fix is retrying `UnprocessedKeys` with exponential backoff and jitter — which the AWS SDK already implements automatically. A GSI doesn't change how `BatchGetItem` behaves or affect `UnprocessedKeys`, more RCUs/Auto Scaling can help with throttling specifically but doesn't address the other causes of partial results, and immediate retries are more likely to fail again than succeed.

---

### Q32 — Development with AWS Services
A startup uses Cognito User Pools' built-in hosted UI for sign-up/sign-in, but the product manager wants the product logo added to the login page.

How should the developer meet this requirement?
- A. Create a login page with the logo and upload it to Amazon Cognito.
- B. Create a login page with the logo, upload it to an S3 bucket, and point the S3 endpoint in Cognito app settings.
- C. Upload the logo to the Cognito app settings and use it on the custom login page.
- D. Upload the logo to an S3 bucket and point the S3 endpoint on the custom login page.

**Answer: C.** Cognito's hosted UI supports built-in customization — you can upload a logo image and adjust CSS directly in the app client's UI customization settings, without needing to build a custom page at all. There's no S3-endpoint option in Cognito app settings, and building your own login page defeats the purpose of using the hosted UI in the first place.

---

## Quick Answer Key

| # | Answer |
|---|--------|
| 1 | D |
| 2 | A |
| 3 | C |
| 4 | A, B |
| 5 | B |
| 6 | C |
| 7 | D |
| 8 | D |
| 9 | A |
| 10 | B, E, F |
| 11 | D |
| 12 | C |
| 13 | D |
| 14 | D |
| 15 | D |
| 16 | C |
| 17 | B, D |
| 18 | A |
| 19 | D |
| 20 | A |
| 21 | B |
| 22 | A |
| 23 | C |
| 24 | B |
| 25 | A |
| 26 | C |
| 27 | C |
| 28 | C |
| 29 | C |
| 30 | A |
| 31 | B, D |
| 32 | C |
