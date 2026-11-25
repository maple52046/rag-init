# Knowledge management rules

The knowledge base in `knowledge/` keeps long-lived knowledge that later work can reuse: procedures,
troubleshooting entries, reference facts, design decisions, and explanations. People and agents both read
it — to look up an answer during a task, to follow steps, or to understand background. It is written in
English.

These are the default rules that ship with rag-init. They assume a small team whose members and agents write
knowledge while they work and review it when something turns out wrong. To fit the rules to your own
situation, run the `create-knowledge-rules` skill (`/create-knowledge-rules`, or `$create-knowledge-rules` in
Codex); it rewrites this document and the `knowledge-rules` section of `AGENTS.md`.

Tier: Standard. The always-on rules live in the `knowledge-rules` section of [`AGENTS.md`](../AGENTS.md); this
document holds the rest. Read it before creating, moving, or restructuring knowledge.

## Layout

| Path | Holds |
| --- | --- |
| `knowledge/README.md` | The root index: every domain and what it covers |
| `knowledge/<domain>/` | Active knowledge, one folder per domain, with a `README.md` index |
| `knowledge/<domain>/decisions/` | Decision records for that domain |
| `knowledge/taxonomy.md`, `knowledge/glossary.md` | The list of domains and their boundaries; term definitions |
| `knowledge/sources/` | External originals and summaries of them, kept apart from the team's own knowledge |

Scratch work — task logs, transcripts, experiments — stays outside `knowledge/`.

## 1. Reading

The core reading rules are in `AGENTS.md`. In addition:

- **R-READ-4** Follow `related` links only when the current document lacks context. Read
  `knowledge/taxonomy.md` only when the domain is unclear and `knowledge/glossary.md` only when a term is.
  *(progressive disclosure, Zettelkasten)*
- **R-READ-10** Versions, prices, limits, and external APIs in knowledge are a baseline. When one decides the
  outcome of a task, confirm it against the live source and update the document if it changed. *(ISO 30401)*

## 2. Distilling

- **R-DIST-1** Capture knowledge while the work happens: when a task ends, after an incident, during research,
  and when a decision is made. Rejected options and exact error messages are gone within days. *(KCS)*
- **R-DIST-2** A usable draft now beats a perfect document later. Mark it `status: draft`; reuse will improve
  it. *(KCS)*
- **R-DIST-3** Keep scratch work out of `knowledge/`. Move only the reusable facts, filed under the domain they
  belong to, instead of copying a whole log into one file. *(KCS, Zettelkasten)*
- **R-DIST-4** Before keeping something, ask: would another task need it in three months? Does it hold for
  another person or time? Is there enough context to understand it without the original conversation?
  *(ISO 30401, KCS)*
- **R-DIST-5** Structure for reuse: state the scope and preconditions; in troubleshooting entries keep the
  exact error text, the cause, and the fix, so a search for the error finds the entry. *(KCS)*
- **R-DIST-6** External material goes to `knowledge/sources/<source>/` unchanged, with its URL, version or
  retrieval date, and license in that folder's `README.md`. Write the team's own summaries as separate
  documents that link to the original. Do not store material whose license forbids redistribution; link to
  it instead.

## 3. Writing

- **R-WRITE-1** One topic per document. Link to the document that owns another topic instead of copying it;
  every fact has one home. *(Zettelkasten, Diátaxis)*
- **R-WRITE-2** Give each document one `doc_type`, chosen with two questions: does it help the reader *do*
  something or *know* something? Is the reader *learning* or *working*? *(Diátaxis)*

| | Learning | Working |
| --- | --- | --- |
| **Doing** | `tutorial` | `how-to` |
| **Knowing** | `explanation` | `reference` |

Other types: `decision`, `glossary`, `taxonomy`, and `index` (a domain `README.md`). A document that teaches,
gives steps, lists facts, and explains reasons at once is split into one document per type.
- **R-WRITE-3** Record a decision in `knowledge/<domain>/decisions/NNNN-title.md` when several reasonable
  options existed and someone will later ask "why not the other one?". Include the context, the options
  considered, the choice and why, and the consequences, including the downsides accepted. *(ADR, MADR)*
- **R-WRITE-4** Every domain folder has a `README.md` index that states what the domain covers and lists its
  documents. Link every new document from its index and add new domains to `knowledge/README.md`; a document
  nothing links to is an orphan. *(Zettelkasten)*
- **R-WRITE-5** Every document starts with frontmatter; the first four fields are required:

```yaml
---
title: ""          # states the topic (and the fix, for troubleshooting)
domain: ""         # a domain listed in knowledge/taxonomy.md
doc_type: ""       # tutorial | how-to | reference | explanation | decision | glossary | taxonomy | index
status: "draft"    # see R-UPD-3
aliases: []        # optional: synonyms, abbreviations, old names
related: []        # optional: relative paths of related documents
references: []     # optional: external sources
---
```

- **R-WRITE-6** File names are lowercase kebab-case after the topic (`connection-pool-limits.md`); domain
  entries are `README.md`; decisions are `NNNN-title-with-dashes.md`, numbered per domain.
- **R-WRITE-7** Open with a paragraph that says what the document covers and what it does not. Be specific and
  verifiable; put commands, paths, and values in code formatting; state preconditions, limits, and defaults;
  keep a neutral tone. *(Google developer documentation style)*
- **R-WRITE-8** When a conclusion depends on versions, operating systems, or external services, add an
  `## Applicability` section with the verified range. *(Google developer documentation style)*
- **R-WRITE-10** Define each term once in `knowledge/glossary.md`, with aliases for synonyms and old names, and
  use the same term everywhere. Add a domain to `knowledge/taxonomy.md` before creating its folder.
- **R-WRITE-12** Link with relative paths and descriptive link text, and list the main links in `related`.
  *(Zettelkasten)*

## 4. Updating

- **R-UPD-1** When two documents say the same thing, keep it in the one that owns the topic and link from the
  other. *(KCS, Zettelkasten)*
- **R-UPD-2** Reuse is review: when using a document shows it is wrong, outdated, or incomplete, fix it on the
  spot or flag it, instead of opening a new document. *(KCS)*
- **R-UPD-3** Every document has a status: *(ISO 30401)*

| Status | Meaning |
| --- | --- |
| `draft` | Not yet verified; usable, but provisional |
| `verified` | Confirmed by running, testing, or checking the source |
| `accepted` | Adopted as the stable reference (common for decisions) |
| `deprecated` | Outdated or no longer recommended; kept with the reason |
| `superseded` | Replaced by another document, which it links to |
| `archived` | No longer maintained; kept as a record |

- **R-UPD-4** Outdated content is updated now, or marked `deprecated` with the reason. Replaced content is
  marked `superseded` and links to its replacement. Known-outdated content is never left unmarked.
  *(ISO 30401)*
- **R-UPD-5** When a decision changes, add a new decision record and mark the old one `superseded`; do not
  rewrite it. *(ADR)*
- **R-UPD-6** Review is triggered by events: a source changes, a reader finds a problem, or a task reports an
  error. A review checks that the content is still correct and its scope still holds. *(ISO 30401)*
- **R-UPD-8** There is no per-file owner; whoever finds a problem fixes or flags it. *(ISO 30401, KCS)*

## Quality checklist

- **R-UPD-10** Before committing a knowledge change, check:

- [ ] Searched the domain, synonyms, and aliases for an existing document (R-PRE-1)
- [ ] `doc_type`, `domain`, and the required frontmatter fields are set (R-WRITE-2, R-WRITE-5)
- [ ] The first paragraph states what the document covers (R-WRITE-7)
- [ ] The domain index links the document, and its links resolve (R-WRITE-4, R-WRITE-12)
- [ ] No fact is duplicated from another document (R-WRITE-1, R-UPD-1)
- [ ] Outdated content is updated or marked (R-UPD-4)
- [ ] No credentials, tokens, or private keys (R-PRE-3)

## Sources

The rules above adapt ideas from these sources; their originals and summaries are in
[`knowledge/sources/`](../knowledge/sources/README.md).

- Diátaxis — <https://diataxis.fr/>
- Knowledge-Centered Service (KCS) — <https://library.serviceinnovation.org/KCS/Principles_and_Core_Concepts>
- Architectural Decision Records and MADR — <https://adr.github.io/>
- Zettelkasten — <https://zettelkasten.de/overview/>
- ISO 30401 knowledge management systems — <https://www.iso.org/standard/68683.html>
- Google developer documentation style guide — <https://developers.google.com/style>
- Progressive disclosure (Agent Skills) — <https://agentskills.io/specification#progressive-disclosure>

## Excluded rules

These rules were left out to stay within the Standard budget or because the default situation does not need
them yet.

| Rule | Why it was left out | Add it back when |
| --- | --- | --- |
| R-DIST-7 One idea per note | Most entries are procedures and references that read best whole | Knowledge arrives as many small notes |
| R-WRITE-9 Templates per type | Fixed templates slow down capture for a small team | Documents of the same type start to look different from each other |
| R-WRITE-11 Reference direction | Few documents outside `knowledge/` link to it yet | Procedures or agent instructions start linking to knowledge and links break |
| R-UPD-9 Dates from version control | Few people are tempted to add dates by hand | Documents start carrying `updated:` fields that drift |
