---
name: discord-send
description: Send messages to Discord channels via webhooks. Covers text, embeds, file attachments, threads.
author: hugobatista
---

## Setup

`curl` must be installed. Webhook URLs stored in `webhooks.json` (`{"webhooks": {"default": "...", "alerts": "..."}}`). If the target is not in `webhooks.json`, ask the user for the URL and persist it there for future use. Reference targets by name.

## Text

```bash
curl -s -o /dev/null -w "%{http_code}" -H "Content-Type: application/json" -d '{"content": "Your message"}' "$WEBHOOK_URL"
```

Supports Discord-flavoured Markdown (bold, code, spoilers, quotes, lists).

## Embed

```bash
curl -s -o /dev/null -w "%{http_code}" -H "Content-Type: application/json" -d '{
  "embeds": [{
    "title": "Build #42 — Success",
    "description": "CI passed",
    "color": 5763719,
    "fields": [{"name": "Branch", "value": "main", "inline": true}],
    "author": {"name": "CI"},
    "footer": {"text": "opencode"},
    "timestamp": "'"$(date -u +%Y-%m-%dT%H:%M:%SZ)"'"
  }]
}' "$WEBHOOK_URL"
```

Fields: `title`, `description`, `color` (decimal), `fields` (name/value/inline, max 25), `author` (name/url/icon_url), `footer` (text/icon_url), `image`/`thumbnail` (url), `timestamp` (ISO 8601).

## File

```bash
curl -s -o /dev/null -w "%{http_code}" -F "payload_json={\"content\":\"See attached\"}" -F "file=@/path/to/file.txt" "$WEBHOOK_URL"
```

Multiple files: repeat `-F "file=@..."`.

## Threads

Send to existing thread:

```bash
curl -s -o /dev/null -w "%{http_code}" -H "Content-Type: application/json" -d '{"content": "Reply"}' "${WEBHOOK_URL}?thread_id=THREAD_ID"
```

Create thread from message (include `thread_name`):

```bash
curl -s -o /dev/null -w "%{http_code}" -H "Content-Type: application/json" -d '{"content": "First post", "thread_name": "Topic"}' "$WEBHOOK_URL"
```

## Status codes

The `-o /dev/null -w "%{http_code}"` flags above return the HTTP status inline:

| Code | Meaning |
|------|---------|
| 204  | Success — message delivered |
| 400  | Bad payload — malformed JSON or content |
| 404  | Bad URL — webhook URL is wrong |
| 429  | Rate limited — check `Retry-After` header |
| 5xx  | Server error — retry after a moment |

Note: do **not** re-run `curl` with the same `-d` payload to "check" delivery — that would send a duplicate message. The `-o /dev/null -w` pattern reports the result on the first (and only) attempt.
