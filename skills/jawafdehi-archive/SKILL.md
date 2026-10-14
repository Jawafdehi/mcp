---
name: jawafdehi-archive
description: Researches Nepali corruption and accountability — published Jawafdehi cases, politicians and government bodies, CIAA and Auditor General material, and Nepal's court record — through the Jawafdehi MCP server. Use when asked about a Nepali public figure or office, a corruption allegation in Nepal, a Nepali court case, or a Bikram Sambat date, and when deciding which Jawafdehi tool answers a question.
---

# Researching the Jawafdehi archive

The server is `https://api.jawafdehi.org/mcp`. Tools below are named without their
client prefix; in Claude they appear as `jawafdehi:search_control_plane` and so on.

## Contents

- Which tool answers which question
- Call order
- Worked example
- Searching in two scripts
- Dates are Bikram Sambat
- Anonymous or signed in
- Traps you must know about
- Citing what you find

## Which tool answers which question

| The question is about | Call |
|---|---|
| Anything, and you don't know which corpus holds it | `search_control_plane` |
| A corruption or accountability case Jawafdehi has published | `search_jawafdehi_cases` → `get_jawafdehi_case` |
| A person, organisation or government body | `search_nes_entities` → `get_nes_entities` |
| Who changed an entity record, and when | `get_nes_entity_versions` |
| A source document — press release, order, report, charge sheet | `browse_materials` |
| A court case, hearing, bench or blacklisted firm | `browse_court_data` |
| A date written in BS, or a Nepali fiscal year | `convert_date` |
| What is *inside* a long document | the `jawafdehi-document-reader` skill |

Four of these take a canonical IRI or slug rather than a name. Get it from a search
result; do not construct one by hand.

## Call order

Search first, read second. The search tools return slugs, IRIs and snippets cheaply;
only the `get_*` and `browse_* action=get` calls return a full record.

```
- [ ] 1. Decide the corpus. Unsure → search_control_plane with type=[...]
- [ ] 2. Search it, in both scripts if the subject is a Nepali name
- [ ] 3. Take the slug / IRI from the result
- [ ] 4. Fetch the one record you need
- [ ] 5. Cite the source URL for every factual claim
```

`search_control_plane` is the one that spans everything: `type` accepts any of
`entity`, `material`, `courtcase`, `case`, and it returns per-type counts plus facets,
so a single call tells you where the subject actually lives before you commit to a
corpus.

## Worked example

**Question:** "Summarise Jawafdehi's case on the Giribandhu Tea Estate land swap."

**Step 1** — `search_control_plane(q="Giribandhu")` → `counts: {material: 4, case: 1,
entity: 1}`. There is a published case, so the case corpus is the right place.

**Step 2** — `search_jawafdehi_cases(search="Giribandhu")`:

```json
{"count": 1, "results": [
  {"slug": "giribandhu-tea-estate-land-swap",
   "title": "Giribandhu Tea Estate Land Swap Scandal",
   "case_type": "CORRUPTION", "date": "2019-02-18",
   "url": "/case/giribandhu-tea-estate-land-swap"}]}
```

**Step 3** — `get_jawafdehi_case(slug="giribandhu-tea-estate-land-swap")` for the
allegations, the entities involved and the evidence cited.

**Step 4** — cite `https://jawafdehi.org/case/giribandhu-tea-estate-land-swap`, plus
the underlying document for any specific figure.

Note what step 2 did **not** need: a case ID. `get_jawafdehi_case` takes the `slug`.
Entity tools take the full canonical IRI instead —
`https://jawafdehi.org/entity/person/ram-chandra-poudel` — and `get_nes_entities`
accepts up to 100 of them in one call, so batch rather than looping.

## Searching in two scripts

The archive is bilingual and Nepali names romanise inconsistently (Shrestha/Shrestha,
Paudel/Poudel/Paudyal, Chaudhary/Chaudhari). A single-script search is the most common
reason a subject looks absent when it is not.

- `search_control_plane` takes `lang` = `ne`, `en` or `both`. It defaults to `both` —
  leave it there unless you are deliberately isolating one language.
- Search the Devanagari spelling as well as the Roman one when the subject is a Nepali
  person, office or place.
- Offices are often known by an acronym in English and a full name in Nepali — try
  "CIAA" and "अख्तियार दुरुपयोग अनुसन्धान आयोग" both.

## Dates are Bikram Sambat

Nepali sources date everything in BS, and fiscal years are written `2081/82`. Do not
convert in your head — LLMs get this wrong, and the archive's date filters
(`date_from`, `date_to`) take AD.

Call `convert_date` with `mode="bs_to_ad"` or `"ad_to_bs"`; it takes up to 100 dates at
a time in `YYYY-MM-DD` form and works in Asia/Kathmandu. A BS year starts in mid-April,
so "2081" spans parts of both 2024 and 2025 — never map a BS year onto a single AD year.

## Anonymous or signed in

The server never returns `401`. An unauthenticated caller gets a successful response
and a read-only catalogue of 12 tools. So:

- If a write tool (case drafting, entity writes, uploads) or a document-reading tool is
  "missing", you are anonymous. Reconnecting will not change that; the user has to
  authenticate.
- `get_current_user` is the check. `"authenticated": true` plus roles means you are in;
  `false` means you are on the public catalogue.
- Seeing a tool is not permission to use it. Writes still answer to the caller's roles
  (Caseworker, Moderator, Admin) at the API.

Never attempt a write on the user's behalf without being asked to. Case and entity
records are public, attributed, and versioned — every change is recorded as that
person.

## Traps you must know about

**The court corpus is bulk record, not curated.** It holds millions of court case rows
scraped from the court system. Party names are stored as the court wrote them and only
a small fraction are resolved to entity records, so `browse_court_data` is excellent
for *"what happened in this case number"* and unreliable for *"every case involving
this person"*. Answer person-level questions from the case and entity corpora.

**Absence is not evidence.** Nothing found means the archive does not hold it — not
that it did not happen. Say which corpus you searched and in which scripts.

**Published cases and raw corpora are different things.** A Jawafdehi case has been
researched and carries citations. A row in the court or materials corpus is raw public
record with no editorial claim attached. Do not present the second as the first.

**Documents are large.** The archive holds hundreds of thousands of documents, some of
them several hundred pages of Devanagari — one Auditor General report is roughly
600k–900k tokens. Never fetch a whole document to answer one question; use the
`jawafdehi-document-reader` skill, which navigates to one page range, table or figure.

**Figures deserve the source.** When a number matters, trace it to the document it came
from and cite that document's own link, not just the archive page.

## Citing what you find

Every factual claim gets a source. Prefer, in order:

1. The underlying document (`source_links` on a material, the court order, the CIAA
   press release).
2. The Jawafdehi case page — `https://jawafdehi.org/case/<slug>`.
3. The entity record — `https://jawafdehi.org/entity/<prefix>/<slug>`.

Name the person or body as the archive spells it, and keep the Nepali spelling
alongside the English one where the answer will be read in Nepali.
