# Microservice Boundary Design (Draft)

> Version: v0.1
> Status: Draft
> Principle: Shared Database, Separated Business Ownership

---

# Overall Architecture

```
                    +-------------+
                    | API Gateway |
                    +-------------+
                           |
        +------------------+------------------+
        |                                     |
        v                                     v
+--------------------+              +------------------+
| public-api-service |              | delivery-service |
+--------------------+              +------------------+
        |                                     |
        |                                     |
        +------------------+------------------+
                           |
                           v
              +-------------------------+
              | Core Business Services  |
              +-------------------------+
              | finance-service         |
              | label-service           |
              | report-service          |
              | integration-service     |
              +-------------------------+
                           |
                           v
              +-------------------------+
              | Shared Supporting       |
              +-------------------------+
              | file-service            |
              | audit/log component     |
              | notification later      |
              +-------------------------+
```
External Entry
--------------
API Gateway
public-api-service
OP / Driver / Warehouse / Portal APIs

Core Business
-------------
delivery-service
finance-service

Supporting Capabilities
-----------------------
file-service
label-service
report-service
integration-service

Shared Components
-----------------
audit log
security
common DTO
constants

The project uses **Shared Database + Service Ownership** instead of database-per-service.

Each service is responsible for its own business logic and write ownership.

---

# Core Principles

## 1. Shared Database

All services share the same database schema.

Benefits:

* No distributed joins
* Easier migration from monolith
* Existing SQL remains usable

---

## 2. Write Ownership

Reading across modules is allowed.

Writing is restricted.

Example:

| Table            | Write Owner      |
| ---------------- | ---------------- |
| shipment         | delivery-service |
| shipment_package | delivery-service |
| manifest         | delivery-service |
| tracking         | delivery-service |
| invoice          | finance-service  |
| invoice_line     | finance-service  |
| payment          | finance-service  |
| file_repo        | file-service     |

A service should **never directly update another service's owned tables**.

---

# public-api-service

## Responsibilities

Acts as the external API boundary.

It owns:

* API Key / Secret validation
* Signature verification
* Client permission validation
* Rate limiting
* External reference deduplication
* Idempotency key
* Request schema validation
* API version management (v1 / v2)
* Customer-specific request mapping
* Public DTO <-> Internal DTO mapping
* Unified error response
* Response formatting
* Webhook callback handling
* API request logging

## Does NOT own

Business logic.

It should NOT directly create or update:

* shipment
* package
* manifest
* invoice

Instead it calls business services.

Example:

```
POST /shipments

↓

Validate API Request

↓

Map to CreateShipmentCommand

↓

delivery-service.createShipment(...)
```

---

# delivery-service

The core business service.

Responsible for all logistics workflows.

Owns:

* Shipment
* Shipment Package
* Manifest
* Delivery Schedule
* Driver Assignment
* Warehouse Workflow
* Tracking
* Shipment State Machine
* Cargo Process
* POD
* Scan Events
* Audit Log

Responsibilities:

* Shipment lifecycle
* Manifest lifecycle
* Delivery workflow
* Warehouse workflow
* Shipment state transitions
* Business validation
* Tracking generation

Delivery Service is the owner of shipment-related business rules.

---

# finance-service

Responsible for financial domain.

Owns:

* Invoice
* Invoice Line
* Payment
* Billing
* Charge Code
* Customer Rate
* Supplier Invoice
* Reconciliation

Responsibilities:

* Invoice generation
* Invoice cancellation
* Payment processing
* Customer billing
* Supplier billing
* Financial reports
* Reconciliation

May READ:

* Shipment
* Manifest
* Manifest Shipment

Should NOT directly modify shipment or manifest.

---

# file-service

Responsible for all file management.

Owns:

* File Repository
* File Metadata

Responsibilities:

* Upload
* Download
* Delete
* Thumbnail generation
* S3 Integration
* Temporary file management
* Signed URL generation

All file storage should be abstracted through this service.

---

# label-service

Optional.

Can be separated later.

Responsibilities:

* Shipping label generation
* Barcode generation
* PDF rendering
* Printer integration
* Carrier-specific templates
* ZPL generation

Reason:

Heavy PDF/image rendering should not affect delivery-service.

---

# report-service

Optional.

Can be implemented later.

Responsibilities:

* Scheduled reports
* Dashboard cache
* BI aggregation
* Long-running statistics
* Warehouse daily reports
* Customer KPI reports

Reason:

Reporting queries are usually heavy and should not compete with transactional workloads.

---

# integration-service

Optional.

Handles third-party integrations.

Responsibilities:

* Carrier APIs
* SMS Gateway
* Email Gateway
* AWS integrations
* Payment Gateway
* External ERP/WMS
* Webhook consumers

Reason:

External dependencies change frequently.

Keeping them isolated reduces impact on core business services.

---

# Tracking API

Tracking can initially stay inside public-api-service.

Reason:

Tracking is externally facing.

Typical flow:

```
Customer

↓

public-api-service

↓

delivery-service

↓

Tracking Response
```

Tracking may become an independent service in the future if traffic becomes very large.

---

# Service Communication

Services should communicate using coarse-grained business commands.

Good:

```
generateInvoiceFromManifest()

cancelInvoice()

createShipment()

assignDriver()

markShipmentDelivered()
```

Avoid:

```
insertInvoice()

updateManifest()

insertTracking()

updateShipmentStatus()
```

Business actions should be exposed rather than CRUD operations.

---

# Future Services (Not Yet Needed)

* notification-service
* authentication-service
* search-service
* pricing-service
* routing-service

These should only be extracted when there is a clear business or operational need.

---

# Current Recommendation

Priority:

1. delivery-service
2. public-api-service
3. finance-service
4. file-service
5. label-service
6. report-service
7. integration-service

Avoid over-engineering.

Business ownership is more important than the number of microservices.
