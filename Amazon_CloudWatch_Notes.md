# 📊 Amazon CloudWatch — Complete Simplified Notes
> *DVA-C02 Developer Associate | Merged from TutorialsDojo + arkalim Notion + AWS Docs*

---

## What is Amazon CloudWatch?

CloudWatch is AWS's **central monitoring and observability service**. It collects metrics, logs, and events from AWS resources and your own applications — and lets you visualize, alert on, and react to them.

> 💡 **Analogy:** Think of CloudWatch as the dashboard of a car. It shows your speed (metrics), records a trip log (logs), alerts you when fuel is low (alarms), and can automatically trigger actions like calling roadside assistance (EventBridge rules).

**Four main pillars:**
- **Metrics** — Numerical data over time (CPU usage, error count, etc.)
- **Logs** — Text output from your applications and AWS services
- **Alarms** — Trigger actions when a metric crosses a threshold
- **Events / EventBridge** — React to changes in your AWS environment

---

## ✅ Gap Analysis — What Each Source Covered

| Topic | TutorialsDojo | arkalim Notion |
|---|---|---|
| Metrics, Namespaces, Dimensions | ✅ | ✅ |
| Standard vs High-Resolution metrics | ✅ | ✅ |
| Custom metrics via PutMetricData | ✅ | ✅ |
| Embedded Metric Format (EMF) | ❌ Missing | ❌ Missing (DVA-C02 key topic) |
| CloudWatch Agent (EC2 / on-premises) | ✅ | ✅ |
| CloudWatch Logs (groups, streams, retention) | ✅ | ✅ |
| Metric Filters | ✅ | ✅ |
| Subscription Filters | ✅ | Partial |
| CloudWatch Logs Insights | ✅ | ✅ |
| Alarms (states, thresholds, actions) | ✅ | ✅ |
| Composite Alarms | Partial | ✅ |
| CloudWatch Dashboards | ✅ | Partial |
| CloudWatch Events / EventBridge | ✅ | ✅ |
| Lambda Insights | ❌ | ❌ (DVA-C02 topic) |
| Container Insights | ✅ | ❌ |
| CloudWatch pricing | ✅ | ❌ |

---

## 1. CloudWatch Metrics

### What is a Metric?
A metric is a **time-ordered set of data points** representing one measurable value — for example, EC2 CPU usage over time. Every metric lives in a **namespace** and can be filtered using **dimensions**.

### Key Terms

| Term | Plain English |
|---|---|
| **Namespace** | A folder/category for metrics. AWS uses `AWS/EC2`, `AWS/Lambda`, etc. You create your own like `MyApp/Orders` |
| **Dimension** | A label/filter for a metric — e.g. `InstanceId=i-1234`. Up to 30 dimensions per metric |
| **Metric Name** | The name of what you're measuring — e.g. `CPUUtilization`, `ErrorCount` |
| **Data Point** | One value at one timestamp |
| **Period** | The time window used to aggregate data points (e.g. 60 seconds) |
| **Statistics** | How data is aggregated: Average, Sum, Min, Max, SampleCount |

> 💡 **Dimensions are critical:** CloudWatch treats each unique combination of dimensions as a *separate* metric — even if the metric name is the same. So `CPUUtilization` for instance A and instance B are two separate metrics.

### Standard vs High-Resolution Metrics

| Type | Resolution | Use case |
|---|---|---|
| **Standard** (default) | 1-minute granularity | Most AWS services publish at this level for free |
| **High-Resolution** | 1-second granularity | Custom metrics you publish for sub-minute monitoring |

- Standard metrics: alarm period must be **60 seconds or multiple of 60**
- High-resolution metrics: alarm period can be **10s or 30s** (more expensive)

### Built-in EC2 Metrics (Free)
Available by default, reported every **5 minutes**:
- CPU Utilization, Network In/Out
- Disk Read/Write (for instance store only)
- Status Check metrics

> ⚠️ **EC2 does NOT send RAM, Disk Space, or Swap utilization by default.** These require the CloudWatch Agent.

Enable **Detailed Monitoring** on EC2 to get **1-minute** granularity (costs extra).

---

## 2. Custom Metrics

When AWS's built-in metrics aren't enough, you publish your own.

### How to Publish Custom Metrics

**Method 1 — AWS CLI / SDK (`PutMetricData`)**
```bash
aws cloudwatch put-metric-data \
  --namespace "MyApp/Orders" \
  --metric-name "FailedOrders" \
  --value 3 \
  --dimensions Environment=Production
```

**Method 2 — CloudWatch Agent** (for system-level metrics like RAM, disk)

**Method 3 — Embedded Metric Format / EMF** (see below — key DVA-C02 topic)

### Metric Resolution for Custom Metrics
- Default: **Standard (60-second)** resolution
- Optional: **High-resolution (1-second)** — specify `StorageResolution=1` in `PutMetricData`

### Timestamps
- Custom metrics can be published with timestamps up to **2 weeks in the past** or **2 hours in the future**
- Useful for backfilling data

---

## 3. Embedded Metric Format (EMF) — Key DVA-C02 Topic

### What is EMF?
A way to **embed custom metric data inside your logs**. Instead of making a separate `PutMetricData` API call, you write structured JSON logs to stdout/CloudWatch Logs. CloudWatch automatically extracts the metrics from those logs.

> 💡 **Why it matters for Lambda:** Lambda already sends stdout to CloudWatch Logs. With EMF, you get custom metrics "for free" without any extra API calls — reducing latency and cost.

### How it works
1. Your app writes a specially-formatted JSON log entry
2. CloudWatch Logs recognizes the `_aws` key and extracts metric values
3. Metrics appear in CloudWatch just like any other custom metric

### EMF Structure (example)
```json
{
  "_aws": {
    "Timestamp": 1574109732004,
    "CloudWatchMetrics": [{
      "Namespace": "MyApp",
      "Dimensions": [["Environment"]],
      "Metrics": [{ "Name": "OrderProcessingTime", "Unit": "Milliseconds" }]
    }]
  },
  "Environment": "Production",
  "OrderProcessingTime": 247
}
```

### EMF Limits
- Up to **100 metrics** per EMF log entry
- Up to **30 dimensions** per metric
- Use AWS EMF libraries (Python, Node.js, Java) to avoid writing raw JSON

### When to use EMF vs PutMetricData
| Scenario | Use |
|---|---|
| Lambda, ECS, or any app already logging to CloudWatch | **EMF** — no extra API call |
| Standalone application or batch job | **PutMetricData** |
| High-cardinality data (many unique dimension values) | **EMF** |

---

## 4. CloudWatch Agent

The CloudWatch Agent is software you install on **EC2 instances or on-premises servers** to send additional metrics and logs that aren't collected by default.

### What it collects
- **System metrics:** RAM, disk usage, swap, number of processes, disk I/O — things EC2 doesn't send by default
- **Logs:** Application logs, system logs, custom log files

### Setup
1. Install the agent on your EC2 instance (via SSM or manually)
2. Create an agent config file (defines what metrics/logs to collect)
3. Attach an IAM role with `CloudWatchAgentServerPolicy` to the EC2 instance
4. Start the agent

> 💡 **On-premises too:** The CloudWatch Agent also works on physical servers in your data centre — useful for hybrid monitoring.

### CloudWatch Agent vs SSM Agent
- **CloudWatch Agent** = collects metrics and logs
- **SSM Agent** = manages instances (run commands, patch, etc.)
- Both can be installed together and work independently

---

## 5. CloudWatch Logs

CloudWatch Logs is where all log data lives. Organized in a hierarchy:

```
Log Group (e.g. /aws/lambda/my-function)
  └── Log Stream (e.g. one stream per Lambda container instance)
        └── Log Events (individual log entries with timestamps)
```

### Key Concepts

**Log Group** — Container for log streams sharing the same retention, access control, and monitoring settings. Define retention at the log group level.

**Log Stream** — Sequence of log events from one source (e.g. one EC2 instance, one Lambda execution environment). No limit on number of streams per group.

**Log Event** — One individual log entry: a timestamp + message.

### Log Retention
- Default: **Never expires** (logs stored forever — a common cost leak!)
- Configurable from **1 day to 10 years** (or indefinite)
- Set at log group level; applies to all streams within it
- Old log events may take up to **72 hours** to actually delete after the retention period

> ⚠️ **Exam tip:** Always set a retention policy to avoid unexpected storage costs.

### Sending Logs to CloudWatch
| Source | How logs get there |
|---|---|
| AWS Lambda | Automatically — no setup needed |
| EC2 | Install CloudWatch Agent |
| ECS / EKS | Use `awslogs` log driver or CloudWatch Agent |
| API Gateway | Enable access logging in stage settings |
| On-premises servers | Install CloudWatch Agent |
| Any app | Use AWS SDK / `PutLogEvents` API |

---

## 6. Metric Filters

Turn log data into CloudWatch metrics — without changing your application code.

### How it works
1. You define a **filter pattern** (e.g. match lines containing `ERROR`)
2. CloudWatch counts how many log events match per minute
3. That count becomes a CloudWatch metric
4. You can set alarms on that metric

### Example use cases
- Count how many `ERROR` or `EXCEPTION` entries appear per minute
- Extract the HTTP response code from access logs and track 5xx errors
- Track order amounts from structured JSON logs

> ⚠️ **Important limit:** Metric filters only process **new log data** going forward — they do NOT retroactively process old log events.

---

## 7. CloudWatch Logs Insights

An **interactive query tool** for your log data. Think SQL for logs.

### What you can do
- Search across multiple log groups simultaneously
- Filter, aggregate, sort, and count log events
- Visualize results as time-series graphs or bar charts
- Pin queries to CloudWatch Dashboards

### Query Syntax Basics
```sql
-- Find errors in the last hour
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20

-- Count log events per 5-minute window
fields @timestamp
| stats count(*) as eventCount by bin(5m)
| sort @timestamp asc
```

### Useful Built-in Fields
| Field | Meaning |
|---|---|
| `@timestamp` | Time the log event was ingested |
| `@message` | Full raw log message |
| `@logStream` | Name of the log stream |
| `@log` | Log group identifier |

> 💡 **DVA-C02 tip:** If a question asks how to search/query/analyze logs, the answer is almost always **CloudWatch Logs Insights**.

---

## 8. Subscription Filters

**Stream log data in real-time** to another AWS service for processing.

### Destinations
- **AWS Lambda** — Process/transform logs in real time
- **Amazon Kinesis Data Streams** — High-volume log streaming
- **Amazon Data Firehose** — Deliver logs to S3, Redshift, OpenSearch
- **Amazon OpenSearch** — Full-text log search

### Limits
- Max **2 subscription filters per log group**
- Cross-account delivery is supported

### Use cases
- Trigger a Lambda function when a specific error appears in logs
- Archive all logs to S3 via Firehose for long-term storage
- Real-time log analysis with Kinesis + custom consumers

---

## 9. CloudWatch Alarms

Alarms **watch a single metric** and take action when it crosses a threshold.

### Three Alarm States

| State | Meaning | Console colour |
|---|---|---|
| **OK** | Metric is within the acceptable range | Green / no colour |
| **ALARM** | Metric has breached the threshold | Red |
| **INSUFFICIENT_DATA** | Not enough data yet (new alarm, or metric stopped reporting) | Grey |

### Alarm Actions — What can trigger
- **SNS notification** — Email, SMS, HTTP, trigger Lambda via SNS
- **EC2 actions** — Stop, terminate, reboot, or recover an instance
- **Auto Scaling** — Scale out or scale in
- **Systems Manager** — Create OpsItem or incident

### Key Configuration Settings

| Setting | What it means |
|---|---|
| **Period** | Length of each evaluation window (e.g. 300 seconds) |
| **Evaluation Periods** | How many consecutive periods to evaluate |
| **Datapoints to Alarm** | How many of those periods must breach the threshold before ALARM triggers |
| **Threshold** | The value to compare against (e.g. CPU > 80%) |
| **Missing Data Treatment** | What to do when data is missing: treat as OK, ALARM, Breaching, or Ignore |

> 💡 **Example:** Period=5min, EvaluationPeriods=3, DatapointsToAlarm=2 means: "If 2 out of the last 3 five-minute windows exceed the threshold, go into ALARM."

### Composite Alarms

Combine multiple alarms using **AND / OR / NOT** logic to reduce alert noise.

```
ALARM("HighCPU") AND ALARM("HighMemory")
```

Only fires when BOTH conditions are true — prevents false positives.

- Can reference up to **100 child alarms**
- Actions: SNS notifications, OpsItems, incidents only (NOT EC2 or Auto Scaling actions)
- Useful for **maintenance windows**: create a suppressor alarm that silences the composite alarm during deployments

> 💡 **DVA-C02 tip:** If a question asks how to avoid alert storms or reduce alarm noise, the answer is **Composite Alarms**.

---

## 10. CloudWatch Dashboards

Visual display of metrics and alarms — shareable across teams.

### Key Points
- Dashboards are **global** — can show metrics from multiple regions on one screen
- Add widgets: line charts, number displays, bar charts, alarms, log query results
- Can be shared publicly (read-only URL) without AWS login
- Automatic refresh: every 10 seconds, 1 minute, 2 minutes, 5 minutes, or 15 minutes

### DVA-C02 Relevance
- You can add **CloudWatch Logs Insights** query results to a dashboard
- Add alarm states to dashboards for a health overview

---

## 11. CloudWatch Events → Amazon EventBridge

> ⚠️ **CloudWatch Events has been renamed to Amazon EventBridge.** They are the same underlying service. The exam may use either name.

### What it does
Reacts to **events** (state changes, scheduled times, API calls) and routes them to targets.

### Event Sources
- **AWS services** — EC2 state change, CodeDeploy success/fail, S3 object upload, etc.
- **CloudWatch Alarms** — State change from OK → ALARM → OK
- **Scheduled events** — Cron or rate expressions (like a cron job in the cloud)
- **Custom events** — Your application publishes custom events

### Targets (what can be triggered)
- Lambda functions
- SNS topics / SQS queues
- Step Functions
- CodePipeline
- Kinesis Data Streams
- ECS tasks

### Scheduled Events (Cron jobs)
```
# Every 5 minutes
rate(5 minutes)

# Every weekday at 9 AM UTC
cron(0 9 ? * MON-FRI *)
```

> 💡 **DVA-C02 tip:** If a question asks how to run a Lambda on a schedule, the answer is **EventBridge scheduled rule**.

---

## 12. CloudWatch Logs Insights — Lambda-Specific Metrics

AWS Lambda automatically sends these useful **built-in log fields** to CloudWatch Logs:

| Field | Meaning |
|---|---|
| `@duration` | How long the function ran |
| `@billedDuration` | Duration rounded up to nearest ms (what you're charged) |
| `@memorySize` | Configured memory limit |
| `@maxMemoryUsed` | Actual memory used during invocation |
| `@initDuration` | Cold start initialization time |
| `@xrayTraceId` | X-Ray trace ID (if tracing enabled) |

> 💡 Query for cold starts:
> ```sql
> filter @type = "REPORT" and @initDuration > 0
> | stats count(*) as coldStarts by bin(1h)
> ```

---

## 13. Lambda Insights (DVA-C02 Topic)

A feature of CloudWatch that gives **enhanced monitoring for Lambda functions** — beyond basic metrics.

### What it adds
- Memory usage, CPU usage, network I/O per invocation
- Cold start count and duration
- Errors and throttles
- Custom metrics via EMF

### How to enable
- Add the **Lambda Insights layer** to your Lambda function
- Attach `CloudWatchLambdaInsightsExecutionRolePolicy` to the Lambda IAM role

---

## 14. Container Insights

Enhanced monitoring for **ECS, EKS, and Kubernetes** clusters.

- Collects CPU, memory, disk, and network metrics at container level
- Aggregates by cluster, service, task, and pod
- Requires the CloudWatch Agent deployed as a sidecar or DaemonSet

---

## 15. CloudWatch Contributor Insights

Analyzes **high-cardinality log data** to find top contributors to problems.

### Example use cases
- Find the top 10 IP addresses making the most requests
- Find the most erroring Lambda function ARNs
- Identify the heaviest DynamoDB partition keys

### How it works
- You define rules pointing at a log group + fields to analyze
- CloudWatch builds a time-series report of top-N contributors
- Results show up as CloudWatch metrics and can trigger alarms

---

## Quick-Reference: Important Numbers & Limits

| Item | Value |
|---|---|
| Default EC2 metric interval | 5 minutes |
| Detailed monitoring EC2 metric interval | 1 minute |
| High-resolution custom metric interval | 1 second |
| Standard alarm period minimum | 60 seconds |
| High-resolution alarm period options | 10s or 30s |
| Default log retention | Never expires (infinite) |
| Max subscription filters per log group | 2 |
| Max dimensions per metric | 30 |
| Max metrics per EMF log entry | 100 |
| CloudWatch data retention for metrics | 15 months |
| Composite alarm max child alarms | 100 |
| Custom metric timestamp range | Up to 2 weeks past, 2 hours future |

---

## DVA-C02 Exam Cheat Sheet

| Scenario | Answer |
|---|---|
| EC2 RAM/Disk not showing in CloudWatch | Install **CloudWatch Agent** |
| Publish custom app metrics without separate API call | **Embedded Metric Format (EMF)** |
| Search/query log data interactively | **CloudWatch Logs Insights** |
| Stream logs to Lambda / Kinesis in real time | **Subscription Filters** |
| Create a metric from log data (e.g. count errors) | **Metric Filters** |
| Run Lambda on a schedule | **EventBridge scheduled rule** |
| Reduce alarm noise / avoid false positives | **Composite Alarms** |
| Monitor Lambda cold starts & memory | **Lambda Insights** |
| Identify top-N callers causing throttling | **Contributor Insights** |
| React to EC2 state change automatically | **EventBridge rule → Lambda/SNS** |
| View metrics from multiple regions in one place | **CloudWatch Dashboard** |
| Alarm on metric only during business hours | **Composite Alarm with suppressor** |

---

## CloudWatch vs CloudTrail — Don't Confuse Them

| | CloudWatch | CloudTrail |
|---|---|---|
| **Purpose** | Monitor performance & health | Audit API calls |
| **Answers** | "How is my app performing?" | "Who did what, when?" |
| **Data type** | Metrics + Logs + Events | API call history |
| **DVA-C02 use** | Debugging, alerting, dashboards | Security auditing, compliance |

> 💡 They complement each other: CloudTrail writes API call logs → CloudWatch Logs, where metric filters can alarm on suspicious patterns like `ConsoleLoginFailed`.

---

*Sources: [TutorialsDojo CloudWatch](https://tutorialsdojo.com/amazon-cloudwatch/) · [arkalim Notion Notes](https://arkalim.notion.site/CloudWatch-0c16b355ef424fba8b31a606f95fe8f4) · [AWS CloudWatch Docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/) · DVA-C02 Exam Guide*
