# High-Concurrency Batch Processing System

Hangfire + ASP.NET Core + Oracle

A backend system designed to process large Excel uploads using asynchronous job queues, controlled concurrency, and parallel execution. The system replaces sequential processing with a scalable architecture capable of handling high-volume data workloads reliably.

---

## Overview

This system processes uploaded files in a non-blocking manner by delegating heavy workloads to background jobs. It ensures efficient handling of large datasets while maintaining system stability under concurrent load.

Flow:

Client → API → Hangfire Queue → Worker Pool → Parallel Processing → Database → Report Generation

---

## Key Features

### Asynchronous Job Processing

* Upload requests are enqueued using Hangfire
* API remains non-blocking and responsive
* Background workers handle execution

### Parallel Processing

* Uses `Parallel.ForEach` with controlled concurrency
* Configured `MaxDegreeOfParallelism = 4`
* Enables concurrent processing of records within a batch

### Controlled Worker Concurrency

* Hangfire worker count configured to 4
* Multiple jobs processed in parallel
* Prevents uncontrolled thread explosion

### Bulk Data Ingestion

* Uses `OracleBulkCopy` for high-speed inserts
* Efficient staging of large datasets

### Retry Mechanism

* Automatic retries configured using Hangfire
* Failed jobs retried up to 3 times
* Improves resilience against transient failures

### Rate Limiting

* Fixed window limiter applied at API level
* Restricts uploads to prevent system overload
* Ensures stable ingestion under high traffic

### Logging and Observability

* Integrated with log4net
* Logs job lifecycle, execution steps, and failures
* Enables traceability and debugging

### Report Generation

* Generates Excel reports using ClosedXML
* Stores results and updates processing status

---

## Architecture

[Client]
↓
[ASP.NET Core API]
↓
[Hangfire Queue]
↓
[Worker Pool (4 Workers)]
↓
[Parallel Processing Engine]
↓
[Oracle Database (Stored Procedures)]
↓
[Excel Report Generation]

---

## Tech Stack

Backend: ASP.NET Core
Queue: Hangfire
Storage (Demo): LiteDB
Database: Oracle
Concurrency: Parallel.ForEach
Logging: log4net
Excel Processing: ClosedXML
Rate Limiting: ASP.NET RateLimiter

---

## Performance Impact

* Converted sequential workflow to concurrent processing
* Reduced processing time by 80–90%
* Saved approximately 400–500 operational hours annually
* Enabled scalable handling of large batch uploads

---

## Concurrency Design

* Worker-level concurrency via Hangfire (WorkerCount = 4)
* Record-level concurrency using Parallel.ForEach
* Thread-safe aggregation using locks
* Atomic counters for tracking progress

---

## Failure Handling Behavior

### Scenario: Job Queued

* Jobs are persisted in storage
* If application restarts, jobs remain intact
* Workers resume processing after restart

### Scenario: Job In Progress During Crash (IIS Reset / App Pool Recycle)

* Running job is interrupted
* Hangfire marks job as failed or incomplete
* Job is retried based on retry policy

---

## Limitations

### 1. Non-Idempotent Processing Risk

If a job fails mid-execution and retries:

* Records may be reprocessed
* Duplicate writes or inconsistent state may occur

### 2. Partial Processing

* Job may complete partially before failure
* No built-in checkpointing for progress recovery

### 3. Application Lifecycle Dependency

* If hosted within IIS, app pool recycle stops workers
* Background processing depends on application uptime

### 4. Single-Instance Storage (Demo Setup)

* LiteDB is not suitable for distributed or scaled environments
* Limits horizontal scalability

### 5. Fixed Concurrency

* Worker count and parallelism are statically configured
* No dynamic scaling based on system load

---

## Solutions and Improvements

### Idempotency Enforcement

* Use file-level or record-level identifiers
* Skip already processed records
* Ensure safe re-execution of jobs

### Checkpointing

* Process data in smaller chunks
* Persist progress in database
* Resume from last successful checkpoint

### Job State Tracking

* Maintain job status (STARTED / PROCESSING / COMPLETED)
* Prevent duplicate execution of completed jobs

### External Worker Service

* Move Hangfire server to a dedicated worker service
* Avoid dependency on IIS lifecycle

### Production Storage Upgrade

* Replace LiteDB with SQL Server or Redis
* Enable distributed job processing

### Dynamic Scaling

* Adjust worker count based on load
* Introduce queue prioritization if needed

### Timeout and Circuit Handling

* Add timeouts for database calls
* Implement retry with backoff strategies

---

## Note

This repository is a simplified and self-contained representation of a production system built to handle large-scale batch processing and concurrency challenges.

The core design, concurrency model, and failure-handling strategies reflect real-world implementation, while infrastructure dependencies have been minimized for clarity and demonstration purposes.

## Summary

This system demonstrates:

* Practical concurrency handling in real-world scenarios
* Queue-based asynchronous processing
* Parallel execution for high-performance workloads
* Awareness of failure modes and recovery strategies

It is designed not just to process data, but to do so reliably under load and failure conditions.

---

## Author

Abhilasha Deshmukh
Backend Engineer — Distributed Systems and Concurrency
