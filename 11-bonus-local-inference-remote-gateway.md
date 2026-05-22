# 11, Bonus, Local Inference and Remote Gateway

This is an optional advanced guide.

Do not do this during your first Hermes setup.

First make the normal Hermes Agent setup work with Telegram, automations, and a hosted model provider.

Then come back to this if you want to experiment with a local sidecar that handles simple routing decisions, while a remote model still handles deeper reasoning.

## What this setup does

This guide builds a small local gateway.

The local gateway can:

- Ask a small local Ollama model to choose a likely tool for routine requests.
- Send complex requests to a remote OpenAI-compatible gateway such as OpenRouter.
- Keep the local service bound to `127.0.0.1` so it is not public.
- Return a routing decision, not execute dangerous actions by itself.

That last point matters.

Do not give a small local model automatic permission to read, write, delete, deploy, or run shell commands. Let Hermes keep its normal approval and tool safety flow.

## How this fits with Hermes

Hermes already supports many model providers, including hosted providers, custom OpenAI-compatible endpoints, and fallback providers.

Hermes can also expose its own OpenAI-compatible API server.

This guide is not replacing those features.

Use this sidecar only if you want to test a local "front desk" pattern:

- Local model: cheap routing and simple tool-plan suggestions.
- Remote model: hard reasoning, long context, planning, coding, and final answers.
- Hermes: the agent that actually owns tools, approvals, memory, and workflows.

For most users, the simpler option is still:

```bash
hermes model
```

Then choose a capable hosted provider.

## When this is worth trying

Try this if:

- Your normal Hermes setup already works.
- You are comfortable running Python services.
- Your VPS has enough RAM for a local model.
- You want to test local routing before calling a remote model.
- You understand that local models may choose the wrong tool.

Do not try this if:

- You are still setting up Hermes for the first time.
- You only have a 1 GB or 2 GB VPS.
- You expect a tiny model to replace a capable remote model.
- You want fully automatic command execution.
- You are not comfortable reviewing service logs.

## Requirements

Use a development VPS or local machine with:

- Ubuntu 24.04 LTS or similar.
- 4 GB RAM minimum for small local models.
- 8 GB RAM preferred.
- Python 3.10 or newer.
- Ollama.
- A remote model provider API key, such as OpenRouter.
- A working Hermes Agent setup.

No GPU is required for a small test, but CPU inference may be slow.

## Step 1, Install basic tools

Connect to your VPS as the `hermes` user.

Then run:

```bash
sudo apt update
sudo apt install -y curl python3 python3-pip python3-venv
```

## Step 2, Install Ollama

Run:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Start and check Ollama:

```bash
sudo systemctl start ollama
sudo systemctl status ollama
ollama -v
```

If your install does not use systemd, you can start it manually:

```bash
ollama serve
```

Keep that terminal open while testing.

## Step 3, Pull a local model

Use a model that supports tool calling.

This guide uses `qwen3:4b` because the Ollama tool-calling docs use Qwen 3 examples.

Run:

```bash
ollama pull qwen3:4b
```

Check that it is available:

```bash
ollama list
```

If the model is too slow for your server, pick a smaller model only after testing that it can return reliable JSON or tool-call decisions.

Do not assume every small model supports tool calling well.

## Step 4, Test Ollama directly

Run:

```bash
curl http://127.0.0.1:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen3:4b",
    "stream": false,
    "messages": [
      {"role": "user", "content": "Reply with JSON: {\"ok\": true}"}
    ],
    "format": "json"
  }'
```

If Ollama is working, you should get a JSON response from the model server.

## Step 5, Create the local gateway folder

Run:

```bash
mkdir -p ~/hermes_local_gateway
cd ~/hermes_local_gateway
```

## Step 6, Create a Python environment

Run:

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -U pip
pip install fastapi uvicorn pydantic requests
```

When you come back later, activate it with:

```bash
cd ~/hermes_local_gateway
. .venv/bin/activate
```

## Step 7, Add remote gateway settings

Create a local `.env` file:

```bash
cat > ~/hermes_local_gateway/.env <<'EOF'
LOCAL_MODEL=qwen3:4b
OPENROUTER_API_KEY=replace-this-with-your-key
OPENROUTER_MODEL=replace-this-with-your-openrouter-model
EOF

chmod 600 ~/hermes_local_gateway/.env
```

Use a model you already know works through your provider.

Do not commit this file to Git.

## Step 8, Create the gateway script

Create the sidecar service:

```bash
cat > ~/hermes_local_gateway/local_gateway.py <<'PY'
#!/usr/bin/env python3
from __future__ import annotations

import json
import os
from pathlib import Path
from typing import Any

import requests
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

BASE_DIR = Path.home() / "hermes_local_gateway"
ENV_FILE = BASE_DIR / ".env"


def load_env_file() -> None:
    if not ENV_FILE.exists():
        return

    for line in ENV_FILE.read_text().splitlines():
        line = line.strip()
        if not line or line.startswith("#") or "=" not in line:
            continue
        key, value = line.split("=", 1)
        os.environ.setdefault(key.strip(), value.strip().strip('"').strip("'"))


load_env_file()

OLLAMA_BASE_URL = os.getenv("OLLAMA_BASE_URL", "http://127.0.0.1:11434")
OPENROUTER_BASE_URL = os.getenv("OPENROUTER_BASE_URL", "https://openrouter.ai/api/v1")
LOCAL_MODEL = os.getenv("LOCAL_MODEL", "qwen3:4b")
OPENROUTER_API_KEY = os.getenv("OPENROUTER_API_KEY", "")
OPENROUTER_MODEL = os.getenv("OPENROUTER_MODEL", "")


def remote_gateway_ready() -> bool:
    return bool(
        OPENROUTER_API_KEY
        and OPENROUTER_API_KEY != "replace-this-with-your-key"
        and OPENROUTER_MODEL
        and OPENROUTER_MODEL != "replace-this-with-your-openrouter-model"
    )

ALLOWED_TOOLS = {
    "file_read",
    "file_search",
    "server_status",
    "memory_search",
    "web_search",
    "none",
}

LOCAL_HINTS = (
    "file",
    "directory",
    "find",
    "search",
    "list",
    "check",
    "status",
    "memory",
    "log",
    "note",
)


class RouteRequest(BaseModel):
    prompt: str
    available_tools: list[str] = Field(default_factory=lambda: sorted(ALLOWED_TOOLS))


class ChatRequest(BaseModel):
    prompt: str
    context: str | None = None
    force_remote: bool = False


app = FastAPI(title="Hermes Local Inference Gateway")


def should_try_local(prompt: str) -> bool:
    prompt_lower = prompt.lower()
    return any(hint in prompt_lower for hint in LOCAL_HINTS)


def parse_json_response(text: str) -> dict[str, Any]:
    try:
        parsed = json.loads(text)
    except json.JSONDecodeError as exc:
        raise HTTPException(status_code=502, detail=f"Local model did not return JSON: {exc}") from exc

    if not isinstance(parsed, dict):
        raise HTTPException(status_code=502, detail="Local model returned JSON, but not an object")

    return parsed


def call_ollama(messages: list[dict[str, str]], *, json_mode: bool = False) -> dict[str, Any]:
    payload: dict[str, Any] = {
        "model": LOCAL_MODEL,
        "messages": messages,
        "stream": False,
    }
    if json_mode:
        payload["format"] = "json"

    try:
        response = requests.post(
            f"{OLLAMA_BASE_URL}/api/chat",
            json=payload,
            timeout=60,
        )
    except requests.RequestException as exc:
        raise HTTPException(status_code=503, detail=f"Ollama is not reachable: {exc}") from exc

    if response.status_code >= 400:
        raise HTTPException(status_code=502, detail=response.text)

    return response.json()


def call_remote(prompt: str, context: str | None = None) -> dict[str, Any]:
    if not OPENROUTER_API_KEY or OPENROUTER_API_KEY == "replace-this-with-your-key":
        raise HTTPException(status_code=503, detail="OPENROUTER_API_KEY is not configured")
    if not OPENROUTER_MODEL or OPENROUTER_MODEL == "replace-this-with-your-openrouter-model":
        raise HTTPException(status_code=503, detail="OPENROUTER_MODEL is not configured")

    messages = [{"role": "system", "content": "You are a careful assistant."}]
    if context:
        messages.append({"role": "system", "content": f"Context:\n{context}"})
    messages.append({"role": "user", "content": prompt})

    try:
        response = requests.post(
            f"{OPENROUTER_BASE_URL}/chat/completions",
            headers={
                "Authorization": f"Bearer {OPENROUTER_API_KEY}",
                "Content-Type": "application/json",
            },
            json={
                "model": OPENROUTER_MODEL,
                "messages": messages,
                "max_tokens": 1000,
            },
            timeout=60,
        )
    except requests.RequestException as exc:
        raise HTTPException(status_code=503, detail=f"Remote gateway is not reachable: {exc}") from exc

    if response.status_code >= 400:
        raise HTTPException(status_code=502, detail=response.text)

    return response.json()


@app.get("/health")
def health() -> dict[str, Any]:
    ollama_ok = False
    try:
        response = requests.get(f"{OLLAMA_BASE_URL}/api/tags", timeout=5)
        ollama_ok = response.status_code < 400
    except requests.RequestException:
        ollama_ok = False

    return {
        "ok": True,
        "ollama_ok": ollama_ok,
        "local_model": LOCAL_MODEL,
        "remote_gateway_ready": remote_gateway_ready(),
    }


@app.post("/route")
def route_request(request: RouteRequest) -> dict[str, Any]:
    allowed = sorted(set(request.available_tools) & ALLOWED_TOOLS)
    if not allowed:
        allowed = ["none"]

    system_prompt = (
        "You choose the safest tool plan for Hermes Agent. "
        "Return only JSON with keys: tool, arguments, confidence, reason. "
        f"Allowed tools: {allowed}. "
        "If the request needs reasoning, planning, coding, deployment, secrets, "
        "or a risky action, choose tool 'none'."
    )

    ollama_response = call_ollama(
        [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": request.prompt},
        ],
        json_mode=True,
    )

    content = ollama_response.get("message", {}).get("content", "{}")
    decision = parse_json_response(content)
    tool = str(decision.get("tool", "none"))

    if tool not in allowed:
        decision = {
            "tool": "none",
            "arguments": {},
            "confidence": 0,
            "reason": f"Rejected unsupported tool: {tool}",
        }

    return {
        "source": "local_router",
        "routing_decision": decision,
    }


@app.post("/chat")
def chat(request: ChatRequest) -> dict[str, Any]:
    if not request.force_remote and should_try_local(request.prompt):
        route = route_request(RouteRequest(prompt=request.prompt))
        return {
            "source": "local_router",
            "note": "This is a routing suggestion only. Hermes should still apply normal tool approvals.",
            "data": route,
        }

    return {
        "source": "remote_gateway",
        "data": call_remote(request.prompt, request.context),
    }
PY

chmod +x ~/hermes_local_gateway/local_gateway.py
```

This script does not execute tools.

It only suggests a tool plan or forwards complex prompts to the remote gateway.

## Step 9, Start the local gateway

Run:

```bash
cd ~/hermes_local_gateway
. .venv/bin/activate
uvicorn local_gateway:app --host 127.0.0.1 --port 8080
```

Keep this terminal open while testing.

The gateway should only listen on `127.0.0.1`.

Do not bind it to `0.0.0.0` unless you have a firewall, authentication, and a clear reason.

## Step 10, Test the health endpoint

In another terminal, run:

```bash
curl http://127.0.0.1:8080/health
```

You should see:

```json
{
  "ok": true,
  "ollama_ok": true,
  "local_model": "qwen3:4b",
  "remote_gateway_ready": true
}
```

If `remote_gateway_ready` is false, edit `~/hermes_local_gateway/.env` and set `OPENROUTER_API_KEY` and `OPENROUTER_MODEL`.

## Step 11, Test local routing

Run:

```bash
curl http://127.0.0.1:8080/route \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Find nginx error logs on this server",
    "available_tools": ["file_search", "server_status", "web_search"]
  }'
```

The response should include:

```json
{
  "source": "local_router",
  "routing_decision": {
    "tool": "file_search"
  }
}
```

The exact reason and confidence may vary.

## Step 12, Test remote fallback

This requires a valid OpenRouter key and model in `~/hermes_local_gateway/.env`.

Run:

```bash
curl http://127.0.0.1:8080/chat \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explain the tradeoffs between local inference and remote hosted reasoning for an AI agent.",
    "force_remote": true
  }'
```

You should see a normal remote model response under `data`.

## Step 13, Use it from Hermes

You can ask Hermes to query the sidecar before deciding what to do.

Example:

```text
Before answering, call http://127.0.0.1:8080/route with this request: "Find the latest nginx error log". Treat the result as a routing suggestion only. Do not run any command without normal approval.
```

For most real workflows, Hermes should still use its own tools directly.

This gateway is useful when you want to experiment with local pre-routing, not when you want to bypass Hermes safety.

## Step 14, Optional, run it as a user service

After the manual test works, create a systemd user service:

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/hermes-local-gateway.service <<'EOF'
[Unit]
Description=Hermes Local Inference Gateway
After=network.target

[Service]
Type=simple
WorkingDirectory=%h/hermes_local_gateway
ExecStart=%h/hermes_local_gateway/.venv/bin/uvicorn local_gateway:app --host 127.0.0.1 --port 8080
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
EOF

systemctl --user daemon-reload
systemctl --user enable hermes-local-gateway
systemctl --user start hermes-local-gateway
```

Check it:

```bash
systemctl --user status hermes-local-gateway
curl http://127.0.0.1:8080/health
```

## Maintenance commands

Update Ollama:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Update the local model:

```bash
ollama pull qwen3:4b
```

List local models:

```bash
ollama list
```

Remove the local model:

```bash
ollama rm qwen3:4b
```

Check Ollama logs:

```bash
journalctl -e -u ollama
```

Check the user service logs:

```bash
journalctl --user -u hermes-local-gateway -e
```

Stop the user service:

```bash
systemctl --user stop hermes-local-gateway
```

## Troubleshooting

If Ollama is not responding:

```bash
sudo systemctl status ollama
curl http://127.0.0.1:11434/api/tags
```

If the local gateway is not responding:

```bash
cd ~/hermes_local_gateway
. .venv/bin/activate
uvicorn local_gateway:app --host 127.0.0.1 --port 8080
```

If local routing returns bad JSON:

- Try the same prompt again.
- Use a model with better tool-calling support.
- Make the available tool list smaller and clearer.
- Route the request to the remote model instead.

If remote gateway calls fail:

- Check `OPENROUTER_API_KEY`.
- Check `OPENROUTER_MODEL`.
- Test your provider account and billing.
- Make sure the model name still exists on your provider.

If local responses are slow:

- Use a smaller tool-capable model.
- Add more RAM or CPU.
- Use local routing only for narrow, routine requests.
- Let Hermes call the remote provider for hard tasks.

## Practical recommendation

Use local inference only for cheap routing suggestions.

Use the remote model for reasoning, coding, planning, and final answers.

Keep Hermes approvals enabled. Keep the sidecar private. Treat every local routing decision as a suggestion, not an instruction.

## Sources

- Hermes AI Providers: https://hermes-agent.nousresearch.com/docs/integrations/providers
- Hermes Fallback Providers: https://hermes-agent.nousresearch.com/docs/user-guide/features/fallback-providers
- Hermes API Server: https://hermes-agent.nousresearch.com/docs/user-guide/features/api-server/
- Ollama Linux install: https://docs.ollama.com/linux
- Ollama tool calling: https://docs.ollama.com/capabilities/tool-calling
- Ollama chat API: https://docs.ollama.com/api/chat
- OpenRouter chat completions: https://openrouter.ai/docs/api-reference/chat-completion
