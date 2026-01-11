# Risk & Fraud Policy: BNPL Implementation
**Confidentiality**: Internal Use Only

## 1. Goal
Balance user acquisition with acceptable loss rates. Target loss rate: < 1.5% of GMV.

## 2. Fraud Detection Rules (Pre-Check)
The following rules trigger an immediate **REJECTION**:
1. **Device Velocity**: More than 3 different user accounts logging in from the same DeviceID within 24 hours.
2. **IP Velocity**: More than 10 orders from same IP in 1 hour (unless whitelisted corporate IP).
3. **Shipping Address Mismatch**: Shipping Country != IP Geolocation Country.
4. **High Value First Time**: New user attempting to buy item > $400 on first transaction.
5. **Disposable Email**: Email domain is in `blacklist_domains.txt` (e.g., yopmail.com).

## 3. Credit Assessment Logic (The "Black Box")
We calculate a simplified "Trust Score" (0-1000) internally since we don't want to pay for Experian/Equifax calls for every user.

**Formula Inputs:**
- Account Age (30%)
- Order History Value (40%)
- Return Rate (30%)

**The Hole in Policy (For Socratic Review):**
- We currently do **NOT** check external debt. A user could be bankrupt externally but have a good history with us.
- We assume "Account Age" is a proxy for stability. ID buying (black market accounts) is a known vector we haven't addressed.

## 4. Collection Strategy
What happens when people don't pay?

**Stage 1: Soft Dunning (Day 1-7 Overdue)**
- Day 1: Email reminder + Push notification.
- Day 3: Email warning about late fee.
- Day 4: Late fee applied ($10).
- Day 7: Account "Suspended" (Cannot buy anything else).

**Stage 2: Hard Dunning (Day 8-30)**
- SMS reminders (Twilio).
- Phone call from CS team (Manual list export).

**Stage 3: Write-Off (Day 60)**
- Mark loan as `DEFAULTED`.
- Sell debt to external agency? (Decision pending).
- Ban user permanently (Shadow ban device ID + Email + Phone).

## 5. VIP Exceptions
- "Whales" (Spend > $10k/year) are exempt from Rule 1 (Device Velocity) because they often have assistants buying for them.
- **Risk**: Assistants going rogue?

## 6. Operational Manual Reviews
- Orders > $1000 flagged for manual review if Trust Score is "Medium".
- Review SLA: 4 hours.
- If review isn't completed in 4 hours, system **Auto-Approves** (to prevent user churn). 
  - *Note*: This is a business decision to prioritize GMV over Risk.

## 7. Data Retention
- We keep transaction logs for 7 years.
- We keep failed fraud attempts for 1 year (to train ML models).
