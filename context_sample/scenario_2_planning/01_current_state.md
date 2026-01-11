# Current State Analysis: Logistics Monolith
**date:** 2026-01-11
**System:** Legacy "LogiTrack"

## 1. System Overview
"LogiTrack" is a 5-year-old Monolithic application built on Java Spring Boot 2.x, utilizing a single PostgreSQL database instance (3TB). It handles all core business functions:
- Order Management
- Driver Dispatch
- Route Optimization
- Customer Billing
- Inventory Tracking

## 2. Current Pain Points (The "Burning Platform")

### 2.1 Performance & Scalability
- **Database Locks**: During peak operational hours (8 AM - 10 AM, dispatch window), database CPU hits 95%. Table locks on `orders` table cause timeouts for the create_order API.
- **Reporting Delay**: The `daily_sales_report` query takes 45 minutes to run and often crashes the read replica.
- **Single Point of Failure**: Last month, a memory leak in the "Image Processing" module (used for Proof of Delivery) caused the entire backend to crash, taking down Order Entry capabilities for 2 hours.

### 2.2 Developer Velocity
- **Build Times**: CI/CD pipeline takes 25 minutes to run tests and build the JAR.
- **Merge Hell**: With 40 developers working on the same repo, we have an average of 5 active merge conflicts per PR.
- **Deployment Fear**: We deploy only once every 2 weeks (Bi-weekly train) because regression testing is manual and painful.

### 2.3 Coupled Logic
- **Example**: The `OrderService` class has 5000+ lines of code.
- Changing a field in the `Driver` entity requires updates in Billing, Routing, and Notification modules because they all access the DB Entity directly.

## 3. Statistics
- **Daily Active Users**: 2,000 internal staff, 15,000 drivers.
- **Requests Per Minute (RPM)**: 12,000 RPM (peak).
- **Database**: 350 Tables. The `audit_log` table alone is 1.2 TB.

## 4. Codebase Analysis (SonarQube)
- **Technical Debt Ratio**: 18.5% (High).
- **Test Coverage**: 45% (mostly integration tests, few unit tests).
- **Duplication**: 12%.

## 5. Infrastructure Cost
- AWS Bill: $15,000/month.
- Primary cost driver: oversized RDS `db.r5.12xlarge` instance to handle the locking issues.

## 6. Business Impact
- We cannot launch "Instant Delivery" (1-hour promise) because the system takes 5 minutes just to calculate a route due to queue backlogs.
- Competitors are deploying features daily; we are moving bi-weekly.
