# dotclaude

Personal AI toolkit — Claude skills, prompts, and automation workflows.

## Structure

```
dotclaude/
├── skills/          # Claude Code skills (~/.claude/skills/)
│   ├── job-evaluator/
│   │   └── SKILL.md
│   └── sdlc/
│       └── SKILL.md
├── agents/          # Claude Code subagents (~/.claude/agents/)
│   └── sdlc-*.md
├── prompts/         # Reusable prompt templates
└── docs/            # Notes, setup guides, decisions
```

## Skills

Skills are modular instruction packages for [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview). Drop them into `~/.claude/skills/` and they trigger automatically from natural language.

| Skill | Description | Language |
|-------|-------------|----------|
| [job-evaluator](./skills/job-evaluator/) | Evaluates companies against a personal career profile using Glassdoor, Kununu, Levels.fyi, Comprehensive.io, LinkedIn, Xing, Indeed.de, Monster.de, Remotely.de, Layoffs.fyi | 🇬🇧 English |
| [sdlc](./skills/sdlc/) | Runs a full software development lifecycle (Analyze → Plan → Dev → Monitor) as a six-persona software company; orchestrates the `sdlc-*` subagents. Stack-agnostic. | 🇬🇧 English |

## Agents

Subagents are specialised personas that skills (or you) delegate to. Drop them into
`~/.claude/agents/`. The `sdlc-*` agents power the [sdlc](./skills/sdlc/) skill.

| Agent | Role |
|-------|------|
| `sdlc-ceo` | Direction, value/cost/ROI, go/no-go |
| `sdlc-product-manager` | Scope, requirements, acceptance criteria |
| `sdlc-architect` | Tech stack, architecture, ADRs |
| `sdlc-developer` | Implementation, tests, docs |
| `sdlc-reviewer` | Quality / correctness / performance review gate |
| `sdlc-secops` | Security review gate |

The specification these are generated from lives at [`prompts/sdlc.md`](./prompts/sdlc.md).

## Installation

### Install a skill globally (available in all projects)

```bash
# Clone the repo
git clone https://github.com/odalabasmaz/dotclaude.git ~/dotclaude

# Symlink a skill into Claude Code's skills directory
mkdir -p ~/.claude/skills

mkdir -p ~/.claude/skills/job-evaluator
ln -s ~/dotclaude/skills/job-evaluator/SKILL.md ~/.claude/skills/job-evaluator/SKILL.md
```

Or copy manually:

```bash
cp -r ~/dotclaude/skills/job-evaluator ~/.claude/skills/
```

Then restart Claude Code — the skill auto-loads on session start.

### Install agents globally

Some skills delegate to subagents. Install them into `~/.claude/agents/`:

```bash
mkdir -p ~/.claude/agents

# Symlink all sdlc-* persona agents (used by the sdlc skill)
for f in ~/dotclaude/agents/sdlc-*.md; do
  ln -s "$f" ~/.claude/agents/"$(basename "$f")"
done
```

The `sdlc` skill needs both its skill directory and the `sdlc-*` agents installed.

### Install a skill per-project

```bash
mkdir -p .claude/skills
ln -s ~/dotclaude/skills/job-evaluator .claude/skills/job-evaluator
```

## Dependencies

Some skills require external tools or API keys. See each skill's README for details.

| Skill | Requires |
|-------|----------|
| job-evaluator | Tavily MCP for web search (see [setup guide](./docs/tavily-mcp-setup.md)) |

## Adding a New Skill

```
skills/
└── your-skill-name/
    ├── SKILL.md       # required — frontmatter + instructions
    ├── README.md      # optional — usage notes, examples
    └── scripts/       # optional — helper scripts
```

Refer to the [Claude Code skill authoring docs](https://support.claude.com/en/articles/12512198-how-to-create-custom-skills) for the full spec.
