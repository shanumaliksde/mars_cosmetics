# MARS Cosmetics — Order & Inventory Sync Service

## Overview

This repository contains the system design for a centralized Order and Inventory Sync Service for MARS Cosmetics.

The goal is to manage orders and inventory across four sales channels: **Website, Nykaa, Flipkart, and Blinkit**, while preventing overselling, synchronizing inventory, and reliably sending confirmed orders to Microsoft Business Central ERP.

## Technology Stack

- **Backend:** Node.js, NestJS
- **Database:** PostgreSQL
- **Queue Management:** Redis + BullMQ
- **Infrastructure:** AWS
- **ERP:** Microsoft Business Central

## My Approach

The proposed solution focuses on four key areas:

### 1. Centralized Order Processing

A NestJS application receives orders from all four sales channels using webhooks and scheduled polling.

All incoming orders are normalized into a common format and stored in PostgreSQL.

### 2. Inventory Consistency

PostgreSQL atomic transactions are used to reserve inventory safely.

Conditional updates prevent concurrent orders from reserving the same available stock.

### 3. Asynchronous Stock Synchronization

Redis and BullMQ handle background inventory updates across sales channels.

Workers respect channel-specific API rate limits and automatically retry failed requests.

The target is to synchronize stock changes within 60 seconds under normal operating conditions.

### 4. Reliable ERP Integration

The Transactional Outbox Pattern ensures that confirmed orders are reliably queued for delivery to Microsoft Business Central.

If the ERP is unavailable, pending events remain in PostgreSQL and are retried after recovery. Idempotency support is required to prevent duplicate ERP orders.

## Repository Structure

```text
mars-order-inventory-sync/
├── README.md
├── DESIGN.md
└── architecture.png
```

- **README.md:** Project overview, approach, and review instructions.
- **DESIGN.md:** Detailed architecture, database design, inventory reservation, failure handling, monitoring, assumptions, and trade-offs.
- **architecture.png:** High-level system architecture diagram.

## Setup / How to Review

This repository is a **system design submission**, not a runnable application. No installation, environment configuration, or database setup is required.

To review the solution:

1. Clone the repository or open it directly on GitHub.
2. Read `DESIGN.md` for the complete technical design.
3. Open `architecture.png` to understand the system architecture.

## Key Design Decisions

| Decision | Reason |
|---|---|
| NestJS Modular Monolith | Simple to develop and maintain with a small team |
| PostgreSQL Atomic Transactions | Prevent concurrent stock reservation conflicts |
| Redis + BullMQ | Reliable asynchronous processing and retries |
| Transactional Outbox | Prevent loss of ERP delivery events during failures |

## Assumptions and Limitations

- All channels use a common canonical SKU mapping.
- PostgreSQL is the source of truth for inventory.
- External marketplace stock updates may be delayed by API limits or outages.
- Exactly-once ERP effects depend on ERP-side idempotency or equivalent deduplication.

## AI Usage

ChatGPT was used to assist with documentation structure and design review.
