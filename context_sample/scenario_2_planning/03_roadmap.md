# Roadmap & Team Assessment (Internal Only)

## 1. Resource Plan
We need 3 "Squads" to execute Project Speedboat.

**Squad A (Platform)**
- 2 DevOps Engineers (Kubernetes experts).
- 1 Backend Principal (Architect).

**Squad B (Migration - Orders & Dispatch)**
- 3 Senior Backend Devs.
- 1 QA Automation.

**Squad C (Migration - Driver & Billing)**
- 3 Fullstack Devs.

## 2. Skills Gap Analysis
We surveyed the current engineering team (40 people).
- **Java Spring**: 90% proficiency.
- **Microservices Patterns**: 30% proficiency.
- **Kafka**: 10% (Only 4 people have used it in prod).
- **Go / Python**: 15%.
- **Kubernetes**: 20%.

**Risk**: We are proposing a stack (Go/Python/Kafka) that 80% of our team doesn't know well.
**Mitigation Plan**:
- Hire 2 consulting architects for the first 6 months ($40k/month cost).
- "Lunch and Learn" sessions every Friday.

## 3. Detailed Timeline (Estimated)

| Month | Milestone | Deliverable |
|---|---|---|
| M1 | Foundation | K8s Cluster Ready, Kafka POC, CI/CD Templates |
| M2 | CDC Setup | Debezium pipe running. Legacy data flowing to Topics |
| M3 | Pilot Service | "Notification Service" extracted (Low risk) |
| M4 | Driver Svc Dev | New Driver API Alpha |
| M5 | Dual Run | Driver App writes to BOTH new and old backend (Shadow mode) |
| M6 | Cutover | Driver App only talks to New Backend |
| ... | ... | ... |
| M12 | Code Freeze | Complete Decommission of Monolith |

## 4. Budget Estimation
- **Cloud Costs**: Increase by 30% initially (running parallel stacks), then drop by 20% after Month 12.
- **Hiring/Consulting**: $200,000 one-off.
- **Tooling (Datadog, Kong Enterprise)**: $50,000/year.

## 5. Unknowns
- How hard is it to decouple the "Billing" logic? (It looks like spaghetti SQL code).
- Will the latency of Kafka introduce lag in "Real-time Dispatching"?
  - If latency > 500ms, the dispatch algorithm might produce suboptimal routes.

## 6. Political/Management Landscape
- CTO is fully supportive.
- CFO is skeptical about the cost bump in Q1/Q2.
- Sales team is angry that "Feature Freeze" means no new features implementation for 6 months.
