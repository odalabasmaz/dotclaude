# job-evaluator

A Claude Code skill that evaluates companies against a personal career profile. Given a company name or URL, it searches Glassdoor, Kununu, Levels.fyi, Comprehensive.io, LinkedIn, Xing, Indeed.de, Monster.de, Remotely.de, and Layoffs.fyi, and produces a structured English-language report.

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

## Output

Each report includes:

- **⚡ Quick Overview** — Glassdoor/Kununu scores, CEO approval, recommendation rate, recent layoffs
- **⚠️ Layoff History** — Events from Layoffs.fyi with dates, size, and reasons
- **💰 Salary & Package** — Market ranges from Comprehensive.io and Levels.fyi, equity (RSU/ESOP), bonus, benefits
- **✅ Pros / ❌ Cons** — Most common employee feedback
- **🎯 Candidate Fit** — Stack, role, location, salary, culture, and stability fit
- **💼 Open Positions** — Relevant roles consolidated from LinkedIn, Xing, Indeed.de, Monster.de, Remotely.de, and the company careers page — with title, location, work model, salary (if listed), and direct apply links
- **🔗 Sources** — Reference links for all data sources
- **🏁 Decision** — 🟢 Apply / 🟡 Research / 🔴 Skip

Multiple companies get a comparison table at the end.

## Requirements

This skill uses web search to fetch live data. You need the **Tavily MCP server** configured in Claude Code.

### Setup

```bash
claude mcp add tavily-mcp \
  -e TAVILY_API_KEY=tvly-YOUR_KEY_HERE \
  -- npx -y tavily-mcp
```

Get a free API key at [tavily.com](https://tavily.com) (1000 searches/month free).

## Installation

```bash
cp -r skills/job-evaluator ~/.claude/skills/
# or symlink:
ln -s $(pwd)/skills/job-evaluator ~/.claude/skills/job-evaluator
```

Restart Claude Code after installation.
