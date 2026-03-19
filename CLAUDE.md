# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pipecat AI voice agent for outbound phone calls using a cascade pipeline (STT → LLM → TTS). Uses Daily for PSTN transport, Deepgram for speech services, and AWS Bedrock for the LLM. All source code lives in `server/`.

## Commands

All commands run from the `server/` directory:

```bash
# Install dependencies
uv sync

# Start the webhook server (port 8080) - handles dial-out requests
uv run server.py

# Start the bot server (port 7860) - handles conversation logic
uv run bot.py -t daily

# Lint (import sorting only)
uv run ruff check .

# Lint with auto-fix
uv run ruff check --fix .

# Type check
uv run pyright
```

Both servers must be running simultaneously for local development. Test with:
```bash
curl -X POST "http://localhost:8080/dialout" \
  -H "Content-Type: application/json" \
  -d '{"dialout_settings": {"phone_number": "+1234567890"}}'
```

## Architecture

**Two-process local architecture:**
- `server.py` — FastAPI webhook server. Receives `/dialout` requests, creates Daily rooms via `server_utils.py`, then starts the bot by calling the bot server's `/start` endpoint (local) or Pipecat Cloud API (production). Uses a shared `aiohttp.ClientSession` via FastAPI lifespan.
- `bot.py` — Pipecat pipeline runner. Contains `DialoutManager` (retry logic for outbound calls) and `run_bot()` which assembles the pipeline: `transport.input() → STT → user_aggregator → LLM → TTS → transport.output() → assistant_aggregator`. Entry point is the `bot()` function which receives `RunnerArguments` from Pipecat's runner.

**Data flow:** `DialoutRequest` → server creates room → `AgentRequest` (room_url + token + dialout_settings) → bot joins room → bot dials out to phone number.

**Environment modes (`ENV` variable):**
- `local` (default): bot runs on local machine at `LOCAL_SERVER_URL` (default `http://localhost:7860`)
- `production`: bot deployed to Pipecat Cloud, requires `PIPECAT_API_KEY` and `PIPECAT_AGENT_NAME`

Krisp noise cancellation is only available in production (Pipecat Cloud).

## Code Style

- Ruff with `line-length = 100`, lint rule `select = ["I"]` (import sorting only)
- Python ≥ 3.10, uses `uv` for dependency management
- Pydantic models for request/response data (`DialoutSettings`, `DialoutRequest`, `AgentRequest`)
- Loguru for logging
