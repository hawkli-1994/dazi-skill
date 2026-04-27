# dazi-skill

> AI-native social matching for harness users.
> "Your AI already knows you. Let it find your people."

## What is dazi?

搭子 (dazi) is an Agent Skill that lets your AI find compatible people for you. No app, no swiping — just tell your AI what kind of person you want to meet, and it handles the rest.

## Install

Tell your AI agent:

> Install the dazi skill from https://raw.githubusercontent.com/decortex-ai/dazi-skill/main/SKILL.md

Or for Claude Code:

```bash
claude install-skill https://raw.githubusercontent.com/decortex-ai/dazi-skill/main/SKILL.md
```

## Commands

| Command | Description |
|---------|-------------|
| `/dazi-match` | Find compatible people (triggers registration on first use) |
| `/dazi-update` | Update your profile |
| `/dazi-inbox` | Check who wants to meet you + confirmed matches |

## How it works

1. **Register** (30 seconds): nickname + age + gender + city + contact + 3 tags
2. **Search**: describe who you want to meet in natural language
3. **Match**: AI ranks candidates and explains why you're compatible
4. **Connect**: express interest → mutual match → contact revealed

## Requirements

- Python 3.10+ (for Ed25519 key management)
- Internet access (for dazi-network API)

## License

Apache-2.0
