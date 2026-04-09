# High-Concurrency Batch Processing System (Hangfire + .NET + Oracle)

A production-grade backend system designed to process large Excel uploads at scale using **asynchronous job queues, parallel execution, and controlled concurrency**.

This system eliminates sequential bottlenecks and enables **high-throughput, fault-tolerant batch processing** for data-intensive workflows such as AML verification.

---

## Problem

Traditional Excel upload systems fail at scale due to:

* Sequential processing → extremely slow execution
* Blocking HTTP requests → poor user experience
* No retry mechanism → data loss on failure
* Lack of concurrency → underutilized resources

---

## Solution

This system introduces a **queue-driven, parallel processing architecture** using Hangfire:

```
User Upload → API → Queue (Hangfire) → Worker Pool → Parallel Processing → Results Export
```

---

## Key Features

### Asynchronous Job Processing

* File uploads are **queued using Hangfire**
* Non-blocking architecture ensures fast API response
* Background workers handle heavy processing

---

### Parallel Execution Engine

* Uses `Parallel.ForEach` with controlled concurrency
* Configured with `MaxDegreeOfParallelism = 4` 
* Enables simultaneous processing of multiple records

---

### High-Performance Data Processing

* Processes large datasets from staging tables efficiently
* Batch execution reduces latency significantly
* Designed for **high-throughput AML search workflows**

---

### Fault Tolerance & Retries

* Automatic retries with Hangfire
* `[AutomaticRetry(Attempts = 3)]` ensures resilience 
* Failures are logged and retried safely

---

### Rate Limiting (System Protection)

* Prevents system overload during uploads
* Fixed window limiter:

  * 10 uploads per 60 seconds
* Ensures **controlled ingestion under load**

---

### Structured Logging

* Integrated with `log4net`
* Logs:

  * Job lifecycle
  * Parallel execution progress
  * Stored procedure calls
  * Failures and retries

---

### Bulk Data Ingestion

* Uses `OracleBulkCopy` for high-speed inserts 
* Efficient staging of uploaded data before processing

---

### Excel Report Generation

* Generates result reports using `ClosedXML`
* Automatically stores and updates processed file metadata

---

## Architecture Overview

```
[Client]
   ↓
[ASP.NET API]
   ↓
[Hangfire Queue]
   ↓
[Worker Pool (4 Workers)]
   ↓
[Parallel Processing Engine]
   ↓
[Oracle DB (Stored Procedures)]
   ↓
[Excel Report Generation]
```

---

## Tech Stack

* **Backend:** ASP.NET Core
* **Queue:** Hangfire (LiteDB storage)
* **Database:** Oracle
* **Concurrency:** Parallel.ForEach
* **Logging:** log4net
* **Excel Processing:** ClosedXML
* **Rate Limiting:** ASP.NET RateLimiter

---

## Performance Impact

* Converted **sequential processing → parallel execution**
* Reduced processing time by **80–90%**
* Saved **400–500 operational hours annually**
* Enabled scalable handling of large batch uploads

---

## Reliability & Safety

* Retry mechanism for failed jobs
* Controlled concurrency to prevent resource exhaustion
* Rate limiting to protect system under load
* Structured logging for traceability

---

## Core Workflow

### 1. Upload

* User uploads Excel file
* File is saved and parsed into a DataTable

### 2. Staging

* Data is bulk inserted into Oracle staging tables

### 3. Queueing

* Job is enqueued using Hangfire

### 4. Processing

* Worker picks job
* Executes parallel search across records
* Merges results safely (thread-safe operations)

### 5. Output

* Generates Excel report
* Updates database with processing status

---

## Concurrency Design

* Controlled worker count (`WorkerCount = 4`)
* Parallel execution with bounded threads
* Thread-safe result aggregation using locks
* Atomic counters for tracking progress

---

## Future Improvements

* Distributed queue (Redis / SQL Server)
* Dynamic worker scaling
* Circuit breaker for DB calls
* Metrics integration (Prometheus + Grafana)

---

## Summary

This system demonstrates:

* Real-world **concurrency handling**
* Production-grade **queue-based architecture**
* High-performance **data processing at scale**
* Strong focus on **reliability and fault tolerance**

---

## Author

**Abhilasha Deshmukh**
Backend Engineer 
