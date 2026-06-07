# 🚌 Amazon EventBridge — Complete Simplified Notes
> *DVA-C02 Developer Associate | Merged from TutorialsDojo + arkalim Notion Notes*

---

## What is Amazon EventBridge?

Amazon EventBridge is a **serverless event bus** — a central hub that receives events from different sources and routes them to the right services automatically.

> 💡 **Analogy:** Think of EventBridge like a postal sorting office. Letters (events) arrive from many senders (AWS services, your app, SaaS tools). The sorting office reads the address (event pattern) and delivers each letter to the correct recipient (target) — automatically, with no manual effort.

**Key idea:** EventBridge decouples producers (who send events) from consumers (who react to events). The producer doesn't need to know who is listening — EventBridge handles the routing.

> ⚠️ **Name history:** CloudWatch Events was the old name. EventBridge is the new, more powerful version. The exam may use either name — they are the same service.

---

## ✅ Gap Analysis — What Each Source Covered

| Topic | TutorialsDojo | arkalim Notion |
|---|---|---|
| What is EventBridge / core idea | ✅ | ✅ |
| Event Bus types (default / custom / partner) | ✅ | ✅ |
| Events — structure and JSON format | ✅ | ✅ |
| Rules — event pattern vs schedule | ✅ | ✅ |
| Targets list | ✅ | ✅ |
| Input Transformer | ✅ | ✅ |
| Content-based filtering / event patterns | ✅ | ✅ |
| Schema Registry | ✅ | ✅ |
| Archive & Replay | ✅ | ✅ |
| API Destinations | ✅ | ✅ |
| EventBridge Pipes | ✅ | Partial |
| EventBridge Scheduler | ✅ | ❌ |
| Cross-account / cross-region routing | ✅ | ✅ |
| Resource-based policies | Partial | ✅ |
| Dead Letter Queue (DLQ) | Partial | ✅ |
| Sandbox (testing tool) | ✅ | ❌ |
| Pricing model | ✅ | ❌ |

---

## 1. Core Concepts — Plain English

### Event
A **JSON object** that describes something that happened. Every event has the same outer structure:

```json
{
  "version": "0",
  "id": "abc-123",
  "source": "aws.ec2",
  "account": "123456789012",
  "time": "2024-01-01T10:00:00Z",
  "region": "us-east-1",
  "detail-type": "EC2 Instance State-change Notification",
  "detail": {
    "instance-id": "i-1234567890",
    "state": "running"
  }
}
```

| Field | What it means |
|---|---|
| `source` | Who sent the event (e.g. `aws.ec2`, `aws.s3`, `myapp.orders`) |
| `detail-type` | What type of event it is (e.g. "EC2 Instance State-change Notification") |
| `detail` | The actual payload — the event-specific data |

### Event Bus
The pipeline that events flow through. Think of it as the sorting office itself.

| Type | Description |
|---|---|
| **Default Event Bus** | Automatically exists in every AWS account. Receives events from all AWS services. Cannot be deleted. |
| **Custom Event Bus** | You create it for your own application's events. Isolate domains (e.g. separate bus per team or service). |
| **Partner Event Bus** | Receives events from third-party SaaS providers (e.g. Datadog, Zendesk, PagerDuty). Must be set up through the AWS Partner Network. |

### Rule
A rule **watches the event bus** and decides what to do when a matching event arrives. Two types:

| Type | How it works |
|---|---|
| **Event Pattern Rule** | "If an event matches this JSON pattern, send it to these targets" |
| **Scheduled Rule** | "At this time (or interval), trigger these targets" |

- Each rule can have up to **5 targets**
- One event can match multiple rules — each matching rule fires independently
- If no rule matches, EventBridge does nothing

### Target
The service or endpoint that **receives the event** when a rule fires.

**Supported targets include:**
- AWS Lambda (most common)
- Amazon SQS
- Amazon SNS
- Amazon Kinesis Data Streams / Firehose
- AWS Step Functions
- Amazon ECS tasks
- AWS CodePipeline / CodeBuild
- Another Event Bus (same account, cross-account, or cross-region)
- API Destinations (external HTTP endpoints)
- CloudWatch Logs

---

## 2. Event Patterns — How Filtering Works

An event pattern is a JSON document that defines which events a rule matches. EventBridge compares the pattern to the incoming event — if they align, the rule fires.

> 💡 **Key rule:** The event pattern only needs to match a **subset** of the event's fields. Any field not in the pattern is ignored.

### Example: Match all EC2 instance terminations
```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}
```

### Content-Based Filtering Options

| Filter Type | Example | Meaning |
|---|---|---|
| **Exact match** | `"state": ["running"]` | Value must equal "running" |
| **Prefix match** | `"prefix": "prod-"` | Value must start with "prod-" |
| **Anything-but** | `"anything-but": ["stopped"]` | Any value EXCEPT "stopped" |
| **Numeric range** | `"numeric": [">", 100, "<=", 200]` | Between 100 and 200 |
| **Exists** | `"exists": true` | Field must be present |
| **Null** | `"exists": false` | Field must not exist |
| **IP address (CIDR)** | `"cidr": "10.0.0.0/8"` | IP must fall in this range |

---

## 3. Input Transformer — Reshaping Events Before Delivery

By default, EventBridge sends the **full raw event JSON** to the target. The Input Transformer lets you **customise and reshape** the event before it arrives at the target.

> 💡 **Analogy:** The sorting office can open the letter, extract only the relevant paragraph, and rewrite it in a new format before delivering it.

### Two parts:
1. **Input Path** — A JSON path map that extracts values from the event
2. **Input Template** — A template using those extracted values to create the new payload

### Example
**Event:**
```json
{ "detail": { "instance-id": "i-1234", "state": "terminated" } }
```
**Input Path:**
```json
{ "instance": "$.detail.instance-id", "status": "$.detail.state" }
```
**Input Template:**
```json
"Instance <instance> has changed state to <status>"
```
**What the target receives:**
```
"Instance i-1234 has changed state to terminated"
```

> ⚠️ Input Transformer is supported for most targets (Lambda, SQS, SNS, API Destinations). It is **NOT supported** when the target is a cross-account event bus.

---

## 4. Schema Registry

The Schema Registry is a **catalogue of event structures** — like a dictionary of all the event "shapes" flowing through your system.

### What it gives you
- Stores the JSON schema (structure) of events in OpenAPI 2.0 format
- **Schema Discovery** — EventBridge can automatically detect and register schemas from events flowing through a bus
- **Code Bindings** — Generate code (Java, Python, TypeScript) from a schema so your app already knows the data structure of incoming events
- Shareable across accounts and teams using resource policies

### Two registries
| Registry | What it contains |
|---|---|
| **AWS Events Registry** | Pre-built schemas for all AWS service events |
| **Custom Registry** | Schemas for your own application events |

> 💡 **DVA-C02 tip:** Schema Registry = faster development because you skip writing boilerplate deserialization code. EventBridge generates it for you.

---

## 5. Archive & Replay

### Archive
Store a **copy of all events** (or filtered events) that pass through an event bus — indefinitely or for a set retention period.

- Configure at the event bus level
- Can apply an event pattern filter (archive only specific events)
- Default retention: indefinite (or specify days)

### Replay
**Re-send archived events** back through the event bus as if they just happened.

> 💡 **Use cases:**
> - You deployed a bug fix — replay events from before the fix to reprocess them correctly
> - You added a new consumer — replay historical events so it catches up
> - Debugging — replay a specific time window to investigate an issue

> ⚠️ Replayed events go through the **same rules** as live events — so any matching rule will fire again, including alerting rules. Be careful!

---

## 6. API Destinations

API Destinations let EventBridge **send events to external HTTP endpoints** (third-party APIs, on-premises systems, or any REST API).

### How it works
1. Create a **Connection** — stores the auth credentials (API key, OAuth, Basic auth) in Secrets Manager
2. Create an **API Destination** — the HTTP endpoint URL + HTTP method + connection
3. Create a rule targeting the API Destination
4. EventBridge calls the external API when the rule fires, authenticating automatically

### Rate limiting
- Set a maximum invocation rate (requests/second) to avoid overwhelming the target API
- EventBridge queues excess events and retries

> 💡 **Common DVA-C02 scenario:** "Send EventBridge events to a third-party ticketing system like PagerDuty or ServiceNow" → answer is **API Destinations**.

---

## 7. EventBridge Pipes

Pipes are for **point-to-point** (one source → one target) integrations with built-in filtering, enrichment, and transformation — all without writing custom glue code.

> 💡 **Bus vs Pipe:**
> - **Event Bus** = many-to-many routing (many sources → many targets via rules)
> - **Pipe** = one-to-one with enrichment (one source → optional filter → optional Lambda enrichment → one target)

### Pipe stages (all optional except source and target)

```
Source → [Filter] → [Enrichment (Lambda/API Gateway/Step Functions)] → Target
```

| Stage | What it does |
|---|---|
| **Source** | Where events come from: SQS, Kinesis, DynamoDB Streams, Kafka, etc. |
| **Filter** | Drop events that don't match a pattern (runs before enrichment — saves cost) |
| **Enrichment** | Call a Lambda function, API Gateway, or Step Functions to add data to the event |
| **Target** | Where the final event goes: Lambda, SQS, EventBus, Step Functions, etc. |

### When to use Pipes vs Bus
| Scenario | Use |
|---|---|
| Route events from SQS to Lambda with custom enrichment | **Pipes** |
| Fan-out one event to 5 different services | **Bus + Rules** |
| One DynamoDB stream → one Lambda, with filtering | **Pipes** |
| React to EC2 state changes across the whole account | **Bus + Rules** |

---

## 8. EventBridge Scheduler

A standalone scheduling service — run tasks on a **fixed schedule or one-time** basis.

> 💡 EventBridge Scheduler is more powerful than scheduled rules on the default bus. It has its own dedicated service and supports more targets.

### Schedule types
| Type | Example | Use |
|---|---|---|
| **Rate** | `rate(5 minutes)` | Repeat every N minutes/hours/days |
| **Cron** | `cron(0 9 ? * MON-FRI *)` | Specific time/day pattern |
| **One-time** | Specific datetime | Run exactly once (e.g. a deferred task) |

### Key features
- Supports **flexible time windows** — run within a time window to spread load
- Built-in **retry with DLQ** — if target fails, retries automatically
- Supports **over 200 AWS service targets** directly (no Lambda needed as middleman)
- Cross-account and cross-region scheduling

---

## 9. Cross-Account & Cross-Region Routing

EventBridge can send events to event buses in **other AWS accounts or regions**.

### Cross-account
1. The receiving account adds a **resource-based policy** to its event bus, granting the sending account permission to `PutEvents`
2. The sending account creates a rule that targets the receiving account's event bus ARN

### Cross-region
- Create a rule on the source event bus and set the target as an event bus in a different region
- Same account, different region

> 💡 **Common architecture pattern:** Aggregate events from multiple accounts into one **central event bus** in a "logging" or "security" account for unified monitoring.

---

## 10. Dead Letter Queue (DLQ)

If EventBridge **fails to deliver an event to a target** after retries, it can send the failed event to an SQS queue (DLQ) so you don't lose it.

- Configured per rule target (not per event bus)
- EventBridge retries delivery for up to **24 hours** before sending to DLQ
- Retry policy is configurable: max attempts and max age of event

> 💡 **DVA-C02 tip:** If a question asks "what happens to events that EventBridge can't deliver?" — the answer is **DLQ (SQS)**.

---

## 11. EventBridge Sandbox

A **testing tool** in the AWS console that lets you:
- Paste a sample event JSON
- Test your event pattern against it to see if it matches — **before** creating the real rule
- Iterate on your pattern without deploying anything

> 💡 Use the Sandbox during development to validate your event patterns before going live. It saves debugging time.

---

## 12. IAM & Permissions

### Sending events to a bus (PutEvents)
- Your app's IAM user/role needs `events:PutEvents` permission on the target event bus

### EventBridge calling a target
- EventBridge needs an **IAM execution role** with permission to invoke the target
- Example: if the target is Lambda → role needs `lambda:InvokeFunction`
- Example: if the target is SNS → role needs `sns:Publish`
- Example: if the target is SQS → role needs `sqs:SendMessage`

### Cross-account access
- The receiving event bus needs a **resource-based policy** granting `events:PutEvents` to the sending account

---

## 13. Pricing

| Usage | Cost |
|---|---|
| Events published to default/custom bus | Per million events |
| Events published to partner bus | Per million events (slightly different rate) |
| Schema discovery (auto-detection) | Per million events analyzed |
| Replayed events | Per million events replayed |
| EventBridge Pipes | Per million events processed through a pipe |
| EventBridge Scheduler | Per scheduled invocation |

> ✅ **Free tier:** First 14 million custom/partner events per month are free.

---

## EventBridge vs SNS vs SQS — Don't Confuse Them

| | EventBridge | SNS | SQS |
|---|---|---|---|
| **Best for** | Event routing with rich filtering | Fan-out notifications | Decoupled queue processing |
| **Filtering** | Content-based (JSON pattern) | Basic attribute filters | No filtering |
| **Sources** | AWS services, SaaS, custom apps | Your app via SDK | Your app via SDK |
| **Targets** | 20+ AWS services + HTTP | Lambda, SQS, HTTP, email | One consumer at a time |
| **Replay** | ✅ Archive + Replay | ❌ | ❌ |
| **Schema Registry** | ✅ | ❌ | ❌ |
| **DVA-C02 trigger** | React to AWS service events, schedule tasks, route to multiple targets | Push notification to many subscribers | Buffer messages for a consumer |

---

## DVA-C02 Exam Quick-Reference

| Scenario | Answer |
|---|---|
| React when an EC2 instance stops | EventBridge rule with `EC2 Instance State-change` event pattern |
| Run a Lambda every 5 minutes | EventBridge scheduled rule — `rate(5 minutes)` |
| Send events to a Slack webhook or PagerDuty API | **API Destinations** |
| Replay past events after a bug fix | **Archive + Replay** |
| Add extra data to an event before it reaches the target | **Input Transformer** (or Pipes enrichment) |
| Route SQS events to Lambda with filtering, no code | **EventBridge Pipes** |
| Centralise events from multiple accounts | Cross-account event bus + resource policy |
| Events that fail delivery should not be lost | **Dead Letter Queue (SQS)** |
| Auto-detect and catalogue event structure | **Schema Registry with Schema Discovery** |
| Test event pattern without deploying | **EventBridge Sandbox** |
| Run a one-time scheduled task at a future time | **EventBridge Scheduler (one-time schedule)** |
| Decouple producer from consumer with rich filtering | **EventBridge custom event bus** |

---

*Sources: [TutorialsDojo EventBridge](https://tutorialsdojo.com/amazon-eventbridge/) · [arkalim Notion Notes](https://arkalim.notion.site/EventBridge-bf30bc5c66724bca9b2ebeffcd30621b) · [AWS EventBridge Docs](https://docs.aws.amazon.com/eventbridge/latest/userguide/)*
