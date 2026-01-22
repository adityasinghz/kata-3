# FlashKart Quick Commerce - Class Diagram

## Table of Contents
1. [Core Domain Classes](#core-domain-classes)
2. [Service Layer Classes](#service-layer-classes)
3. [Complete Class Diagram](#complete-class-diagram)
4. [Class Relationships](#class-relationships)

---

## Core Domain Classes

### Order Management Domain

```java
class Order {
    - UUID id
    - UUID userId
    - UUID storeId
    - UUID deliveryPartnerId
    - OrderStatus status
    - Address deliveryAddress
    - BigDecimal totalAmount
    - int slaMinutes
    - DateTime createdAt
    - DateTime updatedAt
    
    + createOrder(userId, items, address): Order
    + updateStatus(newStatus): void
    + cancelOrder(reason): void
    + calculateTotal(): BigDecimal
    + validateOrder(): boolean
    + getRemainingSLA(): int
}

class OrderItem {
    - UUID id
    - UUID orderId
    - UUID productId
    - int quantity
    - BigDecimal unitPrice
    - BigDecimal totalPrice
    
    + calculateTotal(): BigDecimal
}

class OrderStatus {
    <<enumeration>>
    PENDING
    CONFIRMED
    ASSIGNED_TO_STORE
    PICKING
    PACKED
    ASSIGNED_TO_PARTNER
    PICKED_UP
    IN_TRANSIT
    DELIVERED
    CANCELLED
    REFUNDED
}

class Address {
    - String street
    - String city
    - String state
    - String pincode
    - BigDecimal latitude
    - BigDecimal longitude
    
    + validate(): boolean
    + getFullAddress(): String
}
```

### Product & Inventory Domain

```java
class Product {
    - UUID id
    - String name
    - String description
    - String category
    - List<String> images
    - BigDecimal basePrice
    - Map<UUID, BigDecimal> storePrices
    - ProductStatus status
    
    + getPrice(storeId): BigDecimal
    + isAvailable(storeId): boolean
}

class Inventory {
    - UUID id
    - UUID storeId
    - UUID productId
    - int availableQuantity
    - int reservedQuantity
    - DateTime lastUpdated
    
    + getAvailableQuantity(): int
    + reserve(quantity, orderId): boolean
    + release(orderId): void
    + updateQuantity(quantity): void
    + checkAvailability(quantity): boolean
}

class InventoryReservation {
    - UUID id
    - UUID orderId
    - UUID storeId
    - UUID productId
    - int quantity
    - DateTime expiresAt
    - ReservationStatus status
    
    + isValid(): boolean
    + expire(): void
}
```

### Store Domain

```java
class DarkStore {
    - UUID id
    - String name
    - Address location
    - BigDecimal deliveryRadiusKm
    - StoreStatus status
    - StoreCapacity capacity
    - DateTime createdAt
    
    + isWithinRadius(customerLocation): boolean
    + getCapacityStatus(): CapacityStatus
    + updateCapacity(): void
}

class StoreCapacity {
    - UUID storeId
    - int currentPendingOrders
    - int maxCapacity
    - int averageFulfillmentTimeMinutes
    - DateTime lastUpdated
    
    + canAcceptOrder(): boolean
    + calculateFulfillmentTime(items): int
    + updatePendingOrders(count): void
}
```

### Delivery Domain

```java
class DeliveryPartner {
    - UUID id
    - String name
    - String phone
    - VehicleType vehicleType
    - Location currentLocation
    - boolean isAvailable
    - int currentOrdersCount
    - int maxConcurrentOrders
    - BigDecimal rating
    - int totalDeliveries
    
    + updateLocation(location): void
    + setAvailability(available): void
    + canAcceptOrder(): boolean
    + calculateScore(order): BigDecimal
}

class DeliveryAssignment {
    - UUID id
    - UUID orderId
    - UUID partnerId
    - DateTime assignedAt
    - DateTime pickedUpAt
    - DateTime deliveredAt
    - DeliveryStatus status
    
    + assign(partner): void
    + confirmPickup(): void
    + confirmDelivery(): void
}

class Location {
    - BigDecimal latitude
    - BigDecimal longitude
    - DateTime timestamp
    
    + calculateDistance(other): BigDecimal
    + isValid(): boolean
}

class Route {
    - Location start
    - Location end
    - List<Location> waypoints
    - BigDecimal distanceKm
    - int estimatedTimeMinutes
    
    + optimize(): Route
    + calculateETA(): int
}
```

### Payment Domain

```java
class Payment {
    - UUID id
    - UUID orderId
    - PaymentMethod method
    - BigDecimal amount
    - PaymentStatus status
    - String transactionId
    - DateTime processedAt
    
    + process(): PaymentResult
    + verify(): boolean
    + refund(): Refund
}

class Refund {
    - UUID id
    - UUID paymentId
    - UUID orderId
    - BigDecimal amount
    - RefundStatus status
    - DateTime processedAt
    
    + process(): RefundResult
    + verify(): boolean
}

enum PaymentMethod {
    UPI
    CREDIT_CARD
    DEBIT_CARD
    WALLET
    COD
}

enum PaymentStatus {
    PENDING
    PROCESSING
    SUCCESS
    FAILED
    REFUNDED
}
```

---

## Service Layer Classes

### Order Service

```java
class OrderService {
    - OrderRepository orderRepository
    - InventoryService inventoryService
    - PaymentService paymentService
    - NotificationService notificationService
    
    + createOrder(userId, items, address): Order
    + getOrder(orderId): Order
    + cancelOrder(orderId, reason): boolean
    + updateOrderStatus(orderId, status): void
    + getOrderHistory(userId): List<Order>
}

class OrderRepository {
    + save(order): Order
    + findById(id): Order
    + findByUserId(userId): List<Order>
    + findByStatus(status): List<Order>
    + update(order): Order
}

class OrderValidator {
    + validateOrder(order): ValidationResult
    + checkInventoryAvailability(order): boolean
    + validateAddress(address): boolean
}

class OrderStateMachine {
    + transition(order, newStatus): boolean
    + getValidTransitions(currentStatus): List<OrderStatus>
    + canTransition(from, to): boolean
}
```

### Inventory Service

```java
class InventoryService {
    - InventoryRepository inventoryRepository
    - InventoryCache cache
    - EventPublisher eventPublisher
    
    + getInventory(storeId, productId): Inventory
    + reserveInventory(orderId, items): boolean
    + releaseInventory(orderId): void
    + updateInventory(storeId, productId, quantity): void
    + checkAvailability(storeId, productId, quantity): boolean
}

class InventoryRepository {
    + save(inventory): Inventory
    + findByStoreAndProduct(storeId, productId): Inventory
    + findByStore(storeId): List<Inventory>
    + update(inventory): Inventory
}

class StockReserver {
    + reserve(storeId, productId, quantity, orderId): Reservation
    + release(reservationId): void
    + checkAvailability(storeId, productId): int
    + validateReservation(reservation): boolean
}

class InventorySyncService {
    + syncInventory(storeId): void
    + publishInventoryUpdate(update): void
    + reconcileInventory(storeId): ReconciliationResult
}
```

### Fulfillment Service

```java
class FulfillmentService {
    - StoreSelector storeSelector
    - SLAValidator slaValidator
    - EventPublisher eventPublisher
    
    + assignStore(order): DarkStore
    + calculateFulfillmentTime(storeId, items): int
    + checkStoreCapacity(storeId): CapacityStatus
    + validateSLA(order): boolean
}

class StoreSelector {
    - LocationService locationService
    - InventoryService inventoryService
    - StoreCapacityService capacityService
    
    + selectBestStore(order): DarkStore
    + rankStores(order): List<DarkStore>
    + calculateScore(store, order): BigDecimal
}

class FulfillmentEngine {
    + assignStore(order): DarkStore
    + reRouteOrder(order, reason): DarkStore
    + optimizeAssignment(orders): Map<Order, DarkStore>
}

class SLAValidator {
    + validateSLA(order): boolean
    + calculateRemainingTime(order): int
    + isAtRisk(order): boolean
    + getSLACompliance(orders): ComplianceMetrics
}
```

### Delivery Service

```java
class DeliveryService {
    - PartnerMatcher partnerMatcher
    - RouteOptimizer routeOptimizer
    - TrackingService trackingService
    
    + assignPartner(order): DeliveryPartner
    + reassignPartner(orderId, reason): DeliveryPartner
    + optimizeRoute(orders): Route
    + trackDelivery(orderId): DeliveryStatus
}

class PartnerMatcher {
    - LocationService locationService
    - PartnerRepository partnerRepository
    
    + findAvailablePartners(location, radius): List<DeliveryPartner>
    + assignPartner(order): DeliveryPartner
    + calculateAssignmentScore(partner, order): BigDecimal
    + rankPartners(partners, order): List<DeliveryPartner>
}

class RouteOptimizer {
    - MapsService mapsService
    
    + optimizeRoute(start, end): Route
    + calculateETA(pickup, delivery): int
    + optimizeMultiStopRoute(orders): Route
    + getOptimalRoute(start, end): Route
}

class AssignmentEngine {
    + assign(order): DeliveryAssignment
    + reassign(orderId, reason): DeliveryAssignment
    + release(orderId): void
    + updateAssignmentStatus(assignmentId, status): void
}
```

### Location Service

```java
class LocationService {
    - LocationRepository locationRepository
    - GeocodingService geocodingService
    - MapsService mapsService
    
    + resolveLocation(coordinates): Location
    + validateLocation(location): boolean
    + getDeliveryAddress(userId): Address
    + findNearestStores(location, radius): List<DarkStore>
    + calculateDistance(location1, location2): BigDecimal
}

class LocationResolver {
    + resolveFromGPS(coordinates): Location
    + resolveFromAddress(address): Location
    + validateCoordinates(lat, lng): boolean
}

class StoreSelector {
    + findStoresInRadius(location, radius): List<DarkStore>
    + selectOptimalStore(location, items): DarkStore
    + calculateDistance(location1, location2): BigDecimal
}

class GeofenceManager {
    + isWithinGeofence(location, storeId): boolean
    + getGeofenceRadius(storeId): BigDecimal
    + validateDeliveryAddress(address, storeId): boolean
}
```

### Payment Service

```java
class PaymentService {
    - PaymentGatewayAdapter gatewayAdapter
    - PaymentRepository paymentRepository
    - RefundService refundService
    
    + processPayment(orderId, method, details): Payment
    + verifyPayment(transactionId): boolean
    + processRefund(paymentId, amount): Refund
    + getPaymentHistory(orderId): List<Payment>
}

class PaymentGatewayAdapter {
    + charge(amount, method, details): PaymentResult
    + refund(transactionId, amount): RefundResult
    + verify(transactionId): PaymentStatus
}

class PaymentGatewayFactory {
    + createGateway(type): PaymentGatewayAdapter
}

class RefundProcessor {
    + processRefund(paymentId, amount): Refund
    + verifyRefund(refundId): boolean
    + getRefundStatus(refundId): RefundStatus
}
```

---

## Complete Class Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Domain Layer                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Order ──────┐                                                  │
│    │         │                                                  │
│    │         └─── OrderItem                                     │
│    │                                                             │
│    ├─── OrderStatus (enum)                                      │
│    │                                                             │
│    └─── Address                                                 │
│                                                                 │
│  Product ────┐                                                  │
│    │         │                                                  │
│    │         └─── Inventory                                     │
│    │                │                                           │
│    │                └─── InventoryReservation                   │
│                                                                 │
│  DarkStore ──┐                                                  │
│    │         │                                                  │
│    │         └─── StoreCapacity                                 │
│                                                                 │
│  DeliveryPartner ──┐                                            │
│    │               │                                            │
│    │               └─── DeliveryAssignment                      │
│    │                      │                                      │
│    │                      └─── Location                         │
│    │                             │                              │
│    │                             └─── Route                     │
│                                                                 │
│  Payment ────┐                                                  │
│    │         │                                                  │
│    │         └─── Refund                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        Service Layer                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  OrderService ──────┐                                           │
│    │                 │                                           │
│    ├─── OrderRepository                                          │
│    ├─── OrderValidator                                           │
│    └─── OrderStateMachine                                        │
│                                                                 │
│  InventoryService ──┐                                           │
│    │                 │                                           │
│    ├─── InventoryRepository                                      │
│    ├─── StockReserver                                            │
│    └─── InventorySyncService                                     │
│                                                                 │
│  FulfillmentService ─┐                                           │
│    │                 │                                           │
│    ├─── StoreSelector                                            │
│    ├─── FulfillmentEngine                                        │
│    └─── SLAValidator                                             │
│                                                                 │
│  DeliveryService ────┐                                           │
│    │                 │                                           │
│    ├─── PartnerMatcher                                           │
│    ├─── RouteOptimizer                                           │
│    └─── AssignmentEngine                                         │
│                                                                 │
│  LocationService ────┐                                           │
│    │                 │                                           │
│    ├─── LocationResolver                                         │
│    ├─── StoreSelector                                            │
│    └─── GeofenceManager                                          │
│                                                                 │
│  PaymentService ─────┐                                           │
│    │                 │                                           │
│    ├─── PaymentGatewayAdapter                                    │
│    ├─── PaymentGatewayFactory                                    │
│    └─── RefundProcessor                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        Infrastructure Layer                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Repository Layer                                               │
│  ├─── OrderRepository (implements IOrderRepository)            │
│  ├─── InventoryRepository (implements IInventoryRepository)     │
│  ├─── ProductRepository (implements IProductRepository)         │
│  └─── PartnerRepository (implements IPartnerRepository)        │
│                                                                 │
│  External Services                                              │
│  ├─── MapsService (Google Maps API)                            │
│  ├─── PaymentGateway (Razorpay, Stripe)                        │
│  ├─── NotificationService (SMS, Push, Email)                  │
│  └─── GeocodingService (Address to Coordinates)                │
│                                                                 │
│  Cache Layer                                                    │
│  ├─── InventoryCache (Redis)                                   │
│  ├─── LocationCache (Redis)                                     │
│  └─── ProductCache (Redis)                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Class Relationships

### Inheritance
- `OrderItem` extends `BaseEntity`
- `InventoryReservation` extends `BaseEntity`
- `DeliveryAssignment` extends `BaseEntity`
- `Refund` extends `BaseEntity`

### Composition
- `Order` contains `List<OrderItem>`
- `Order` contains `Address`
- `DarkStore` contains `StoreCapacity`
- `DeliveryAssignment` contains `Location` and `Route`
- `Payment` may contain `Refund`

### Aggregation
- `OrderService` uses `OrderRepository`
- `InventoryService` uses `InventoryRepository`
- `FulfillmentService` uses `StoreSelector`
- `DeliveryService` uses `PartnerMatcher`

### Association
- `Order` → `DarkStore` (assigned to)
- `Order` → `DeliveryPartner` (assigned to)
- `Order` → `Payment` (has payment)
- `Inventory` → `Product` (belongs to)
- `Inventory` → `DarkStore` (belongs to)
- `DeliveryAssignment` → `Order` (assigned to)
- `DeliveryAssignment` → `DeliveryPartner` (assigned to)

### Dependency
- `OrderService` depends on `InventoryService`
- `OrderService` depends on `PaymentService`
- `FulfillmentService` depends on `LocationService`
- `DeliveryService` depends on `MapsService`
- `PaymentService` depends on `PaymentGatewayAdapter`

---

**Last Updated**: January 2025
**Version**: 1.0
**Status**: Design Complete, Implementation Pending
