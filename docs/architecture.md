````md
# Bacha Cafe Architecture

## 1. Overview

Bacha Cafe is designed using a **microservices architecture** to support scalability, maintainability, independent deployment, and clear separation of business responsibilities.

The system is split into:

- **Frontend** for customer interaction
- **Backend microservices** for business capabilities
- **Databases** owned by each service
- **Asynchronous messaging** for event-driven communication
- **API Gateway / edge routing** for controlled external access
- **Observability stack** for logs, metrics, and tracing
- **Containerized deployment** using Docker

This architecture is suitable for a growing online cafe platform where users can browse products, place orders, make payments, and receive notifications.

---

## 2. Architecture Goals

The main goals of this architecture are:

- Keep services **loosely coupled**
- Allow each service to be **developed and deployed independently**
- Improve **fault isolation**
- Make scaling easier for high-traffic areas like catalog and ordering
- Support future growth such as delivery tracking, loyalty points, promotions, and analytics
- Enable both **synchronous** and **asynchronous** communication patterns

---

## 3. High-Level System Components

### Frontend
The frontend is a web application used by customers to:

- sign up and log in
- browse coffee and food items
- search products
- manage cart
- place orders
- make payments
- view order history and order status

Recommended stack:

- React
- TypeScript
- Axios
- React Router
- Context API or Redux Toolkit

---

### Backend Services

The backend is divided into multiple services:

- **Auth Service**
- **Catalog Service**
- **Order Service**
- **Inventory Service**
- **Payment Service**
- **Notification Service**

Each service owns its own business logic and data.

---

## 4. High-Level Architecture Diagram

```text
                        +----------------------+
                        |      Frontend        |
                        |   React Web App      |
                        +----------+-----------+
                                   |
                                   |
                            HTTP / HTTPS
                                   |
                                   v
                        +----------------------+
                        |     API Gateway      |
                        |  Routing / Security  |
                        +---+----+----+----+---+
                            |    |    |    |
        --------------------+    |    |    +---------------------
        |                        |    |                          |
        v                        v    v                          v
+---------------+      +---------------+      +---------------+      +------------------+
| Auth Service  |      | Catalog Svc   |      | Order Service |      | Payment Service  |
+-------+-------+      +-------+-------+      +-------+-------+      +---------+--------+
        |                      |                      |                          |
        v                      v                      v                          v
+---------------+      +---------------+      +---------------+      +------------------+
| Auth DB       |      | Catalog DB    |      | Order DB      |      | Payment DB       |
+---------------+      +---------------+      +---------------+      +------------------+

                                   |
                                   |
                                   v
                         +----------------------+
                         | Inventory Service    |
                         +----------+-----------+
                                    |
                                    v
                             +--------------+
                             | Inventory DB |
                             +--------------+

                                   |
                                   |
                            Event / Message Bus
                                   |
               ------------------------------------------------
               |                                              |
               v                                              v
     +----------------------+                      +------------------------+
     | Notification Service |                      | Future Services        |
     | email / sms / push   |                      | loyalty / analytics    |
     +----------+-----------+                      +------------------------+
                |
                v
      +----------------------+
      | Notification DB      |
      +----------------------+
````

---

## 5. Microservices and Responsibilities

### 5.1 Auth Service

**Purpose:**
Handles authentication and authorization.

**Responsibilities:**

* user registration
* login
* password hashing
* JWT token generation
* token validation
* role management
* refresh token handling

**Owns data such as:**

* users
* roles
* credentials
* refresh tokens

**Notes:**

* This service should be the source of truth for identity
* Other services should trust user identity from validated tokens instead of storing credentials themselves

---

### 5.2 Catalog Service

**Purpose:**
Manages cafe menu items and product browsing.

**Responsibilities:**

* list products
* search/filter products
* group items by category
* manage product details
* expose pricing and availability metadata
* support admin product management later

**Owns data such as:**

* products
* categories
* prices
* product images
* descriptions

**Notes:**

* Frequently read-heavy service
* Good candidate for caching

---

### 5.3 Order Service

**Purpose:**
Handles customer orders from creation through status tracking.

**Responsibilities:**

* create order
* validate order items
* calculate totals
* maintain order lifecycle
* track order status
* store order history
* coordinate with payment and inventory

**Typical statuses:**

* PENDING
* CONFIRMED
* PAID
* PREPARING
* READY
* COMPLETED
* CANCELLED

**Owns data such as:**

* orders
* order items
* order totals
* order statuses
* timestamps

**Notes:**

* Central business workflow service
* Must be designed carefully for consistency

---

### 5.4 Inventory Service

**Purpose:**
Tracks available stock for products and ingredients.

**Responsibilities:**

* maintain stock levels
* reserve stock during order flow
* reduce stock after successful payment
* restore stock on cancellation/failure
* alert low-stock conditions

**Owns data such as:**

* stock quantities
* inventory movements
* reservation records
* threshold settings

**Notes:**

* Important for preventing overselling
* Could later evolve into ingredient-level inventory instead of only product-level stock

---

### 5.5 Payment Service

**Purpose:**
Handles payment processing and payment state tracking.

**Responsibilities:**

* initiate payment
* integrate with payment provider
* record payment results
* handle success/failure callbacks
* maintain transaction history
* support refunds later

**Owns data such as:**

* payments
* transaction references
* payment status
* payment timestamps
* provider response metadata

**Notes:**

* Never store raw sensitive card data unless fully PCI-compliant
* Prefer tokenized payments through external provider

---

### 5.6 Notification Service

**Purpose:**
Sends communication to customers.

**Responsibilities:**

* order confirmation email
* payment success/failure notification
* order ready notification
* password reset email
* future SMS/push notifications

**Owns data such as:**

* notification requests
* delivery status
* retry attempts
* templates
* audit logs

**Notes:**

* Best triggered asynchronously through events
* Should not block main order flow

---

## 6. Communication Patterns

The system uses two communication styles:

### 6.1 Synchronous Communication

Used when an immediate response is required.

Examples:

* frontend → auth service for login
* frontend → catalog service for browsing menu
* order service → inventory service for stock check
* order service → payment service to initiate payment

Recommended approach:

* REST APIs over HTTP/HTTPS
* JSON payloads
* timeout and retry policies
* circuit breaker in production-grade setup

---

### 6.2 Asynchronous Communication

Used when work can happen in the background.

Examples:

* order placed event
* payment completed event
* order status changed event
* notification requested event
* inventory updated event

Recommended approach:

* message broker such as RabbitMQ or Kafka

Benefits:

* reduces tight coupling
* improves resilience
* supports retries
* enables future services like analytics

---

## 7. Database Ownership Pattern

Each microservice owns its own database.

### Why this matters

This avoids:

* tight coupling at data layer
* accidental cross-service schema dependencies
* unsafe direct reads/writes between services

### Example

* Auth Service → Auth DB
* Catalog Service → Catalog DB
* Order Service → Order DB
* Inventory Service → Inventory DB
* Payment Service → Payment DB
* Notification Service → Notification DB

### Rule

A service should never directly modify another service’s database.

If data is needed from another service, it should be accessed through:

* API calls
* events
* replicated read models later if needed

---

## 8. Request Flow Example

### Example: Customer places an order

#### Step 1: User logs in

* Frontend sends credentials to Auth Service
* Auth Service validates user
* Auth Service returns JWT token

#### Step 2: User browses products

* Frontend calls Catalog Service
* Catalog Service returns menu items

#### Step 3: User creates cart and checks out

* Frontend sends order request to Order Service with JWT token
* Order Service validates items and calculates total

#### Step 4: Order Service checks inventory

* Order Service calls Inventory Service
* Inventory Service confirms stock availability or reserves stock

#### Step 5: Payment is processed

* Order Service calls Payment Service
* Payment Service communicates with external payment provider
* Payment Service updates payment result

#### Step 6: Order status is updated

* If payment succeeds:

    * Order becomes `PAID` or `CONFIRMED`
    * Inventory is reduced
    * event is published
* If payment fails:

    * Order becomes `CANCELLED` or `PAYMENT_FAILED`
    * reserved stock is released

#### Step 7: Notification is sent

* Notification Service consumes event
* Customer receives confirmation email or message

---

## 9. Security Architecture

Security should be built into the system from the beginning.

### Main security measures

* HTTPS everywhere
* JWT-based authentication
* password hashing using BCrypt or Argon2
* role-based authorization
* input validation on every service
* API rate limiting at gateway level
* CORS configuration for frontend
* secrets stored securely using environment variables or secret manager
* audit logging for sensitive actions

### Roles

Possible roles:

* CUSTOMER
* ADMIN
* STAFF

Example:

* CUSTOMER can browse and place orders
* ADMIN can manage products
* STAFF can update order preparation status

---

## 10. Scalability Considerations

The system is designed so services can scale independently.

### Services likely to scale first

* **Catalog Service** → high read traffic
* **Order Service** → business-critical traffic
* **Notification Service** → burst traffic after events

### Scaling strategies

* horizontal scaling with containers
* load balancing
* cache frequently accessed catalog data
* offload slow background work to message queue
* use read replicas later if necessary

---

## 11. Reliability and Fault Tolerance

To make the system resilient:

* use retries for transient failures
* configure request timeouts
* add circuit breaker for unstable downstream dependencies
* handle idempotency for payment/order operations
* use dead-letter queues for failed async messages
* persist important events and audit logs
* degrade gracefully when non-critical services fail

### Example

If Notification Service is down:

* order should still complete
* notification event should remain in queue and be retried later

---

## 12. Observability

The platform should be observable from early stages.

### Logging

Use structured logs with:

* request ID
* user ID when available
* service name
* error code
* timestamp

### Metrics

Track:

* request count
* error rate
* response time
* payment failure rate
* stock reservation failures
* queue lag
* order throughput

### Tracing

Distributed tracing helps follow requests across:

* frontend
* gateway
* order service
* inventory service
* payment service
* notification service

Recommended tools:

* Prometheus
* Grafana
* ELK stack
* OpenTelemetry
* Jaeger

---

## 13. Deployment Architecture

The project should be containerized for consistency.

### Local development

Use:

* Docker Compose
* one container per service
* one container per database
* optional local message broker
* shared network

### Production direction

Can later run on:

* Kubernetes
* AWS ECS
* AWS EKS
* Azure Kubernetes Service
* GCP Cloud Run / GKE

---

## 14. Suggested Technology Stack

### Frontend

* React
* TypeScript
* Axios
* Tailwind CSS or Material UI

### Backend

* Java + Spring Boot for core services
  or
* Node.js + Express/NestJS for selected lightweight services

### Databases

* PostgreSQL for transactional services
* Redis for caching
* MongoDB only if a document use case appears later

### Messaging

* RabbitMQ for simpler event-driven workflow
  or
* Kafka for larger-scale streaming/event platform

### DevOps

* Docker
* Docker Compose
* GitHub Actions or Jenkins
* Nginx or API Gateway
* Prometheus + Grafana

---

## 15. Future Evolution

This architecture supports future expansion such as:

* cart service
* delivery service
* loyalty/rewards service
* review/rating service
* recommendation engine
* analytics/reporting service
* admin dashboard
* coupon/promotion service

The current design keeps those future additions possible without major redesign.

---

## 16. Key Architectural Decisions

### Decision 1: Microservices instead of monolith

**Reason:** better separation of concerns and easier scaling.

### Decision 2: Database per service

**Reason:** stronger boundaries and independent evolution.

### Decision 3: REST for direct calls

**Reason:** simple, widely used, easy for frontend integration.

### Decision 4: Event-driven communication for notifications and future workflows

**Reason:** loose coupling and better resilience.

### Decision 5: JWT-based authentication

**Reason:** stateless and works well for distributed services.

### Decision 6: Dockerized services

**Reason:** consistent local and deployment environments.

---

## 17. Summary

Bacha Cafe uses a modern microservices architecture where each service has a focused responsibility, independent data ownership, and clear communication patterns.

This design gives the project:

* clean separation of concerns
* easier team collaboration
* better scalability
* better fault isolation
* production-ready growth path

It is a strong foundation for building a real-world online cafe ordering platform.


