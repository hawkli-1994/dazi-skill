# dazi-network API Reference

Base URL: `https://api.dazi.decortex.ai` (or `DAZI_API_URL` env var)

Authentication: Every request includes DID signature headers:

```
X-DID: did:key:z6Mk...
X-Signature: <base64 Ed25519 signature of (timestamp + "\n" + body)>
X-Timestamp: 2026-04-27T12:00:00Z
```

Signature payload: `timestamp + "\n" + request_body_string`. For GET requests with no body, sign `timestamp + "\n"`.

---

## POST /profile

Create or update user profile. Server generates ai_extracted + embeddings from tags.

**Request:**
```json
{
  "nickname": "微扰",
  "basic": { "birth_year": 1995, "gender": "M", "city": "北京" },
  "tags": [
    "前大厂逃兵，现在全职独立开发",
    "INTP 社恐但话多，想多做少",
    "精酿、摄影、深度游，夜猫子"
  ],
  "contact": { "type": "wechat", "value": "wx_12345" }
}
```

**Validation:**
- `nickname`: 1-20 chars
- `basic.birth_year`: integer 1920-2010
- `basic.gender`: "M" | "F" | "O"
- `basic.city`: 1-50 chars
- `tags`: exactly 3 strings, each 1-200 chars
- `contact.type`: "wechat" | "telegram" | "twitter" | "jike" | "email"
- `contact.value`: 1-100 chars

**Response 201 (created) / 200 (updated):**
```json
{
  "did": "did:key:z6Mk...",
  "nickname": "微扰",
  "version": 1,
  "created_at": "2026-04-27T12:00:00Z"
}
```

**Errors:** 400 (validation), 429 (rate limited)

---

## POST /search

Search for matching candidates. Server parses intent, applies filters, runs embedding sort.

**Request:**
```json
{
  "intent": "找个五一去大理的搭子，能接受我社恐但会做攻略"
}
```

**Validation:** `intent`: 1-500 chars

**Response 200:**
```json
{
  "candidates": [
    {
      "nickname": "赛博游民",
      "tags": ["ENFP 行动派，想到就干", "产品经理转创业，做旅行社区", "徒步 + 精酿 + 猫奴"]
    }
  ],
  "total": 30
}
```

- Returns max 30 candidates
- Only nickname + tags returned (no ai_extracted, no age, no contact, no did)
- Empty candidates array if no matches

**Errors:** 400 (intent too long/empty), 404 (no profile — must POST /profile first), 429

---

## POST /interest

Express interest or decline. Default action is "accept".

**Request (accept — default):**
```json
{
  "target_nickname": "赛博游民"
}
```

**Request (decline):**
```json
{
  "target_nickname": "赛博游民",
  "action": "decline"
}
```

**Response 200 (pending):**
```json
{
  "status": "pending",
  "message": "Interest sent. You'll be notified when they respond."
}
```

**Response 200 (mutual match — other party already expressed interest):**
```json
{
  "status": "matched",
  "contact": { "type": "telegram", "value": "@cybernomad" },
  "message": "Mutual match! Here's their contact info."
}
```

**Response 200 (declined):**
```json
{
  "status": "declined",
  "message": "Declined interest from '赛博游民'."
}
```

**Errors:** 400 (self-interest), 404 (target not found / no pending interest to decline), 409 (already expressed), 429 (max 5/day)

---

## GET /connections

Get incoming interests + confirmed matches.

**Request:** No body. DID in auth header.

**Response 200:**
```json
{
  "pending_incoming": [
    {
      "nickname": "咖啡续命",
      "tags": ["设计师转产品", "ENFJ 社牛", "爱做饭的宅人"]
    }
  ],
  "pending_outgoing": [
    {
      "nickname": "赛博游民",
      "tags": ["ENFP 行动派", "产品转创业", "徒步精酿猫奴"]
    }
  ],
  "matched": [
    {
      "nickname": "清迈续命中",
      "tags": ["数字游民", "INFP 安静靠谱", "摄影咖啡慢旅行"],
      "contact": { "type": "wechat", "value": "nomad_cm" },
      "matched_at": "2026-04-27T15:30:00Z"
    }
  ]
}
```

---

## DELETE /profile

Delete account and all data. Irreversible.

**Request:** No body. DID in auth header.

**Response 200:**
```json
{
  "message": "Profile and all associated data deleted."
}
```

---

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| POST /profile | 10/hour |
| POST /search | 20/hour |
| POST /interest | 5/day |
| GET /connections | 60/hour |
| DELETE /profile | 1/day |

Rate limit headers on every response:
```
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 3
X-RateLimit-Reset: 2026-04-27T13:00:00Z
```

---

## Error Format

All errors follow:
```json
{
  "error": "error_code",
  "message": "Human readable description"
}
```

Error codes: `validation_error` (400), `auth_failed` (401), `not_found` (404), `conflict` (409), `rate_limited` (429), `server_error` (500)

429 response includes `retry_after` (seconds).
