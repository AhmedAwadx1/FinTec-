# Fintech Money Transfer Backend (Laravel)

This project is a minimal backend service built with **Laravel (PHP)** that supports money transfers between accounts, similar to a simplified fintech core system.

The goal of this project is to demonstrate:
- Backend system design
- Data modeling and transactional thinking
- Safe and consistent money transfer logic
- Clean code structure using Repository Pattern
- Clear and well-designed APIs

No frontend or third-party integrations are included.

---

## Tech Stack

- PHP 8+
- Laravel
- Relational Database (MySQL / PostgreSQL)
- RESTful API

---

## Core Features

### 1. Users & Accounts
- A user can have one or more accounts
- Each account belongs to exactly one user
- Each account has a balance stored as an integer (cents)

### 2. Money Transfer
- Money can be transferred between two accounts
- A transfer **fails** if:
  - The source account balance is insufficient
  - The source and destination accounts are the same
  - Either account is inactive
- Transfers are processed **atomically**
- Account balances always remain consistent
- Concurrent transfers are handled safely using row-level locking

### 3. Transaction History
- Every money movement is recorded
- Transfers are traceable through transaction records
- Transfer status is stored (`PENDING`, `SUCCESS`, `FAILED`)
- A ledger system is used to track actual money movements

---

## Database Design

The system consists of the following core tables:

- `users`
- `accounts`
- `transfers`
- `ledger_entries`

### Key Design Decisions

- **Integer-based money storage**  
  All monetary values (`balance`, `amount`) are stored as integers representing the smallest currency unit (cents) to avoid floating-point precision issues.

- **Transfers vs Ledger Entries**
  - `transfers` table records transfer attempts (including failed ones)
  - `ledger_entries` table records actual money movements
  - Each successful transfer creates **two ledger entries**:
    - DEBIT (negative amount) for source account
    - CREDIT (positive amount) for destination account

- **Atomicity & Consistency**
  - All transfers are executed inside a database transaction
  - Accounts are locked using `SELECT ... FOR UPDATE` (`lockForUpdate()` in Laravel)
  - Accounts are locked in sorted order by ID to prevent deadlocks

- **Traceability**
  - Each transfer has a unique human-readable reference
  - Full audit trail is available via ledger entries

> See the ERD diagram for full schema details: [https://dbdiagram.io/d/FinTec-697b8124bd82f5fce20aa66f](https://dbdiagram.io/d/FinTec-697b8124bd82f5fce20aa66f)

