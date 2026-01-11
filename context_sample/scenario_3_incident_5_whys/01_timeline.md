# Incident Timeline: Major Collaboration Outage
**Incident ID:** INC-20250412-001
**Service**: CollabWriter (Real-time Document Editor)
**Severity**: Sev-1
**Date**: April 12, 2025

## Summary
For 45 minutes, 30% of users in the EU and NA regions could not save their documents. "Saving..." indicator stuck indefinitely. Data loss confirmed for ~2000 documents (last 5-10 minutes of edits lost).

## Timeline (All times UTC)

**14:00** - Deployment pipeline `deploy-websocket-service-v34` starts.
- Change: Optimization of WebSocket heartbeat interval from 30s to 5s.

**14:05** - Deployment completes. 100% of traffic shifted to new pods.

**14:12** - Customer Support receives first ticket from Enterprise User (Acme Corp).
- "We are typing but the changes aren't saving."

**14:15** - Alert Firing: `HighErrorRate` on `save-service` (Not the websocket service!).
- 500 Error rate spikes to 15%.

**14:20** - On-call Engineer (Alice) acknowledges pager.
- Checks `save-service` logs. Sees database connection timeouts.
- Hypothesis: Database is overloaded.

**14:25** - Alice checks DB metrics.
- CPU is 100%.
- Active Connections: 5000/5000 (Maxed out).

**14:30** - Alice scales up Database (Vertical Scaling).
- Takes 10 minutes to propagate.
- Incident is getting worse. Connection metrics don't drop.

**14:35** - Lead Engineer (Bob) joins.
- Bob notices `websocket-service` traffic is weird.
- The number of "Reconnect" events is 10x normal.

**14:40** - Root Cause Theory Formed.
- The new `v34` deployment with 5s heartbeat is causing clients to think connection is unstable (logic bug?), forcing them to Reconnect aggressively.
- Each Reconnect triggers a "Full State Sync" (Reading the whole doc from DB).
- This Thundering Herd is killing the DB.

**14:42** - **Decision**: Rollback `websocket-service` to `v33`.

**14:45** - Rollback complete.
- Client reconnection storm subsides.
- DB Load drops to 40%.
- Save functionality restored.

**15:00** - Incident Closed. Cleanup begins.
