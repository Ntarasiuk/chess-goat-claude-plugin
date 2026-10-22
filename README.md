# Chess Goat for Claude

Same MCP server as the ChatGPT app: https://trychessgoat.com/mcp (stateless Streamable HTTP, OAuth 2.1 for Goat Pass and reports). This directory is the Claude Code plugin; claude.ai and Claude Desktop need no package.

How the server tells hosts apart: `initialize` returns an `Mcp-Session-Id` that encodes the client (from `clientInfo.name`) and whether it declared the MCP Apps UI extension; hosts echo it on every call (worker/mcp/host.ts). claude.ai and Claude Desktop render the card and board widgets through MCP Apps; Claude Code gets boards in text (`show_position` returns the FEN, an /analyze/ link and, for exercises, the answers, hints and refutations for the model) and matching server instructions. Links and progress records carry `src=claude`.

## claude.ai / Claude Desktop

Settings → Connectors → Add custom connector → URL `https://trychessgoat.com/mcp`, no client ID or secret. Claude registers dynamically (callbacks `https://claude.ai/api/mcp/auth_callback`, `https://claude.com/api/mcp/auth_callback`; worker/mcp/callbacks.ts). Public tools work without connecting; Pass lessons and `my_report` prompt to connect through the same consent page as ChatGPT. The public connector directory is an Anthropic partner listing (not self-serve); apply once the custom connector has been exercised.

## Claude Code

Direct: `claude mcp add --transport http chess-goat https://trychessgoat.com/mcp`. OAuth uses a loopback callback (`http://localhost:<port>/callback`), allowed on any port.

Plugin (this directory, agent-plugins layout: `.claude-plugin/plugin.json`, `.mcp.json`, `skills/chess-coach/SKILL.md`). Local test:

```
claude plugin marketplace add ./claude-plugin
claude plugin install chess-goat@chess-goat
```

Publishing needs a public repo holding this directory's contents at its root (the chess-goat repo is private, so a marketplace pointed at it cannot be fetched by anyone else); then `claude plugin marketplace add <owner>/<repo>`.

Manual acceptance (Claude Code):
1. `/mcp` lists chess-goat with the public tools; `goat_card hikaru` returns the card summary with a `?src=claude` link.
2. "Teach me the forks lesson": `show_position` returns FEN + analyze link + accepted moves; Claude asks for the move before revealing it.
3. A Pass lesson says locked; authenticate via `/mcp`, consent page shows the Goat Pass sign-in, lesson opens; `record_lesson` shows on /learn/ with device `claude`.
4. claude.ai custom connector: same flow with the board and card widgets rendered.
