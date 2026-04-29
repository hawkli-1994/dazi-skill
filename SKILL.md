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

## ENTRY POINT — Execute This Immediately on Activation

**Do NOT show a command table. Do NOT ask "what do you want to do". Execute the steps below.**

### Step A: Setup identity (silent — no user-visible output)

Run the identity generation script below to ensure `~/.dazi/private-key.jwk` exists. Do not tell the user about cryptography or keys.

### Step B: Check server profile (MANDATORY)

Having a local keypair does NOT mean the user has a profile. You MUST call the server:

1. Sign an empty body (`''`) using the signing script
2. Call `GET ${DAZI_API_URL}/connections` with the signed auth headers
3. Check the response:
   - **404** → User has NO profile. Go directly to **"Flow: Registration"** below. Do NOT show any menu first.
   - **200** → Profile exists. Proceed to Step C.
   - **Network error** → Tell user "dazi-network 暂时无法连接" and stop.

### Step C: Route to the right flow

Determine what the user wants based on their input:

| User input | Action |
|------------|--------|
| `/dazi-match`, `/dazi`, "帮我找搭子", "find me a dazi", "推荐几个搭子" | → Go to **Flow: /dazi-match** |
| `/dazi-inbox`, "有人想认识我吗", "有没有新的匹配", "check my matches" | → Go to **Flow: /dazi-inbox** |
| `/dazi-update`, "更新资料" | → Go to **Flow: /dazi-update** |
| Ambiguous or just `/dazi` with no clear intent | → Ask: "想找搭子，还是看看收件箱？" |

---

## User Intent Routing (internal — do not show this table to users)

Users will express intent in natural language. Route accordingly:

| User says | Flow |
|-----------|------|
| "帮我找搭子", "想认识...", "find someone who...", any search intent | → /dazi-match |
| "有人想认识我吗", "查看收件箱", "check inbox", "有没有新匹配" | → /dazi-inbox |
| "更新资料", "改一下标签", "update profile" | → /dazi-update |

**NEVER tell users to type `/dazi-match` or any slash command.** Users should describe what they want naturally. The skill name `/dazi` is the only command they need to know.

---

## Setup: Identity Management

On first use, silently ensure the user has a did:key identity. Do NOT explain cryptography to the user — just make it work.

### Key file lookup (IMPORTANT)

For all scripts below, resolve the private key file in this order:
1. `DAZI_KEY_FILE` env var (absolute or relative path)
2. `./.dazi/private-key.jwk` (only if the `./.dazi/` directory exists)
3. `~/.dazi/private-key.jwk` (fallback)

### Step 1: Check for existing identity

```bash
python3 -c "
import json, os
from pathlib import Path
def resolve_key_file():
    env_path = os.environ.get('DAZI_KEY_FILE')
    if env_path:
        return Path(env_path).expanduser()
    cwd_dir = Path.cwd() / '.dazi'
    if cwd_dir.exists():
        return cwd_dir / 'private-key.jwk'
    return Path.home() / '.dazi' / 'private-key.jwk'

key_file = resolve_key_file()
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

def resolve_key_file():
    env_path = os.environ.get("DAZI_KEY_FILE")
    if env_path:
        return Path(env_path).expanduser()
    cwd_dir = Path.cwd() / ".dazi"
    if cwd_dir.exists():
        return cwd_dir / "private-key.jwk"
    return Path.home() / ".dazi" / "private-key.jwk"

KEY_FILE = resolve_key_file()

KEY_FILE.parent.mkdir(parents=True, exist_ok=True)

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
import os

BODY = '__BODY_PLACEHOLDER__'  # Replace with actual JSON body string

from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
from cryptography.hazmat.primitives.serialization import Encoding, NoEncryption, PrivateFormat, PublicFormat

def resolve_key_file():
    env_path = os.environ.get("DAZI_KEY_FILE")
    if env_path:
        return Path(env_path).expanduser()
    cwd_dir = Path.cwd() / ".dazi"
    if cwd_dir.exists():
        return cwd_dir / "private-key.jwk"
    return Path.home() / ".dazi" / "private-key.jwk"

KEY_FILE = resolve_key_file()
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
DAZI_API_URL = https://dazi-network-production.up.railway.app
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

**BLOCKER: If the user has not registered (no profile on server), you CANNOT search. You MUST register first.**

### Step 1: Verify registration (MANDATORY — never skip, never assume)

If you have NOT already verified profile existence in this session via a successful API call, you MUST do it now:

1. Ensure identity exists (run identity script if needed — silent)
2. Sign an empty body and call `GET ${DAZI_API_URL}/connections` with auth headers
3. Check response:
   - **404** → **STOP. Do not proceed to search.** Run "Flow: Registration" first. After registration completes, come back here and continue to Step 2.
   - **200** → Profile confirmed. Continue to Step 2.
   - **Network error** → Tell user and stop.

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

Ask user who they want to connect with. Before sending, generate a short message (1-2 sentences) explaining why the user wants to connect — based on the recommendation reason you generated. This message will be visible to the other person.

Call `POST /interest`:
```json
{
  "target_nickname": "<selected nickname>",
  "message": "<1-2 sentence reason, e.g. '都是独立开发者，都爱精酿，想聊聊'>"
}
```

If response is `"status": "matched"` — contact info is revealed! Show it to the user.
If `"status": "pending"` — tell user they'll be notified.

---

## Flow: Registration (first-time)

Triggered when /dazi-match detects no profile.

### Step 1: Nickname
Ask: "Pick a nickname? (this is what others will see)"

### Step 2: Basic info
Ask for birth year, gender (M/F/O), and city. Can be collected in one message:
"出生年份、性别（M/F/O）、城市？"

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
  "basic": { "birth_year": N, "gender": "M/F/O", "city": "..." },
  "tags": ["...", "...", "..."],
  "contact": { "type": "wechat", "value": "..." }
}
```

### Validation (check before calling API)
- nickname: 1-20 chars
- birth_year: 1920-2010
- gender: M, F, or O
- city: 1-50 chars
- tags: exactly 3 strings, each 1-200 chars
- contact.type: wechat | telegram | twitter | jike | email
- contact.value: 1-100 chars

### Step 6: Post-registration guidance

After successful registration, DO NOT mention slash commands. Instead say something like:

> 注册成功！你现在可以直接告诉我你想找什么样的搭子，比如：
> - "想认识喜欢 vibe coding 的小伙伴"
> - "找个北京一起喝精酿的人"
> - "随便看看有没有聊得来的"
>
> 也可以随时问我"有没有人想认识我"来查看收件箱。

Then immediately ask the user what kind of person they want to meet, and proceed to Flow: /dazi-match Step 2.

---

## Flow: /dazi-inbox

### Step 1: Call GET /connections

### Step 2: Present results

Show four sections:

**有人想认识你 (Incoming)**
For each: show nickname + tags + message (the reason they want to meet you). Then YOU (the local LLM) should read the incoming person's tags + message, compare with the user's own tags, and generate a brief compatibility analysis to help the user decide. Ask user to accept or decline.

Display format:
```
[nickname] 想认识你
  - "[tag 1]"
  - "[tag 2]"
  - "[tag 3]"
  对方说："[message]"
  AI 分析：[1-2 sentences about compatibility based on both profiles]
  → 接受 / 拒绝？
```

**等待回复 (Outgoing)**
For each: show nickname + tags + your message + "waiting for response". Offer to withdraw.

**被拒绝 (Declined)**
For each: show nickname + tags. These are your outgoing interests that were declined.

**已匹配 (Matched)**
For each: show nickname + tags + contact info + match time.

### Step 3: Handle accept/decline/withdraw

- **Accept**: call `POST /interest` with `{"target_nickname": "...", "action": "accept"}`. If mutual, contact is revealed.
- **Decline**: call `POST /interest` with `{"target_nickname": "...", "action": "decline"}`. The other person will see the decline.
- **Withdraw**: call `POST /interest` with `{"target_nickname": "...", "action": "withdraw"}`. Removes your outgoing interest.

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
