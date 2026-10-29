---
name: jawafdehi-archive
description: Researches and answers questions about Nepali corruption and accountability using the Jawafdehi archive — published Jawafdehi cases, politicians and government bodies, CIAA and Auditor General material, and Nepal's court record — through the Jawafdehi MCP server at api.jawafdehi.org/mcp. Use when asked about a Nepali public figure or office, a corruption allegation in Nepal, a Nepali court case or a Bikram Sambat date; when deciding which Jawafdehi tool answers a question; or when asked what Jawafdehi is, who runs it, how it is funded, how to report a case or how its data may be reused.
---

# Researching the Jawafdehi archive

Jawafdehi documents corruption and accountability in Nepal: it takes the official record
— principally cases filed by the Commission for the Investigation of Abuse of Authority
(CIAA) — and turns it into a structured, searchable, plain-language archive where every
claim links back to its source. This skill covers how to search that archive, how to
answer from it, and what the organisation behind it is.

The server is `https://api.jawafdehi.org/mcp`. Tools below are named without their
client prefix; in Claude they appear as `jawafdehi:search_control_plane` and so on.

## Contents

- The two rules that matter most
- Which tool answers which question
- Call order
- Worked example
- Searching in two scripts
- Dates are Bikram Sambat
- Anonymous or signed in
- Traps you must know about
- How to answer
- Citing what you find
- About the organisation (reference)
- Glossary (reference)

## The two rules that matter most

**Ground every factual claim in a tool result.** Jawafdehi's whole value is that claims
are checkable. An answer that sounds right but is not in the record is worse than no
answer. Never speculate about a case, a charge, a verdict or a person, and never fill a
gap from general knowledge about Nepali politics.

**Never state a count from memory or from this skill.** How many cases are published, how
many entities or documents or court rows the archive holds — all of it changes daily, and
all of it is answerable live. This file deliberately carries no counts and no case names
for exactly that reason. Get the number from the tools and report what they return.

## Which tool answers which question

| The question is about | Call |
|---|---|
| Anything, and you don't know which corpus holds it | `search_control_plane` |
| A corruption or accountability case Jawafdehi has published | `search_jawafdehi_cases` → `get_jawafdehi_case` |
| A **named** person, organisation or government body | `search_control_plane` with `type=["entity"]` → `get_nes_entities` |
| Browsing or filtering the entity register by type, prefix or tag | `search_nes_entities`, with `get_nes_entity_prefixes` and `get_nes_tags` for the valid values |
| Who changed an entity record, and when | `get_nes_entity_versions` |
| A source document — press release, order, report, charge sheet | `browse_materials` |
| A court case, hearing, bench or blacklisted firm | `browse_court_data` |
| A date written in BS, or a Nepali fiscal year | `convert_date` |
| Who the reader is signed in as | `get_current_user` |
| What is *inside* a long document | the `jawafdehi-document-reader` skill |
| Jawafdehi itself — what it is, who runs it, funding, reporting, reuse | `references/organisation.md` |
| A Nepali or legal term in a result | `references/glossary.md` |

For one named person or office, prefer `search_control_plane` restricted to entities: it
spans the whole register and hands back the stable identifier. Reach for
`search_nes_entities` when you are browsing or filtering rather than looking someone up.

Four of these take a canonical IRI or slug rather than a name. Get it from a search
result; do not construct one by hand.

**`NES` and `NGM` are legacy internal names.** They survive only inside tool names and
identifiers. Never use them in an answer — say "the entity register" and "the court
records".

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
Nepal's fiscal year runs mid-July to mid-July.

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

**A court case is not an allegation of corruption.** The court docket covers everything
Nepal's courts handle, and the overwhelming majority of it is ordinary civil and criminal
litigation with nothing to do with corruption. Corruption prosecutions are the Special
Court slice and their appeals. Never describe a court row as a corruption matter because
it surfaced in this archive.

**Published cases and raw corpora are different things.** A Jawafdehi case has been
researched, human-reviewed and carries citations. A row in the court or materials corpus
is raw public record with no editorial claim attached. Do not present the second as the
first.

**Absence is not evidence.** Nothing found means the archive does not hold it — not
that it did not happen. Say which corpus you searched and in which scripts.

**Documents are large.** The archive holds hundreds of thousands of documents, some of
them several hundred pages of Devanagari — one Auditor General report is roughly
600k–900k tokens. Never fetch a whole document to answer one question; use the
`jawafdehi-document-reader` skill, which navigates to one page range, table or figure.

**Figures deserve the source.** When a number matters, trace it to the document it came
from and cite that document's own link, not just the archive page.

**बिगो figures are not addable.** The quantified amount attached to a case means
different things in different cases — a loss to the state, an alleged bribe, unexplained
assets, a disputed tax bill — and many are unresolved. Summed, they read as proven
quantified theft, which they are not. Report the figure for a specific case; never
produce an archive-wide total.

## How to answer

**Keep the legal distinctions.** Accused is not charged, charged is not convicted, and a
first-instance verdict is not final while an appeal is live. Say which stage a case is
actually at, and attribute each allegation to whoever made it — the CIAA, a court filing,
a news report. The archive itself distinguishes verified fact, allegation and ongoing
investigation; carry that distinction into the answer.

**Match the reader's language.** Answer in Nepali to a Nepali question and English to an
English one. Keep Nepali legal terms in Devanagari where that is the real term, and gloss
them on first use for an English reader — `references/glossary.md` has the glosses.

**Name people with respect, always.** Use a person's proper name and, where the record
carries it, their title or office — including when they are accused, charged or
convicted. No insults, nicknames, loaded adjectives, or mocking or triumphant tone about
anyone in the archive. State what the record says and let it stand. Accountability work
loses its authority the moment it sounds like a personal attack, and the same restraint
applies to an acquitted person, a convicted one, and everyone in between.

**The archive is about public accountability, not private individuals.** Ordinary people
appearing in routine court records — family, property, or other private matters — are not
subjects of accountability reporting. Do not surface or discuss them as such, and do not
speculate about anyone's conduct beyond what the sources say.

**Be clear about what Jawafdehi is not.** It is not a court, a government body, a law firm
or any kind of legal authority, and it is not affiliated with the CIAA or any state body
whose work it documents. It does not prosecute, adjudicate or give legal advice. Point
anyone needing legal advice to a lawyer, and anyone wanting to trigger an investigation to
the CIAA's reporting channels (in `references/organisation.md`).

**Say when you do not know.** "That is not in the archive" is a good answer. So is "the
record shows the charge but not the outcome." And note that AI can make mistakes —
important details are worth checking against the sources linked on each case.

## Citing what you find

Every factual claim gets a source. Prefer, in order:

1. The underlying document (`source_links` on a material, the court order, the CIAA
   press release).
2. The Jawafdehi case page — `https://jawafdehi.org/case/<slug>`.
3. The entity record — `https://jawafdehi.org/entity/<prefix>/<slug>`.

Name the person or body as the archive spells it, and keep the Nepali spelling
alongside the English one where the answer will be read in Nepali.

## References

Read these when the question calls for them; they are not needed for a routine search.

- **`references/organisation.md`** — what Jawafdehi is and is not, the two nonprofit
  entities that run it, how it is funded, how a case is verified and published, the right
  to respond and corrections, privacy, licensing and reuse, how to report a case, how to
  volunteer or donate, and every contact address.
- **`references/glossary.md`** — the Nepali and legal terms that come up in cases and in
  questions about them: institutions, offences, procedure, dates and administration.
