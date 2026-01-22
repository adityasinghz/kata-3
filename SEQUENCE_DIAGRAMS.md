# FlashKart Quick Commerce - Sequence Diagrams

## Table of Contents
1. [Order Placement Flow](#order-placement-flow)
2. [Order Fulfillment Flow](#order-fulfillment-flow)
3. [Delivery Assignment Flow](#delivery-assignment-flow)
4. [Inventory Update Flow](#inventory-update-flow)
5. [Payment Processing Flow](#payment-processing-flow)
6. [Order Tracking Flow](#order-tracking-flow)

---

## Order Placement Flow

### Sequence Diagram: Complete Order Placement

```
Customer    Customer App    API Gateway    Order Service    Inventory Service    Payment Service    Notification Service
    |            |                |               |                  |                    |                      |
    |--Browse Products-->|            |               |                  |                    |                      |
    |<--Products List----|            |               |                  |                    |                      |
    |            |                |               |                  |                    |                      |
    |--Add to Cart-->|            |               |                  |                    |                      |
    |            |--POST /cart-->|            |               |                  |                    |                      |
    |            |                |--Validate-->|            |               |                  |                      |
    |            |                |               |--Check Inventory-->|                    |                      |
    |            |                |               |<--Available--------|                    |                      |
    |            |<--Cart Updated--|            |               |                  |                    |                      |
    |<--Cart Updated----|            |               |                  |                    |                      |
    |            |                |               |                  |                    |                      |
    |--Checkout-->|            |               |                  |                    |                      |
    |            |--POST /orders-->|            |               |                  |                    |                      |
    |            |                |--Validate-->|            |               |                  |                      |
    |            |                |               |--Reserve Inventory-->|                    |                      |
    |            |                |               |<--Reserved----------|                    |                      |
    |            |                |               |--Create Order------|                    |                      |
    |            |                |               |--Process Payment-->|                    |                      |
    |            |                |               |               |                  |--Charge Payment-->|                      |
    |            |                |               |               |                  |<--Success----------|                      |
    |            |                |               |<--Payment Success--|                    |                      |
    |            |                |               |--Confirm Order----|                    |                      |
    |            |                |               |--Send Notification-->|                      |
    |            |                |               |               |                  |                    |--Send SMS/Push-->|
    |            |<--Order Created--|            |               |                  |                    |                      |
    |<--Order Confirmed--|            |               |                  |                    |                      |
```

### Key Steps:
1. Customer browses products and adds to cart
2. System checks inventory availability
3. Customer proceeds to checkout
4. System reserves inventory
5. System creates order
6. Payment is processed
7. Order is confirmed
8. Notification is sent to customer

---

## Order Fulfillment Flow

### Sequence Diagram: Store Fulfillment Process

```
Order Service    Fulfillment Service    Dark Store System    Dark Store Staff    Inventory Service    Notification Service
      |                  |                      |                    |                      |                      |
      |--Order Created-->|                      |                    |                      |                      |
      |                  |--Assign Store------->|                    |                      |                      |
      |                  |<--Store Assigned-----|                    |                      |                      |
      |                  |--Send Notification-->|                    |                      |                      |
      |                  |                      |                    |                      |                      |--Notify Staff-->|
      |                  |                      |                    |                      |                      |
      |                  |                      |<--View Order Queue--|                    |                      |
      |                  |                      |--Order Details----->|                    |                      |
      |                  |                      |                    |                      |                      |
      |                  |                      |<--Start Picking---->|                    |                      |
      |                  |                      |--Update Status----->|                    |                      |
      |                  |                      |                    |                      |                      |
      |                  |                      |<--Pick Product------|                    |                      |
      |                  |                      |--Scan Barcode------>|                    |                      |
      |                  |                      |                    |--Verify Inventory-->|                      |
      |                  |                      |                    |<--Verified----------|                      |
      |                  |                      |<--Item Picked------>|                    |                      |
      |                  |                      |--Update Inventory-->|                    |                      |
      |                  |                      |                    |--Deduct Stock------->|                      |
      |                  |                      |                    |<--Updated-----------|                      |
      |                  |                      |                    |                      |                      |
      |                  |                      |<--Pack Order--------|                    |                      |
      |                  |                      |--Mark Packed------->|                    |                      |
      |                  |                      |--Update Status----->|                    |                      |
      |                  |                      |                    |                      |                      |
      |                  |                      |<--Mark Ready--------|                    |                      |
      |                  |                      |--Order Ready------->|                    |                      |
      |                  |<--Order Ready--------|                    |                      |                      |
      |--Status Updated--|                      |                    |                      |                      |
      |                  |--Send Notification-->|                    |                      |                      |
      |                  |                      |                    |                      |                      |--Notify Customer-->|
```

### Key Steps:
1. Order is assigned to dark store
2. Store staff views order queue
3. Staff picks products and scans barcodes
4. Inventory is updated
5. Order is packed
6. Order is marked as ready
7. Notification is sent to customer

---

## Delivery Assignment Flow

### Sequence Diagram: Partner Assignment and Delivery

```
Fulfillment Service    Delivery Service    Partner Matcher    Delivery Partner App    Maps Service    Tracking Service
         |                   |                   |                      |                      |                      |
         |--Order Ready----->|                   |                      |                      |                      |
         |                   |--Find Partners---->|                      |                      |                      |
         |                   |                   |--Query Available---->|                      |                      |
         |                   |                   |<--Partners List------|                      |                      |
         |                   |                   |--Calculate Scores-->|                      |                      |
         |                   |                   |--Select Best--------|                      |                      |
         |                   |<--Best Partner----|                      |                      |                      |
         |                   |--Assign Order---->|                      |                      |                      |
         |                   |                   |                      |--Notification------>|                      |
         |                   |                   |                      |                      |                      |
         |                   |                   |                      |<--Accept Assignment--|                      |
         |                   |<--Accepted--------|                      |                      |                      |
         |                   |--Get Route--------|                      |                      |--Get Route--------->|
         |                   |                   |                      |                      |<--Route Data--------|
         |                   |<--Route Data------|                      |                      |                      |
         |                   |--Send Navigation-->|                      |                      |                      |
         |                   |                   |                      |--Show Navigation---->|                      |
         |                   |                   |                      |                      |                      |
         |                   |                   |                      |--Update Location----|                      |
         |                   |                   |                      |--Location Update---->|                      |
         |                   |                   |                      |                      |                      |
         |                   |                   |                      |--Arrive at Store----|                      |
         |                   |                   |                      |--Scan QR Code------->|                      |
         |                   |<--Pickup Confirmed-|                      |                      |                      |
         |                   |--Update Status---->|                      |                      |                      |
         |                   |                   |                      |                      |                      |
         |                   |--Get Customer Route|                      |                      |--Get Route--------->|
         |                   |                   |                      |                      |<--Route Data--------|
         |                   |<--Route Data------|                      |                      |                      |
         |                   |--Send Navigation-->|                      |                      |                      |
         |                   |                   |                      |--Show Navigation---->|                      |
         |                   |                   |                      |                      |                      |
         |                   |                   |                      |--Update Location----|                      |
         |                   |                   |                      |--Location Update---->|                      |
         |                   |                   |                      |                      |                      |
         |                   |                   |                      |--Arrive at Customer-|                      |
         |                   |                   |                      |--Deliver Order------>|                      |
         |                   |<--Delivery Confirmed|                      |                      |                      |
         |                   |--Update Status---->|                      |                      |                      |
         |                   |--Update Earnings-->|                      |                      |                      |
```

### Key Steps:
1. Order is ready for delivery
2. System finds available delivery partners
3. System calculates assignment scores
4. Best partner is selected and assigned
5. Partner accepts and navigates to store
6. Partner picks up order
7. Partner navigates to customer
8. Partner delivers order
9. Order status updated, earnings calculated

---

## Inventory Update Flow

### Sequence Diagram: Real-Time Inventory Sync

```
Dark Store Staff    Dark Store System    Inventory Service    Cache (Redis)    Catalog Service    Customer App
        |                   |                      |                  |                  |                  |
        |--Scan Barcode---->|                      |                  |                  |                  |
        |--Update Quantity->|                      |                  |                  |                  |
        |                   |--Update Inventory-->|                  |                  |                  |
        |                   |                      |--Validate--------|                  |                  |
        |                   |                      |--Update DB------>|                  |                  |
        |                   |                      |--Update Cache--->|                  |                  |
        |                   |                      |                  |--Set Inventory----|                  |
        |                   |                      |--Publish Event-->|                  |                  |
        |                   |                      |                  |                  |                  |
        |                   |                      |                  |                  |--Update Catalog-->|
        |                   |                      |                  |                  |--Notify Apps----->|
        |                   |                      |                  |                  |                  |--Update UI------>|
        |                   |<--Success------------|                  |                  |                  |
        |<--Updated---------|                      |                  |                  |                  |
```

### Key Steps:
1. Store staff scans barcode and updates quantity
2. Inventory service validates and updates database
3. Cache is updated for real-time access
4. Event is published for other services
5. Catalog service updates product availability
6. Customer apps are notified of inventory changes

---

## Payment Processing Flow

### Sequence Diagram: Payment and Refund

```
Customer App    Order Service    Payment Service    Payment Gateway    Refund Service
      |               |                  |                    |                  |
      |--Place Order->|                  |                    |                  |
      |               |--Create Order--->|                    |                  |
      |               |--Process Payment|                    |                  |
      |               |                  |--Charge Payment-->|                  |
      |               |                  |                    |--Process-------->|
      |               |                  |                    |<--Success--------|
      |               |                  |<--Payment Success-|                  |
      |               |--Confirm Order--|                    |                  |
      |<--Order Confirmed|                  |                    |                  |
      |               |                  |                    |                  |
      |               |                  |                    |                  |
      |--Cancel Order->|                  |                    |                  |
      |               |--Cancel Order--->|                    |                  |
      |               |--Process Refund->|                    |                  |
      |               |                  |--Refund Request-->|                  |
      |               |                  |                    |--Process-------->|
      |               |                  |                    |<--Refund Success-|
      |               |                  |<--Refund Success---|                  |
      |               |--Release Inventory|                    |                  |
      |<--Refund Processed|                  |                    |                  |
```

### Key Steps:
1. Customer places order
2. Payment service charges payment gateway
3. Payment gateway processes payment
4. Order is confirmed
5. If cancellation, refund is processed
6. Refund is credited to customer
7. Inventory is released

---

## Order Tracking Flow

### Sequence Diagram: Real-Time Order Tracking

```
Customer App    Tracking Service    Order Service    Delivery Service    Delivery Partner App    Maps Service
      |                 |                 |                  |                      |                      |
      |--Track Order--->|                 |                  |                      |                      |
      |                 |--Get Order Status|                  |                      |                      |
      |                 |                 |--Fetch Status---->|                      |                      |
      |                 |                 |<--Status Data-----|                      |                      |
      |                 |<--Status Data---|                  |                      |                      |
      |                 |--Get ETA--------|                  |                      |                      |
      |                 |                 |--Calculate ETA--->|                      |                      |
      |                 |                 |                  |--Get Partner Location|                      |
      |                 |                 |                  |                      |--Get Location------>|
      |                 |                 |                  |                      |<--Location Data-----|
      |                 |                 |                  |<--Location Data------|                      |
      |                 |                 |--Calculate ETA----|                      |                      |
      |                 |<--ETA Data------|                  |                      |                      |
      |<--Status & ETA--|                 |                  |                      |                      |
      |                 |                 |                  |                      |                      |
      |                 |--Subscribe------|                 |                  |                      |                      |
      |                 |                 |                  |                      |                      |
      |                 |                 |                  |--Location Update---->|                      |
      |                 |                 |                  |                      |--Update Location----|
      |                 |                 |                  |--Status Update------>|                      |
      |                 |                 |--Status Changed--|                      |                      |
      |                 |--Push Update---->|                 |                  |                      |                      |
      |<--Status Update-|                 |                  |                      |                      |
```

### Key Steps:
1. Customer requests order tracking
2. System fetches current order status
3. System calculates ETA based on partner location
4. Customer subscribes to real-time updates
5. System pushes updates when status/location changes
6. Customer sees live tracking

---

## Additional Sequence Diagrams

### Order Cancellation Flow

```
Customer    Customer App    Order Service    Inventory Service    Payment Service    Refund Service    Notification Service
    |            |                |                  |                    |                  |                      |
    |--Cancel-->|            |                  |                    |                  |                      |
    |            |--POST /orders/{id}/cancel-->|                  |                    |                  |                      |
    |            |                |--Validate Cancellation-->|                    |                  |                      |
    |            |                |<--Cancellable------------|                    |                  |                      |
    |            |                |--Cancel Order----------->|                    |                  |                      |
    |            |                |--Release Inventory------>|                    |                  |                      |
    |            |                |                  |--Release Reserved-->|                  |                      |
    |            |                |                  |<--Released----------|                  |                      |
    |            |                |--Process Refund--------->|                  |                      |
    |            |                |                    |--Refund Request---->|                  |                      |
    |            |                |                    |                  |--Process Refund-->|                      |
    |            |                |                    |                  |<--Refund Success-|                      |
    |            |                |                    |<--Refund Success---|                  |                      |
    |            |                |--Send Notification------>|                      |                      |
    |            |                |                  |                    |                  |                      |--Send Notification-->|
    |            |<--Cancelled-----|                  |                    |                  |                      |
    |<--Cancelled-|            |                  |                    |                  |                      |
```

### Partner Assignment Algorithm Flow

```
Fulfillment Service    Delivery Service    Partner Matcher    Location Service    Partner Management Service
         |                   |                   |                      |                          |
         |--Order Ready----->|                   |                      |                          |
         |                   |--Find Partners---->|                      |                          |
         |                   |                   |--Query Available--->|                          |
         |                   |                   |                      |--Get Partners-------->|
         |                   |                   |                      |<--Partners List-------|
         |                   |                   |--Get Locations------>|                          |
         |                   |                   |                      |--Get Partner Locations|                          |
         |                   |                   |                      |<--Locations-----------|
         |                   |                   |--Calculate Distances-|                          |
         |                   |                   |--Get Workloads----->|                          |
         |                   |                   |                      |--Get Workloads-------->|
         |                   |                   |                      |<--Workloads------------|
         |                   |                   |--Calculate Scores--->|                          |
         |                   |                   |--Rank Partners------|                          |
         |                   |                   |--Select Best--------|                          |
         |                   |<--Best Partner----|                      |                          |
         |                   |--Assign Order----->|                      |                          |
         |<--Assigned--------|                   |                      |                          |
```

---

**Last Updated**: January 2025
**Version**: 1.0
**Status**: Design Complete, Implementation Pending
