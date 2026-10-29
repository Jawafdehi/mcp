# Jawafdehi MCP

**Nepal's open accountability archive, served over the [Model Context Protocol](https://modelcontextprotocol.io).**

```
https://api.jawafdehi.org/mcp
```

Point any MCP client at that URL and your AI assistant can read every published
Jawafdehi case, the people and offices behind them, the source documents each case is
built on, and Nepal's court record — 2,600+ accountability cases and 2.2M+ court case
records.

This is a **hosted, remote server**. There is nothing to install, no API key to issue,
and no account to create for the public tools.

## Contents

- [Try it without a client](#try-it-without-a-client)
- [Install](#install)
- [The skill](#the-skill)
- [Signing in](#signing-in)
- [What you can ask](#what-you-can-ask)
- [Tools](#tools)

## Try it without a client

The public catalogue needs no token, no session and no handshake beyond `initialize`:

```bash
curl -s https://api.jawafdehi.org/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call",
       "params":{"name":"search_control_plane","arguments":{"q":"Giribandhu"}}}'
```

## Install

### Claude Code

```bash
claude mcp add --transport http jawafdehi https://api.jawafdehi.org/mcp -s user
```

Then run `/mcp` to confirm it connected.

### Claude

Settings → **Connectors** → **Add custom connector**, and paste the server URL.

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

## The skill

Having the tools is not the same as using them well. The archive is large enough that
an agent will burn a context window on one document, or answer a question about a
person from the court corpus when the case corpus is the right place to look.

[`skills/jawafdehi-archive`](skills/jawafdehi-archive) is an [agent skill](https://docs.claude.com/en/docs/claude-code/skills) that teaches the call
order, which tool answers which question, the traps in the data, and how to answer from
it — keeping accused apart from convicted, never summing बिगो figures across cases, and
citing a source for every claim. Two reference files load on demand: `organisation.md`
(what Jawafdehi is, who runs it, funding, reporting a case, licensing and reuse) and
`glossary.md` (the Nepali and legal terms that appear in results).

Install it in Claude Code:

```
/plugin install jawafdehi-archive --marketplace Jawafdehi/mcp
```

Requires Claude Code 2.1.275 or later; on earlier versions run
`/plugin marketplace add Jawafdehi/mcp` first.

For Cursor, Codex and anything else that reads `SKILL.md`, use the
[`skills`](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add Jawafdehi/mcp --skill jawafdehi-archive
```

Add `-g` to install it for every project rather than this one, `-a claude-code` to pick
a single agent, or `--copy` if your agent does not follow symlinks. Failing all that,
copy `skills/jawafdehi-archive/SKILL.md` into wherever your agent keeps its
instructions.

Reading *inside* one document — a page range, a table or a chart at a time — is covered
by a companion skill shipped from the API repo:

```
/plugin install jawafdehi-document-reader --marketplace Jawafdehi/JawafdehiAPI
```

## Signing in

**Signing in is optional.**

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

### Public

| Tool | What it does |
|---|---|
| `search_control_plane` | One search across entities, materials, court cases and published cases — start here when you don't know which archive holds the answer |
| `search_jawafdehi_cases` | Search published accountability cases by keywords, tags or case type |
| `get_jawafdehi_case` | One case in full: allegations, entities involved, evidence cited |
| `search_nes_entities` | Search people, organisations and government bodies by text, type, prefix or tag |
| `get_nes_entities` | Complete profiles for one or more entities, by ID |
| `get_nes_entity_versions` | The edit history of one entity record |
| `get_nes_entity_prefixes` | The valid entity prefixes and their metadata |
| `get_nes_tags` | Every entity tag in use |
| `browse_materials` | List source documents, or fetch one by its canonical IRI |
| `browse_court_data` | Courts, court cases, hearings, case parties and blacklisted firms |
| `convert_date` | Convert between Bikram Sambat and Gregorian dates, in Asia/Kathmandu |
| `get_current_user` | Who the server thinks you are, and with which roles |

### Authenticated

| Tool | What it does |
|---|---|
| `list_document_sources` | Every document shelf — CIAA and Auditor General reports, audit reports, charge sheets, court orders — with its size |
| `get_document_overview` | One document: page, table and figure counts, readability, links to the original PDF |
| `list_document_contents` | Its page map or table captions, without returning any text |
| `read_document_part` | One slice of it: a page range, a table, or a figure |
| `ngm_extract_case_data` | One judicial case extracted to Markdown |
| `ngm_query_judicial` | `SELECT` queries against the court tables over the gated SQL plane (needs the ReadOnly role) |

## Support

- Website: [jawafdehi.org](https://jawafdehi.org)
- Email: `inquiry@jawafdehi.org`
