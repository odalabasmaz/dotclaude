# Tavily MCP Setup Guide

Tavily provides web search capability to Claude Code skills. The free tier offers 1000 searches/month.

## 1. Get an API Key

1. Go to [tavily.com](https://tavily.com)
2. Sign up for a free account
3. Copy your API key from the dashboard (starts with `tvly-`)

## 2. Add to Claude Code

```bash
claude mcp add tavily-mcp \
  -e TAVILY_API_KEY=tvly-YOUR_KEY_HERE \
  -- npx -y tavily-mcp
```

This command:
- Downloads the Tavily MCP server package automatically via `npx`
- Stores your API key in `~/.claude/claude.json`
- Makes web search available in every Claude Code session

## 3. Verify

Start a new Claude Code session and test:

```
Search for Zalando Munich engineer reviews on Glassdoor
```

If results come back, setup is complete.

## Troubleshooting

**MCP not loading:** Run `claude mcp list` to confirm `tavily-mcp` appears.

**Auth errors:** Double-check the API key — it must start with `tvly-`.

**Rate limits:** Free tier is 1000 requests/month. Each company evaluation uses ~6 searches.
