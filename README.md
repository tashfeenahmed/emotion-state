# emotion-state

An [OpenClaw](https://github.com/openclaw) hook that evaluates user and agent emotions as short natural-language phrases, tracks them across sessions, and injects a compact `<emotion_state>` block into the system prompt at bootstrap.

## How it works

1. On each `agent:bootstrap` event the hook extracts the latest user and assistant messages from the session.
2. Messages are classified via a configurable backend (custom HTTP endpoint or OpenAI-compatible API).
3. Results are stored in a per-agent JSON state file with exponential-decay trending.
4. A formatted emotion block is injected into the system prompt so the agent is aware of the current emotional context.

### Example output

```xml
<emotion_state>
  <user>
    2026-02-05 09:15: Felt moderately frustrated because of deployment delays.
    Trend (last 24h): mostly frustrated.
  </user>
  <agent>
    2026-02-05 09:10: Felt mildly focused because tasks were clearly defined.
  </agent>
</emotion_state>
```

## Installation

1. Copy the hook into your workspace:

```bash
cp -R ./hooks/emotion-state /path/to/your/hooks/
```

2. Enable it:

```bash
openclaw hooks enable emotion-state
```

3. Restart the OpenClaw gateway.

## Configuration

Set these as environment variables or under `hooks.internal.entries.emotion-state.env` in your OpenClaw config:

| Variable | Default | Description |
|---|---|---|
| `EMOTION_CLASSIFIER_URL` | — | Custom HTTP classification endpoint |
| `OPENAI_API_KEY` | — | OpenAI API key (used when no classifier URL is set) |
| `OPENAI_BASE_URL` | `https://api.openai.com/v1` | Base URL for OpenAI-compatible API |
| `EMOTION_MODEL` | `gpt-4o-mini` | Model used for classification |
| `EMOTION_CONFIDENCE_MIN` | `0.35` | Minimum confidence threshold |
| `EMOTION_HISTORY_SIZE` | `100` | Max entries kept in history |
| `EMOTION_HALF_LIFE_HOURS` | `12` | Decay half-life for trend calculation |
| `EMOTION_TREND_WINDOW_HOURS` | `24` | Time window for trend analysis |
| `EMOTION_MAX_USER_ENTRIES` | `3` | User entries shown in prompt |
| `EMOTION_MAX_AGENT_ENTRIES` | `2` | Agent entries shown in prompt |
| `EMOTION_MAX_OTHER_AGENTS` | `3` | Other agents' emotions shown |
| `EMOTION_TIMEZONE` | system default | IANA timezone (e.g. `America/Los_Angeles`) |

## State storage

Emotion state is persisted at:

```
~/.openclaw/agents/<agentId>/agent/emotion-state.json
```

Only model-inferred emotion labels and reasons are stored — no raw user text.

## License

MIT
