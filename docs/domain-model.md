```md
# Bacha Cafe Domain Model

## 1. Overview

The domain model defines the core business entities used in the Bacha Cafe system and the relationships between them.

Each microservice owns the entities that belong to its domain.  
Other services should access data only through APIs or events, not through direct database access.

The system is divided into the following domains:

- Authentication Domain
- Catalog Domain
- Order Domain
- Inventory Domain
- Payment Domain
- Notification Domain

Each domain has its own entities and database tables.

---

# 2. Authentication Domain

This domain manages users, authentication, and authorization.

Owned by **Auth Service**.

---

## 2.1 User

Represents a registered customer or staff member.

### Fields

| Field | Type | Description |
|------|------|-------------|
| id | Long | unique identifier |
| firstName | String | user's first name |
| lastName | String | user's last name |
| email | String | login email |
| passwordHash | String | encrypted password |
| role | Enum | CUSTOMER / ADMIN / STAFF |
| status | Enum | ACTIVE / DISABLED |
| createdAt | Timestamp | account creation time |
| updatedAt | Timestamp | last update |

### Example

```

## User

id: 1
firstName: Bacha
lastName: Wondimu
email: [bacha@example.com](mailto:bacha@example.com)
role: CUSTOMER
status: ACTIVE
createdAt: 2026-03-10

```

---

## 2.2 RefreshToken

Used to issue new JWT access tokens without logging in again.

### Fields

| Field | Type |
|------|------|
| id | Long |
| userId | Long |
| token | String |
| expiresAt | Timestamp |
| createdAt | Timestamp |

### Relationship

```

User (1) ------ (N) RefreshToken

```

A user can have multiple active refresh tokens.

---

# 3. Catalog Domain

This domain manages the cafe menu and products.

Owned by **Catalog Service**.

---

## 3.1 Product

Represents an item sold by the cafe.

### Fields

| Field | Type | Description |
|------|------|-------------|
| id | Long | product ID |
| name | String | product name |
| description | String | product details |
| price | Decimal | product price |
| categoryId | Long | category reference |
| imageUrl | String | product image |
| available | Boolean | available for sale |
| createdAt | Timestamp | creation date |
| updatedAt | Timestamp | last update |

### Example

```

## Product

id: 101
name: Caffe Latte
price: 4.99
category: Coffee
available: true

```

---

## 3.2 Category

Groups products into logical menu sections.

### Fields

| Field | Type |
|------|------|
| id | Long |
| name | String |
| description | String |
| createdAt | Timestamp |

### Example Categories

- Coffee
- Tea
- Pastries
- Sandwiches
- Desserts

### Relationship

```

Category (1) ------ (N) Product

```

---

# 4. Order Domain

This domain manages customer orders.

Owned by **Order Service**.

---

## 4.1 Order

Represents a customer purchase.

### Fields

| Field | Type |
|------|------|
| id | Long |
| customerId | Long |
| status | Enum |
| totalAmount | Decimal |
| notes | String |
| createdAt | Timestamp |
| updatedAt | Timestamp |

### Order Status

```

PENDING
CONFIRMED
PAID
PREPARING
READY
COMPLETED
CANCELLED
PAYMENT_FAILED

```

### Example

```

## Order

id: 5001
customerId: 1
status: PAID
totalAmount: 14.97
createdAt: 2026-03-10

```

---

## 4.2 OrderItem

Represents an item inside an order.

### Fields

| Field | Type |
|------|------|
| id | Long |
| orderId | Long |
| productId | Long |
| productName | String |
| quantity | Integer |
| unitPrice | Decimal |
| subtotal | Decimal |

### Relationship

```

Order (1) ------ (N) OrderItem

```

### Example

```

## OrderItem

orderId: 5001
productId: 101
productName: Caffe Latte
quantity: 2
unitPrice: 4.99
subtotal: 9.98

```

---

# 5. Inventory Domain

This domain tracks stock levels.

Owned by **Inventory Service**.

---

## 5.1 Inventory

Tracks product availability.

### Fields

| Field | Type |
|------|------|
| productId | Long |
| availableQuantity | Integer |
| reservedQuantity | Integer |
| updatedAt | Timestamp |

### Example

```

## Inventory

productId: 101
availableQuantity: 50
reservedQuantity: 5

```

---

## 5.2 StockReservation

Temporary reservation of inventory during order processing.

### Fields

| Field | Type |
|------|------|
| id | Long |
| orderId | Long |
| productId | Long |
| quantity | Integer |
| status | Enum |
| createdAt | Timestamp |

### Status

```

RESERVED
CONFIRMED
RELEASED

```

### Relationship

```

Order (1) ------ (N) StockReservation

```

---

# 6. Payment Domain

This domain manages payments.

Owned by **Payment Service**.

---

## 6.1 Payment

Represents a payment attempt.

### Fields

| Field | Type |
|------|------|
| id | Long |
| orderId | Long |
| amount | Decimal |
| status | Enum |
| paymentMethod | Enum |
| transactionReference | String |
| createdAt | Timestamp |
| paidAt | Timestamp |

### Payment Status

```

PENDING
SUCCESS
FAILED
REFUNDED

```

### Payment Methods

```

CARD
APPLE_PAY
GOOGLE_PAY
CASH

```

### Example

```

## Payment

id: 9001
orderId: 5001
amount: 14.97
status: SUCCESS
transactionReference: TXN123456

```

---

# 7. Notification Domain

This domain handles customer communication.

Owned by **Notification Service**.

---

## 7.1 Notification

Represents a notification sent to a user.

### Fields

| Field | Type |
|------|------|
| id | Long |
| type | Enum |
| channel | Enum |
| recipient | String |
| status | Enum |
| payload | JSON |
| createdAt | Timestamp |
| sentAt | Timestamp |

---

### Notification Types

```

ORDER_CONFIRMED
PAYMENT_SUCCESS
PAYMENT_FAILED
ORDER_READY
PASSWORD_RESET

```

---

### Notification Channels

```

EMAIL
SMS
PUSH

```

---

### Notification Status

```

QUEUED
SENT
FAILED
RETRYING

```

---

# 8. Domain Relationships Overview

High-level relationships between domains.

```

User
|
| 1
|
| N
Order
|
| 1
|
| N
OrderItem ---- Product
|
|
Category

Order ---- Payment

Product ---- Inventory

Order ---- StockReservation

Order ---- Notification

```

---

# 9. Service Ownership

| Domain | Service | Database |
|------|------|------|
| Authentication | Auth Service | Auth DB |
| Catalog | Catalog Service | Catalog DB |
| Orders | Order Service | Order DB |
| Inventory | Inventory Service | Inventory DB |
| Payments | Payment Service | Payment DB |
| Notifications | Notification Service | Notification DB |

Each service manages its own data and schema.

---

# 10. Domain Events

The system uses events to communicate between services.

Examples:

### OrderCreated

```

{
orderId
customerId
items
totalAmount
}

```

---

### PaymentSucceeded

```

{
orderId
paymentId
amount
}

```

---

### PaymentFailed

```

{
orderId
reason
}

```

---

### OrderReady

```

{
orderId
customerId
}

```

---

### InventoryReserved

```

{
orderId
productId
quantity
}

```

These events allow services to remain loosely coupled.

---

# 11. Aggregate Boundaries

Each service maintains strong aggregate boundaries.

### Auth Aggregate
```

User
RefreshToken

```

### Catalog Aggregate
```

Category
Product

```

### Order Aggregate
```

Order
OrderItem

```

### Inventory Aggregate
```

Inventory
StockReservation

```

### Payment Aggregate
```

Payment

```

### Notification Aggregate
```

Notification

```

---

# 12. Key Domain Rules

Important business rules:

- A user must be authenticated to place an order
- Orders must have at least one item
- Payment must succeed before order moves to PREPARING
- Inventory must be reserved before payment
- Stock is deducted only after payment success
- Notifications should never block order processing

---

# 13. Summary

The Bacha Cafe domain model defines the core business entities and relationships used across the system.

The model supports:

- microservice ownership of data
- clear service boundaries
- scalable event-driven architecture
- clean entity design
- future expansion of the system

This domain model will guide:

- database schema design
- entity classes
- API DTOs
- service logic
- event structures
```

Next we should create **`observability.md`**, which explains:

* logging strategy
* distributed tracing
* metrics
* monitoring dashboards
* alerting

This is something **real production systems always include**, and it will make your project look **very senior-level**.
