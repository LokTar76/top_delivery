# Local Delivery System Design

A domain-driven system design for a local delivery and last-mile logistics platform.

This project models the complete operational lifecycle of a local delivery business, including:

- Customer shipment creation
- Manifest receiving
- Warehouse reconciliation
- Sorting and storage
- Delivery scheduling
- Driver pickup verification
- Package-level delivery execution
- Exception handling
- POD capture
- Auditability and operational traceability

---

# System Goals

This system is designed to solve common operational challenges in local delivery logistics:

- Prevent missing-package pickup issues
- Support package-level operational truth
- Enable split delivery attempts
- Track full shipment execution history
- Support route optimization
- Provide operational transparency
- Make exception handling explicit and traceable

---

# Architecture Overview

The platform is organized into five bounded domains.

---

## 1. Shipment Domain

The shipment is the canonical business entity.

It represents the customer-facing consignment and survives across:

- Schedule reassignment
- Delivery retries
- Split attempts
- Operational recovery

### Responsibilities

- Shipment creation
- Customer shipment ownership
- Consignment metadata
- Package aggregation
- Lifecycle state tracking

### Core Entities

- Shipment
- ShipmentPackage
- ShipmentExtraData
- ShipmentEvent

---

## 2. Manifest Domain

A manifest represents a customer submission batch.

It defines how shipments enter the operational system.

### Responsibilities

- Batch receiving
- Customer handoff reconciliation
- Manifest closure
- Receiving discrepancy handling

### Core Entities

- Manifest
- ManifestShipment

---

## 3. Execution Domain

Execution attempts are modeled separately from shipment identity.

This allows operational retries without mutating the shipment itself.

### Examples

- Partial pickup
- Failed delivery retry
- Split execution
- Return-to-warehouse reattempt

### Core Entities

- CargoProcess
- ShipmentAttemptPackage

---

## 4. Scheduling Domain

Schedules are operational execution containers.

A schedule groups cargo processes into a route execution plan.

### Responsibilities

- Route generation
- Route optimization
- Driver assignment
- DSP assignment
- Schedule lifecycle management

### Core Entities

- DeliverySchedule
- DeliveryScheduleRelation
- ScheduleEvent

---

## 5. Audit Domain

Operational actions are fully traceable.

The audit domain records all important state transitions and user actions.

### Responsibilities

- User action recording
- Operational traceability
- State transition history
- Debugging and replay support

### Core Entities

- Log
- ShipmentEvent
- ScheduleEvent

---

# Documentation Structure

---

## 1. ERD / Domain Model

Defines:

- Core entities
- Relationships
- Aggregate boundaries
- Ownership rules

**File**

```text
ERD_Domain_Model.mmd
```

---

## 2. Shipment Lifecycle

Defines the complete end-to-end shipment lifecycle.

**File**

```text
Shipment_Lifecycle.mmd
```

### Covers

- Shipment creation
- Manifest generation
- Receiving
- Sorting
- Scheduling
- Delivery execution
- Completion / return

---

## 3. Schedule Lifecycle

Defines schedule generation and operational execution.

**File**

```text
Schedule_Lifecycle.mmd
```

### Covers

- Schedule generation
- Assignment
- Pickup validation
- Execution state transitions
- Completion rules

---

## 4. Delivery Flow

Defines detailed driver-side operational execution.

**File**

```text
Delivery_Flow.mmd
```

### Covers

- Pickup scanning
- Package verification
- Force-start exception logic
- Delivery confirmation
- POD capture
- Completion validation

---

## 5. Exception Flows

Defines operational exception handling.

Examples:

- Manifest mismatch
- Missing pickup package
- Partial delivery
- Failed delivery retry
- Manual intervention

---

## 6. UI Journey

Defines operational user interaction flows.

Includes:

- Warehouse operator flow
- Driver app flow
- Dispatcher workflow
- Exception handling screens

---

## 7. Data Flow / System Integration

Defines system-level integration boundaries.

Includes:

- API interactions
- Driver app communication
- Scan event propagation
- Notification triggers
- External integrations

---

# Core Design Principles

---

## Shipment Is the Business Truth

Shipment identity never changes.

Operational retries must not mutate shipment identity.

---

## CargoProcess Represents Execution Attempts

A shipment may have multiple execution attempts.

Each attempt tracks operational execution separately.

This supports:

- Retry delivery
- Split pickup
- Partial execution recovery

---

## Package Is the Scan Unit

Operational truth is package-level.

This enables:

- Missing package detection
- Partial pickup validation
- Partial delivery tracking
- Accurate operational reconciliation

---

## Schedule Is an Execution Container

Schedules are temporary execution plans.

They do not own shipment identity.

---

## Event-Driven Traceability

All critical operational transitions are recorded as immutable events.

Benefits:

- Auditability
- Operational debugging
- Historical reconstruction
- KPI analytics

---

# State Modeling

The system uses layered state tracking.

---

## Shipment State

Business lifecycle state.

Example:

```text
Created
→ Manifested
→ Received
→ Ready
→ Scheduled
→ Delivered
→ Completed
```

---

## CargoProcess State

Execution attempt state.

Example:

```text
Pending
→ Picked Up
→ Out For Delivery
→ Completed
```

---

## Schedule State

Operational route state.

Example:

```text
Draft
→ Assigned
→ Picked Up
→ In Progress
→ Completed
```

---

## Package State

Physical scan-level state.

Example:

```text
Expected
→ Picked Up
→ Delivered
```

---

# Exception Handling Philosophy

Exceptions are modeled explicitly.

The system avoids hidden operational ambiguity.

---

## Receiving Discrepancy

Shipment does not match manifest.

---

## Pickup Exception

Driver missing expected packages.

---

## Partial Delivery

Only subset of packages delivered.

---

## Retry Workflow

Return-to-warehouse and reattempt generation.

---

## Manual Recovery

Operator-driven intervention path.

---

# Future Extensions

Planned extension points:

- Billing integration
- Customer self-service portal
- Real-time vehicle tracking
- POD OCR
- Dynamic route re-optimization
- SLA monitoring
- Automated anomaly detection
- Predictive dispatching

---

# Reading Order

Recommended reading sequence:

1. ERD_Domain_Model.mmd
2. Shipment_Lifecycle.mmd
3. Schedule_Lifecycle.mmd
4. Delivery_Flow.mmd
5. Exception Flows
6. UI Journey
7. Data Flow / Integration

---

# Design Philosophy

This system is built around one core principle:

**Separate business truth from operational execution.**

This separation keeps the system:

- Maintainable
- Auditable
- Extensible
- Operationally predictable

while supporting the real-world messiness of local delivery logistics.
