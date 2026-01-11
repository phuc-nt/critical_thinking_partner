# Product Requirement Document: BNPL (Buy Now Pay Later) Core Service
**Version:** 1.0.0
**Status:** Draft
**Owner:** Product Team A

## 1. Executive Summary
The logic behind this feature is to increase GMV (Gross Merchandise Value) by allowing users to split their payments into 4 interest-free installments. This document outlines the core functional requirements for the MVP.

## 2. User User Stories & Acceptance Criteria

### US-001: User Eligibility Check
**As a** checkout user,
**I want** to see BNPL as a payment option only if I am eligible,
**So that** I don't get rejected after selecting it.

**Acceptance Criteria:**
1. System must check user 'trust score' in real-time.
2. If Trust Score > 600, show BNPL option.
3. If Trust Score < 600, hide BNPL option.
4. If User has active overdue debts, hide BNPL option.
5. Check must complete within 200ms.

### US-002: Loan Activation (Checkout)
**As a** eligible user,
**I want** to confirm the payment plan,
**So that** my order is placed immediately.

**Acceptance Criteria:**
1. User selects "Pay in 4 installments".
2. System displays a repayment schedule:
   - Payment 1: Today (25%)
   - Payment 2: T+14 days (25%)
   - Payment 3: T+28 days (25%)
   - Payment 4: T+42 days (25%)
3. User agrees to T&C.
4. System authorizes the first 25% on the linked card.
5. Order status moves to "Processing".

### US-003: Auto-Repayment Deduction
**As a** system,
**I want** to automatically charge the user's card on due dates,
**So that** collection effort is minimized.

**Acceptance Criteria:**
1. Cron job runs at 00:00 UTC daily.
2. Identify all installments due today.
3. Attempt charge on primary card.
4. If success: Send email "Payment Successful" and update Ledger.
5. If fail: Retry 3 times (every 6 hours).
6. If all retries fail: Mark loan as "Overdue" and apply $10 late fee.

## 3. Detailed Logic Specification

### 3.1 Credit Limit Calculation
- **New Users**: Fixed limit of $200.
- **Existing Users**: 
  - If 3+ successful repayments: Limit = $500.
  - If 10+ successful repayments: Limit = $1000.
  - NOTE: Limit does not decrease unless a default occurs.

### 3.2 Late Fee Logic
- Fee: Flat $10.
- Grace Period: 3 days.
- Cap: Late fees cannot exceed 25% of the loan value.
- **Exception**: VIP users (Total Spend > $5000) get one free pass per year.

### 3.3 Refund Handling
- If user returns an item, the refund amount is deducted from the *last* installment first.
- If refund > remaining balance, the difference is refunded to the card.
- Loan status is updated to "Closed" if balance hits 0.

## 4. Edge Cases (To be refined)
- User changes card before due date.
- Card expires mid-loan.
- Insufficient funds during auto-charge.
- Partial refunds.
- System downtime during checkout.

## 5. UI Requirements (Mobile)
- **Checkout Screen**: 
  - Highlight "0% Interest" prominently.
  - Show precise dates for future payments.
- **Dashboard**:
  - Progress bar for repayment.
  - "Pay Early" button.

## 6. Open Questions
- Do we need to do a hard credit check with external bureaus? (Currently assuming No to save cost/speed).
- How do we handle users who delete their account while having active loans?
- Should we allow users to change the repayment date?
