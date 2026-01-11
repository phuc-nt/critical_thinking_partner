# Technical Specification: BNPL Integration
**Date:** 2026-01-10
**Author:** Backend Architect

## 1. System Architecture
The BNPL feature will be implemented as a new Microservice `finance-core`.

**Interactions:**
- `checkout-service` -> GRPC -> `finance-core` (Create Loan)
- `finance-core` -> REST -> `stripe-gateway` (Payment Processing)
- `finance-core` -> REST -> `fraud-detection-service` (Risk Check)
- `finance-core` -> Kafka -> `notification-service` (Emails/Push)

## 2. API Definitions

### POST /v1/loans/eligibility
**Request:**
```json
{
  "user_id": "u_123456",
  "cart_value": 150.00,
  "currency": "USD",
  "device_id": "dev_hash_abc"
}
```
**Response:**
```json
{
  "eligible": true,
  "max_limit": 200.00,
  "reason": "OK"
}
```

### POST /v1/loans/create
**Request:**
```json
{
  "order_id": "ord_987",
  "user_id": "u_123456",
  "amount": 100.00,
  "tenure_days": 42,
  "payment_method_id": "pm_card_visa_111"
}
```

**Logic Flow:**
1. **Validate Param**: Check if amount <= user's available limit.
2. **Lock Funds**: Call Stripe `payment_intents.create` with `capture_method=manual` for the first installment ($25).
3. **DB Transaction**:
   - Insert into `loans` table (status: PENDING).
   - Insert 4 rows into `installments` table.
4. **Capture**: Call Stripe `payment_intents.capture`.
5. **Finalize**: Commit DB, return Success.

## 3. Database Schema (PostgreSQL)

### Table: `users_credit_profile`
| Column | Type | Notes |
|Names| Type| Description|
|---|---|---|
| user_id | UUID | PK |
| total_limit | DECIMAL | Max exposure |
| available_limit | DECIMAL | Current balance |
| trust_score | INT | 0-1000 |
| is_blocked | BOOLEAN | Hard block |

### Table: `loans`
| Column | Type | Notes |
|---|---|---|
| loan_id | UUID | PK |
| user_id | UUID | FK |
| total_amount | DECIMAL | |
| outstanding_amount | DECIMAL | |
| status | ENUM | ACTIVE, PAID, DEFAULTED |
| created_at | TIMESTAMP | |

### Table: `installments`
| Column | Type | Notes |
|---|---|---|
| id | UUID | PK |
| loan_id | UUID | FK |
| due_date | DATE | |
| amount | DECIMAL | |
| status | ENUM | PENDING, PAID, FAILED |
| retry_count | INT | Default 0 |

## 4. Security & Compliance
- **PCI-DSS**: We will NOT store raw card numbers. Only tokens (`pm_xxx`) from Stripe.
- **Idempotency**: All payment endpoints must support `Idempotency-Key` header to prevent double charges.
- **Audit Logs**: Any change to `users_credit_profile` must be logged in `audit_logs` table.

## 5. Performance Constraints
- Eligibility check must happen < 200ms (P99).
- Currently, `fraud-detection-service` takes avg 150ms. We may need to cache trust scores in Redis to meet the requirement.
- **Risks**: Redis cache might be stale. If a user defaults on a loan, but cache says "Good Score", they might get another loan instantly. We accept this risk for now for performance.

## 6. Depenedencies
- Stripe API v2025-01-01
- Internal `user-service` for KYB status.
