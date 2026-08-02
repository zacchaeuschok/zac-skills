# zac-skills

Agent skills for design critique, structured writing, strategy evaluation, code architecture, Pydantic patterns, and frontend development.

Compatible with **Claude Code**, **OpenAI Codex**, **Cursor**, and any tool supporting the [Agent Skills standard](https://agentskills.io).

## Skills

### Code Architecture & Review

| Skill | Trigger | Description |
|-------|---------|-------------|
| `zac-critique` | `/zac-critique` | Dominated-design detection + Ousterhout red flags. Finds implementations more complex than the requirements demand. |
| `konmari` | `/konmari` | Code review through Marie Kondo's KonMari principles. Spark joy, discard before organizing, tidy by category. |
| `cosmic-python` | `/cosmic-python` | Review code against Architecture Patterns with Python (DDD, Repository, Service Layer, UoW, Aggregates). |
| `cosmic-python-pt1` | `cosmic python`, `DDD` | Write Python backends following Cosmic Python Part 1 patterns. The generative counterpart to `cosmic-python`. |

### Python & Frameworks

| Skill | Trigger | Description |
|-------|---------|-------------|
| `pydantic-v2-idiomatic` | `pydantic`, `BaseModel` | Write idiomatic Pydantic v2 — models, validators, serialization, fields, types, config. |
| `building-pydantic-ai-agents` | `pydantic ai`, `build agent` | Build AI agents with Pydantic AI — tools, capabilities, structured output, streaming, testing. |
| `frontend` | `/frontend` | Build and review frontend features in React + TS + Vite + Tailwind + shadcn/ui + TanStack stacks. |

### Writing & Communication

| Skill | Trigger | Description |
|-------|---------|-------------|
| `minto` | `/minto` | Minto Pyramid Principle. Restructures writing with SCQA, MECE groupings, and inductive reasoning. |
| `abstract` | `/abstract` | ML/research paper abstracts using Neel Nanda's cold-start structure with AI-writing decontamination. |

### Strategy & Evaluation

| Skill | Trigger | Description |
|-------|---------|-------------|
| `strategy` | `/strategy` | Rumelt's Good Strategy Bad Strategy. Tests for the kernel and four hallmarks of bad strategy. |

### Meta

| Skill | Trigger | Description |
|-------|---------|-------------|
| `skill-creator` | `/skill-creator` | Eval-driven skill creation workflow with blind A/B testing and grading agents. |

## Installation

### All agents at once (recommended)

Install to every detected agent (Claude Code, Codex, Cursor, etc.) with one command:

```bash
npx skills add zacchaeuschok/zac-skills --agent claude-code codex cursor
```

Or install to all detected agents automatically:

```bash
npx skills add zacchaeuschok/zac-skills
```

Install a single skill:

```bash
npx skills add zacchaeuschok/zac-skills --skill zac-critique
```

Uses [`skills`](https://github.com/vercel-labs/skills) by Vercel Labs — the universal agent skills package manager.

### Claude Code only

```
/plugin install zacchaeuschok/zac-skills
```

### Codex only

```bash
git clone https://github.com/zacchaeuschok/zac-skills.git
ln -s "$(pwd)/zac-skills/skills"/* ~/.agents/skills/
```

### Manual (any agent)

All skills follow the [agentskills.io](https://agentskills.io) standard. Copy or symlink from `skills/` into your tool's skill discovery path.

## Structure

```
zac-skills/
├── .claude-plugin/             # Claude Code plugin manifest
│   └── plugin.json
├── .agents/plugins/            # Codex registry
│   └── marketplace.json
├── .cursor-plugin/             # Cursor registry
│   └── marketplace.json
├── skills/                     # Skills (single source of truth)
│   ├── abstract/
│   ├── building-pydantic-ai-agents/
│   ├── cosmic-python/
│   ├── cosmic-python-pt1/
│   ├── frontend/
│   ├── konmari/
│   ├── minto/
│   ├── pydantic-v2-idiomatic/
│   ├── skill-creator/
│   ├── strategy/
│   └── zac-critique/
└── README.md
```

## Attribution

- `skill-creator` — [Anthropic](https://github.com/anthropics/skills) (Apache-2.0)
- `building-pydantic-ai-agents` — [Pydantic](https://github.com/pydantic/skills) (MIT)
- Ousterhout red flags in `zac-critique` are based on John Ousterhout's *A Philosophy of Software Design*
- `cosmic-python` and `cosmic-python-pt1` patterns from *Architecture Patterns with Python* by Harry Percival & Bob Gregory
- `abstract` structure based on Neel Nanda's "Highly Opinionated Advice on How to Write ML Papers"
- `pydantic-v2-idiomatic` distilled from [Pydantic v2 official docs](https://pydantic.dev/docs)

## Requirements

- Any AI coding agent supporting the [Agent Skills standard](https://agentskills.io)
- Opus model recommended for complex skills (set in skill frontmatter where applicable)
