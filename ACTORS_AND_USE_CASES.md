# FlashKart Quick Commerce - Actors and Use Cases

## Table of Contents
1. [Actors](#actors)
2. [Use Case Diagram](#use-case-diagram)
3. [Detailed Use Cases](#detailed-use-cases)
4. [Use Case Relationships](#use-case-relationships)

---

## Actors

### Primary Actors

#### 1. Customer
**Description**: End user who places orders for groceries, daily essentials, and snacks
**Characteristics**:
- Uses mobile app or web application
- Has delivery address
- Makes payments
- Tracks orders

**Responsibilities**:
- Browse products
- Place orders
- Make payments
- Track orders
- Manage account

---

#### 2. Dark Store Staff
**Description**: Staff working at dark stores who fulfill orders
**Characteristics**:
- Uses tablet/web interface
- Has access to inventory
- Picks and packs orders
- Updates order status

**Responsibilities**:
- View order queue
- Pick products
- Pack orders
- Update inventory
- Mark orders as ready

---

#### 3. Delivery Partner
**Description**: Independent delivery partner who delivers orders to customers
**Characteristics**:
- Uses mobile app
- Has vehicle (bike/scooter)
- Tracks location
- Manages deliveries

**Responsibilities**:
- Accept order assignments
- Pick up orders from stores
- Deliver orders to customers
- Update delivery status
- Track earnings

---

### Secondary Actors

#### 4. System Administrator
**Description**: Internal staff managing the platform
**Characteristics**:
- Has admin access
- Manages configurations
- Monitors system health
- Handles escalations

**Responsibilities**:
- Manage dark stores
- Manage delivery partners
- Configure pricing
- Monitor operations
- Handle exceptions

---

#### 5. Payment Gateway
**Description**: External payment processing service
**Characteristics**:
- Processes payments
- Handles refunds
- Provides transaction status

**Responsibilities**:
- Process payment requests
- Return payment status
- Process refunds
- Provide transaction history

---

#### 6. Maps Service
**Description**: External mapping and routing service (e.g., Google Maps)
**Characteristics**:
- Provides geocoding
- Calculates routes
- Provides navigation

**Responsibilities**:
- Geocode addresses
- Calculate distances
- Optimize routes
- Provide navigation

---

#### 7. Notification Service
**Description**: External service for sending notifications (SMS, Push, Email)
**Characteristics**:
- Sends SMS
- Sends push notifications
- Sends emails

**Responsibilities**:
- Send order notifications
- Send status updates
- Send promotional messages

---

## Use Case Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        FlashKart System                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Customer Use Cases                     │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  UC1: Browse Products                                    │  │
│  │  UC2: Search Products                                    │  │
│  │  UC3: View Product Details                               │  │
│  │  UC4: Add to Cart                                        │  │
│  │  UC5: View Cart                                          │  │
│  │  UC6: Place Order                                        │  │
│  │  UC7: Make Payment                                       │  │
│  │  UC8: Track Order                                         │  │
│  │  UC9: Cancel Order                                        │  │
│  │  UC10: View Order History                                 │  │
│  │  UC11: Manage Addresses                                   │  │
│  │  UC12: Rate Order                                         │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Dark Store Staff Use Cases                   │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  UC13: View Order Queue                                  │  │
│  │  UC14: Pick Products                                      │  │
│  │  UC15: Pack Order                                         │  │
│  │  UC16: Update Inventory                                   │  │
│  │  UC17: Mark Order Ready                                   │  │
│  │  UC18: Scan Barcode                                       │  │
│  │  UC19: View Inventory                                      │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │            Delivery Partner Use Cases                     │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  UC20: Accept Order Assignment                            │  │
│  │  UC21: Navigate to Store                                  │  │
│  │  UC22: Pick Up Order                                      │  │
│  │  UC23: Navigate to Customer                               │  │
│  │  UC24: Deliver Order                                      │  │
│  │  UC25: Update Location                                    │  │
│  │  UC26: View Earnings                                      │  │
│  │  UC27: Update Availability                                │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │          System Administrator Use Cases                  │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │  UC28: Manage Dark Stores                                 │  │
│  │  UC29: Manage Delivery Partners                           │  │
│  │  UC30: Configure Pricing                                  │  │
│  │  UC31: View Analytics                                     │  │
│  │  UC32: Handle Exceptions                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

         Customer ──────┐
                        │
         Dark Store Staff ──────┐
                                │
         Delivery Partner ──────┼─────► FlashKart System
                                │
         System Admin ───────────┘
                                │
         Payment Gateway ───────┘
         Maps Service ───────────┘
         Notification Service ───┘
```

---

## Detailed Use Cases

### Customer Use Cases

#### UC1: Browse Products
**Actor**: Customer
**Preconditions**: Customer is logged in, location is set
**Main Flow**:
1. Customer opens app/web
2. System detects location
3. System shows products from nearby stores
4. Customer browses products by category
5. System displays products with availability and pricing

**Postconditions**: Customer can see products available for delivery

---

#### UC2: Search Products
**Actor**: Customer
**Preconditions**: Customer is logged in, location is set
**Main Flow**:
1. Customer enters search query
2. System searches products matching query
3. System filters by location and availability
4. System displays search results
5. Customer can filter results (category, price, etc.)

**Postconditions**: Customer sees relevant products

---

#### UC3: View Product Details
**Actor**: Customer
**Preconditions**: Customer is browsing products
**Main Flow**:
1. Customer clicks on a product
2. System displays product details (images, description, price, availability)
3. Customer can see nutritional info, reviews
4. Customer can add to cart

**Postconditions**: Customer has product details

---

#### UC4: Add to Cart
**Actor**: Customer
**Preconditions**: Customer is viewing product details
**Main Flow**:
1. Customer selects quantity
2. Customer clicks "Add to Cart"
3. System checks inventory availability
4. System adds product to cart
5. System shows cart count

**Postconditions**: Product added to cart

**Alternative Flow**:
- 3a. If inventory unavailable, system shows error message

---

#### UC5: View Cart
**Actor**: Customer
**Preconditions**: Customer has items in cart
**Main Flow**:
1. Customer opens cart
2. System displays cart items with prices
3. Customer can update quantities
4. Customer can remove items
5. System calculates total amount

**Postconditions**: Customer sees cart contents

---

#### UC6: Place Order
**Actor**: Customer
**Preconditions**: Customer has items in cart, delivery address is set
**Main Flow**:
1. Customer clicks "Checkout"
2. System validates cart and inventory
3. System assigns order to optimal dark store
4. System reserves inventory
5. System creates order
6. System redirects to payment

**Postconditions**: Order created, inventory reserved

**Alternative Flow**:
- 2a. If inventory unavailable, system removes unavailable items
- 3a. If no store available, system shows error message

---

#### UC7: Make Payment
**Actor**: Customer, Payment Gateway
**Preconditions**: Order is created
**Main Flow**:
1. Customer selects payment method
2. Customer enters payment details
3. System sends payment request to Payment Gateway
4. Payment Gateway processes payment
5. Payment Gateway returns status
6. If successful, system confirms order
7. System sends confirmation notification

**Postconditions**: Payment processed, order confirmed

**Alternative Flow**:
- 5a. If payment fails, system shows error, allows retry

---

#### UC8: Track Order
**Actor**: Customer
**Preconditions**: Customer has placed an order
**Main Flow**:
1. Customer opens order tracking
2. System fetches order status
3. System displays current status (PENDING, CONFIRMED, PICKING, PACKED, IN_TRANSIT, DELIVERED)
4. System shows ETA
5. System shows delivery partner location (if assigned)
6. System updates status in real-time

**Postconditions**: Customer sees order status

---

#### UC9: Cancel Order
**Actor**: Customer
**Preconditions**: Order is placed and not yet delivered
**Main Flow**:
1. Customer requests cancellation
2. System checks order status
3. If cancellable, system cancels order
4. System releases reserved inventory
5. System processes refund (if payment made)
6. System sends cancellation notification

**Postconditions**: Order cancelled, inventory released, refund processed

**Alternative Flow**:
- 3a. If order cannot be cancelled (already in transit), system shows error

---

#### UC10: View Order History
**Actor**: Customer
**Preconditions**: Customer is logged in
**Main Flow**:
1. Customer opens order history
2. System fetches past orders
3. System displays orders with status, date, amount
4. Customer can filter by date, status
5. Customer can view order details

**Postconditions**: Customer sees order history

---

#### UC11: Manage Addresses
**Actor**: Customer
**Preconditions**: Customer is logged in
**Main Flow**:
1. Customer opens address management
2. System displays saved addresses
3. Customer can add new address
4. Customer can edit address
5. Customer can delete address
6. Customer can set default address

**Postconditions**: Addresses updated

---

#### UC12: Rate Order
**Actor**: Customer
**Preconditions**: Order is delivered
**Main Flow**:
1. Customer receives rating prompt
2. Customer rates order (1-5 stars)
3. Customer provides feedback (optional)
4. System saves rating
5. System updates product/store ratings

**Postconditions**: Rating saved

---

### Dark Store Staff Use Cases

#### UC13: View Order Queue
**Actor**: Dark Store Staff
**Preconditions**: Staff is logged in
**Main Flow**:
1. Staff opens order queue
2. System displays pending orders
3. System shows order priority (SLA-based)
4. Staff can filter by status
5. Staff can select order to fulfill

**Postconditions**: Staff sees order queue

---

#### UC14: Pick Products
**Actor**: Dark Store Staff
**Preconditions**: Order is assigned to store
**Main Flow**:
1. Staff selects order
2. System displays order items
3. Staff picks products from shelves
4. Staff scans barcode to verify
5. System updates inventory
6. System marks item as picked

**Postconditions**: Products picked, inventory updated

---

#### UC15: Pack Order
**Actor**: Dark Store Staff
**Preconditions**: All products are picked
**Main Flow**:
1. Staff packs products in bag
2. Staff scans order barcode
3. System verifies all items packed
4. System marks order as packed
5. System updates order status

**Postconditions**: Order packed, status updated

---

#### UC16: Update Inventory
**Actor**: Dark Store Staff
**Preconditions**: Staff is logged in
**Main Flow**:
1. Staff scans product barcode
2. Staff enters quantity change
3. Staff selects reason (restock, damaged, etc.)
4. System updates inventory
5. System syncs with central inventory

**Postconditions**: Inventory updated

---

#### UC17: Mark Order Ready
**Actor**: Dark Store Staff
**Preconditions**: Order is packed
**Main Flow**:
1. Staff marks order as ready
2. System updates order status
3. System assigns delivery partner
4. System sends notification to delivery partner
5. System sends notification to customer

**Postconditions**: Order ready, delivery partner assigned

---

#### UC18: Scan Barcode
**Actor**: Dark Store Staff
**Preconditions**: Staff is logged in
**Main Flow**:
1. Staff scans barcode
2. System identifies product
3. System displays product information
4. System allows inventory update or order fulfillment

**Postconditions**: Product identified

---

#### UC19: View Inventory
**Actor**: Dark Store Staff
**Preconditions**: Staff is logged in
**Main Flow**:
1. Staff opens inventory view
2. System displays current inventory
3. Staff can search products
4. Staff can filter by low stock
5. Staff can see inventory history

**Postconditions**: Staff sees inventory status

---

### Delivery Partner Use Cases

#### UC20: Accept Order Assignment
**Actor**: Delivery Partner
**Preconditions**: Partner is available, order is ready
**Main Flow**:
1. Partner receives order assignment notification
2. Partner views order details
3. Partner accepts assignment
4. System updates order status
5. System provides navigation to store

**Postconditions**: Order assigned to partner

**Alternative Flow**:
- 3a. If partner rejects, system assigns to another partner

---

#### UC21: Navigate to Store
**Actor**: Delivery Partner, Maps Service
**Preconditions**: Order is assigned
**Main Flow**:
1. Partner clicks "Navigate to Store"
2. System gets store location
3. System calls Maps Service for route
4. Maps Service returns route
5. System displays navigation

**Postconditions**: Partner navigates to store

---

#### UC22: Pick Up Order
**Actor**: Delivery Partner
**Preconditions**: Partner is at store
**Main Flow**:
1. Partner arrives at store
2. Partner scans order QR code
3. System verifies order
4. Partner receives order bag
5. Partner confirms pickup
6. System updates order status
7. System provides navigation to customer

**Postconditions**: Order picked up, status updated

---

#### UC23: Navigate to Customer
**Actor**: Delivery Partner, Maps Service
**Preconditions**: Order is picked up
**Main Flow**:
1. Partner clicks "Navigate to Customer"
2. System gets customer location
3. System calls Maps Service for route
4. Maps Service returns route
5. System displays navigation
6. System updates ETA

**Postconditions**: Partner navigates to customer

---

#### UC24: Deliver Order
**Actor**: Delivery Partner
**Preconditions**: Partner is at customer location
**Main Flow**:
1. Partner arrives at customer location
2. Partner calls customer (if needed)
3. Partner hands over order
4. Partner confirms delivery
5. System updates order status to DELIVERED
6. System processes payment (if COD)
7. System sends delivery notification
8. System updates partner earnings

**Postconditions**: Order delivered, status updated

---

#### UC25: Update Location
**Actor**: Delivery Partner
**Preconditions**: Partner app is running
**Main Flow**:
1. System tracks partner location (GPS)
2. System sends location updates periodically
3. System updates partner location in database
4. System uses location for order assignment

**Postconditions**: Partner location updated

---

#### UC26: View Earnings
**Actor**: Delivery Partner
**Preconditions**: Partner is logged in
**Main Flow**:
1. Partner opens earnings view
2. System displays earnings summary
3. Partner can see daily, weekly, monthly earnings
4. Partner can see per-delivery earnings
5. Partner can see payment history

**Postconditions**: Partner sees earnings

---

#### UC27: Update Availability
**Actor**: Delivery Partner
**Preconditions**: Partner is logged in
**Main Flow**:
1. Partner toggles availability status
2. System updates partner availability
3. If available, system includes partner in assignment pool
4. If unavailable, system excludes partner from assignments

**Postconditions**: Availability updated

---

### System Administrator Use Cases

#### UC28: Manage Dark Stores
**Actor**: System Administrator
**Preconditions**: Admin is logged in
**Main Flow**:
1. Admin opens dark store management
2. Admin can add new store (location, capacity, inventory)
3. Admin can edit store details
4. Admin can deactivate store
5. Admin can view store performance

**Postconditions**: Dark stores managed

---

#### UC29: Manage Delivery Partners
**Actor**: System Administrator
**Preconditions**: Admin is logged in
**Main Flow**:
1. Admin opens partner management
2. Admin can onboard new partner
3. Admin can verify partner documents
4. Admin can suspend/activate partner
5. Admin can view partner performance

**Postconditions**: Delivery partners managed

---

#### UC30: Configure Pricing
**Actor**: System Administrator
**Preconditions**: Admin is logged in
**Main Flow**:
1. Admin opens pricing configuration
2. Admin can set base prices
3. Admin can configure surge pricing rules
4. Admin can set promotional pricing
5. Admin can configure delivery fees

**Postconditions**: Pricing configured

---

#### UC31: View Analytics
**Actor**: System Administrator
**Preconditions**: Admin is logged in
**Main Flow**:
1. Admin opens analytics dashboard
2. System displays key metrics (orders, revenue, SLA compliance)
3. Admin can filter by date, city, store
4. Admin can export reports
5. Admin can view trends

**Postconditions**: Admin sees analytics

---

#### UC32: Handle Exceptions
**Actor**: System Administrator
**Preconditions**: Exception occurs (order failed, delivery issue)
**Main Flow**:
1. System flags exception
2. Admin receives notification
3. Admin reviews exception details
4. Admin takes action (refund, reassign, etc.)
5. Admin resolves exception
6. System updates status

**Postconditions**: Exception resolved

---

## Use Case Relationships

### Include Relationships
- **UC6 (Place Order)** includes **UC4 (Add to Cart)**
- **UC6 (Place Order)** includes **UC7 (Make Payment)**
- **UC8 (Track Order)** includes **UC25 (Update Location)**
- **UC14 (Pick Products)** includes **UC18 (Scan Barcode)**
- **UC22 (Pick Up Order)** includes **UC21 (Navigate to Store)**
- **UC24 (Deliver Order)** includes **UC23 (Navigate to Customer)**

### Extend Relationships
- **UC6 (Place Order)** extends **UC9 (Cancel Order)** if cancellation needed
- **UC24 (Deliver Order)** extends **UC12 (Rate Order)** after delivery

### Generalization Relationships
- **UC1 (Browse Products)** and **UC2 (Search Products)** are specialized forms of product discovery
- **UC21 (Navigate to Store)** and **UC23 (Navigate to Customer)** are specialized forms of navigation

---

**Last Updated**: January 2025
**Version**: 1.0
**Status**: Design Complete, Implementation Pending
