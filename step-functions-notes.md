# 🔄 AWS Step Functions — DVA-C02 Study Notes

---

## What is Step Functions?

A fully managed AWS service to **orchestrate multiple AWS services** into a visual workflow called a **State Machine**. Think of it as a flowchart that runs your cloud services in the right order — handling retries, branching, and error handling automatically.

- Defined using **Amazon States Language (ASL)** — a JSON-based language
- Provides a **visual diagram** of the workflow in the console
- Built-in support for **CloudWatch metrics** and **X-Ray tracing**

---

## State Types

These are the building blocks of a state machine:

| State | What it does |
|---|---|
| **Task** | Does actual work — calls Lambda, ECS, DynamoDB, SQS, Batch, etc. |
| **Choice** | Conditional branching (like an if/else) |
| **Wait** | Pauses for a set time or until a specific date/time |
| **Pass** | Passes input to output; can inject fixed/default data |
| **Parallel** | Runs multiple branches simultaneously |
| **Map** | Loops over an array and runs steps for each item |
| **Succeed** | Terminates execution as a success |
| **Fail** | Terminates execution as a failure |

---

## Standard vs Express Workflows

| Feature | Standard | Express |
|---|---|---|
| **Max duration** | 1 year | 5 minutes |
| **Execution model** | Exactly-once ✅ | At-least-once ⚠️ |
| **Execution rate** | 2,000/sec | 100,000/sec |
| **Pricing** | Per state transition | Per execution + duration + memory |
| **Audit history** | Yes (90 days via API) | No (use CloudWatch Logs) |
| **Use case** | Long-running, critical workflows | High-volume, short-lived tasks |

> ⚠️ **Workflow type CANNOT be changed after creation!**

### Express Sub-Types

- **Synchronous Express** — waits for the result before returning; ideal for **API Gateway / Lambda** integrations requiring a real-time response
- **Asynchronous Express** — fire-and-forget; result is not waited on

---

## Integration Modes (for Task States)

| Mode | How it works |
|---|---|
| **Request / Response** | Calls the service and moves on — does NOT wait for result |
| **Sync (`.sync`)** | Waits for the job to complete (e.g., ECS task, Glue job) |
| **Wait for Task Token** | Pauses execution until an external system sends back the token — used for **human approval flows** |

---

## Error Handling

By default, if **any state fails → the entire execution fails**.

### Two Ways to Handle Errors

**1. Retry** — automatic retries with exponential backoff:

| Parameter | Default | Description |
|---|---|---|
| `IntervalSeconds` | 1s | Wait time before the first retry |
| `MaxAttempts` | 3 | Maximum number of retry attempts |
| `BackoffRate` | 2× | Multiplier applied between retries |

**2. Catch** — if all retries fail, redirect to a fallback state:

- `ErrorEquals` — which error type(s) to catch
- `Next` — which state to transition to next

### Common Error Types

- **State machine definition issues** — e.g., no matching rule in a Choice state
- **Task failures** — e.g., Lambda throws an exception
- **Transient failures** — e.g., network timeout or partition

---

## Key Service Integrations

Step Functions can natively integrate with:

- AWS Lambda
- Amazon ECS / Fargate
- Amazon DynamoDB
- Amazon SQS / SNS
- Amazon API Gateway
- AWS Batch
- AWS Glue
- On-premises systems (via **Activities**)

---

## Exam Tips 🎯

- **Standard** = long-running + exactly-once + auditable → think **order fulfillment, financial transactions**
- **Express** = short, high-volume, idempotent → think **IoT ingestion, streaming, mobile backends**
- **Synchronous Express** = used with API Gateway for real-time responses
- **Retry + Catch** = the two error handling mechanisms; know the retry parameters cold
- **Wait for Task Token** = the pattern for human approval or async external systems
- **At-least-once** (Express) means your actions should be **idempotent** (safe to run more than once)
- Step Functions has **built-in CloudWatch + X-Ray** support — no extra setup needed
- Workflow type **cannot be changed** after the state machine is created
