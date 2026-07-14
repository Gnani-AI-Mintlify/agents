# Gnani smoke integration (LiveKit Agents)

End-to-end smoke checks and a runnable voice bot for the `livekit-plugins-gnani`
plugin inside the [LiveKit Agents](https://github.com/livekit/agents) monorepo.

Mirrors the Pipecat layout at `all_repos/integrations/pipecat/scripts/smoke-gnani/`.

## Prerequisites

- Python 3.10+
- [uv](https://docs.astral.sh/uv/)
- [LiveKit CLI](https://docs.livekit.io/home/cli/) (`lk`) — for `create-token.sh` and playground workflows
- `GNANI_API_KEY` — from [app.gnani.ai/voice](https://app.gnani.ai/voice)
- `GROQ_API_KEY` — Groq chat model for the LLM leg
- LiveKit Cloud project (`LIVEKIT_URL`, `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`) — for `dev` / `start` modes

## 1. Install

From the **agents repo root** (`all_repos/integrations/livekit/agents/`):

```bash
chmod +x ./scripts/smoke-gnani/*.sh
./scripts/smoke-gnani/setup.sh
```

This runs `uv venv` + editable installs for `livekit-agents`, `livekit-plugins-gnani`,
`livekit-plugins-groq`, and `livekit-plugins-silero`, and copies `env.example` → `examples/.env`.

To test against the standalone Gnani fork instead of the in-tree plugin:

```bash
GNANI_PLUGIN_SRC=../python-package/livekit-plugins-gnani ./scripts/smoke-gnani/setup.sh
```

Edit `examples/.env` with your keys.

## 2. Quick smoke (imports + unit tests)

```bash
./scripts/smoke-gnani/run.sh
```

Runs:

1. Import check (`livekit.plugins.gnani`, `voice_gnani.py` loads)
2. Gnani plugin unit tests from `python-package/livekit-plugins-gnani/tests` (or in-tree fallback)

Pass `--live` to attempt live integration tests when `GNANI_API_KEY` is set.

## 3. Run the voice bot

```bash
# Local console — mic + speaker, no LiveKit server (fastest sanity check)
./scripts/smoke-gnani/run-bot.sh console

# Dev worker — connects to LiveKit Cloud; use Agents Playground
./scripts/smoke-gnani/run-bot.sh dev

# Production worker
./scripts/smoke-gnani/run-bot.sh start
```

Equivalent direct command:

```bash
uv run examples/voice_agents/voice_gnani.py console
```

## 4. Join token (playground / custom client)

```bash
./scripts/smoke-gnani/create-token.sh
# or
ROOM=test IDENTITY=sandeep ./scripts/smoke-gnani/create-token.sh
```

Requires `lk` CLI and `LIVEKIT_*` vars in `examples/.env`.

Open [Agents Playground](https://agents-playground.livekit.io) and paste the token, or run `dev` mode and connect from the playground UI.

## Pipeline

```
room audio → Silero VAD → Gnani STT (WebSocket) → Groq LLM → Gnani TTS → room audio
```

Implemented in `examples/voice_agents/voice_gnani.py`.

Optional env overrides:

| Variable | Default | Purpose |
|----------|---------|---------|
| `GNANI_STT_LANGUAGE` | `en-IN` | BCP-47 STT language |
| `GNANI_TTS_VOICE` | `Pranav` | TTS voice |
| `GROQ_MODEL` | `llama-3.1-8b-instant` | Groq chat model |

## PyPI note

The public package at [pypi.org/project/livekit-plugins-gnani](https://pypi.org/project/livekit-plugins-gnani/)
is published by **LiveKit** (currently `1.6.x`). This smoke harness uses the **workspace plugin**
under `livekit-plugins/livekit-plugins-gnani/` (or the Gnani fork via `GNANI_PLUGIN_SRC`).
Upstream merges go through `github.com/livekit/agents`.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `No .venv found` | Run `./scripts/smoke-gnani/setup.sh` |
| `GNANI_API_KEY` errors | Set in `examples/.env` |
| `lk: command not found` | Install LiveKit CLI |
| Console mode silent | Grant microphone permission; check logs |
| `dev` won't connect | Verify `LIVEKIT_URL` / API key / secret in `examples/.env` |
