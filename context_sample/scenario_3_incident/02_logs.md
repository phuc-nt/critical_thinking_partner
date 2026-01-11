# System Logs & Metrics Snippets
**Date**: 2025-04-12
**Context**: During the outage window (14:15 - 14:45)

## 1. Application Logs (Save Service)
```text
[ERROR] 2025-04-12 14:15:23.112 [pool-1-thread-4] c.c.s.ConnectionManager - Unable to acquire JDBC Connection. Reason: Connection pool exhausted.
[ERROR] 2025-04-12 14:15:23.115 [pool-1-thread-4] c.c.s.DocumentService - Failed to save document chunk doc_id=88291. Retrying in 200ms...
[WARN ] 2025-04-12 14:15:23.320 [pool-1-thread-4] c.c.s.CircuitBreaker - Circuit [DatabaseWrite] is OPEN. Fast-failing requests.
```

## 2. Access Logs (Nginx Gateway)
```text
10.0.1.2 - - [12/Apr/2025:14:18:01] "POST /api/v1/sync HTTP/1.1" 503 145 "-" "CollabClient/3.4.0"
10.0.1.5 - - [12/Apr/2025:14:18:01] "POST /api/v1/sync HTTP/1.1" 503 145 "-" "CollabClient/3.4.0"
10.0.1.8 - - [12/Apr/2025:14:18:02] "POST /api/v1/sync HTTP/1.1" 503 145 "-" "CollabClient/3.4.0"
... (Repeated 50,000 times per minute)
```

## 3. Database Metrics (AWS RDS Performance Insights)
- **Wait Event**: `Client:ClientRead` (High), `LWLock:buffer_mapping` (High).
- **Explanation**: The DB was overwhelmed by SELECT queries.
- **Top Query**:
```sql
SELECT content_blob, version, lock_id 
FROM documents 
WHERE id = ? 
FOR UPDATE;
```
- **Frequency**: Jumped from 1,000 ops/sec to 25,000 ops/sec.

## 4. WebSocket Service Config (The Change)
**Old Config (v33)**
```yaml
socket:
  heartbeat_interval: 30s
  reconnect_backoff: exponential
  max_reconnect_attempts: 10
```

**New Config (v34 - The Culprit)**
```yaml
socket:
  heartbeat_interval: 5s  # <--- Changed to improve "perceived" liveness
  reconnect_backoff: linear # <--- Changed to "reconnect faster"
  reconnect_delay: 1s
```

## 5. Client Side Log (Sample submitted by user)
```json
{
  "event": "socket_error",
  "msg": "Heartbeatack missing",
  "action": "reconnecting_now",
  "timestamp": 1744467600
}
{
  "event": "connect",
  "action": "request_full_sync", // <--- THE KILLER
  "doc_id": "doc_xyz"
}
```
**Analysis**: The client logic triggers a "Full Sync" (Heavy Read) every time the socket reconnects. With the new aggressive retry logic, 100k clients hammered the DB for full docs simultaneously.
