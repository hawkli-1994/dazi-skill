---
name: dazi
description: >-
  AI-native social matching for harness users. Find compatible people through
  your AI — no app, no swiping. Your AI already knows you; let it find your people.
  Use when: /dazi-match, /dazi-update, /dazi-inbox, "帮我找搭子", "find me a dazi",
  "有没有聊得来的人", "推荐几个搭子", "有人想认识我吗".
license: Apache-2.0
compatibility: Requires Python 3.10+ and internet access for matching API.
metadata:
  author: decortex
  version: "0.1.0"
  homepage: https://github.com/decortex-ai/dazi-skill
allowed-tools: Read Write Bash
---

# dazi — AI-Native Social Matching

> "Your AI already knows you. Let it find your people."

Auto-detect the user's language from their first message. Respond in the same language throughout.

---

## Commands

| Command | Description |
|---------|-------------|
| `/dazi-match` | Find compatible people (triggers registration on first use) |
| `/dazi-update` | Update your profile |
| `/dazi-inbox` | Check who wants to meet you + confirmed matches |

### First-Time Behavior (CRITICAL)

**On ANY command or dazi-related trigger**, you MUST first:
1. Silently set up identity (generate keypair if needed)
2. Call `GET /connections` with signed headers to check if profile exists on server
3. If 404 → **immediately start Registration flow** (ask nickname, basic info, tags, contact). Do NOT show a command menu. Do NOT ask "what do you want to do".
4. Only after registration is complete (or if profile already exists) → proceed with the requested command.

### Natural Language Routing

Route these intents to the matching command:

- `/dazi-match`: "帮我找搭子", "find me a dazi", "推荐几个搭子", "find someone compatible"
- `/dazi-inbox`: "有人想认识我吗", "有没有新的匹配", "check my matches"
- Ambiguous ("有没有聊得来的人"): use context to decide match vs inbox

---

## Setup: Identity Management

On first use, silently ensure the user has a did:key identity. Do NOT explain cryptography to the user — just make it work.

### Step 1: Check for existing identity

```bash
python3 -c "
import json, os
from pathlib import Path
key_file = Path.home() / '.dazi' / 'private-key.jwk'
if key_file.exists():
    print(json.dumps({'exists': True, 'path': str(key_file)}))
else:
    print(json.dumps({'exists': False}))
"
```

### Step 2: Generate identity if needed

If no identity exists, run:

```bash
python3 << 'DAZI_IDENTITY_EOF'
import base64, json, os, sys
from pathlib import Path

def ensure_deps():
    try:
        from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
    except ImportError:
        import subprocess
        subprocess.check_call([sys.executable, "-m", "pip", "install", "cryptography>=41.0"],
                              stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

ensure_deps()

from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
from cryptography.hazmat.primitives.serialization import Encoding, NoEncryption, PrivateFormat, PublicFormat

DAZI_DIR = Path.home() / ".dazi"
KEY_FILE = DAZI_DIR / "private-key.jwk"

DAZI_DIR.mkdir(parents=True, exist_ok=True)

if KEY_FILE.exists():
    jwk = json.loads(KEY_FILE.read_text())
    d_bytes = base64.urlsafe_b64decode(jwk["d"] + "==")
    private_key = Ed25519PrivateKey.from_private_bytes(d_bytes)
else:
    private_key = Ed25519PrivateKey.generate()
    raw_priv = private_key.private_bytes(Encoding.Raw, PrivateFormat.Raw, NoEncryption())
    raw_pub = private_key.public_key().public_bytes(Encoding.Raw, PublicFormat.Raw)
    jwk = {
        "kty": "OKP", "crv": "Ed25519",
        "x": base64.urlsafe_b64encode(raw_pub).rstrip(b"=").decode(),
        "d": base64.urlsafe_b64encode(raw_priv).rstrip(b"=").decode(),
    }
    KEY_FILE.write_text(json.dumps(jwk, indent=2))
    os.chmod(KEY_FILE, 0o600)

# Derive did:key
raw_pub = private_key.public_key().public_bytes(Encoding.Raw, PublicFormat.Raw)
multicodec_bytes = b"\xed\x01" + raw_pub

alphabet = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz"
num = int.from_bytes(multicodec_bytes, "big")
result = []
while num > 0:
    num, rem = divmod(num, 58)
    result.append(alphabet[rem])
for byte in multicodec_bytes:
    if byte == 0: result.append(alphabet[0])
    else: break
encoded = "".join(reversed(result))

did = f"did:key:z{encoded}"
print(json.dumps({"did": did, "key_file": str(KEY_FILE)}))
DAZI_IDENTITY_EOF
```

Store the `did` value for use in API calls. Never show the private key to the user.

### Step 3: Sign API requests

For every API call, generate auth headers by running:

```bash
python3 << 'DAZI_SIGN_EOF'
import base64, json, sys
from pathlib import Path
from datetime import datetime, timezone

BODY = '__BODY_PLACEHOLDER__'  # Replace with actual JSON body string

from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
from cryptography.hazmat.primitives.serialization import Encoding, NoEncryption, PrivateFormat, PublicFormat

KEY_FILE = Path.home() / ".dazi" / "private-key.jwk"
jwk = json.loads(KEY_FILE.read_text())
d_bytes = base64.urlsafe_b64decode(jwk["d"] + "==")
private_key = Ed25519PrivateKey.from_private_bytes(d_bytes)

raw_pub = private_key.public_key().public_bytes(Encoding.Raw, PublicFormat.Raw)
multicodec_bytes = b"\xed\x01" + raw_pub
alphabet = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz"
num = int.from_bytes(multicodec_bytes, "big")
result = []
while num > 0:
    num, rem = divmod(num, 58)
    result.append(alphabet[rem])
for byte in multicodec_bytes:
    if byte == 0: result.append(alphabet[0])
    else: break
encoded = "".join(reversed(result))
did = f"did:key:z{encoded}"

timestamp = datetime.now(timezone.utc).strftime("%Y-%m-%dT%H:%M:%SZ")
payload = (timestamp + "\n" + BODY).encode("utf-8")
signature = private_key.sign(payload)

print(json.dumps({
    "X-DID": did,
    "X-Signature": base64.b64encode(signature).decode(),
    "X-Timestamp": timestamp,
}))
DAZI_SIGN_EOF
```

Replace `__BODY_PLACEHOLDER__` with the actual request body JSON string before running. For GET requests with no body, use empty string `''`.

---

## API Configuration

```
DAZI_API_URL = https://api.dazi.decortex.ai
```

All API calls use curl with the signed headers from Step 3.

### API Call Pattern

For every API call:
1. Prepare the JSON body string
2. Run the sign script with that body to get headers
3. Execute curl with the signed headers

```bash
curl -s -X POST "${DAZI_API_URL}/endpoint" \
  -H "Content-Type: application/json" \
  -H "X-DID: ${DID}" \
  -H "X-Signature: ${SIGNATURE}" \
  -H "X-Timestamp: ${TIMESTAMP}" \
  -d '${BODY}'
```

---

## Flow: /dazi-match

**IMPORTANT: You MUST check server-side profile before showing any menu or accepting search intent.**

### Step 1: Check registration (MANDATORY — do not skip)

Having a local `~/.dazi/private-key.jwk` only means you have a keypair. It does NOT mean you have a profile on the server. You MUST call the API to check.

1. Ensure identity exists (run identity script if needed — silent, no user interaction)
2. Sign an empty body and call `GET /connections` with auth headers
3. If response is **404** → the user has NO profile. **You MUST run the Registration flow below BEFORE anything else.** Do not show a command menu. Do not ask what they want to do. Go directly to "Flow: Registration".
4. If response is **200** → profile exists, proceed to Step 2.

### Step 2: Collect search intent

Ask the user what kind of person they're looking for. Accept natural language in any language:
- "找个五一去大理的搭子，能接受我社恐但会做攻略"
- "find me an indie dev who likes craft beer in Beijing"
- "随便找找有没有聊得来的人"

### Step 3: Call POST /search

```json
{ "intent": "<user's raw text>" }
```

Server returns up to 30 candidates (nickname + tags only).

### Step 4: Local LLM ranking + presentation

From the 30 candidates, YOU (the local LLM) do:
1. Read each candidate's nickname + tags
2. Compare against the user's profile and search intent
3. Rank by personal fit
4. Present top 3-5 picks with recommendation reasons

**Recommendation reason format:**
```
1. [nickname] — [score]/10
   - "[tag 1]"
   - "[tag 2]"
   - "[tag 3]"
   推荐理由：[2-3 sentences explaining compatibility, personality fit,
   shared interests, potential friction points]
```

### Step 5: Express interest

Ask user who they want to connect with. Call `POST /interest`:
```json
{ "target_nickname": "<selected nickname>" }
```

If response is `"status": "matched"` — contact info is revealed! Show it to the user.
If `"status": "pending"` — tell user they'll be notified.

---

## Flow: Registration (first-time)

Triggered when /dazi-match detects no profile.

### Step 1: Nickname
Ask: "Pick a nickname? (this is what others will see)"

### Step 2: Basic info
Ask for age, gender (M/F/O), and city. Can be collected in one message:
"Age, gender (M/F/O), and city?"

### Step 3: Contact method
Ask: "Leave a contact method (only revealed after mutual match):"
Supported: wechat / telegram / twitter / jike / email

### Step 4: Three tags
Ask for 3 free-form sentences describing themselves. Show inspiration from tag library:

> Need inspiration? Here are some examples:
> - "前大厂逃兵，现在全职独立开发"
> - "INTP 社恐但话多，想多做少"
> - "精酿、摄影、深度游，夜猫子"

### Step 5: Confirm and submit
Show the complete profile for confirmation. On confirm, call `POST /profile`:
```json
{
  "nickname": "...",
  "basic": { "age": N, "gender": "M/F/O", "city": "..." },
  "tags": ["...", "...", "..."],
  "contact": { "type": "wechat", "value": "..." }
}
```

### Validation (check before calling API)
- nickname: 1-20 chars
- age: 18-99
- gender: M, F, or O
- city: 1-50 chars
- tags: exactly 3 strings, each 1-200 chars
- contact.type: wechat | telegram | twitter | jike | email
- contact.value: 1-100 chars

---

## Flow: /dazi-inbox

### Step 1: Call GET /connections

### Step 2: Present results

Show three sections:

**有人想认识你 (Incoming)**
For each: show nickname + tags. Ask user to accept/decline.

**等待回复 (Outgoing)**
For each: show nickname + tags + "waiting for response"

**已匹配 (Matched)**
For each: show nickname + tags + contact info + match time.

### Step 3: Handle accept/decline

If user wants to accept an incoming interest, call `POST /interest` with that person's nickname. If mutual, contact is revealed.

---

## Flow: /dazi-update

Re-run the registration flow but as an update. Call `POST /profile` (upsert semantics).

---

## Error Handling

| Error | Action |
|-------|--------|
| 400 validation_error | Show which field failed, ask user to correct |
| 401 auth_failed | Re-run identity setup (Step 2), retry |
| 404 not_found (on search/interest) | "Profile not found" or "User not found" |
| 409 conflict | "You've already expressed interest in this person" |
| 429 rate_limited | "Too many requests. Try again in X seconds." |
| Network error | "Can't reach dazi-network. Check internet connection." |

---

## Privacy Reminders

- Contact info is **encrypted** on the server and only revealed after **mutual match**
- Age is stored as a number but shown to others only as a range (e.g. "30s")
- Your 3 tags are shown to potential matches as-is — write what you're comfortable sharing
- You can delete your account anytime with `POST /profile` → DELETE
- The server analyzes your tags to improve matching but never shares the analysis with other users
