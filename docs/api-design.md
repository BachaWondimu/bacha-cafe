````md
# Bacha Cafe API Design

## 1. Overview

This document defines the API design for Bacha Cafe.  
The goal is to provide clear, consistent, and scalable REST APIs for the frontend and for inter-service communication.

The API design follows these principles:

- resource-oriented endpoints
- predictable naming
- JSON request/response format
- proper HTTP methods
- clear status codes
- validation for all inputs
- secure access using JWT
- version-ready structure

Base path example:

```text
/api/v1
````

---

## 2. General API Standards

### 2.1 Content Type

All APIs use JSON.

**Request**

```http
Content-Type: application/json
```

**Response**

```http
Content-Type: application/json
```

---

### 2.2 Common HTTP Methods

* `GET` → retrieve data
* `POST` → create resource or trigger action
* `PUT` → full update
* `PATCH` → partial update
* `DELETE` → remove resource

---

### 2.3 Naming Conventions

Use:

* nouns, not verbs, for resources
* plural resource names where appropriate
* lowercase words with hyphens only if needed
* path variables for specific resources

Examples:

```text
/api/v1/products
/api/v1/products/{productId}
/api/v1/orders
/api/v1/orders/{orderId}
```

---

### 2.4 Response Envelope

A consistent response structure makes frontend integration easier.

### Success response

```json
{
  "success": true,
  "message": "Products fetched successfully",
  "data": []
}
```

### Error response

```json
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "error": "Email is invalid"
    }
  ]
}
```

---

### 2.5 Pagination Format

For list APIs:

```json
{
  "success": true,
  "message": "Products fetched successfully",
  "data": {
    "items": [],
    "page": 0,
    "size": 10,
    "totalItems": 100,
    "totalPages": 10
  }
}
```

---

### 2.6 Authentication Header

Protected APIs require JWT in the Authorization header:

```http
Authorization: Bearer <token>
```

---

## 3. Service API Boundaries

The system exposes APIs through these services:

* Auth Service
* Catalog Service
* Order Service
* Inventory Service
* Payment Service
* Notification Service

Not every service must be directly exposed to the frontend.

### Frontend-facing services

* Auth Service
* Catalog Service
* Order Service
* Payment Service

### Mostly internal services

* Inventory Service
* Notification Service

---

## 4. Auth Service APIs

Base path:

```text
/api/v1/auth
```

### 4.1 Register User

**POST** `/api/v1/auth/register`

#### Request

```json
{
  "firstName": "Bacha",
  "lastName": "Wondimu",
  "email": "bacha@example.com",
  "password": "Password123"
}
```

#### Response

```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "userId": 1,
    "email": "bacha@example.com"
  }
}
```

#### Validation

* firstName required
* lastName required
* email required and unique
* password required with minimum rules

---

### 4.2 Login

**POST** `/api/v1/auth/login`

#### Request

```json
{
  "email": "bacha@example.com",
  "password": "Password123"
}
```

#### Response

```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "accessToken": "jwt-token",
    "refreshToken": "refresh-token",
    "user": {
      "id": 1,
      "firstName": "Bacha",
      "lastName": "Wondimu",
      "email": "bacha@example.com",
      "role": "CUSTOMER"
    }
  }
}
```

---

### 4.3 Refresh Token

**POST** `/api/v1/auth/refresh`

#### Request

```json
{
  "refreshToken": "refresh-token"
}
```

#### Response

```json
{
  "success": true,
  "message": "Token refreshed successfully",
  "data": {
    "accessToken": "new-jwt-token"
  }
}
```

---

### 4.4 Get Current User

**GET** `/api/v1/auth/me`

#### Headers

```http
Authorization: Bearer <token>
```

#### Response

```json
{
  "success": true,
  "message": "User fetched successfully",
  "data": {
    "id": 1,
    "firstName": "Bacha",
    "lastName": "Wondimu",
    "email": "bacha@example.com",
    "role": "CUSTOMER"
  }
}
```

---

### 4.5 Logout

**POST** `/api/v1/auth/logout`

#### Request

```json
{
  "refreshToken": "refresh-token"
}
```

#### Response

```json
{
  "success": true,
  "message": "Logged out successfully",
  "data": null
}
```

---

## 5. Catalog Service APIs

Base path:

```text
/api/v1/products
```

### 5.1 Get All Products

**GET** `/api/v1/products`

#### Query params

* `page`
* `size`
* `category`
* `search`
* `available`

Example:

```text
/api/v1/products?page=0&size=10&category=coffee&search=latte
```

#### Response

```json
{
  "success": true,
  "message": "Products fetched successfully",
  "data": {
    "items": [
      {
        "id": 101,
        "name": "Caffe Latte",
        "description": "Fresh espresso with steamed milk",
        "price": 4.99,
        "category": "COFFEE",
        "imageUrl": "/images/latte.jpg",
        "available": true
      }
    ],
    "page": 0,
    "size": 10,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

---

### 5.2 Get Product By ID

**GET** `/api/v1/products/{productId}`

#### Response

```json
{
  "success": true,
  "message": "Product fetched successfully",
  "data": {
    "id": 101,
    "name": "Caffe Latte",
    "description": "Fresh espresso with steamed milk",
    "price": 4.99,
    "category": "COFFEE",
    "imageUrl": "/images/latte.jpg",
    "available": true
  }
}
```

---

### 5.3 Create Product

**POST** `/api/v1/products`

Protected: `ADMIN`

#### Request

```json
{
  "name": "Mocha",
  "description": "Chocolate flavored coffee",
  "price": 5.49,
  "category": "COFFEE",
  "imageUrl": "/images/mocha.jpg",
  "available": true
}
```

#### Response

```json
{
  "success": true,
  "message": "Product created successfully",
  "data": {
    "id": 102,
    "name": "Mocha"
  }
}
```

---

### 5.4 Update Product

**PUT** `/api/v1/products/{productId}`

Protected: `ADMIN`

#### Request

```json
{
  "name": "Mocha",
  "description": "Premium chocolate flavored coffee",
  "price": 5.99,
  "category": "COFFEE",
  "imageUrl": "/images/mocha.jpg",
  "available": true
}
```

---

### 5.5 Delete Product

**DELETE** `/api/v1/products/{productId}`

Protected: `ADMIN`

#### Response

```json
{
  "success": true,
  "message": "Product deleted successfully",
  "data": null
}
```

---

## 6. Order Service APIs

Base path:

```text
/api/v1/orders
```

### 6.1 Create Order

**POST** `/api/v1/orders`

Protected: authenticated user

#### Request

```json
{
  "items": [
    {
      "productId": 101,
      "quantity": 2
    },
    {
      "productId": 205,
      "quantity": 1
    }
  ],
  "notes": "No sugar please"
}
```

#### Response

```json
{
  "success": true,
  "message": "Order created successfully",
  "data": {
    "orderId": 5001,
    "status": "PENDING",
    "totalAmount": 14.97,
    "items": [
      {
        "productId": 101,
        "productName": "Caffe Latte",
        "quantity": 2,
        "unitPrice": 4.99,
        "subtotal": 9.98
      },
      {
        "productId": 205,
        "productName": "Blueberry Muffin",
        "quantity": 1,
        "unitPrice": 4.99,
        "subtotal": 4.99
      }
    ]
  }
}
```

---

### 6.2 Get Order By ID

**GET** `/api/v1/orders/{orderId}`

Protected: authenticated user or admin/staff

#### Response

```json
{
  "success": true,
  "message": "Order fetched successfully",
  "data": {
    "orderId": 5001,
    "customerId": 1,
    "status": "PAID",
    "totalAmount": 14.97,
    "createdAt": "2026-03-10T10:30:00Z",
    "items": [
      {
        "productId": 101,
        "productName": "Caffe Latte",
        "quantity": 2,
        "unitPrice": 4.99,
        "subtotal": 9.98
      }
    ]
  }
}
```

---

### 6.3 Get Current User Orders

**GET** `/api/v1/orders/my-orders`

#### Query params

* `page`
* `size`
* `status`

Example:

```text
/api/v1/orders/my-orders?page=0&size=10&status=PAID
```

---

### 6.4 Update Order Status

**PATCH** `/api/v1/orders/{orderId}/status`

Protected: `STAFF` or `ADMIN`

#### Request

```json
{
  "status": "PREPARING"
}
```

#### Response

```json
{
  "success": true,
  "message": "Order status updated successfully",
  "data": {
    "orderId": 5001,
    "status": "PREPARING"
  }
}
```

---

### 6.5 Cancel Order

**PATCH** `/api/v1/orders/{orderId}/cancel`

Protected: authenticated user

#### Response

```json
{
  "success": true,
  "message": "Order cancelled successfully",
  "data": {
    "orderId": 5001,
    "status": "CANCELLED"
  }
}
```

---

## 7. Payment Service APIs

Base path:

```text
/api/v1/payments
```

### 7.1 Initiate Payment

**POST** `/api/v1/payments`

#### Request

```json
{
  "orderId": 5001,
  "paymentMethod": "CARD"
}
```

#### Response

```json
{
  "success": true,
  "message": "Payment initiated successfully",
  "data": {
    "paymentId": 9001,
    "orderId": 5001,
    "status": "PENDING",
    "paymentUrl": "https://payment-gateway.example/checkout/abc123"
  }
}
```

---

### 7.2 Get Payment By ID

**GET** `/api/v1/payments/{paymentId}`

#### Response

```json
{
  "success": true,
  "message": "Payment fetched successfully",
  "data": {
    "paymentId": 9001,
    "orderId": 5001,
    "status": "SUCCESS",
    "amount": 14.97,
    "transactionReference": "TXN123456",
    "paidAt": "2026-03-10T10:35:00Z"
  }
}
```

---

### 7.3 Payment Callback Webhook

**POST** `/api/v1/payments/webhook`

This endpoint is called by the external payment provider.

#### Request example

```json
{
  "transactionReference": "TXN123456",
  "orderId": 5001,
  "status": "SUCCESS",
  "amount": 14.97
}
```

#### Response

```json
{
  "success": true,
  "message": "Webhook processed successfully",
  "data": null
}
```

#### Notes

* verify signature from provider
* make processing idempotent
* do not trust raw payload blindly

---

## 8. Inventory Service APIs

Base path:

```text
/api/v1/inventory
```

These APIs are mostly internal or admin-facing.

### 8.1 Check Product Stock

**GET** `/api/v1/inventory/{productId}`

#### Response

```json
{
  "success": true,
  "message": "Inventory fetched successfully",
  "data": {
    "productId": 101,
    "availableQuantity": 25,
    "reservedQuantity": 3,
    "inStock": true
  }
}
```

---

### 8.2 Reserve Stock

**POST** `/api/v1/inventory/reservations`

Internal use by Order Service.

#### Request

```json
{
  "orderId": 5001,
  "items": [
    {
      "productId": 101,
      "quantity": 2
    }
  ]
}
```

#### Response

```json
{
  "success": true,
  "message": "Stock reserved successfully",
  "data": {
    "reservationId": 7001,
    "orderId": 5001,
    "status": "RESERVED"
  }
}
```

---

### 8.3 Release Stock

**POST** `/api/v1/inventory/release`

Internal use when payment fails or order is cancelled.

#### Request

```json
{
  "orderId": 5001
}
```

---

### 8.4 Deduct Stock

**POST** `/api/v1/inventory/deduct`

Internal use after successful payment.

#### Request

```json
{
  "orderId": 5001
}
```

---

### 8.5 Update Stock

**PATCH** `/api/v1/inventory/{productId}`

Protected: `ADMIN` or `STAFF`

#### Request

```json
{
  "availableQuantity": 50
}
```

---

## 9. Notification Service APIs

Base path:

```text
/api/v1/notifications
```

Mostly internal APIs.

### 9.1 Send Notification

**POST** `/api/v1/notifications`

#### Request

```json
{
  "type": "ORDER_CONFIRMED",
  "channel": "EMAIL",
  "recipient": "bacha@example.com",
  "payload": {
    "orderId": 5001,
    "customerName": "Bacha"
  }
}
```

#### Response

```json
{
  "success": true,
  "message": "Notification queued successfully",
  "data": {
    "notificationId": 3001,
    "status": "QUEUED"
  }
}
```

---

### 9.2 Get Notification Status

**GET** `/api/v1/notifications/{notificationId}`

#### Response

```json
{
  "success": true,
  "message": "Notification fetched successfully",
  "data": {
    "notificationId": 3001,
    "status": "SENT",
    "channel": "EMAIL",
    "recipient": "bacha@example.com"
  }
}
```

---

## 10. Common Status Codes

### Success codes

* `200 OK` → successful retrieval/update
* `201 Created` → resource created
* `202 Accepted` → async request accepted
* `204 No Content` → delete success without body

### Client error codes

* `400 Bad Request` → invalid request
* `401 Unauthorized` → missing/invalid token
* `403 Forbidden` → no permission
* `404 Not Found` → resource not found
* `409 Conflict` → duplicate or invalid state conflict
* `422 Unprocessable Entity` → validation/business rule issue

### Server error codes

* `500 Internal Server Error`
* `502 Bad Gateway`
* `503 Service Unavailable`

---

## 11. Error Handling Design

### Validation error example

```json
{
  "success": false,
  "message": "Validation failed",
  "errors": [
    {
      "field": "password",
      "error": "Password must be at least 8 characters"
    }
  ]
}
```

### Not found example

```json
{
  "success": false,
  "message": "Order not found",
  "errors": []
}
```

### Unauthorized example

```json
{
  "success": false,
  "message": "Invalid or expired token",
  "errors": []
}
```

---

## 12. Security Design for APIs

### Authentication

Use JWT access token for protected routes.

### Authorization

Use role-based access control.

Examples:

* CUSTOMER → place order, view own orders
* STAFF → update order preparation status
* ADMIN → manage products and stock

### Additional protections

* input validation
* password hashing
* API rate limiting
* CORS configuration
* request logging
* secure secret management
* webhook signature verification

---

## 13. API Versioning

Use URI versioning:

```text
/api/v1/...
```

This makes it easier to introduce breaking changes later:

```text
/api/v2/...
```

---

## 14. Idempotency Considerations

Some APIs should be idempotent or protected from duplicate processing.

Important examples:

* payment webhook processing
* payment initiation retry handling
* order creation retry from frontend
* inventory deduction

Possible solution:

* use idempotency key
* store processed transaction reference
* reject duplicate payment updates

---

## 15. Inter-Service API Considerations

Internal service-to-service APIs should include:

* internal authentication or service token
* trace ID / correlation ID
* retry rules
* timeout settings
* structured error responses

Example headers:

```http
X-Correlation-Id: 12345-abcde
X-Service-Name: order-service
```

---

## 16. Example End-to-End Flow

### Place Order Flow

#### 1. Login

`POST /api/v1/auth/login`

#### 2. Browse products

`GET /api/v1/products`

#### 3. Create order

`POST /api/v1/orders`

#### 4. Reserve stock

Internal call:
`POST /api/v1/inventory/reservations`

#### 5. Initiate payment

`POST /api/v1/payments`

#### 6. Payment provider callback

`POST /api/v1/payments/webhook`

#### 7. Update order status

Internal workflow updates order to `PAID`

#### 8. Send notification

Internal event triggers notification service

---

## 17. Suggested DTO Groups

To keep API design clean, define DTOs clearly.

### Auth DTOs

* RegisterRequest
* LoginRequest
* AuthResponse
* RefreshTokenRequest

### Product DTOs

* ProductRequest
* ProductResponse
* ProductSummaryResponse

### Order DTOs

* CreateOrderRequest
* OrderItemRequest
* OrderResponse
* UpdateOrderStatusRequest

### Payment DTOs

* PaymentRequest
* PaymentResponse
* PaymentWebhookRequest

### Inventory DTOs

* InventoryResponse
* ReserveStockRequest
* UpdateStockRequest

### Notification DTOs

* NotificationRequest
* NotificationResponse

---

## 18. Summary

The Bacha Cafe API design is built around simple, consistent REST principles.

It provides:

* clear endpoint structure
* clean request/response contracts
* secure token-based access
* strong validation and error handling
* service boundaries aligned with microservices
* room for future growth and versioning

This API structure is a strong foundation for implementing the backend and integrating the frontend cleanly.


