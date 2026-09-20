---
name: saas-anti-pattern-review
description: Investigate code or an issue for SaaS-unfriendly patterns.
author: hugobatista
---

Investigate code or an issue for SaaS-unfriendly patterns. Use when the user asks to review something for SaaS readiness or to check if a pattern scales horizontally.

## Trigger phrases

"review for SaaS", "SaaS readiness", "SaaS anti-pattern", "SaaS-friendly", "is this SaaS-ready", "check this for SaaS", "will this scale for SaaS"

## Investigation checklist

For each file or area being reviewed, check these patterns:

### 1. State locality
- Is in-memory state (plain dict, module-level variable, `@lru_cache` singleton) used for:
  - WebSocket connections?
  - MFA / login / lockout state?
  - Rate limiting counters?
  - OAuth state or PKCE verifiers?
  - Failed login attempt tracking?
- In-memory state is invisible to other instances. Requires Redis or a shared KV store for multi-instance deployments.

### 2. Blocking request patterns
- Does any endpoint use `asyncio.to_thread` or `run_in_executor` for long-running sync work?
- Is there a `future.result(timeout=N)` blocking > a few seconds?
- Are background tasks submitted via `BackgroundTasks` (not durable — lost on restart)?
- Long blocking requests exhaust the thread/connection pool. Should be async job queues.

### 3. WebSocket dependency
- Is WebSocket the only delivery mechanism for a critical user-facing event?
- Is there a fallback (polling, job status endpoint) when WS is unavailable?
- WS should be an optimization, never a requirement for core flows.

### 4. Scheduler / job duplication
- Are periodic tasks started in the API process (APScheduler, startup hooks)?
- Is there a distributed lock or leader election?
- Without coordination, each replica runs the same job — duplicates imports, notifications, API calls.

### 5. Filesystem dependency
- Are uploads, media, temp files written to local paths?
- Is there an S3 / object-storage abstraction already?
- Local filesystem prevents horizontal scaling. Needs shared or object storage.

### 6. Database pool pressure
- What are the pool size and max overflow settings?
- Are COUNT queries loading full rowsets (`len(results)` instead of `COUNT(*)`)?
- Pool exhaustion is a common failure mode when scaling from single-instance to multi-worker.

### 7. Request cancellation
- Does the frontend set client-side timeouts on HTTP requests (`AbortController`)?
- Can the backend cancel in-flight work when the client disconnects?
- Without timeouts, a hanging request occupies a thread/connection indefinitely.

### 8. Credentials in URLs
- Are access tokens passed in WebSocket URLs (`ws?token=...`)?
- Are API keys accepted via query parameter?
- URLs end up in logs, browser history, referrer headers, support screenshots.

### 9. Multiple WebSocket consumers (frontend)
- Do multiple components assign `websocket.onmessage = handler` (collision-prone)?
- Should use `addEventListener('message', handler)` instead.

## Output format

Present findings as a structured list:

### Source
{issue / PR / code path}

### Blockers found

{N}. **[severity] Title** — `file:line`
- Current behavior: ...
- Why it fails in SaaS: ...
- Migration path: ...

### Not affected
- {patterns checked and found clear}

### Cross-reference notes
- {any connections to existing architecture docs or previous findings}

## Severity legend

- 🔴 Critical — blocks core SaaS capabilities (horizontal scaling, zero-downtime)
- 🟡 Moderate — affects UX, maintainability, or observability
- ⚪ Low — nice to fix but not blocking
