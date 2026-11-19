# Jawafdehi MCP

**Nepal's open accountability archive, served over the [Model Context Protocol](https://modelcontextprotocol.io).**

```
https://api.jawafdehi.org/mcp
```

Point any MCP client at that URL and your AI assistant can read every published
Jawafdehi case, the people and offices behind them, the source documents each case is
built on, and Nepal's court record — 2,600+ accountability cases and 2.2M+ court case
records.

This is a **hosted, remote server**. There is nothing to install and no API key to
issue. Every tool needs sign-in with a Google account; no Jawafdehi account or paid plan
is needed, and one is created the first time you sign in.

## Contents

- [Install](#install)
- [The plugin and its skills](#the-plugin-and-its-skills)
- [Signing in](#signing-in)
- [What you can ask](#what-you-can-ask)
- [Tools](#tools)

## Install

### Claude Code

```bash
claude mcp add --transport http jawafdehi https://api.jawafdehi.org/mcp -s user
```

Then run `/mcp` to confirm it connected.

Or install the plugin, which adds the server and both skills in one step (see
[below](#the-plugin-and-its-skills)).

### Claude

Settings → **Connectors** → **Add custom connector**, and paste the server URL. Sign in
with Google when Claude asks.

### ChatGPT

Turn on developer mode, then Settings → **Connectors** → add a connector, and paste
the server URL.

### Codex CLI

```bash
codex mcp add jawafdehi --url https://api.jawafdehi.org/mcp
```

### Cursor

Settings → **MCP** → **Add new global MCP server**, or edit `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "jawafdehi": {
      "url": "https://api.jawafdehi.org/mcp"
    }
  }
}
```

### VS Code (GitHub Copilot)

```bash
code --add-mcp '{"name":"jawafdehi","type":"http","url":"https://api.jawafdehi.org/mcp"}'
```

Or run **MCP: Add Server** from the Command Palette, choose **HTTP**, and paste the
URL. In a workspace it lands in `.vscode/mcp.json` under `"servers"`.

### Windsurf

Edit `~/.codeium/windsurf/mcp_config.json`. Windsurf names the field `serverUrl`, not
`url` — a Cursor-shaped block fails silently there:

```json
{
  "mcpServers": {
    "jawafdehi": {
      "serverUrl": "https://api.jawafdehi.org/mcp"
    }
  }
}
```

### Any other client

Add a remote MCP server over streamable HTTP pointing at
`https://api.jawafdehi.org/mcp`.

## The plugin and its skills

Having the tools is not the same as using them well. The archive is large enough that
an agent will burn a context window on one document, or answer a question about a
person from the court corpus when the case corpus is the right place to look.

[`plugins/jawafdehi`](plugins/jawafdehi) is a Claude plugin that bundles the server
(its `.mcp.json` points at `https://api.jawafdehi.org/mcp`) with two
[agent skills](https://docs.claude.com/en/docs/claude-code/skills):

- **`jawafdehi-archive`** teaches the call order, which tool answers which question, the
  traps in the data, and how to answer from it — keeping accused apart from convicted,
  never summing बिगो figures across cases, and citing a source for every claim. Two
  reference files load on demand: `organisation.md` (what Jawafdehi is, who runs it,
  funding, reporting a case, licensing and reuse) and `glossary.md` (the Nepali and legal
  terms that appear in results).
- **`jawafdehi-document-reader`** covers reading *inside* one document — a page range, a
  table or a chart at a time — without loading the whole file.

Install it in Claude Code:

```
/plugin marketplace add Jawafdehi/mcp
/plugin install jawafdehi@jawafdehi-mcp
```

For Cursor, Codex and anything else that reads `SKILL.md`, use the
[`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add Jawafdehi/mcp --skill jawafdehi-archive
npx skills add Jawafdehi/mcp --skill jawafdehi-document-reader
```

Add `-g` to install for every project rather than this one, `-a claude-code` to pick a
single agent, or `--copy` if your agent does not follow symlinks. Failing all that, copy
the `SKILL.md` files from `plugins/jawafdehi/skills/` into wherever your agent keeps its
instructions.

## Signing in

**Signing in is required for every tool.** Clients that support MCP sign-in (Claude, ChatGPT) prompt for it on the first call. For command-line clients:

```bash
claude mcp add --transport http jawafdehi https://api.jawafdehi.org/mcp \
  --client-id 380811001584419184 -s user
```

```bash
codex mcp add jawafdehi --url https://api.jawafdehi.org/mcp \
  --oauth-client-id 380811001584419184 && codex mcp login jawafdehi
```

In Claude Code, run `/mcp`, select **jawafdehi**, and choose **Authenticate**. On a
machine with no browser use `claude mcp login jawafdehi --no-browser`; the callback to
`localhost` failing is expected — paste that failed URL back into the terminal.

## What you can ask

- *Which corruption cases has Jawafdehi published about hydropower projects?*
- *Summarise Jawafdehi's case on the Giribandhu Tea Estate land swap.*
- *How many corruption cases did the Special Court decide in 2080 BS?*
- *Show me the CIAA press releases a case is built on.*
- *What is 15 Shrawan 2081 BS in the Gregorian calendar?*

## Tools

A signed-in reader sees these 20 tools. Staff accounts also see tools that create, change or
delete records; they are not listed here.

### Read

| Tool | What it does |
|---|---|
| `search_archive` | One search across entities, documents, court cases and published cases — start here when you don't know which part of the archive holds the answer |
| `search_jawafdehi_cases` | Search published accountability cases by keywords, tags or case type |
| `get_jawafdehi_case` | One case in full: allegations, entities involved, evidence cited |
| `search_entities` | Search people, organisations and government bodies by text, type or prefix |
| `get_entities` | Complete profiles for one or more entities, by ID |
| `get_entity_history` | The edit history of one entity record |
| `list_entity_prefixes` | The valid entity types and their metadata |
| `browse_materials` | List source documents, or fetch one by its canonical IRI |
| `list_document_sources` | Every document collection — CIAA and Auditor General reports, charge sheets, court orders — with its size and how much has been read |
| `get_document_overview` | One document: page, table and figure counts, readability, links to the original PDF |
| `list_document_contents` | Its page map or table captions, without returning any text |
| `search_document` | Where a term appears inside one document, with snippets |
| `read_document_part` | One slice of it: a page range, a table, or a figure |
| `browse_court_data` | Courts, court cases, hearings, case parties and blacklisted firms |
| `query_court_records` | Read-only `SELECT` queries over the court tables |
| `convert_date` | Convert between Bikram Sambat and Gregorian dates, in Asia/Kathmandu |
| `convert_to_markdown` | Fetch a document from a public web address and convert it to text |
| `get_current_user` | Who you are signed in as, and with which roles |

### Change something

| Tool | What it does |
|---|---|
| `send_feedback` | Send a correction, a suggested source, a bug report or a tip to the Jawafdehi team |
| `request_document_read` | Ask Jawafdehi to read (and if needed OCR) a document it holds but has not read |

## Support

- Website: [jawafdehi.org](https://jawafdehi.org)
- Email: `inquiry@jawafdehi.org`
- Documentation: [jawafdehi.org/mcp](https://jawafdehi.org/mcp/)
