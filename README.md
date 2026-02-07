# bouncer

Name's not on the list.

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that scans every installed skill, plugin, and MCP server on your machine, checks their credentials at the door, and walks you through setting up anything that showed up without ID.

## Install

```bash
npx skills add timmyg/bouncer -g -y
```

Or manually:

```bash
# Copy SKILL.md to your Claude Code skills directory
mkdir -p ~/.claude/skills/bouncer
curl -o ~/.claude/skills/bouncer/SKILL.md https://raw.githubusercontent.com/timmyg/bouncer/main/SKILL.md
```

## Usage

In Claude Code, run:

```
/bouncer
```

## What it does

1. **Scans the guest list** — Reads `~/.claude/skills/`, `~/.agents/skills/`, `~/.claude/plugins/`, `~/.claude/settings.json`, and `.mcp.json` to find every skill, plugin, and MCP server you have installed.

2. **Reads every skill's SKILL.md** — Doesn't just check names. Actually opens each skill's definition and looks for mentioned CLI tools, env vars, API keys, and external service dependencies.

3. **Cross-references a known registry** — Has built-in knowledge of common MCP server requirements (Playwright, Puppeteer, GitHub, Gmail, Context7, Atlassian/Jira, Slack, Linear, Notion, Supabase, and more).

4. **Checks IDs at the door** — Runs `command -v` for CLIs, checks env var presence (never prints actual secrets), verifies auth status where possible (e.g. `gh auth status`).

5. **Prints the report** — Groups results by status:
   - `✋ NOT GETTING IN` — Missing dependencies with exact fix steps
   - `🤝 YOU'RE IN` — Everything checks out
   - `🤷 NO ID ON FILE` — Unknown servers it can't auto-check

6. **Walks you through fixes** — Doesn't just dump a report and leave. Interviews you one failure at a time: "Want me to install this?", "Do you actually use this?", "Should I add a placeholder to your zshrc?"

7. **Re-checks after fixes** — Verifies each fix worked before moving on.

## Example output

```
╔══════════════════════════════════════════════╗
║              THE BOUNCER REPORT              ║
╠══════════════════════════════════════════════╣
║  Checked: 12  |  In: 8  |  Out: 3           ║
╚══════════════════════════════════════════════╝

NOT GETTING IN (3)
──────────────────

✋ MCP Server: gmail
   Missing: GMAIL_OAUTH_CLIENT_ID, GMAIL_OAUTH_CLIENT_SECRET
   Fix:
     1. Go to https://console.cloud.google.com/apis/credentials
     2. Create OAuth 2.0 credentials for Gmail API
     3. Add to ~/.zshrc:
        export GMAIL_OAUTH_CLIENT_ID="your-client-id"
        export GMAIL_OAUTH_CLIENT_SECRET="your-client-secret"
     4. Run: source ~/.zshrc

✋ MCP Server: context7
   Missing: CONTEXT7_API_KEY
   Fix:
     1. Go to https://context7.com/settings
     2. Copy your API key
     3. Add to ~/.zshrc:
        export CONTEXT7_API_KEY="your-key"
     4. Run: source ~/.zshrc

YOU'RE IN (8)
─────────────

🤝 MCP Server: github
   gh v2.74.0 installed, authenticated as @timmyg

🤝 MCP Server: playwright
   Playwright v1.51.0, browsers ready

🤝 Plugin: frontend-design
   Enabled and loaded

🤝 Skill: bouncer
   That's me. I'm always on the list.
```

## Known MCP servers

Bouncer has built-in knowledge of these MCP servers and their requirements:

| Server | Checks for |
|--------|-----------|
| `playwright` | `npx playwright --version`, browser binaries |
| `puppeteer` | `npx puppeteer --version` |
| `github` | `gh` CLI, `gh auth status`, `GITHUB_TOKEN` |
| `gmail` | `GMAIL_OAUTH_CLIENT_ID`, `GMAIL_OAUTH_CLIENT_SECRET` |
| `context7` | `CONTEXT7_API_KEY` |
| `mcp-atlassian` | `JIRA_URL`, `JIRA_USERNAME`, `JIRA_API_TOKEN` |
| `slack` | `SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN` |
| `linear` | `LINEAR_API_KEY` |
| `notion` | `NOTION_API_KEY` |
| `supabase` | `npx supabase --version`, `SUPABASE_URL`, `SUPABASE_ANON_KEY` |
| `desktop-commander` | `npx @anthropic/desktop-commander --version` |
| `filesystem` | Built-in, always passes |

Unknown MCP servers are flagged but not blocked — bouncer just tells you to check the docs.

## How it works

Bouncer is a single `SKILL.md` file. It instructs Claude Code to:

- Read the filesystem to discover all installed skills, plugins, and MCP configs
- Parse each skill's SKILL.md to extract dependency requirements
- Run non-destructive shell checks (`command -v`, env var presence, `--version` flags)
- Never print actual secrets — only reports SET or NOT SET
- Use `AskUserQuestion` to interactively walk through fixes

No binaries. No dependencies. Just a markdown file that tells Claude what to look for.

## License

MIT
