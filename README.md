# TradeStream — High-Throughput Trade Data Processing Platform

Production-style reference implementation for high-frequency trade ingestion, Redis Streams fan-out, asynchronous processing, market analytics, durable storage, monitoring, and reconciliation.

## Architecture

```text
Clients
   |
   v
Java Spring Boot API
   |
   v
Redis Streams
   |------------|-------------|
   v            v             v
 Risk       Compliance     Analytics
   |            |             |
   +------------+-------------+
                |
                v
              SQS
                |
          +-----+------+
          |            |
         Lambda       DLQ
          |
          v
      DynamoDB
          |
          v
   Reconciliation
          |
          v
 PostgreSQL positions
```

## Stack
Java, Spring Boot, Python, PostgreSQL, Redis Streams, AWS Lambda, SQS, DynamoDB, Docker, CloudWatch, asyncio, pandas, NumPy.

## Local run

```bash
docker compose up --build
```

API: http://localhost:8080  
Analytics: http://localhost:8000/docs  
LocalStack: http://localhost:4566

## Example

```bash
curl -X POST http://localhost:8080/api/v1/trades   -H "Content-Type: application/json"   -d '{
    "eventId":"evt-001",
    "accountId":"ACC-100",
    "instrumentId":"AAPL",
    "eventType":"EXECUTION",
    "side":"BUY",
    "quantity":100,
    "price":190.50,
    "orderState":"ORDER_RECEIVED",
    "eventTime":1760000000000
  }'
```

## Correctness
- `eventId` is the idempotency key.
- Processing follows at-least-once semantics.
- DynamoDB order transitions use conditional writes.
- Invalid transitions are rejected.
- Failed SQS messages are sent to a DLQ.
- Reconciliation compares DynamoDB trade state against PostgreSQL positions.

## Monitoring
CloudWatch-ready metrics:
- events/sec
- consumer lag
- p99 processing latency
- processing failures
- DLQ depth

Consumer lag alarm threshold is configured at 500 unprocessed events.

Performance numbers should be benchmarked under a controlled workload before being claimed on a resume.
# TradeStream
