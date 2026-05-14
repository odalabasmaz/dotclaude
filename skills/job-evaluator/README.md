# job-evaluator

A Claude Code skill that evaluates companies against a personal career profile. Given a company name or URL, it searches Glassdoor, Kununu, Levels.fyi, Comprehensive.io, LinkedIn, Greenhouse, Xing, Indeed.de, Monster.de, Remotely.de, Layoffs.fyi, Hiring.cafe, Builtin.com, and Wellfound.com — and produces a structured, source-cited English-language report.

The skill is **profile-driven**: it reads `PROFILE.md` before every run and tailors all fit assessments, salary thresholds, and role filters to whoever is using it. Anyone can use it by editing their own `PROFILE.md`.

## Usage

Just type naturally in Claude Code:

```
Evaluate Zalando
```
```
Prepare a report for N26, Celonis, and HelloFresh
```
```
https://www.personio.com — should I apply here?
```

The skill triggers automatically — no slash command needed.

## Setup Your Profile

`PROFILE.md` is gitignored — your personal data (salary, employer, contact info) never gets committed.

```bash
cp PROFILE.example.md PROFILE.md
```

Then open `PROFILE.md` and fill in your details:

- Name and current role
- Target roles (Principal, Staff, SRE, EM, etc.)
- Tech stack and domain expertise
- Work model preferences (remote / hybrid / city)
- Minimum base salary and equity expectations
- Language requirements
- Industry preferences

The skill reads this file at runtime. **All fit assessments are derived from your profile** — the skill will not use hardcoded assumptions.

## Output

Each report includes:

- **⚡ Quick Overview** — Glassdoor/Kununu scores, CEO approval, recommendation rate, recent layoffs summary
- **⚠️ Layoff History** — Events from Layoffs.fyi with dates, size, and reasons (if found)
- **💰 Salary & Package** — Market ranges from Comprehensive.io and Levels.fyi, equity (RSU/ESOP), bonus, and benefits
- **✅ Pros / ❌ Cons** — Most common employee feedback from Glassdoor/Kununu
- **🎯 Candidate Fit** — Stack, role availability, location, salary, equity, culture, and stability — assessed against your PROFILE.md
- **💼 Open Positions** — Relevant roles consolidated from LinkedIn, Greenhouse, Xing, Indeed.de, Monster.de, Remotely.de, Hiring.cafe, Builtin.com, Wellfound.com, and the company careers page — with title, location, work model, salary (if listed), and direct apply links
- **🔗 Sources** — Every source searched, with link or explicit "no results found"
- **🏁 Decision** — 🟢 Apply / 🟡 Research More / 🔴 Skip

Multiple companies produce individual reports followed by a side-by-side comparison table.

## Data Integrity

This skill operates in **zero-hallucination mode**:

- Every data point is cited with a source URL.
- Missing data is always written as `"data not available"` — never inferred or estimated.
- Tech stack, salary, and work model assessments are only marked ✅ or ❌ when explicitly confirmed by a source. Uncertain cases use ⚠️.
- The skill does not use training-data knowledge to fill gaps — only what live web searches return.

## Requirements

This skill uses web search to fetch live data. You need the **Tavily MCP server** configured in Claude Code.

### Setup

```bash
claude mcp add tavily-mcp \
  -e TAVILY_API_KEY=tvly-YOUR_KEY_HERE \
  -- npx -y tavily-mcp
```

Get a free API key at [tavily.com](https://tavily.com) (1,000 searches/month free).

## Installation

```bash
cp -r skills/job-evaluator ~/.claude/skills/
# or symlink:
ln -s $(pwd)/skills/job-evaluator ~/.claude/skills/job-evaluator
```

Restart Claude Code after installation.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill definition — instructions, research steps, report template |
| `PROFILE.example.md` | Generic profile template — copy to `PROFILE.md` and fill in your details |
| `PROFILE.md` | Your personal profile — **gitignored**, never committed |
| `README.md` | This file |

## Search Tools & Configuration

The skill uses two layers of web search: Claude Code's built-in tools (free) and the Tavily MCP server (paid, more powerful).

### Tool Overview

| Tool | Provider | Cost | Best For |
|------|----------|------|----------|
| `WebSearch` | Claude Code (Anthropic) | Free | Quick lookups, news, Glassdoor links |
| `WebFetch` | Claude Code (Anthropic) | Free | Reading a specific URL (job posting, salary page) |
| `tavily_search` | Tavily MCP | 1–2 credits | Targeted web search with domain/date filtering |
| `tavily_research` | Tavily MCP | 4–250 credits | Deep multi-step research with AI synthesis |
| `tavily_extract` | Tavily MCP | 1–2 credits per 5 URLs | Extracting full content from specific pages |
| `tavily_crawl` | Tavily MCP | map + extract cost | Indexing entire sites (e.g. careers pages) |

### Tavily Pricing

| Plan | Monthly Cost | Credits/Month |
|------|-------------|---------------|
| **Researcher (Free)** | $0 | 1,000 |
| **Project** | $30 | 4,000 |
| **Bootstrap** | $100 | 15,000 |
| **Startup** | $220 | 38,000 |
| **Growth** | $500 | 100,000 |
| **Pay-as-you-go** | — | $0.008/credit |

**Credit cost by tool:**

| Tool | Depth | Credits |
|------|-------|---------|
| `tavily_search` | basic | 1 credit |
| `tavily_search` | advanced | 2 credits |
| `tavily_extract` | basic | 1 credit per 5 URLs |
| `tavily_extract` | advanced | 2 credits per 5 URLs |
| `tavily_research` | mini model | 4–110 credits |
| `tavily_research` | pro model | 15–250 credits |
| `tavily_map` | — | 1 credit per 10 pages |

> For typical runs (6–8 searches per company), expect ~10–20 credits per company with `tavily_search` at basic depth.

### Permissions

Add to `.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "WebSearch",
      "WebFetch(domain:www.glassdoor.com)",
      "mcp__tavily-mcp__tavily_search",
      "mcp__tavily-mcp__tavily_research",
      "mcp__tavily-mcp__tavily_extract"
    ]
  }
}
```

#### Verify the MCP server is running

```bash
claude mcp list
```

You should see `tavily-mcp` listed as connected.

---

## License

MIT
