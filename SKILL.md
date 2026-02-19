---
name: atlassian-mcp
description: Jira and Confluence via Atlassian's official MCP server (mcporter CLI). Search, read, create, and update Jira issues and Confluence pages.
---

# Atlassian MCP (via mcporter)

Access Jira and Confluence through the official Atlassian Rovo MCP server, wrapped as CLI calls via mcporter.

## Setup

Before using, you need:

1. mcporter config at `~/.mcporter/mcporter.json` (see README.md)
2. Completed OAuth flow (run `npx -y mcp-remote https://mcp.atlassian.com/v1/mcp` once)
3. Your Cloud ID — get it with: `npx mcporter call atlassian.getAccessibleAtlassianResources`

## Usage

All calls go through `npx mcporter call atlassian.<toolName>`. Two syntax options:

```bash
# Function-call style
npx mcporter call 'atlassian.search(query: "onboarding docs")'

# Colon-delimited style
npx mcporter call atlassian.search query:"onboarding docs"
```

## Key Tools

### Universal Search (Rovo)

Use `search` as the default — it searches both Jira and Confluence:

```bash
npx mcporter call 'atlassian.search(query: "quarterly planning")'
```

To get full details from a search result, use `fetch` with the ARI:

```bash
npx mcporter call 'atlassian.fetch(id: "ari:cloud:jira:CLOUD_ID:issue/10107")'
```

### Jira

```bash
# Search with JQL
npx mcporter call 'atlassian.searchJiraIssuesUsingJql(cloudId: "CLOUD_ID", jql: "assignee = currentUser() AND status != Done")'

# Get issue
npx mcporter call 'atlassian.getJiraIssue(cloudId: "CLOUD_ID", issueIdOrKey: "PROJ-123")'

# Create issue
npx mcporter call 'atlassian.createJiraIssue(cloudId: "CLOUD_ID", projectKey: "PROJ", issueTypeName: "Task", summary: "Fix login bug")'

# Add comment
npx mcporter call 'atlassian.addCommentToJiraIssue(cloudId: "CLOUD_ID", issueIdOrKey: "PROJ-123", commentBody: "Looking into this")'

# Get transitions then transition
npx mcporter call 'atlassian.getTransitionsForJiraIssue(cloudId: "CLOUD_ID", issueIdOrKey: "PROJ-123")'

# List projects
npx mcporter call 'atlassian.getVisibleJiraProjects(cloudId: "CLOUD_ID")'

# Lookup user by name
npx mcporter call 'atlassian.lookupJiraAccountId(cloudId: "CLOUD_ID", searchString: "Werner")'
```

### Confluence

```bash
# Search with CQL
npx mcporter call 'atlassian.searchConfluenceUsingCql(cloudId: "CLOUD_ID", cql: "title ~ \"meeting\" AND type = page")'

# Get page (markdown)
npx mcporter call 'atlassian.getConfluencePage(cloudId: "CLOUD_ID", pageId: "123456789", contentFormat: "markdown")'

# List spaces
npx mcporter call 'atlassian.getConfluenceSpaces(cloudId: "CLOUD_ID")'

# Get pages in a space
npx mcporter call 'atlassian.getPagesInConfluenceSpace(cloudId: "CLOUD_ID", spaceId: "SPACE_ID")'

# Create page
npx mcporter call 'atlassian.createConfluencePage(cloudId: "CLOUD_ID", spaceId: "SPACE_ID", title: "New Page", body: "# Hello\nContent here", contentFormat: "markdown")'

# Update page
npx mcporter call 'atlassian.updateConfluencePage(cloudId: "CLOUD_ID", pageId: "123456789", body: "Updated content", contentFormat: "markdown")'
```

## All Available Tools (28)

| Tool | Description |
|------|-------------|
| `search` | **Rovo Search** — searches both Jira & Confluence (use by default) |
| `fetch` | Get details by ARI from search results |
| `atlassianUserInfo` | Current user info |
| `getAccessibleAtlassianResources` | Get cloud IDs |
| **Jira** | |
| `searchJiraIssuesUsingJql` | JQL search |
| `getJiraIssue` | Get issue details |
| `createJiraIssue` | Create issue |
| `editJiraIssue` | Update issue fields |
| `getTransitionsForJiraIssue` | Get available transitions |
| `transitionJiraIssue` | Change issue status |
| `addCommentToJiraIssue` | Add comment |
| `getJiraIssueRemoteIssueLinks` | Get remote links |
| `getVisibleJiraProjects` | List projects |
| `getJiraProjectIssueTypesMetadata` | Get issue types for project |
| `getJiraIssueTypeMetaWithFields` | Get field metadata |
| `lookupJiraAccountId` | Lookup user by name |
| `addWorklogToJiraIssue` | Log work time |
| **Confluence** | |
| `searchConfluenceUsingCql` | CQL search |
| `getConfluencePage` | Get page content |
| `getConfluenceSpaces` | List spaces |
| `getPagesInConfluenceSpace` | Pages in space |
| `getConfluencePageDescendants` | Child pages |
| `getConfluencePageFooterComments` | Footer comments |
| `getConfluencePageInlineComments` | Inline comments |
| `createConfluencePage` | Create page |
| `updateConfluencePage` | Update page |
| `createConfluenceFooterComment` | Add footer comment |
| `createConfluenceInlineComment` | Add inline comment |

## Tips

- Use `search` for general queries; use JQL/CQL tools when you need precise filtering
- Always request `contentFormat: "markdown"` for readable Confluence pages
- If auth expires, run `npx -y mcp-remote https://mcp.atlassian.com/v1/mcp` and re-authorize in browser
- To see all tools with full parameter details: `npx mcporter list atlassian --all-parameters`
