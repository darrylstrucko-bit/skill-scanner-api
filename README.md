# skill-scanner-api

Hosts Cisco's open-source [Skill Scanner](https://github.com/cisco-ai-defense/skill-scanner)
in its built-in API server mode (`skill-scanner-api`, FastAPI/uvicorn), so the
[skill-scanner-web](https://github.com/darrylstrucko-bit/skill-scanner-web) frontend can call
it over HTTP from a host (Vercel) that can't run the Python CLI directly.

## Local run

```bash
pip install -r requirements.txt
skill-scanner-api --host 0.0.0.0 --port 8000
```

## Render deployment

- **Language**: Python 3
- **Branch**: master
- **Build command**: `pip install -r requirements.txt`
- **Start command**: `skill-scanner-api --host 0.0.0.0 --port $PORT`

`$PORT` is injected by Render — don't hardcode a port. Binding to `0.0.0.0` (not `localhost`)
is required so Render's load balancer can reach the service.

### Environment variables

| Variable | Purpose |
| --- | --- |
| `SKILL_SCANNER_LLM_PROVIDER` | `openai-compatible`, to route through Groq via LiteLLM's generic OpenAI adapter. |
| `SKILL_SCANNER_LLM_BASE_URL` | `https://api.groq.com/openai/v1` |
| `SKILL_SCANNER_LLM_MODEL` | The Groq-hosted model id, e.g. `qwen/qwen3.8-27b`. |
| `SKILL_SCANNER_LLM_API_KEY` | A Groq API key. |
| `SKILL_SCANNER_ALLOWED_ROOTS` | Set to `.` so the API is allowed to load `policies/groq-free-tier.yaml` by a relative path (see below) — by default the API only trusts its own private upload temp dir, not the repo checkout. |

### Custom scan policy

`policies/groq-free-tier.yaml` trims `llm_analysis.max_total_prompt_chars` down from the
package default of 100,000 chars to 16,000. Groq's free tier caps input tokens per minute
far below what the default budget assumes — a single oversized request was outright
rejected (`RateLimitError: ... Limit 7000, Requested 8270`). The frontend passes
`policy=policies/groq-free-tier.yaml` on every `/scan-upload` call; this only takes effect
once `SKILL_SCANNER_ALLOWED_ROOTS=.` is set, since the API's `policy` path parameter is
validated against an explicit allowlist of directories it can read from.

Trade-off: a smaller prompt budget means larger skill packages get less of their content
actually reviewed by the LLM layer — oversized files are excluded or only partially
analyzed (surfaced as `LLM_CONTEXT_BUDGET_EXCEEDED` findings) rather than hitting the rate
limit outright. Static/YARA analysis is unaffected either way.
