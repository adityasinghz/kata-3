# FlashKart Quick Commerce - Database Schema

## Table of Contents
1. [Entity Relationship Diagram](#entity-relationship-diagram)
2. [Database Tables](#database-tables)
3. [Indexes](#indexes)
4. [Relationships](#relationships)
5. [Data Types and Constraints](#data-types-and-constraints)

---

## Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        ER Diagram                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐         ┌──────────────┐                    │
│  │    Users     │         │   Addresses  │                    │
│  ├──────────────┤         ├──────────────┤                    │
│  │ id (PK)      │────────│ id (PK)      │                    │
│  │ name         │   1:N   │ user_id (FK) │                    │
│  │ email        │         │ street       │                    │
│  │ phone        │         │ city         │                    │
│  │ role         │         │ latitude     │                    │
│  └──────────────┘         │ longitude    │                    │
│                           └──────────────┘                    │
│                                                                 │
│  ┌──────────────┐         ┌──────────────┐                    │
│  │   Products   │         │  Categories  │                    │
│  ├──────────────┤         ├──────────────┤                    │
│  │ id (PK)      │────────│ id (PK)      │                    │
│  │ name         │   N:1   │ name         │                    │
│  │ description  │         │ description  │                    │
│  │ category_id  │         └──────────────┘                    │
│  │ base_price   │                                              │
│  └──────────────┘                                              │
│                                                                 │
│  ┌──────────────┐         ┌──────────────┐                    │
│  │ Dark Stores  │         │  Inventory   │                    │
│  ├──────────────┤         ├──────────────┤                    │
│  │ id (PK)      │────────│ id (PK)      │                    │
│  │ name         │   1:N   │ store_id (FK)│                    │
│  │ latitude     │         │ product_id(FK)│                   │
│  │ longitude    │         │ available_qty│                    │
│  │ radius_km    │         │ reserved_qty │                    │
│  └──────────────┘         └──────────────┘                    │
│                                                                 │
│  ┌──────────────┐         ┌──────────────┐                    │
│  │    Orders    │         │  Order Items │                    │
│  ├──────────────┤         ├──────────────┤                    │
│  │ id (PK)      │────────│ id (PK)      │                    │
│  │ user_id (FK) │   1:N   │ order_id (FK)│                    │
│  │ store_id(FK) │         │ product_id(FK)│                   │
│  │ status       │         │ quantity     │                    │
│  │ total_amount │         │ unit_price   │                    │
│  └──────────────┘         └──────────────┘                    │
│                                                                 │
│  ┌──────────────┐         ┌──────────────┐                    │
│  │   Payments   │         │   Refunds    │                    │
│  ├──────────────┤         ├──────────────┤                    │
│  │ id (PK)      │────────│ id (PK)      │                    │
│  │ order_id (FK)│   1:1   │ payment_id(FK)│                   │
│  │ amount       │         │ amount       │                    │
│  │ method       │         │ status       │                    │
│  │ status       │         └──────────────┘                    │
│  └──────────────┘                                              │
│                                                                 │
│  ┌──────────────┐         ┌──────────────┐                    │
│  │   Partners   │         │ Assignments  │                    │
│  ├──────────────┤         ├──────────────┤                    │
│  │ id (PK)      │────────│ id (PK)      │                    │
│  │ name         │   1:N   │ order_id (FK)│                    │
│  │ phone        │         │ partner_id(FK)│                   │
│  │ vehicle_type │         │ status       │                    │
│  │ is_available │         │ assigned_at │                    │
│  └──────────────┘         └──────────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Database Tables

### Users Table

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL DEFAULT 'CUSTOMER', -- CUSTOMER, ADMIN, STORE_STAFF, PARTNER
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_users_role ON users(role);
```

### Addresses Table

```sql
CREATE TABLE addresses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    street TEXT NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    pincode VARCHAR(10) NOT NULL,
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    is_default BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_addresses_user_id ON addresses(user_id);
CREATE INDEX idx_addresses_location ON addresses(latitude, longitude);
```

### Categories Table

```sql
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    parent_category_id UUID REFERENCES categories(id),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_categories_parent ON categories(parent_category_id);
```

### Products Table

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category_id UUID NOT NULL REFERENCES categories(id),
    base_price DECIMAL(10, 2) NOT NULL,
    images TEXT[], -- Array of image URLs
    nutritional_info JSONB, -- JSON for nutritional information
    status VARCHAR(20) DEFAULT 'ACTIVE', -- ACTIVE, INACTIVE, DISCONTINUED
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_products_name ON products(name);
```

### Dark Stores Table

```sql
CREATE TABLE dark_stores (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    address TEXT NOT NULL,
    city VARCHAR(100) NOT NULL,
    pincode VARCHAR(10) NOT NULL,
    delivery_radius_km DECIMAL(5, 2) NOT NULL DEFAULT 3.0,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_stores_location ON dark_stores(latitude, longitude);
CREATE INDEX idx_stores_city ON dark_stores(city);
CREATE INDEX idx_stores_active ON dark_stores(is_active);
```

### Store Capacity Table

```sql
CREATE TABLE store_capacity (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    store_id UUID NOT NULL UNIQUE REFERENCES dark_stores(id) ON DELETE CASCADE,
    current_pending_orders INT DEFAULT 0,
    max_capacity INT NOT NULL DEFAULT 100,
    average_fulfillment_time_minutes INT DEFAULT 15,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_store_capacity_store_id ON store_capacity(store_id);
```

### Inventory Table

```sql
CREATE TABLE inventory (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    store_id UUID NOT NULL REFERENCES dark_stores(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    available_quantity INT NOT NULL DEFAULT 0,
    reserved_quantity INT NOT NULL DEFAULT 0,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(store_id, product_id)
);

CREATE INDEX idx_inventory_store_product ON inventory(store_id, product_id);
CREATE INDEX idx_inventory_store ON inventory(store_id);
CREATE INDEX idx_inventory_product ON inventory(product_id);
CREATE INDEX idx_inventory_available ON inventory(available_quantity);
```

### Inventory Reservations Table

```sql
CREATE TABLE inventory_reservations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL,
    store_id UUID NOT NULL REFERENCES dark_stores(id),
    product_id UUID NOT NULL REFERENCES products(id),
    quantity INT NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    status VARCHAR(20) DEFAULT 'RESERVED', -- RESERVED, RELEASED, EXPIRED
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_reservations_order ON inventory_reservations(order_id);
CREATE INDEX idx_reservations_expires ON inventory_reservations(expires_at);
CREATE INDEX idx_reservations_status ON inventory_reservations(status);
```

### Inventory History Table

```sql
CREATE TABLE inventory_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    store_id UUID NOT NULL REFERENCES dark_stores(id),
    product_id UUID NOT NULL REFERENCES products(id),
    change_type VARCHAR(20) NOT NULL, -- IN, OUT, ADJUSTMENT, RESERVED, RELEASED
    quantity INT NOT NULL,
    previous_quantity INT,
    new_quantity INT,
    reason TEXT,
    order_id UUID,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_inventory_history_store_product ON inventory_history(store_id, product_id);
CREATE INDEX idx_inventory_history_created ON inventory_history(created_at);
CREATE INDEX idx_inventory_history_order ON inventory_history(order_id);
```

### Orders Table

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    store_id UUID REFERENCES dark_stores(id),
    delivery_partner_id UUID REFERENCES delivery_partners(id),
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING', -- PENDING, CONFIRMED, ASSIGNED_TO_STORE, PICKING, PACKED, ASSIGNED_TO_PARTNER, PICKED_UP, IN_TRANSIT, DELIVERED, CANCELLED, REFUNDED
    total_amount DECIMAL(10, 2) NOT NULL,
    delivery_fee DECIMAL(10, 2) DEFAULT 0,
    delivery_address TEXT NOT NULL,
    delivery_latitude DECIMAL(10, 8),
    delivery_longitude DECIMAL(11, 8),
    sla_minutes INT DEFAULT 20,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    confirmed_at TIMESTAMP,
    delivered_at TIMESTAMP,
    cancelled_at TIMESTAMP
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_store_id ON orders(store_id);
CREATE INDEX idx_orders_partner_id ON orders(delivery_partner_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created ON orders(created_at);
CREATE INDEX idx_orders_delivery_location ON orders(delivery_latitude, delivery_longitude);
```

### Order Items Table

```sql
CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id),
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    total_price DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
```

### Order Status History Table

```sql
CREATE TABLE order_status_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    status VARCHAR(20) NOT NULL,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    changed_by VARCHAR(100), -- user_id or system
    notes TEXT
);

CREATE INDEX idx_order_status_history_order ON order_status_history(order_id);
CREATE INDEX idx_order_status_history_changed ON order_status_history(changed_at);
```

### Payments Table

```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id),
    payment_method VARCHAR(20) NOT NULL, -- UPI, CREDIT_CARD, DEBIT_CARD, WALLET, COD
    amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING', -- PENDING, PROCESSING, SUCCESS, FAILED, REFUNDED
    transaction_id VARCHAR(255),
    gateway_response JSONB, -- Store gateway response
    processed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_payments_order_id ON payments(order_id);
CREATE INDEX idx_payments_transaction_id ON payments(transaction_id);
CREATE INDEX idx_payments_status ON payments(status);
CREATE INDEX idx_payments_created ON payments(created_at);
```

### Refunds Table

```sql
CREATE TABLE refunds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    payment_id UUID NOT NULL REFERENCES payments(id),
    order_id UUID NOT NULL REFERENCES orders(id),
    amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING', -- PENDING, PROCESSING, SUCCESS, FAILED
    refund_reason TEXT,
    transaction_id VARCHAR(255),
    processed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_refunds_payment_id ON refunds(payment_id);
CREATE INDEX idx_refunds_order_id ON refunds(order_id);
CREATE INDEX idx_refunds_status ON refunds(status);
```

### Delivery Partners Table

```sql
CREATE TABLE delivery_partners (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    email VARCHAR(255),
    vehicle_type VARCHAR(50), -- BIKE, SCOOTER, CAR
    current_latitude DECIMAL(10, 8),
    current_longitude DECIMAL(11, 8),
    is_available BOOLEAN DEFAULT true,
    current_orders_count INT DEFAULT 0,
    max_concurrent_orders INT DEFAULT 1,
    rating DECIMAL(3, 2) DEFAULT 0.0,
    total_deliveries INT DEFAULT 0,
    is_verified BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_partners_phone ON delivery_partners(phone);
CREATE INDEX idx_partners_location ON delivery_partners(current_latitude, current_longitude);
CREATE INDEX idx_partners_available ON delivery_partners(is_available);
CREATE INDEX idx_partners_rating ON delivery_partners(rating);
```

### Delivery Assignments Table

```sql
CREATE TABLE delivery_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL UNIQUE REFERENCES orders(id),
    partner_id UUID NOT NULL REFERENCES delivery_partners(id),
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    picked_up_at TIMESTAMP,
    delivered_at TIMESTAMP,
    status VARCHAR(20) DEFAULT 'ASSIGNED', -- ASSIGNED, PICKED_UP, IN_TRANSIT, DELIVERED, CANCELLED
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_assignments_order_id ON delivery_assignments(order_id);
CREATE INDEX idx_assignments_partner_id ON delivery_assignments(partner_id);
CREATE INDEX idx_assignments_status ON delivery_assignments(status);
```

### Partner Location History Table

```sql
CREATE TABLE partner_location_history (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    partner_id UUID NOT NULL REFERENCES delivery_partners(id) ON DELETE CASCADE,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_partner_location_history_partner ON partner_location_history(partner_id);
CREATE INDEX idx_partner_location_history_recorded ON partner_location_history(recorded_at);
```

### Store Prices Table (for dynamic pricing)

```sql
CREATE TABLE store_prices (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    store_id UUID NOT NULL REFERENCES dark_stores(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    price DECIMAL(10, 2) NOT NULL,
    effective_from TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    effective_to TIMESTAMP,
    is_active BOOLEAN DEFAULT true,
    UNIQUE(store_id, product_id, effective_from)
);

CREATE INDEX idx_store_prices_store_product ON store_prices(store_id, product_id);
CREATE INDEX idx_store_prices_effective ON store_prices(effective_from, effective_to);
```

---

## Indexes

### Primary Indexes
- All tables have primary key indexes on `id` column

### Foreign Key Indexes
- All foreign key columns are indexed for join performance

### Composite Indexes
- `inventory(store_id, product_id)` - Unique constraint for inventory lookup
- `inventory_reservations(order_id, status)` - For order reservation queries
- `orders(user_id, status)` - For user order history
- `orders(store_id, status)` - For store order queue
- `delivery_assignments(partner_id, status)` - For partner assignments

### Performance Indexes
- `orders(created_at)` - For time-based queries
- `inventory_history(created_at)` - For audit queries
- `partner_location_history(recorded_at)` - For location tracking

---

## Relationships

### One-to-Many Relationships
- `users` → `addresses` (1 user can have many addresses)
- `users` → `orders` (1 user can have many orders)
- `categories` → `products` (1 category can have many products)
- `dark_stores` → `inventory` (1 store can have many inventory items)
- `dark_stores` → `orders` (1 store can fulfill many orders)
- `products` → `order_items` (1 product can be in many order items)
- `orders` → `order_items` (1 order can have many items)
- `orders` → `order_status_history` (1 order can have many status changes)
- `delivery_partners` → `delivery_assignments` (1 partner can have many assignments)
- `payments` → `refunds` (1 payment can have many refunds)

### One-to-One Relationships
- `dark_stores` → `store_capacity` (1 store has 1 capacity record)
- `orders` → `delivery_assignments` (1 order has 1 assignment)
- `payments` → `orders` (1 order has 1 payment, but can have multiple attempts)

### Many-to-Many Relationships
- `dark_stores` ↔ `products` (through `inventory` table)
- `products` ↔ `categories` (through `category_id` foreign key)

---

## Data Types and Constraints

### UUID
- Used for all primary keys and foreign keys
- Generated using `gen_random_uuid()` function

### DECIMAL
- Used for monetary values (prices, amounts)
- Format: `DECIMAL(10, 2)` for amounts up to ₹99,99,99,999.99
- Format: `DECIMAL(10, 8)` for latitude
- Format: `DECIMAL(11, 8)` for longitude

### TIMESTAMP
- Used for all date/time fields
- Default: `CURRENT_TIMESTAMP`
- Updated automatically using triggers

### VARCHAR
- Used for text fields with known max length
- Indexed for search performance

### TEXT
- Used for longer text fields (descriptions, addresses)
- Not indexed by default

### JSONB
- Used for flexible data structures (gateway responses, nutritional info)
- Indexed using GIN indexes for query performance

### BOOLEAN
- Used for flags (is_active, is_available, is_default)
- Indexed for filtering

### Constraints
- **NOT NULL**: Required fields
- **UNIQUE**: Unique constraints on email, phone, composite keys
- **CHECK**: Validation constraints (e.g., quantity > 0, rating between 0-5)
- **FOREIGN KEY**: Referential integrity
- **DEFAULT**: Default values for optional fields

---

**Last Updated**: January 2025
**Version**: 1.0
**Status**: Design Complete, Implementation Pending
