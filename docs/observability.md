````md
# Bacha Cafe Observability

## 1. Overview

Observability is the ability to understand what is happening inside the system by using:

- logs
- metrics
- traces
- dashboards
- alerts

For Bacha Cafe, observability is important because the system uses multiple services.  
When one request travels through several services, we need a reliable way to answer questions like:

- Which service failed?
- Why is the request slow?
- Did payment succeed but order update fail?
- Is inventory reservation causing errors?
- Are notifications delayed?
- Is one service using too much CPU or memory?

A good observability design helps with:

- debugging
- production monitoring
- incident response
- performance tuning
- capacity planning
- system reliability

---

## 2. Observability Goals

The main goals are:

- detect failures quickly
- identify the exact service causing a problem
- trace one user request across services
- measure latency and error rates
- monitor infrastructure health
- support root cause analysis
- make production issues easier to troubleshoot

---

## 3. Core Observability Pillars

### 3.1 Logs
Logs tell us what happened.

Examples:

- login succeeded
- order created
- payment failed
- inventory reservation timed out
- notification retry started

---

### 3.2 Metrics
Metrics tell us how the system is performing over time.

Examples:

- request count
- error count
- response time
- CPU usage
- memory usage
- queue size
- payment success rate

---

### 3.3 Traces
Traces show how one request moved across services.

Example flow:

- frontend sends order request
- API gateway forwards request
- order service validates order
- inventory service reserves stock
- payment service processes payment
- notification service sends confirmation

Tracing helps us see where time is spent and where failure occurred.

---

## 4. Target Services to Observe

All services should produce observability data:

- API Gateway
- Auth Service
- Catalog Service
- Order Service
- Inventory Service
- Payment Service
- Notification Service
- Databases
- Message broker
- Frontend later if needed

---

## 5. Logging Strategy

## 5.1 Logging Principles

Logs should be:

- structured
- searchable
- consistent across services
- useful for debugging
- safe from sensitive data leakage

Do not log:

- passwords
- raw payment card data
- JWT secrets
- personal sensitive information that is unnecessary

---

## 5.2 Log Format

Use structured JSON logs in backend services.

Example:

```json
{
  "timestamp": "2026-03-10T14:30:00Z",
  "level": "INFO",
  "service": "order-service",
  "traceId": "abc-123",
  "spanId": "xyz-456",
  "requestId": "req-789",
  "userId": 1,
  "message": "Order created successfully",
  "orderId": 5001
}
````

Important fields:

* timestamp
* log level
* service name
* trace ID
* span ID
* request ID
* user ID if available
* operation name
* important domain identifiers like orderId, paymentId, productId

---

## 5.3 Log Levels

### INFO

Normal business events.

Examples:

* user logged in
* product fetched
* order created
* payment completed
* notification sent

### WARN

Unexpected but recoverable situations.

Examples:

* invalid login attempt
* low stock detected
* retry triggered
* slow downstream response

### ERROR

Failures that need attention.

Examples:

* payment provider timeout
* database connection failure
* order update failed
* queue publish failed

### DEBUG

Detailed developer logs for local troubleshooting.
Should usually be reduced or disabled in production.

---

## 5.4 Where to Add Logs

Add logs at important points:

### API entry

When a request enters the service.

### Service logic

When major business action starts or completes.

### External calls

Before and after calling another service or external payment provider.

### Async consumers

When an event is received, processed, retried, or failed.

### Error handling

When exceptions happen.

---

## 5.5 Example Log Events by Service

### Auth Service

* user registration started
* user registration completed
* login success
* login failure
* token refresh
* unauthorized access attempt

### Catalog Service

* product search requested
* product created
* product updated
* cache hit / miss

### Order Service

* order validation started
* order created
* order status changed
* cancellation requested
* duplicate order request detected

### Inventory Service

* stock checked
* stock reserved
* stock deduction completed
* stock release completed
* low stock warning

### Payment Service

* payment initiated
* payment callback received
* callback signature verified
* payment success
* payment failed
* duplicate callback ignored

### Notification Service

* notification queued
* email sent
* retry scheduled
* permanent notification failure

---

## 6. Metrics Strategy

Metrics help answer system health questions continuously.

---

## 6.1 Key Metric Categories

### Application Metrics

* request count
* error count
* response time
* success rate
* business event count

### Infrastructure Metrics

* CPU usage
* memory usage
* disk usage
* container restarts
* database connections

### Messaging Metrics

* queue depth
* consumer lag
* retry count
* dead-letter count

### Business Metrics

* orders created
* completed orders
* cancelled orders
* payment success rate
* low inventory events

---

## 6.2 Golden Signals

A good production system should monitor the four golden signals:

### Latency

How long requests take.

Examples:

* product fetch latency
* order creation latency
* payment processing latency

### Traffic

How many requests are coming in.

Examples:

* login requests per minute
* orders per minute
* product search requests per minute

### Errors

How many requests fail.

Examples:

* 5xx errors
* payment failures
* inventory reservation failures

### Saturation

How close the system is to capacity.

Examples:

* CPU usage near 90%
* memory pressure
* queue backlog growing
* DB connection pool exhaustion

---

## 6.3 Service-Level Metrics

### Auth Service Metrics

* login request count
* login failure count
* token refresh count
* registration count
* unauthorized request count

### Catalog Service Metrics

* product list request count
* search latency
* cache hit rate
* cache miss rate
* admin update count

### Order Service Metrics

* order creation count
* order creation latency
* order cancellation count
* order status transition count
* failed order count

### Inventory Service Metrics

* stock check count
* stock reservation count
* stock reservation failure count
* stock release count
* low stock count

### Payment Service Metrics

* payment initiation count
* payment success count
* payment failure count
* payment callback latency
* duplicate callback count

### Notification Service Metrics

* notification queued count
* notification sent count
* notification failure count
* retry count
* queue delay time

---

## 6.4 Business KPIs

Useful business-level indicators:

* total daily orders
* average order value
* payment success percentage
* most ordered products
* cancellation rate
* out-of-stock rate
* notification delivery rate

These are not only for operations, but also for product insights.

---

## 7. Distributed Tracing

Tracing is very important in microservices.

Without tracing, one user problem may require checking logs in many services manually.

With tracing, we can follow a request from start to finish.

---

## 7.1 Trace Flow Example

A customer places an order:

1. frontend sends request
2. API gateway receives request
3. order service handles order creation
4. inventory service reserves stock
5. payment service processes payment
6. order service updates status
7. notification service sends confirmation

All these steps should share the same **trace ID**.

---

## 7.2 Required Trace Fields

Each request should carry:

* traceId
* spanId
* parentSpanId
* service name
* operation name
* duration
* status

---

## 7.3 Trace Propagation

Headers should be passed between services.

Examples:

```http
traceparent: 00-abcd1234efgh5678ijkl9012mnop3456-1234abcd5678efgh-01
X-Correlation-Id: req-12345
```

Even if simplified locally, the idea is:

* one request gets one trace ID
* child operations create their own spans
* downstream services continue the trace

---

## 7.4 Important Spans to Track

Track spans for:

* incoming HTTP requests
* database queries
* external HTTP calls
* message publish
* message consume
* payment provider interaction
* cache lookup
* order status update

---

## 8. Correlation IDs and Request IDs

Besides tracing, correlation IDs are helpful for log search.

### Why they matter

A single user issue may create many log lines.
Correlation IDs allow us to find all logs related to one request.

### Rule

Each inbound request should get:

* request ID
* trace ID
* correlation ID if available

These values should be:

* logged
* passed to downstream services
* included in async event metadata where possible

---

## 9. Dashboard Design

Dashboards give a quick view of system health.

---

## 9.1 System Overview Dashboard

This dashboard should show:

* total request rate
* error rate
* average response time
* CPU and memory by service
* active alerts
* container restart count
* queue depth

This is the main dashboard for quick health checks.

---

## 9.2 Order Flow Dashboard

Important widgets:

* orders created per minute
* order success rate
* order failure rate
* average order creation latency
* order status distribution
* cancellation rate

This dashboard focuses on the main customer flow.

---

## 9.3 Payment Dashboard

Important widgets:

* payment initiation count
* payment success vs failure
* payment latency percentile
* external provider error count
* callback processing count
* duplicate callback count

This dashboard is critical because payment problems directly affect revenue.

---

## 9.4 Inventory Dashboard

Important widgets:

* stock reservation requests
* stock reservation failures
* low stock products
* out-of-stock products
* inventory update count

This helps prevent overselling and availability problems.

---

## 9.5 Notification Dashboard

Important widgets:

* notifications queued
* sent count
* failed count
* retry count
* average delivery delay

This ensures customers receive important updates.

---

## 10. Alerting Strategy

Alerts should notify the team about real issues, not create noise.

---

## 10.1 Alert Principles

Good alerts should be:

* actionable
* meaningful
* limited to important failures
* based on symptoms and impact

Avoid alerting on every small warning.

---

## 10.2 Critical Alerts

### High Error Rate

Trigger when:

* 5xx error rate crosses threshold

Example:

* order-service error rate > 5% for 5 minutes

### High Latency

Trigger when:

* p95 response time exceeds threshold

Example:

* payment-service p95 latency > 2 seconds for 10 minutes

### Service Down

Trigger when:

* health check fails repeatedly
* container is not running

### Queue Backlog Growing

Trigger when:

* notification queue or event queue grows beyond safe threshold

### Payment Failure Spike

Trigger when:

* payment failures suddenly rise above normal

### Database Connection Problems

Trigger when:

* DB connection pool is exhausted
* query latency spikes heavily

### Low Inventory Alert

Trigger when:

* product stock drops below threshold

---

## 10.3 Warning Alerts

Lower-severity alerts may include:

* cache miss rate unusually high
* retry count increasing
* slow queries detected
* increasing memory usage
* disk space getting low

---

## 11. Health Checks

Each service should expose health endpoints.

Example:

```text
/actuator/health
/actuator/info
/actuator/metrics
```

Health checks should cover:

* service availability
* database connectivity
* message broker connectivity if required
* external dependency status where appropriate

### Types of health checks

#### Liveness

Is the service process running?

#### Readiness

Is the service ready to accept traffic?

Example:

* app is up but DB is unavailable → not ready

---

## 12. Recommended Tools

For a strong real-world setup, use tools like these:

### Logging

* SLF4J + Logback
* ELK Stack
* Loki

### Metrics

* Micrometer
* Prometheus
* Grafana

### Tracing

* OpenTelemetry
* Jaeger
* Tempo

### Health and application stats

* Spring Boot Actuator

### Container/infrastructure monitoring

* Docker metrics
* Kubernetes metrics later if deployed there

---

## 13. Suggested Spring Boot Implementation Direction

For Java Spring Boot services:

### Logging

* use SLF4J
* add structured logging with JSON format
* include traceId and requestId in MDC

### Metrics

* enable Actuator
* export metrics with Micrometer
* scrape through Prometheus

### Tracing

* use OpenTelemetry instrumentation
* propagate trace context across services

### Example Actuator endpoints

* `/actuator/health`
* `/actuator/prometheus`
* `/actuator/metrics`

---

## 14. Message Broker Observability

Since Bacha Cafe uses async communication, the broker also needs monitoring.

Track:

* queue depth
* publish failures
* consumer failure count
* message retry count
* dead-letter queue count
* oldest message age

Important question this answers:

* Are notifications delayed because consumers are slow?
* Is inventory processing stuck?
* Are events being retried too often?

---

## 15. Database Observability

Databases should also be monitored.

Important DB metrics:

* query latency
* active connections
* connection pool usage
* slow queries
* deadlocks
* CPU and memory usage
* storage growth

Important DB logs:

* failed queries
* lock timeouts
* schema migration failures

---

## 16. Example Incident Scenarios

### Scenario 1: Customer says payment succeeded but order still shows pending

Use:

* trace from payment callback to order update
* payment-service logs
* order-service logs
* event publishing logs
* dashboard for payment success vs order status failures

### Scenario 2: Orders are slow

Check:

* order-service latency
* DB query time
* inventory-service response time
* payment-service latency
* traces showing the slow span

### Scenario 3: Customers are not receiving confirmation emails

Check:

* notification queue depth
* notification consumer logs
* retry count
* dead-letter queue
* notification delivery dashboard

---

## 17. Minimum Local Development Observability

Even in local development, include a simple version of observability.

Recommended minimum:

* structured console logs
* trace/correlation ID in logs
* Spring Boot Actuator
* Prometheus metrics endpoint
* basic Grafana dashboard later
* Docker Compose support for monitoring stack if possible

This helps developers build good habits early.

---

## 18. Production Maturity Roadmap

### Phase 1

* basic logs
* Actuator health endpoint
* request metrics
* simple dashboards

### Phase 2

* structured JSON logs
* centralized logging
* Prometheus + Grafana
* service alerts

### Phase 3

* distributed tracing
* queue monitoring
* DB monitoring
* golden signal dashboards

### Phase 4

* business KPI dashboards
* SLOs / SLIs
* advanced alert tuning
* incident review process

---

## 19. Key Design Decisions

### Decision 1: Use structured logs

Reason:

* easier searching and filtering

### Decision 2: Use centralized metrics

Reason:

* compare service health in one place

### Decision 3: Trace requests across services

Reason:

* microservices are hard to debug without tracing

### Decision 4: Monitor both technical and business metrics

Reason:

* a technically healthy system can still fail business goals

### Decision 5: Alerts should be symptom-based

Reason:

* reduce alert fatigue

---

## 20. Summary

The Bacha Cafe observability design gives the system a strong production-style foundation.

It includes:

* structured logging
* application and infrastructure metrics
* distributed tracing
* dashboards for service and business flows
* alerting for critical failures
* health checks for service readiness

With this setup, the team can:

* detect issues faster
* troubleshoot more easily
* understand cross-service request flow
* improve reliability over time

This makes Bacha Cafe look much more like a real professional microservices platform.

```

Next should be `deployment.md`, because now you already have:

- architecture
- APIs
- domain model
- observability

and deployment will complete the core technical design set.
```
