# Post-Mortem Draft Analysis
**Facilitator**: Bob (Lead Eng)
**Participants**: Alice, Charlie (Frontend), Dave (DBA)

## 1. What went wrong?
- We created a DDoS attack on ourselves.
- The change to `heartbeat_interval` and `reconnect_strategy` was intended to make the app feel "snappier", but it had a catastrophic side effect on backend load.
- We did not test this configuration at scale (Scale testing environment only has 100 bots).

## 2. Why was this not caught in PR Review?
- **Reviewer Comment**: "Looks like simple config tweaks. LGTM."
- The reviewer (Charlie) is a frontend dev and didn't realize the backend implication of "Full Sync on Reconnect".
- There was no load-test requriment for config changes.

## 3. Impact Assessment
- **Duration**: 45 mins.
- **Data Loss**: ~2000 versions of documents lost (Users typed, thought it saved, but it failed, then they refreshed page and lost local cache).
- **Reputation**: Enterprise Customer "Acme Corp" has requested an SLA breach credit.

## 4. Remediation Items (TODO)

### Immediate
- [x] Rollback to v33.
- [ ] Add capacity to RDS Read Replicas (Band aid).

### Short Term
- Fix the Client Logic: Do NOT request full document sync on simple socket reconnection. Only sync *deltas* (missing operations).
- Implement "Jitter" in the reconnect backoff (Randomize retry times so they don't sync).

### Long Term
- **Rate Limiting**: Implement rate limiting at the API Gateway for `/sync` endpoint.
- **Circuit Breakers**: The `Save Service` has a circuit breaker, but the `Sync Service` (Read) did not have one protecting the DB.

## 5. The "5 Whys" Preparation (For AI Analysis)
We need to dig deeper.
- Why did the client request full sync? -> Legacy code from 3 years ago.
- Why did we deploy without load testing? -> Config changes constitute "Low Risk" track in our release process.
- Why is the DB the bottleneck? -> We rely on pessimistic locking (`FOR UPDATE`) for document consistency.

We invite the AI to challenge us on the **Process** failures here.
