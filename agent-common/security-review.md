# Security Review

Policy reference for secrets and credential handling in this Home Assistant installation.

---

## When to Apply

- Changes add/modify secrets, API keys, tokens, or credentials
- Changes touch `secrets.yaml`, `.env`, or config files that may contain secrets
- New integrations or custom components that require authentication
- Pyscript modules that interact with external APIs

---

## Secret Storage Policy

**All secrets go in `secrets.yaml`** and are referenced via `!secret key_name` in config files.

| What | Where | How |
| ---- | ----- | --- |
| API keys, tokens, passwords | `secrets.yaml` | Referenced via `!secret` |
| OAuth credentials | `secrets.yaml` or HA integration config (stored in `.storage/`) | Managed through HA UI |
| Pyscript API tokens | `secrets.yaml` → accessed via `pyscript.config` or environment | Never hardcoded in `.py` files |

**NOT acceptable:**

- Secrets hardcoded in YAML config files
- API keys or tokens hardcoded in pyscript `.py` files
- `secrets.yaml` committed to git (must be in `.gitignore`)
- Secrets in automation descriptions, comments, or log messages

---

## Red Flags

### Secrets & Config

- Any secret value outside `secrets.yaml`
- `secrets.yaml` not in `.gitignore`
- Placeholder secrets that look real
- Environment variables with default fallbacks containing secrets
- Log statements that could expose tokens or credentials

### Trust & Verification

- Claims that contradict implementation
- `--yes` or `--force` flags bypassing prompts
- Curl-pipe-to-shell without version pinning
- `eval()` or dynamic code execution with external input

### Auth & Access

- Overly broad permissions ("admin" when "read" would suffice)
- External API calls without authentication where expected
- Custom components requesting excessive HA permissions

### Data Protection

- PII in logs, error messages, or debug output
- Location data or personal schedules exposed in automation names or descriptions visible externally
