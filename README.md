# atlassian-mcp

A [pi coding agent](https://github.com/badlogic/pi-mono) skill that wraps the [official Atlassian Rovo MCP server](https://github.com/atlassian/atlassian-mcp-server) as CLI calls via [mcporter](https://github.com/steipete/mcporter). No native MCP support needed — just bash.

Works with any agent that can run shell commands (pi, Claude Code, Codex CLI, etc).

## What you get

28 tools covering **Jira**, **Confluence**, and **Rovo Search**:

- Search across both Jira and Confluence with Rovo Search
- JQL/CQL queries for precise filtering
- Full CRUD on Jira issues (create, read, update, transition, comment, worklog)
- Full CRUD on Confluence pages (create, read, update, comment)
- Project/space listing, user lookup, and more

## Prerequisites

- [Node.js](https://nodejs.org/) v18+
- [mcporter](https://github.com/steipete/mcporter) (runs via `npx`, no install needed)
- [mcp-remote](https://www.npmjs.com/package/mcp-remote) (runs via `npx`, no install needed)
- An Atlassian Cloud account with Jira and/or Confluence access

## Setup

### 1. Configure mcporter

Create `~/.mcporter/mcporter.json`:

```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.atlassian.com/v1/mcp"]
    }
  }
}
```

### 2. Authenticate

Run once to complete the OAuth flow in your browser:

```bash
npx -y mcp-remote https://mcp.atlassian.com/v1/mcp
```

A browser window will open — log in to Atlassian and authorize. Tokens are cached in `~/.mcp-auth/`.

### 3. Verify

```bash
npx mcporter call atlassian.atlassianUserInfo
```

### 4. Get your Cloud ID

```bash
npx mcporter call atlassian.getAccessibleAtlassianResources
```

Note the `id` field — you'll need it as `cloudId` for most tool calls.

## Install as a pi skill

```bash
# User-level (all projects)
git clone https://github.com/wernerbatt-klz/atlassian-mcp ~/.pi/agent/skills/atlassian-mcp

# Or project-level
git clone https://github.com/wernerbatt-klz/atlassian-mcp .pi/skills/atlassian-mcp
```

### Claude Code

```bash
mkdir -p ~/.claude/skills
ln -s ~/atlassian-mcp ~/.claude/skills/atlassian-mcp
```

## Usage

All calls go through `npx mcporter call atlassian.<tool>`:

```bash
# Rovo search (searches both Jira & Confluence)
npx mcporter call 'atlassian.search(query: "onboarding docs")'

# Jira — search with JQL
npx mcporter call 'atlassian.searchJiraIssuesUsingJql(cloudId: "YOUR_CLOUD_ID", jql: "assignee = currentUser() AND status != Done")'

# Jira — get issue
npx mcporter call 'atlassian.getJiraIssue(cloudId: "YOUR_CLOUD_ID", issueIdOrKey: "PROJ-123")'

# Jira — create issue
npx mcporter call 'atlassian.createJiraIssue(cloudId: "YOUR_CLOUD_ID", projectKey: "PROJ", issueTypeName: "Task", summary: "Fix login bug")'

# Confluence — search with CQL
npx mcporter call 'atlassian.searchConfluenceUsingCql(cloudId: "YOUR_CLOUD_ID", cql: "title ~ \"meeting\" AND type = page")'

# Confluence — get page as markdown
npx mcporter call 'atlassian.getConfluencePage(cloudId: "YOUR_CLOUD_ID", pageId: "123456789", contentFormat: "markdown")'
```

See [SKILL.md](SKILL.md) for the full tool reference.

## Why not native MCP?

This follows [pi's philosophy](https://mariozechner.at/posts/2025-11-02-what-if-you-dont-need-mcp/): CLI tools invoked via bash are composable, token-efficient (tool descriptions load on-demand, not on every session), and work with any agent. mcporter bridges the MCP ecosystem without requiring native MCP support.

## License

MIT
