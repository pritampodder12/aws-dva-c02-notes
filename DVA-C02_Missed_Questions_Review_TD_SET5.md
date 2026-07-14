# DVA-C02 Practice Exam — Missed Questions Review (Set 6)
*Compiled from a Tutorials Dojo practice exam — 14 questions you got wrong, with explanations.*

*Note: A few of these (KMS/SSE-KMS throttling, CloudWatch EMF, Lambda logging permissions, CodeBuild proxy config, cross-account S3 delegation) are the same underlying questions you missed in Set 1 — worth double-checking those topics specifically since they've now shown up twice.*

---

### Q1 — AWS Developer Tools
A development team needs to deploy an application revision into three environments — Test, then Staging, then Production — in that order.

Which approach conveniently allows deploying to different environments?
- A. Create separate CloudFormation templates for each environment.
- B. Create separate S3 buckets for each environment.
- C. Create multiple deployment groups for each environment using CodeDeploy.
- D. Create, configure, and deploy multiple application projects for each environment using CodeBuild.

**Answer: C.** CodeDeploy lets you associate multiple deployment groups with a single application, each targeting a different tagged set of instances (Test, Staging, Production) — so you deploy the same revision sequentially to each group as it passes verification. S3 buckets can hold artifacts but have no built-in deployment orchestration, requiring manual scripting to actually push code out. CodeBuild only compiles and tests code — it doesn't deploy anything. CloudFormation manages infrastructure as code, but isn't designed for sequencing application code deployments across environments the way CodeDeploy is.

---

### Q2 — AWS Security, Identity, & Compliance
A developer working remotely needs temporary credentials via STS, with MFA enforced to protect specific programmatic calls against an EC2 instance that could adversely affect the server.

Which STS API should be used?
- A. `GetSessionToken`
- B. `GetFederationToken`
- C. `AssumeRoleWithSAML`
- D. `AssumeRoleWithWebIdentity`

**Answer: A.** `GetSessionToken` is the only STS API of these four that supports enforcing MFA as a condition on the resulting temporary credentials — a direct fit for "enforce MFA to protect specific calls." `AssumeRoleWithSAML` and `AssumeRoleWithWebIdentity` are both for federated users authenticated via external identity providers (corporate SAML systems or public OIDC providers like Google/Facebook), not for an existing IAM user needing MFA-gated temporary credentials. `GetFederationToken` also issues temporary credentials for federated users but doesn't have the same MFA-enforcement tie-in for this use case.

---

### Q3 — AWS Security, Identity, & Compliance
An application uploads hundreds of thousands of objects per second to S3 using `PutObject`, encrypted with SSE-KMS. Performance degrades noticeably after enabling SSE-KMS.

What is the most likely cause?
- A. S3 throttles `PutObject` for SSE-KMS-encrypted objects.
- B. The KMS key uses AES-256, which is slower than AES-128.
- C. The KMS key lacks an alias required for SSE-KMS.
- D. The API request rate has exceeded the quota for KMS API operations.

**Answer: D.** Every SSE-KMS upload triggers a `GenerateDataKey` call to KMS behind the scenes, and those calls count against KMS's own per-second quota (5,500–30,000 depending on region) — at very high throughput, that quota becomes the actual bottleneck, not S3 itself. S3 scales automatically regardless of encryption. The AES-256 vs. AES-128 speed difference is negligible in practice, and a key alias isn't required at all for SSE-KMS to function.

---

### Q4 — AWS Management & Governance
A team uses Step Functions with Lambda and needs to extract custom metrics (like processing times) directly from Lambda logs, with real-time anomaly alarms.

Which approach should be used?
- A. Use CloudWatch Lambda Insights to extract custom metrics, with alarms/dashboards in Lambda Insights.
- B. Stream logs via Data Firehose to Redshift and use SQL for anomaly alerts.
- C. Send custom metric data directly to EventBridge via `PutMetricData`, with EventBridge rules for actions.
- D. Use AWS's open-source libraries to format logs in CloudWatch embedded metric format (EMF), then monitor via CloudWatch.

**Answer: D.** CloudWatch Embedded Metric Format (EMF) lets CloudWatch automatically parse custom metrics straight out of structured JSON logs — no separate instrumentation code needed — and AWS's open-source libraries make it easy to emit logs in that format from Lambda. `PutMetricData` isn't an EventBridge API at all (it belongs to CloudWatch), Lambda Insights doesn't extract arbitrary custom metrics from log content, and Firehose→Redshift is far more infrastructure than needed for real-time metric extraction and alarming.

---

### Q5 — AWS Compute
A developer building a serverless URL shortener (API Gateway + DynamoDB + Lambda) needs both the application code and the infrastructure stack written in Python, with the stack code reusable for future updates.

Which action meets these requirements?
- A. Use AWS CloudShell to build the stack; Python for Lambda logic.
- B. Use the AWS SDK for Python (boto3) to build the stack; Python for Lambda logic.
- C. Use AWS CDK to build the stack; Python for Lambda logic.
- D. Use AWS CloudFormation to build the stack; Python for Lambda logic.

**Answer: C.** AWS CDK lets you define infrastructure directly in Python (or other supported languages), compiling down to a CloudFormation template under the hood, and its constructs/commands make updating deployed resources straightforward — a perfect match for "stack in Python, reusable for updates." Plain CloudFormation only accepts JSON/YAML, not Python, for defining resources. Boto3 is just an AWS SDK library for making API calls programmatically — it's not an infrastructure-as-code framework. CloudShell is just a browser-based terminal; it doesn't provide any reusable IaC modeling capability.

---

### Q6 — AWS Developer Tools
A Lambda function created via CloudFormation runs without errors but produces no logs — no log group or stream exists. Its execution role trusts the Lambda service but has no permissions, and there's no resource-based policy.

What must be done to fix this?
- A. Add the `AWSLambdaBasicExecutionRole` managed policy to the execution role.
- B. Attach a resource-based policy with `logs:PutLogEvents`.
- C. Attach a resource-based policy with `logs:CreateLogGroup`, `logs:CreateLogStream`, and `logs:PutLogEvents`.
- D. Add the `CloudWatchLambdaInsightsExecutionRolePolicy` managed policy to the execution role.

**Answer: A.** Logging permissions belong on the execution role, not a resource-based policy (which only controls who can invoke the function). The Console auto-attaches `AWSLambdaBasicExecutionRole` when creating a function there, but Infrastructure-as-Code tools like CloudFormation require attaching it explicitly. Lambda Insights' managed policy serves a narrower, different purpose and isn't a substitute for basic logging permissions.

---

### Q7 — AWS Management & Governance
An application in us-east-1 needs to be recreated in us-east-2, ap-northeast-1, and ap-southeast-1 using the same AMI, via a CloudFormation template.

What is the most suitable way to configure the template?
- A. Copy the AMI to each region; use a Mappings section keyed by region, retrieved via `Fn::GetAtt`.
- B. Copy the AMI to each region; use a Parameters section keyed by region, retrieved via `Ref`.
- C. Copy the AMI to each region; use a Mappings section keyed by region, retrieved via `Fn::FindInMap`.
- D. Copy the AMI to each region; use a Mappings section keyed by region, retrieved via `Fn::ImportValue`.

**Answer: C.** Since AMIs are region-specific, they must first be copied into each target region; the CloudFormation `Mappings` section is exactly built for looking up per-region values like this, and `Fn::FindInMap` is the specific intrinsic function used to retrieve values out of a Mappings section. `Fn::ImportValue` only pulls output values exported by a *different stack*, not entries from a Mappings section. `Fn::GetAtt` retrieves an attribute from a resource defined in the template, not a mapping value. A Parameters section could technically hold per-region values, but it doesn't give the same clean per-region key-based lookup structure that Mappings + `FindInMap` provides.

---

### Q8 — AWS Developer Tools
A team uses CodeBuild to compile a website from GitHub source, configured to run through a proxy server. A `RequestError` timeout appears in CloudWatch whenever CodeBuild is accessed.

What is a possible solution?
- A. Modify the `phases` element of `AppSpec.yml` in the source root.
- B. Modify the `proxy` element of `buildspec.yml` in the source root.
- C. Modify the `proxy` element of `AppSpec.yml` in the source root.
- D. Modify the `artifacts` element of `buildspec.yml` in the source root.

**Answer: B.** When CodeBuild runs behind an explicit proxy, `buildspec.yml` needs a `proxy` element (with settings like `upload-artifacts` and `logs`) to route traffic correctly. AppSpec files belong to CodeDeploy, not CodeBuild, so both AppSpec-based options are out of scope entirely, and the `artifacts` element only controls where build output goes — it has nothing to do with proxy configuration.

---

### Q9 — AWS Security, Identity, & Compliance (Select THREE)
Static assets in an S3 bucket in the production account need to be accessed by a user in the development account, with no sharing of full credentials between accounts.

Which steps delegate this access?
- A. On the production account, create an IAM role specifying the development account as a trusted entity.
- B. Set the policy granting S3 access for the IAM role created in the development account.
- C. In the development account, create a policy using STS to assume the production account's IAM role, and attach it to IAM users.
- D. Set the policy granting S3 access for the IAM role created in the production account.
- E. On the development account, create an IAM role specifying the production account as a trusted entity.
- F. In the production account, create a policy using STS to assume the development account's IAM role, and attach it to IAM users.

**Answer: A, D, C.** Since the bucket lives in production, both the role and its S3-access permissions must be created there, trusting the dev account as an entity allowed to assume it. Then, in the development account, IAM users are granted a policy letting them call `sts:AssumeRole` against that production role. Creating the role or its policy in the development account instead (E, B) puts them in the wrong account, and building the assume-role policy in production (F) is also backwards — that policy belongs in the account being delegated access, i.e., development.

---

### Q10 — AWS Security, Identity, & Compliance
A mobile app authenticates with an Identity Provider via its SDK and Amazon Cognito. Once authenticated, the IdP's OAuth/OIDC token is passed to Cognito.

What is returned to eventually provide the user temporary, limited-privilege AWS credentials?
- A. Cognito SDK
- B. Cognito ID
- C. Cognito Key Pair
- D. Cognito API

**Answer: B.** Once the IdP token is handed to Cognito, Cognito returns a unique Cognito identity ID for that user, which the Identity Pool then uses to vend temporary, limited-privilege AWS credentials. The SDK is just a client library for interacting with Cognito, not an identifier returned by the service. There's no such thing as a "Cognito Key Pair" in this flow, and the "Cognito API" is just the interface for interacting with the service, not a returned identifier.

---

### Q11 — AWS Networking & Content Delivery
A media analytics company serves aggregated video metrics (updated every 24 hours) via API Gateway + Lambda, sourced from a precomputed S3 file. Traffic surges have caused latency, and the company wants to improve responsiveness without changing the backend architecture.

Which approach improves responsiveness?
- A. Use Amazon ElastiCache to store frequently requested data in memory.
- B. Enable Amazon API Gateway caching.
- C. Enable CORS on API Gateway.
- D. Configure CloudFront as a caching layer in front of API Gateway.

**Answer: B.** API Gateway's built-in caching stores responses for a configurable TTL — set to match the 24-hour data refresh cycle — letting repeated requests be served instantly without invoking the backend Lambda at all, and it requires no backend architecture changes. ElastiCache would work but requires integrating a new cache layer into the Lambda code itself, which does change the backend. CloudFront could also help, but it's a heavier, more complex setup better suited to globally-distributed static content than a straightforward API caching need that API Gateway already solves natively. CORS is unrelated to performance — it only governs cross-origin browser access rules.

---

### Q12 — AWS Security, Identity, & Compliance (Select TWO)
An app uses AWS KMS to encrypt data before storing it in DynamoDB.

Which features are supported by KMS?
- A. Automatic key rotation for KMS keys in custom key stores.
- B. Importing custom key material to an asymmetric KMS key.
- C. Using AWS Certificate Manager as a custom key store.
- D. Re-enabling disabled keys.
- E. Creation of symmetric encryption and asymmetric KMS keys.

**Answer: D & E.** KMS supports both disabling and re-enabling keys, plus creating both symmetric and asymmetric keys. Custom key stores are backed by AWS CloudHSM, not Certificate Manager. You can only import custom key material into symmetric keys, not asymmetric ones. And automatic key rotation only applies to symmetric keys generated normally by the service — it's not available for keys in custom key stores, asymmetric keys, or keys with imported material.

---

### Q13 — AWS Compute
A developer plans to deploy a microservice application on Elastic Beanstalk using a multi-container Docker environment.

How should the container definitions be configured?
- A. Configure them in the ECS Console when building the Docker environment.
- B. Configure them in `Dockerrun.aws.json.config`, placed inside the `.ebextensions` folder.
- C. Use the `eb config` command to configure them.
- D. Configure them in the `Dockerrun.aws.json` file.

**Answer: D.** Elastic Beanstalk's multi-container Docker platform specifically requires a `Dockerrun.aws.json` file (placed at the same level as your application source, not nested inside `.ebextensions`) to define how multiple containers run together on each instance. Since the deployment must go through Elastic Beanstalk (not raw ECS), configuring container definitions directly in the ECS Console bypasses the required Beanstalk-specific configuration file. `eb config` only manages environment configuration settings, not container definitions.

---

### Q14 — AWS Compute
A developer uses AWS SAM locally to build a serverless Python application, having already defined dependencies in `requirements.txt`.

What are the correct steps to test and deploy?
- A. Build the SAM template locally; run `sam deploy` to package/deploy from AWS CodePipeline.
- B. Run `sam init`; build the SAM template locally; call `sam deploy` to package/deploy from an S3 bucket.
- C. Build the SAM template locally and call `sam deploy` to package/deploy from an S3 bucket.
- D. Upload and build the SAM template on an EC2 instance; run `sam deploy`.

**Answer: C.** Since the app's runtime and dependency structure are already established, the remaining steps are simply to build (`sam build`, implied here) and then run `sam deploy`, which packages the app and deploys it via an S3 bucket and CloudFormation under the hood. `sam init` is only needed to scaffold a brand-new project structure — unnecessary here since that's already done. `sam deploy` doesn't interact directly with CodePipeline (that's a separate CI/CD layer you could add on top, but it's not part of the base deploy flow). Building on an EC2 instance is unnecessary and costly — SAM CLI is designed to build and deploy directly from your local machine.

---

## Quick Answer Key

| # | Answer |
|---|--------|
| 1 | C |
| 2 | A |
| 3 | D |
| 4 | D |
| 5 | C |
| 6 | A |
| 7 | C |
| 8 | B |
| 9 | A, D, C |
| 10 | B |
| 11 | B |
| 12 | D, E |
| 13 | D |
| 14 | C |
