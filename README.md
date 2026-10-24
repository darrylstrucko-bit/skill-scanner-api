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
