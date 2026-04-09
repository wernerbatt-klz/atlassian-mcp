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

# Edit issue (use colon-delimited style — fields must be a JSON object, not a string)
npx mcporter call atlassian.editJiraIssue cloudId:"CLOUD_ID" issueIdOrKey:"PROJ-123" fields:'{"summary": "Updated title", "duedate": "2026-04-11"}'

# Add comment
npx mcporter call 'atlassian.addCommentToJiraIssue(cloudId: "CLOUD_ID", issueIdOrKey: "PROJ-123", commentBody: "Looking into this")'

# Get transitions then transition
npx mcporter call 'atlassian.getTransitionsForJiraIssue(cloudId: "CLOUD_ID", issueIdOrKey: "PROJ-123")'

# List projects
npx mcporter call 'atlassian.getVisibleJiraProjects(cloudId: "CLOUD_ID")'

# Lookup user by name
npx mcporter call 'atlassian.lookupJiraAccountId(cloudId: "CLOUD_ID", searchString: "Werner")'
```

**Note:** `editJiraIssue` requires colon-delimited syntax with `fields` as a JSON object. Function-call style wraps the JSON in a string, which causes a validation error:

```bash
# Wrong — fields becomes a string, not an object
npx mcporter call 'atlassian.editJiraIssue(cloudId: "...", issueIdOrKey: "PROJ-123", fields: "{...}")'
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

## Patterns

### Editing rich Confluence pages (ADF)

**When updating a page that has rich formatting (panels, expand sections, info boxes), use ADF format, not markdown.** Markdown strips all Confluence macros. Only two `contentFormat` options exist: `"markdown"` and `"adf"`.

Workflow:

1. Get the page in ADF: `getConfluencePage(..., contentFormat: "adf")`
2. The body is a JSON object (Atlassian Document Format). Edit the JSON surgically.
3. Push back with `updateConfluencePage(..., contentFormat: "adf", body: <json-string>)`

Key ADF node types:
- `panel` (attrs: `panelType` = info/note/warning/success/error)
- `expand` (collapsible section)
- `table` / `tableRow` / `tableCell` / `tableHeader`
- `inlineCard` (smart links, attrs: `url`)
- `heading` (attrs: `level`)
- `paragraph`, `bulletList`, `listItem`, `text`
- All structural nodes need `localId` (use `uuid4()`)

**When to use which format:**
- `markdown` — fine for creating new pages from scratch, or reading pages for analysis
- `adf` — required when editing existing pages that have rich formatting (panels, macros, expand sections). Markdown round-trips are lossy.

**Markdown `<br>` tags do NOT create separate bullet items in table cells.** Confluence's markdown-to-ADF converter wraps `- item1 <br>- item2` into a single text node with literal `\n-` text inside one `listItem`. To get proper multi-line bullet lists in table cells, you must use ADF: create a `bulletList` node with separate `listItem` children.

**Inline comment annotations split text nodes.** Pages with inline comments have `annotation` marks on text nodes that anchor the comment to specific text. This splits what looks like a continuous sentence into multiple text nodes. A `replace_text_recursive` call targeting the full sentence will silently fail because no single node contains the full string.

Mitigation:
1. Before writing replacement logic, dump the exact node structure of the target cell.
2. Fix nodes individually: match by annotation ID for annotated text, and by position/content for surrounding non-annotated text.
3. After pushing, **always verify edits landed** by re-fetching the page.

**Re-fetch before each edit pass.** If applying fixes in multiple scripts, each script must re-fetch the live page. Never reuse a previous script's output file as the base for the next pass, or earlier fixes will be overwritten.

**Replying to inline comments:** Use `parentCommentId` only. Do not pass both `pageId` and `parentCommentId`; the API rejects it with a 400.

**Not supported by MCP tools:** Resolving inline comments, uploading attachments. These require manual action.

### Creating new Confluence pages

**Do NOT include an H1 title in the markdown body.** Confluence uses the `title` parameter as the page heading. An H1 in the body creates a duplicate heading.

**`parentPageId` only works for page-type parents.** If the target parent has `parentType: "folder"` (common for space sections), the parameter is silently ignored and the page lands under the space root. Check the parent page's type before creating siblings.

### Using --args for large payloads

When body content is too large or complex for CLI arg escaping, use `--args`:

```python
import subprocess, json

args = json.dumps({
    "cloudId": "YOUR_CLOUD_ID",
    "pageId": "123456789",      # must be string, not number
    "contentFormat": "adf",
    "body": json.dumps(adf_doc)  # ADF body as JSON string
})

subprocess.run(
    ['npx', 'mcporter', 'call', 'atlassian.updateConfluencePage', '--args', args],
    capture_output=True, text=True, timeout=60
)
```

**Note:** `pageId` must be a quoted string in the JSON, not a number. mcporter validates types strictly.

### Very large ADF payloads (>128KB): bypass mcporter, use MCP SDK directly

The `--args` approach fails when the ADF body exceeds ~128KB because Linux's `execve` has a hard limit on combined argv + envp size. Shell variable expansion, env vars, and named pipes all hit the same wall.

**Size heuristic:** If a page has >15 tables or >100 top-level ADF nodes, skip `--args` entirely and go straight to the MCP SDK file-based approach. Content-heavy pages with rich formatting almost always exceed 128KB after slimming.

**Solution:** Write a Node.js script that imports the MCP SDK, spawns `mcp-remote` as a stdio transport, and calls the tool directly. The payload is read from a file, never passed through shell args.

Before using this approach, **slim the ADF first** to reduce size:
- Strip `localId` from all nodes (Confluence regenerates these)
- Remove empty `attrs` objects
- Remove `colspan`/`rowspan` when equal to 1
- Use compact JSON (`separators=(',', ':')`)

```javascript
// update-via-mcp.mjs
import { readFileSync } from 'fs';
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

async function main() {
  const adf = readFileSync('/tmp/page-adf-compact.json', 'utf-8');

  const transport = new StdioClientTransport({
    command: 'npx',
    args: ['-y', 'mcp-remote', 'https://mcp.atlassian.com/v1/mcp'],
  });

  const client = new Client({ name: 'update-script', version: '1.0.0' });
  await client.connect(transport);

  const result = await client.callTool({
    name: 'updateConfluencePage',
    arguments: {
      cloudId: 'YOUR_CLOUD_ID',
      pageId: '123456789',
      contentFormat: 'adf',
      body: adf,
      title: 'Page Title'
    }
  });

  console.log('Result:', JSON.stringify(result).substring(0, 2000));
  await client.close();
}

main().catch(e => { console.error(e); process.exit(1); });
```

Run with: `timeout 60 node update-via-mcp.mjs`

This approach also works for any other mcporter call that hits arg size limits (e.g. large Jira description updates).

### Bulk ADF restructuring (reorder, demote, remove sections)

For structural edits that touch many sections (e.g. reordering, demoting heading levels, removing sections, adding new sections), use this workflow:

1. **Fetch ADF** and save to a working file
2. **Print a node index** — list all top-level nodes with index, type, and text preview for orientation
3. **Write a script** that builds a new `content` array by copying/transforming nodes from the original. Use deep copies for each node. Verify source nodes with assertions before transforming.
4. **Clean up** — remove consecutive duplicate rules, slim the ADF (strip `localId`, empty `attrs`, `colspan`/`rowspan` = 1)
5. **Push** via MCP SDK (these pages are almost always >128KB)
6. **Verify** — re-fetch in markdown and check heading structure matches expectations

This is more reliable than surgical find-and-replace for multi-section edits because it avoids index drift from earlier mutations.

### Table width audit

When reviewing or fixing table widths on a Confluence page, run a diagnostic pass first. Fetch the page in ADF, then iterate all table nodes and measure:
- Column count
- Row count
- Max cell text length (longest text node in any cell)
- Current `layout` and `width` attrs

Reference widths:
- `center` with `width: 1400-1600` for 4-5 column tables with long text
- `center` with `width: 1200` for 4-5 column tables with medium text
- `center` with `width: 1000` for data model / schema tables with short text
- `wide` for 2-3 column tables with one long-text column
- `default` is fine for small tables with short text and ≤3 columns

Flag tables where max cell text >150 chars and layout is `default` with no width as needing attention.

## Tips

- Use `search` for general queries; use JQL/CQL tools when you need precise filtering
- Use `contentFormat: "markdown"` for reading pages or creating simple new ones. Use `contentFormat: "adf"` when editing existing pages with rich formatting. Markdown updates are lossy and will strip macros.
- OAuth tokens are cached in `~/.mcp-auth/` — if auth expires, run: `npx -y mcp-remote https://mcp.atlassian.com/v1/mcp` and re-authorize in browser
- To see all tools with full parameter details: `npx mcporter list atlassian --all-parameters`
- When creating epics or stories, check for required custom fields via `getJiraIssueTypeMetaWithFields` to find the correct field IDs and allowed values for your instance.
