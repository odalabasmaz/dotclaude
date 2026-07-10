# SDLC Skill

Runs a full software development lifecycle — **Analyze → Plan → Dev → Monitor** — as a small,
opinionated software company. The skill is the **orchestrator**; the real work is done by six
persona **subagents** it delegates to.

## What it does

You give it product context and requirements. It asks clarifying questions first, right-sizes the
process to the work, then drives the lifecycle: defining scope, choosing a stack and architecture,
implementing with tests, and gating the result through a quality + security review before "go
live". Everything is documented as versioned artifacts under `docs/sdlc/`, and new requirements
loop back into Analyze rather than restarting.

It is **stack-agnostic** — it builds backend, frontend, CLI, service, or library projects in any
language (Java/Kotlin, Python, Go, TypeScript, …). The Architect picks the tech per problem.

## Personas (subagents)

| subagent | Role |
|---|---|
| `sdlc-ceo` | Direction, value/cost/ROI, go/no-go |
| `sdlc-product-manager` | Scope, requirements, acceptance criteria |
| `sdlc-architect` | Tech stack, architecture, ADRs |
| `sdlc-developer` | Implementation, tests, docs |
| `sdlc-reviewer` | Quality/correctness/perf review gate |
| `sdlc-secops` | Security review gate |

## Install

The skill lives in `skills/sdlc/`; the personas live in `agents/` and must be installed as
Claude Code subagents. See the repo root [README](../../README.md#installation) for symlink steps.

## Design

The full specification this skill and its agents are generated from lives at
[`prompts/sdlc.md`](../../prompts/sdlc.md) — keep that as the source of truth and regenerate the
files when the process changes.

## Kickoff

Just describe what you want built, e.g. *"build me a URL-shortener service"* or *"run the SDLC on
this feature"*. The team opens with clarifying questions, then begins at **Analyze**.
