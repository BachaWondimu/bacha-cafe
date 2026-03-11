
# Bacha Café Platform

Bacha Café Platform is a microservices-based café ordering and management system designed for modern coffee shops.

The platform enables customers to browse menu items, customize beverages and food, place online pickup orders, and track order progress. It also supports internal workflows for employees and managers, including order preparation, inventory tracking, payment processing, and operational notifications.

The system is built using a modular microservices architecture powered by Spring Boot services, a React frontend, and a relational database. The architecture emphasizes scalability, service independence, secure authentication, and operational visibility through logging, metrics, and distributed tracing.

The platform is designed to support future expansion including cloud deployment, monitoring infrastructure, and container orchestration.

---

# Main Scenario

The first version focuses on **online ordering for pickup**.

Typical order flow:

1. Customer registers or logs in
2. Customer browses menu items (coffee, tea, smoothies, and food)
3. Customer customizes beverage or food options
4. Customer adds items to a cart
5. Customer places an order
6. Inventory availability is validated
7. Payment is processed
8. Staff receives and prepares the order
9. Customer receives a notification when the order is ready
10. Customer picks up the order at the shop

---

# Product Goals

The platform is designed to support the operational needs of modern café environments while maintaining a flexible service architecture.

Primary objectives include:
- Provide a seamless digital ordering experience for café customers
- Enable efficient order management workflows for staff
- Support structured product catalogs including beverages and food categories
- Maintain inventory awareness to prevent unavailable items from being ordered
- Implement secure authentication and role-based access control
- Deliver a modular microservices architecture where each service owns a clearly defined business capability
- Ensure extensibility so additional services or integrations can be introduced without major architectural changes
- Provide operational visibility through logging, metrics, and distributed tracing

---

# Microservices Overview

The platform is composed of several independent services, each responsible for a specific domain capability.

## Auth Service

Responsible for authentication and identity management.

Responsibilities:

* user registration
* login and authentication
* password hashing
* JWT token generation
* role-based authorization

---

## Catalog Service

Manages the product catalog and menu structure.

Responsibilities:

* category management
* product management
* beverage and food configuration
* customization options
* product availability

Example category structure:

```
Beverages
  Hot
    Coffee
    Tea
  Cold
    Smoothies
    Iced Drinks

Food
  Breakfast
  Snacks
  Lunch
```

---

## Order Service

Manages the full ordering lifecycle.

Responsibilities:

* cart management
* order creation
* order history
* order status updates
* customer order tracking

Example order states:

```
CREATED
CONFIRMED
PREPARING
READY
COMPLETED
CANCELLED
```

---

## Inventory Service

Tracks product availability and stock levels.

Responsibilities:

* stock validation
* stock deduction after confirmed orders
* restocking operations
* availability management

---

## Payment Service

Handles order payment processing.

Responsibilities:

* create payment records
* process payment attempts
* confirm or reject transactions

The initial implementation uses a simplified payment flow that can later integrate with external payment providers.

---

## Notification Service

Handles operational and customer notifications.

Responsibilities:

* order confirmation notifications
* order ready notifications
* operational alerts

Notifications may be delivered through:

* in-app notifications
* email
* push notifications (future extension)

---

# Architecture

Bacha Café Platform follows a **microservices architecture** where each service owns its domain logic and data.

System components include:

Frontend
React + TypeScript

Backend
Spring Boot microservices

Database
MySQL

Security
Spring Security + JWT

Service Communication
REST APIs

Local Development
Docker Compose

Deployment
Render

Future Infrastructure
AWS Cloud

Observability
Structured logging, metrics, and distributed tracing

Container Orchestration
Kubernetes (future extension)

---

# Technology Stack

## Frontend

* React
* TypeScript
* Vite
* React Router
* Axios
* Tailwind CSS

---

## Backend

* Java 21
* Spring Boot
* Spring Web
* Spring Data JPA
* Spring Security
* JWT Authentication
* Hibernate
* Maven

---

## Database

* MySQL
* Flyway for database migrations

---

## Testing

* JUnit 5
* Mockito
* Spring Boot Test

---

# Logging, Metrics, and Tracing

Operational visibility is an important component of the platform.

## Logging

Logging is implemented using:

* SLF4J
* Logback

Logs capture key events including:

* application startup
* authentication attempts
* order creation
* payment failures
* inventory updates
* system errors

Structured logging allows integration with centralized logging systems.

---

## Metrics

Metrics are exposed using:

* Spring Boot Actuator
* Micrometer

Metrics include:

* application health
* HTTP request metrics
* response latency
* JVM memory usage
* CPU utilization

Custom business metrics may include:

* order creation rates
* payment failures
* inventory shortages

---

## Distributed Tracing

Distributed tracing will be introduced using **OpenTelemetry**.

Tracing allows tracking requests across services such as:

```
Order Service
   → Inventory Service
   → Payment Service
   → Notification Service
```

---

## Monitoring Platform

Telemetry data can be aggregated using **New Relic**, providing:

* service performance monitoring
* distributed tracing visualization
* centralized logging
* operational dashboards
* alerting

---

# Security

Security is implemented using **Spring Security with JWT-based authentication**.

Key features include:

* user registration and login
* secure password hashing
* JWT token generation
* token validation for protected endpoints
* role-based authorization

Supported roles:

```
CUSTOMER
EMPLOYEE
MANAGER
ADMIN
```

Authorization policies ensure that:

* customers can place and view their own orders
* employees can manage order preparation
* managers can manage products and inventory

---

# API Design

The platform follows a **RESTful API design**.

Example endpoints:

```
POST /auth/register
POST /auth/login

GET /products
GET /products/{id}

POST /orders
GET /orders/my

PUT /staff/orders/{id}/status
```

Detailed API documentation is available in the `docs/` directory and in each service-level README.

---

# Repository Structure

```
bacha-cafe/

README.md
.env
.env.example
.gitignore

docs/
  product-overview.md
  architecture.md
  api-design.md
  domain-model.md
  observability.md
  deployment.md
  roadmap.md

frontend/

services/
  auth-service/
  catalog-service/
  order-service/
  inventory-service/
  payment-service/
  notification-service/

docker/
  docker-compose.yml

```

---

# Deployment Strategy

Local development environment:

* Docker
* Docker Compose

Initial public deployment:

* Render

Future infrastructure expansion:

* AWS cloud services
* Kubernetes orchestration

---

# Project Status

The platform is currently under active development as services and system capabilities are progressively implemented.

---

If you'd like, the **next important step** (before writing code) is creating:

**`docs/domain-model.md`**

This will define:

* Product
* Category
* Cart
* Order
* OrderItem
* Payment
* Inventory
* Notification

Designing this first will **save you from major refactoring later**.
