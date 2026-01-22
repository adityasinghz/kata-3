# FlashKart Quick Commerce - API Specification

## Table of Contents
1. [API Overview](#api-overview)
2. [Authentication](#authentication)
3. [Customer APIs](#customer-apis)
4. [Order APIs](#order-apis)
5. [Product & Catalog APIs](#product--catalog-apis)
6. [Inventory APIs](#inventory-apis)
7. [Payment APIs](#payment-apis)
8. [Delivery APIs](#delivery-apis)
9. [Dark Store APIs](#dark-store-apis)
10. [Admin APIs](#admin-apis)

---

## API Overview

### Base URL
```
Production: https://api.flashkart.com/v1
Staging: https://api-staging.flashkart.com/v1
```

### API Versioning
- Version included in URL path: `/v1/`
- Version header: `API-Version: 1.0`

### Response Format
All APIs return JSON responses with the following structure:

```json
{
  "success": true,
  "data": {},
  "message": "Success",
  "timestamp": "2025-01-27T10:00:00Z"
}
```

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Error message",
    "details": {}
  },
  "timestamp": "2025-01-27T10:00:00Z"
}
```

### HTTP Status Codes
- `200 OK`: Success
- `201 Created`: Resource created
- `400 Bad Request`: Invalid request
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `409 Conflict`: Resource conflict
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

---

## Authentication

### JWT Token Authentication
All APIs (except public endpoints) require JWT token in Authorization header:

```
Authorization: Bearer <jwt_token>
```

### Login
**POST** `/auth/login`

**Request:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "uuid",
      "name": "John Doe",
      "email": "user@example.com",
      "role": "CUSTOMER"
    },
    "expires_in": 3600
  }
}
```

### Register
**POST** `/auth/register`

**Request:**
```json
{
  "name": "John Doe",
  "email": "user@example.com",
  "phone": "+919876543210",
  "password": "password123",
  "role": "CUSTOMER"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "name": "John Doe",
      "email": "user@example.com"
    }
  }
}
```

---

## Customer APIs

### Get User Profile
**GET** `/customers/profile`

**Headers:**
```
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "John Doe",
    "email": "user@example.com",
    "phone": "+919876543210",
    "default_address": {
      "id": "uuid",
      "street": "123 Main St",
      "city": "Mumbai",
      "pincode": "400001"
    }
  }
}
```

### Update Profile
**PUT** `/customers/profile`

**Request:**
```json
{
  "name": "John Doe Updated",
  "phone": "+919876543211"
}
```

### Get Addresses
**GET** `/customers/addresses`

**Response:**
```json
{
  "success": true,
  "data": {
    "addresses": [
      {
        "id": "uuid",
        "street": "123 Main St",
        "city": "Mumbai",
        "state": "Maharashtra",
        "pincode": "400001",
        "latitude": 19.0760,
        "longitude": 72.8777,
        "is_default": true
      }
    ]
  }
}
```

### Add Address
**POST** `/customers/addresses`

**Request:**
```json
{
  "street": "123 Main St",
  "city": "Mumbai",
  "state": "Maharashtra",
  "pincode": "400001",
  "latitude": 19.0760,
  "longitude": 72.8777,
  "is_default": false
}
```

### Update Address
**PUT** `/customers/addresses/{addressId}`

### Delete Address
**DELETE** `/customers/addresses/{addressId}`

---

## Product & Catalog APIs

### Get Products (Browse)
**GET** `/products`

**Query Parameters:**
- `latitude` (required): Customer latitude
- `longitude` (required): Customer longitude
- `category_id` (optional): Filter by category
- `search` (optional): Search query
- `min_price` (optional): Minimum price
- `max_price` (optional): Maximum price
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 20)

**Response:**
```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "uuid",
        "name": "Milk 1L",
        "description": "Fresh milk",
        "category": {
          "id": "uuid",
          "name": "Dairy"
        },
        "base_price": 50.00,
        "images": ["https://..."],
        "stores": [
          {
            "store_id": "uuid",
            "store_name": "Store A",
            "price": 50.00,
            "available_quantity": 100,
            "estimated_delivery_minutes": 15
          }
        ]
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "total_pages": 8
    }
  }
}
```

### Get Product Details
**GET** `/products/{productId}`

**Query Parameters:**
- `latitude` (required): Customer latitude
- `longitude` (required): Customer longitude

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Milk 1L",
    "description": "Fresh milk",
    "category": {
      "id": "uuid",
      "name": "Dairy"
    },
    "base_price": 50.00,
    "images": ["https://..."],
    "nutritional_info": {
      "calories": 150,
      "protein": "8g",
      "fat": "5g"
    },
    "stores": [
      {
        "store_id": "uuid",
        "store_name": "Store A",
        "price": 50.00,
        "available_quantity": 100,
        "distance_km": 1.5,
        "estimated_delivery_minutes": 15
      }
    ]
  }
}
```

### Search Products
**GET** `/products/search`

**Query Parameters:**
- `q` (required): Search query
- `latitude` (required): Customer latitude
- `longitude` (required): Customer longitude
- `category_id` (optional): Filter by category
- `page` (optional): Page number
- `limit` (optional): Items per page

### Get Categories
**GET** `/categories`

**Response:**
```json
{
  "success": true,
  "data": {
    "categories": [
      {
        "id": "uuid",
        "name": "Dairy",
        "description": "Dairy products",
        "parent_id": null,
        "product_count": 50
      }
    ]
  }
}
```

---

## Order APIs

### Get Cart
**GET** `/orders/cart`

**Headers:**
```
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "product_id": "uuid",
        "product_name": "Milk 1L",
        "quantity": 2,
        "unit_price": 50.00,
        "total_price": 100.00,
        "store_id": "uuid",
        "store_name": "Store A"
      }
    ],
    "subtotal": 100.00,
    "delivery_fee": 30.00,
    "total": 130.00
  }
}
```

### Add to Cart
**POST** `/orders/cart/items`

**Request:**
```json
{
  "product_id": "uuid",
  "store_id": "uuid",
  "quantity": 2
}
```

### Update Cart Item
**PUT** `/orders/cart/items/{itemId}`

**Request:**
```json
{
  "quantity": 3
}
```

### Remove from Cart
**DELETE** `/orders/cart/items/{itemId}`

### Place Order
**POST** `/orders`

**Request:**
```json
{
  "address_id": "uuid",
  "items": [
    {
      "product_id": "uuid",
      "quantity": 2
    }
  ],
  "payment_method": "UPI"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "order": {
      "id": "uuid",
      "status": "CONFIRMED",
      "total_amount": 130.00,
      "store_id": "uuid",
      "store_name": "Store A",
      "estimated_delivery_time": "2025-01-27T10:20:00Z",
      "sla_minutes": 20
    },
    "payment": {
      "id": "uuid",
      "status": "SUCCESS",
      "transaction_id": "txn_123"
    }
  }
}
```

### Get Order Details
**GET** `/orders/{orderId}`

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "IN_TRANSIT",
    "total_amount": 130.00,
    "items": [
      {
        "product_id": "uuid",
        "product_name": "Milk 1L",
        "quantity": 2,
        "unit_price": 50.00,
        "total_price": 100.00
      }
    ],
    "store": {
      "id": "uuid",
      "name": "Store A"
    },
    "delivery_partner": {
      "id": "uuid",
      "name": "Rajesh Kumar",
      "phone": "+919876543210"
    },
    "delivery_address": {
      "street": "123 Main St",
      "city": "Mumbai",
      "pincode": "400001"
    },
    "estimated_delivery_time": "2025-01-27T10:20:00Z",
    "created_at": "2025-01-27T10:00:00Z"
  }
}
```

### Track Order
**GET** `/orders/{orderId}/track`

**Response:**
```json
{
  "success": true,
  "data": {
    "order_id": "uuid",
    "status": "IN_TRANSIT",
    "current_location": {
      "latitude": 19.0760,
      "longitude": 72.8777,
      "timestamp": "2025-01-27T10:15:00Z"
    },
    "estimated_delivery_time": "2025-01-27T10:20:00Z",
    "status_history": [
      {
        "status": "CONFIRMED",
        "timestamp": "2025-01-27T10:00:00Z"
      },
      {
        "status": "PICKING",
        "timestamp": "2025-01-27T10:05:00Z"
      },
      {
        "status": "IN_TRANSIT",
        "timestamp": "2025-01-27T10:10:00Z"
      }
    ]
  }
}
```

### Cancel Order
**POST** `/orders/{orderId}/cancel`

**Request:**
```json
{
  "reason": "Changed my mind"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "order_id": "uuid",
    "status": "CANCELLED",
    "refund": {
      "id": "uuid",
      "amount": 130.00,
      "status": "PROCESSING"
    }
  }
}
```

### Get Order History
**GET** `/orders`

**Query Parameters:**
- `status` (optional): Filter by status
- `page` (optional): Page number
- `limit` (optional): Items per page

**Response:**
```json
{
  "success": true,
  "data": {
    "orders": [
      {
        "id": "uuid",
        "status": "DELIVERED",
        "total_amount": 130.00,
        "created_at": "2025-01-27T10:00:00Z",
        "delivered_at": "2025-01-27T10:18:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 50,
      "total_pages": 3
    }
  }
}
```

---

## Inventory APIs

### Get Inventory (Store)
**GET** `/inventory/stores/{storeId}/products/{productId}`

**Headers:**
```
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "store_id": "uuid",
    "product_id": "uuid",
    "available_quantity": 100,
    "reserved_quantity": 5,
    "last_updated": "2025-01-27T10:00:00Z"
  }
}
```

### Update Inventory
**PUT** `/inventory/stores/{storeId}/products/{productId}`

**Request:**
```json
{
  "quantity": 95,
  "change_type": "OUT",
  "reason": "Order fulfillment"
}
```

### Reserve Inventory
**POST** `/inventory/reserve`

**Request:**
```json
{
  "order_id": "uuid",
  "store_id": "uuid",
  "items": [
    {
      "product_id": "uuid",
      "quantity": 2
    }
  ],
  "expires_in_minutes": 10
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "reservation_id": "uuid",
    "reserved": true,
    "expires_at": "2025-01-27T10:10:00Z"
  }
}
```

### Release Inventory
**POST** `/inventory/release`

**Request:**
```json
{
  "order_id": "uuid"
}
```

---

## Payment APIs

### Process Payment
**POST** `/payments`

**Request:**
```json
{
  "order_id": "uuid",
  "payment_method": "UPI",
  "amount": 130.00,
  "payment_details": {
    "upi_id": "user@paytm"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "payment_id": "uuid",
    "status": "SUCCESS",
    "transaction_id": "txn_123",
    "amount": 130.00,
    "processed_at": "2025-01-27T10:00:00Z"
  }
}
```

### Get Payment Status
**GET** `/payments/{paymentId}`

### Process Refund
**POST** `/payments/{paymentId}/refund`

**Request:**
```json
{
  "amount": 130.00,
  "reason": "Order cancelled"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "refund_id": "uuid",
    "status": "PROCESSING",
    "amount": 130.00,
    "estimated_completion": "2025-01-27T11:00:00Z"
  }
}
```

---

## Delivery APIs

### Get Available Partners
**GET** `/delivery/partners/available`

**Query Parameters:**
- `latitude` (required): Store latitude
- `longitude` (required): Store longitude
- `radius_km` (optional): Search radius (default: 5)

**Response:**
```json
{
  "success": true,
  "data": {
    "partners": [
      {
        "id": "uuid",
        "name": "Rajesh Kumar",
        "current_location": {
          "latitude": 19.0760,
          "longitude": 72.8777
        },
        "distance_km": 1.2,
        "is_available": true,
        "current_orders_count": 0,
        "rating": 4.5
      }
    ]
  }
}
```

### Assign Partner to Order
**POST** `/delivery/assignments`

**Request:**
```json
{
  "order_id": "uuid",
  "partner_id": "uuid"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "assignment_id": "uuid",
    "order_id": "uuid",
    "partner_id": "uuid",
    "status": "ASSIGNED",
    "assigned_at": "2025-01-27T10:00:00Z"
  }
}
```

### Update Delivery Status
**PUT** `/delivery/assignments/{assignmentId}/status`

**Request:**
```json
{
  "status": "PICKED_UP",
  "location": {
    "latitude": 19.0760,
    "longitude": 72.8777
  }
}
```

### Get Route
**GET** `/delivery/route`

**Query Parameters:**
- `start_latitude` (required)
- `start_longitude` (required)
- `end_latitude` (required)
- `end_longitude` (required)

**Response:**
```json
{
  "success": true,
  "data": {
    "distance_km": 2.5,
    "estimated_time_minutes": 8,
    "route": {
      "waypoints": [
        {"latitude": 19.0760, "longitude": 72.8777},
        {"latitude": 19.0770, "longitude": 72.8787}
      ]
    }
  }
}
```

---

## Dark Store APIs

### Get Order Queue
**GET** `/stores/{storeId}/orders/queue`

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `status` (optional): Filter by status
- `page` (optional): Page number
- `limit` (optional): Items per page

**Response:**
```json
{
  "success": true,
  "data": {
    "orders": [
      {
        "id": "uuid",
        "status": "ASSIGNED_TO_STORE",
        "total_amount": 130.00,
        "items": [
          {
            "product_id": "uuid",
            "product_name": "Milk 1L",
            "quantity": 2
          }
        ],
        "sla_minutes": 20,
        "created_at": "2025-01-27T10:00:00Z"
      }
    ]
  }
}
```

### Update Order Status
**PUT** `/stores/{storeId}/orders/{orderId}/status`

**Request:**
```json
{
  "status": "PACKED",
  "notes": "Order ready for pickup"
}
```

### Scan Barcode
**POST** `/stores/{storeId}/scan`

**Request:**
```json
{
  "barcode": "1234567890",
  "action": "PICK", // PICK, PACK, INVENTORY_UPDATE
  "order_id": "uuid" // Required for PICK and PACK
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "product": {
      "id": "uuid",
      "name": "Milk 1L",
      "price": 50.00
    },
    "action": "PICK",
    "order_id": "uuid",
    "status": "SUCCESS"
  }
}
```

---

## Admin APIs

### Get Analytics
**GET** `/admin/analytics`

**Query Parameters:**
- `start_date` (required): Start date (ISO 8601)
- `end_date` (required): End date (ISO 8601)
- `city` (optional): Filter by city
- `store_id` (optional): Filter by store

**Response:**
```json
{
  "success": true,
  "data": {
    "orders": {
      "total": 10000,
      "completed": 9500,
      "cancelled": 500,
      "sla_compliance": 95.5
    },
    "revenue": {
      "total": 5000000.00,
      "average_order_value": 500.00
    },
    "inventory": {
      "accuracy": 99.2,
      "low_stock_items": 50
    }
  }
}
```

### Manage Stores
**POST** `/admin/stores`

**PUT** `/admin/stores/{storeId}`

**DELETE** `/admin/stores/{storeId}`

### Manage Partners
**POST** `/admin/partners`

**PUT** `/admin/partners/{partnerId}`

**GET** `/admin/partners/{partnerId}/performance`

---

## Rate Limiting

### Limits
- **Public APIs**: 100 requests per minute per IP
- **Authenticated APIs**: 1000 requests per minute per user
- **Admin APIs**: 500 requests per minute per user

### Headers
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1643270400
```

---

## Webhooks

### Order Status Updated
**POST** `/webhooks/order-status`

**Payload:**
```json
{
  "event": "order.status.updated",
  "order_id": "uuid",
  "status": "DELIVERED",
  "timestamp": "2025-01-27T10:20:00Z"
}
```

### Payment Processed
**POST** `/webhooks/payment-processed`

**Payload:**
```json
{
  "event": "payment.processed",
  "payment_id": "uuid",
  "order_id": "uuid",
  "status": "SUCCESS",
  "timestamp": "2025-01-27T10:00:00Z"
}
```

---

**Last Updated**: January 2025
**Version**: 1.0
**Status**: Design Complete, Implementation Pending
