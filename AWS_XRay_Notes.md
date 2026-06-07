# 🔍 AWS X-Ray — Complete Simplified Notes

> *Merged from TutorialsDojo + arkalim/DevOps exam notes*

---

## What is AWS X-Ray?

AWS X-Ray is a **debugging and performance analysis tool** for distributed applications (microservices, serverless, etc.). It traces requests end-to-end across all services so you can pinpoint exactly where slowdowns or errors happen.

> 💡 **Analogy:** You ordered food through an app. The order flows through: your app → API Gateway → Lambda → DynamoDB. If something breaks, X-Ray is the GPS tracker showing exactly where in that chain things went wrong.

---

## ✅ Gap Analysis — What Each Source Covered

| Topic | TutorialsDojo | arkalim/DevOps Notes |
|---|---|---|
| Core concepts (segment, trace, etc.) | ✅ | ✅ |
| Sampling rules & reservoir/rate | Partial | ✅ Full detail |
| X-Ray Daemon setup details | Partial | ✅ Full detail |
| Write & Read APIs | ❌ Missing | ✅ |
| Elastic Beanstalk config details | Basic | ✅ Full detail |
| Troubleshooting (EC2 & Lambda) | ❌ Missing | ✅ |
| How to enable X-Ray (step by step) | ❌ Missing | ✅ |
| Adaptive sampling / new features | ✅ | ❌ |
| IAM policy breakdown | ✅ Full | Partial |
| Service graph / service map | ✅ | ✅ |
| OTLP support | ✅ | ❌ |

---

## Key Concepts — Plain English

### Segment
A report from one service about what it did for a request. Minimum info recorded: name, ID, start time, trace ID, end time.

### Subsegment
A smaller breakdown inside a segment. Used to record individual downstream calls — e.g. "within that Lambda, the DynamoDB call took 80ms." Subsegments can nest inside other subsegments.

> 💡 For services that don't send their own segments (like DynamoDB), X-Ray auto-creates **inferred segments** from subsegments so they still appear on the service map.

### Trace
The complete journey of ONE request across ALL services. A trace = all segments + subsegments from that single request stitched together.

### Trace ID
A unique ID that tracks a request as it hops across services. Passed along in the request header so every service knows it's part of the same trace.

### Service Graph
A JSON document describing all services and how they connect. The X-Ray console renders this as a visual **Service Map**. Retained for **30 days**.

### Edges
The arrows on the service map connecting services to each other.

### Annotations
Key-value pairs that are **indexed and searchable**. Use when you want to filter/query traces by that data.
- Example: `userID=12345`, `env=production`

### Metadata
Key-value pairs that are **stored but NOT searchable**. Use for extra context (like a full request body) you want to keep but don't need to query on.

> 💡 **Memory trick:** Annotations = **A**llow searching. Metadata = just **M**ore info.

### Groups
Saved collections of traces defined by a filter expression. Like a saved search.

### Filter Expressions
Queries to find specific traces. Example: find all traces with errors, or all requests from a specific user.

---

## How to Enable X-Ray — Step by Step

### Step 1 — Instrument your application code

Import the X-Ray SDK into your app.

**Supported languages:** Java, Python, Go, Node.js, .NET

The SDK automatically captures:
- Calls to AWS services (via AWS SDK)
- HTTP/HTTPS requests
- Database calls (MySQL, PostgreSQL, DynamoDB)
- Queue calls (SQS)

### Step 2 — Install and run the X-Ray Daemon

The SDK does **NOT** send data directly to X-Ray. It sends JSON segment data over **UDP port 2000** to the **X-Ray Daemon**, which buffers and uploads it in batches.

| Platform | How to set up |
|---|---|
| EC2 | Install via user data script + IAM role with X-Ray write permissions |
| Lambda | Daemon is already provided — just enable X-Ray in config |
| Elastic Beanstalk | Daemon included — enable via console or `.ebextensions/xray-daemon.config` |
| ECS | Run daemon as a sidecar container |

> ⚠️ The X-Ray daemon is **NOT** available for Multicontainer Docker on Elastic Beanstalk.

### Step 3 — Grant IAM permissions

Every component (daemon, Lambda, EC2 instance) must have IAM permission to write to X-Ray (`AWSXRayDaemonWriteAccess`).

---

## Sampling Rules — How X-Ray Decides What to Trace

Sampling prevents overwhelming X-Ray with data (and keeps costs low).

### Default Rule

| Term | Value | Meaning |
|---|---|---|
| **Reservoir** | 1 request/second | First request each second is ALWAYS recorded |
| **Rate** | 5% | 5% of all additional requests beyond the reservoir |

> 💡 Think of it like a bouncer at a club: the first person each second always gets in (reservoir). After that, only 5% of remaining people get in (rate).

### Custom Sampling Rules
- Create your own rules with custom reservoir sizes and rates.
- No code changes needed — changes take effect immediately.

### Adaptive Sampling *(newer feature)*
- **Sampling Boost** — Temporarily increases sampling during anomalies.
- **Anomaly Span Capture** — Captures traces for unusual/error requests even if they'd normally be skipped by the sampling rule.

---

## X-Ray SDK — What It Provides

| Component | What it does |
|---|---|
| **Interceptors** | Auto-trace all incoming HTTP requests to your app |
| **Client handlers** | Trace outgoing AWS SDK calls (to S3, DynamoDB, etc.) |
| **HTTP client** | Trace calls to external HTTP APIs |

---

## X-Ray APIs — Important for Exams

### Write APIs *(used by daemon/app to send data)*

| API | What it does |
|---|---|
| `PutTraceSegments` | Upload segment documents to X-Ray |
| `PutTelemetryRecords` | Upload daemon health telemetry (e.g. SegmentsReceivedCount, BackendConnectionErrors) |
| `GetSamplingRules` | Fetch sampling rules (daemon uses this to know what to send) |
| `GetSamplingTargets` | Get dynamic sampling targets |

### Read APIs *(used to query/view data)*

| API | What it does |
|---|---|
| `GetServiceGraph` | Get the full service map |
| `GetTraceSummaries` | Get IDs + annotations for traces in a time window |
| `BatchGetTraces` | Get full trace details by trace ID |
| `GetTraceGraph` | Get service graph for specific trace IDs |

---

## AWS Service Integrations

| Service | Integration Type | What happens |
|---|---|---|
| AWS Lambda | Active + Passive | Traces requests; adds 2 nodes to service map (Lambda service + function) |
| Amazon API Gateway | Active + Passive | Uses sampling rules; adds gateway stage node to map |
| Application Load Balancer | Request Tracing | Stamps trace ID into request header before forwarding to target group |
| AWS Elastic Beanstalk | Tooling | Daemon included on most platforms; enable via console or `.ebextensions` |
| Amazon ECS | Active | Daemon runs as a sidecar container |
| Amazon EC2 | Tooling | Must install daemon manually via user data script |

### Four Integration Modes Explained

| Mode | What it means |
|---|---|
| **Active** | Instruments and samples requests itself |
| **Passive** | Only traces requests already sampled upstream (relay baton pass) |
| **Request Tracing** | Just stamps a trace ID on the header and passes it along |
| **Tooling** | Just runs the daemon to receive SDK data |

---

## X-Ray with Elastic Beanstalk — Special Notes

- The X-Ray daemon is **bundled** in Elastic Beanstalk platforms.
- Enable it via the **console toggle** OR by adding `.ebextensions/xray-daemon.config` to your project.
- Your **instance profile** (IAM role for EC2 instances) needs X-Ray write permissions.
- Your application code must still be **instrumented with the X-Ray SDK** — the daemon alone isn't enough.
- ⚠️ **Multicontainer Docker on Beanstalk does NOT include the X-Ray daemon.**

---

## Troubleshooting — X-Ray Not Working?

### On EC2
- ✅ Is the X-Ray daemon installed and running?
- ✅ Does the EC2 IAM role have `AWSXRayDaemonWriteAccess`?

### On Lambda
- ✅ Does the Lambda execution role have `AWSXRayWriteOnlyAccess`?
- ✅ Is X-Ray tracing enabled in the Lambda configuration?
- ✅ Is the X-Ray SDK imported in your function code?

---

## IAM Policies — Which One to Use?

| Policy | Use for |
|---|---|
| `AWSXrayReadOnlyAccess` | Developers who only need to VIEW traces and service maps |
| `AWSXRayDaemonWriteAccess` | The daemon / app uploading trace data to X-Ray |
| `AWSXrayFullAccess` | Admin configuring encryption keys and sampling rules |

> 💡 Always follow least privilege. If someone just needs to look at traces in the console, `ReadOnly` is enough.

---

## X-Ray Security

| Layer | How |
|---|---|
| **Authorization** | IAM controls who can call X-Ray APIs |
| **Encryption at rest** | KMS encrypts stored trace data |
| **Encryption in transit** | TLS for all data in motion |

---

## Pricing

| Item | Detail |
|---|---|
| Billing basis | Per trace recorded, retrieved, and scanned |
| Max trace size | 500 KB |
| Data retention | 30 days at no extra cost |
| Free tier | First 100,000 traces recorded per month are free |

---

## Quick-Reference Summary Card

| Term | One-line reminder |
|---|---|
| Segment | One service's report on what it did |
| Subsegment | Breakdown within a segment (e.g. one DB call) |
| Trace | Complete journey of one request across all services |
| Trace ID | Unique ID passed between services to link them |
| Service Graph | Visual map of all services + connections |
| Annotations | **Searchable** key-value tags |
| Metadata | **Non-searchable** key-value data |
| X-Ray Daemon | Local buffer on UDP 2000 that batches + uploads data |
| Reservoir | First 1 req/sec always recorded (default sampling) |
| Rate | 5% of requests beyond reservoir (default sampling) |
| Data retention | 30 days |
| `PutTraceSegments` | API to send segment data to X-Ray |
| `GetServiceGraph` | API to fetch the service map |

---

*Sources: [TutorialsDojo AWS X-Ray](https://tutorialsdojo.com/aws-x-ray/) · [arkalim Notion Notes](https://arkalim.notion.site/Notes-143374c83daa4d4991b07400056a2aa9) · [AWS Documentation](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html)*
