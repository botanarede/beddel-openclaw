# beddel-openclaw

[Beddel](https://github.com/botanarede/beddel) integration with [OpenClaw](https://openclaw.dev) — plugin + skill that allow the agent to run, validate, and inspect declarative YAML AI workflows.

## Repository structure

```
beddel-openclaw/
├── plugin/          # OpenClaw plugin (TypeScript) — registers the "beddel" tool
├── skill/           # OpenClaw skill — SKILL.md + examples + references
│   ├── SKILL.md
│   ├── examples/
│   └── references/
└── TODO.md
```

| Component | What it does |
|-----------|-------------|
| `plugin/` | npm package `@botanarede/beddel` — spawns the `beddel` CLI as a subprocess and exposes the `run`, `validate`, and `list-primitives` actions as agent tools |
| `skill/` | Documentation loaded by the agent at runtime (SKILL.md) with usage instructions, examples, and detailed references on the YAML format |

## Prerequisites

- Node.js ≥ 18
- Python 3.11+
- An LLM API key (any provider supported by [LiteLLM](https://docs.litellm.ai/docs/providers)). Recommended: Gemini

```bash
export GEMINI_API_KEY="your-key"
```

## Installation

### 1. Beddel CLI (Python)

```bash
python3.11 -m pip install "beddel[all]"
beddel version
```

### 2. OpenClaw plugin

```bash
openclaw plugins install @botanarede/beddel
```

### 3. OpenClaw skill

```bash
openclaw skills install --from ./skill
```

> The skill loads `SKILL.md` into the agent context and makes the references in `references/` available.

## Quick start

### Run a workflow

```bash
beddel run workflow.yaml -i topic="AI agents" --json-output
```

### Via OpenClaw agent

After installing the plugin + skill, the agent can invoke the `beddel` tool directly:

- `run` — execute a YAML workflow
- `validate` — validate YAML syntax and schema
- `list-primitives` — list the 7 available primitives

### Example workflow (automatic setup)

The file `skill/examples/setup-beddel.yaml` checks whether the plugin is installed and installs it if needed:

```bash
beddel run skill/examples/setup-beddel.yaml --json-output
```

## Key concepts

A **workflow** is a YAML file with `id`, `name`, `input_schema` (optional), and a list of `steps`. Each step declares a **primitive** and a **config**.

### Primitives

| Primitive | Purpose |
|-----------|---------|
| `llm` | Single-turn LLM call |
| `chat` | Multi-turn conversation with history |
| `output-generator` | Template rendering |
| `guardrail` | Data validation with correction strategies |
| `call-agent` | Nested workflow invocation |
| `tool` | External tool call (`shell_exec` is built-in) |
| `agent-exec` | Delegation to an external agent |

### Variables

| Namespace | Example | Source |
|-----------|---------|--------|
| `$input` | `$input.topic` | Runtime inputs (`-i key=val`) |
| `$stepResult` | `$stepResult.greet.content` | Output from previous steps |
| `$env` | `$env.GEMINI_API_KEY` | Environment variables |

### Execution strategies

| Strategy | Behavior |
|----------|----------|
| `fail` | Stop the workflow on error (default) |
| `skip` | Log the error and continue |
| `retry` | Retry with exponential backoff |
| `fallback` | Execute an alternative step |
| `delegate` | Delegate recovery to an LLM |

## Plugin configuration

Configurable options in `openclaw.plugin.json`:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `defaultTimeoutMs` | number | 120000 | Process timeout in ms |
| `maxStdoutBytes` | number | 1048576 | Maximum captured stdout (1 MB) |
| `beddelPath` | string | `beddel` | Custom path to the CLI |
| `jsonOutput` | boolean | true | Use `--json-output` on run |

## Security

- Never put API keys directly in YAML — use `$env.*`
- `shell_exec` runs with `shell=False` (no shell injection)
- Default timeout of 60s per step, stdout capped at 1 MB
- Beddel does not send telemetry by default — OpenTelemetry is opt-in

## Detailed documentation

See `skill/references/` for full specifications:

- [`workflow-format.md`](skill/references/workflow-format.md) — Complete YAML schema
- [`primitives.md`](skill/references/primitives.md) — All 7 primitives with every option
- [`execution-strategies.md`](skill/references/execution-strategies.md) — 5 strategies with examples
- [`variable-resolution.md`](skill/references/variable-resolution.md) — Namespaces, recursive resolution, errors

## Troubleshooting

| Error | Cause | Solution |
|-------|-------|----------|
| `BEDDEL-PRIM-300` | Tool not found | Use `shell_exec` (built-in) or register with `-t name=module:func` |
| `BEDDEL-RESOLVE-001` | Unresolved variable | Check the step id and the result path |
| `BEDDEL-GUARD-201` | Guardrail validation failed | Check types in the schema; use `strategy: correct` for JSON string inputs |
| `python3.11: not found` | Wrong Python | Install Python 3.11+; the system Python may be 3.10 |

## License

MIT
