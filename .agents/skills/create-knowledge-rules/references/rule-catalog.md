# Rule catalog

Every rule the skill can put into the outputs, grouped by part. Use the rule IDs in the generated document so a
later run can match and update them. Write the rules in the user's language and adapt names (folder, field,
status values) to their answers; keep the intent.

## How to rank

Each rule has a base priority and boosts from situation signals (see [`interview.md`](interview.md)):

- `core` — always kept, in every tier. Shorten the wording before dropping one.
- `high` → `medium` → `low` — kept in this order until the tier limit is reached.
- `n/a` — leave the rule out and do **not** list it as excluded (it does not apply, for example R-READ-8
  without a retrieval tool).

A rule's priority is the highest of its base and every boost whose signal is on. Ties: keep the rule that
prevents the costlier mistake (wrong answer > duplicate > inconsistent format).

Layer: `A` = the always-on section in `AGENTS.md`; `D` = the rules document.

Sources: Diátaxis — <https://diataxis.fr/>; ADR / MADR — <https://adr.github.io/>; Zettelkasten —
<https://zettelkasten.de/overview/>; KCS — <https://library.serviceinnovation.org/KCS/Principles_and_Core_Concepts>;
ISO 30401 — <https://www.iso.org/standard/68683.html>; Google developer style —
<https://developers.google.com/style>; The Good Docs Project — <https://gitlab.com/tgdp/templates>; progressive
disclosure — <https://agentskills.io/specification#progressive-disclosure>.

## Read

| ID | Layer | Rule | Source | Priority |
| --- | --- | --- | --- | --- |
| R-READ-1 | A | When the task or a skill names a knowledge file, start from it. | Progressive disclosure | core |
| R-READ-2 | A | Otherwise search file names, frontmatter, headings, and exact error strings; leave the source archive out of routine searches. | Progressive disclosure | core |
| R-READ-3 | A | Read the frontmatter, first paragraph, and headings of a few candidates; then read only the matching section. | Progressive disclosure | core |
| R-READ-4 | A | Follow related links only when context is missing; open the glossary only for unclear terms. | Progressive disclosure, Zettelkasten | medium; high with `S-LARGE` |
| R-READ-5 | A | Prefer verified content. Name the status when using draft, deprecated, or superseded content, and point out contradictions instead of picking one silently. | ISO 30401 (knowledge quality) | core |
| R-READ-6 | A | When to consult knowledge first: the option the user chose (every task / matching area / on request). | — | core |
| R-READ-7 | A | Cite the knowledge files an answer relies on (or the option the user chose). | Google style (reference hierarchy) | medium; high with `S-CITE` |
| R-READ-8 | A | Query the retrieval tool first, then open the files it returns and read the matching section before relying on it. | — | n/a; high with `S-RAG` |
| R-READ-9 | A | When knowledge is missing or wrong, do what the user chose: note it, propose an entry, or write it. | KCS (capture in the workflow) | high |
| R-READ-10 | D | Treat versions, prices, limits, and external APIs in knowledge as a baseline; confirm them against the live source when they decide the outcome. | ISO 30401 (currency of knowledge) | medium; high with `S-REFERENCE` or `S-EXTERNAL` |

## Before writing (always-on checks)

| ID | Layer | Rule | Source | Priority |
| --- | --- | --- | --- | --- |
| R-PRE-1 | A | Search for an existing document on the same topic (synonyms and aliases too). Extend it rather than adding a new one. | KCS (reuse), Zettelkasten | core |
| R-PRE-2 | A | Write down only what a later task can reuse, with enough context to stand alone; mark unverified content as draft. | KCS, ISO 30401 | core |
| R-PRE-3 | A | Never write credentials, tokens, or private keys; use placeholders such as `<API_TOKEN>`. | — | core |
| R-PRE-4 | A | Read the rules document before creating, moving, or restructuring knowledge. | — | core |

## Distil

| ID | Layer | Rule | Source | Priority |
| --- | --- | --- | --- | --- |
| R-DIST-1 | D | Capture knowledge while the work happens — at the end of a task, after an incident, during research, when deciding — not weeks later. | KCS (capture in the workflow) | medium; high with `S-CAPTURE` or `S-REPEAT` |
| R-DIST-2 | D | Good enough to solve beats perfect: record a usable draft now and improve it when it is reused. | KCS (sufficient to solve) | medium; high with `S-CAPTURE` or `S-FRAGMENTS` |
| R-DIST-3 | D | Keep scratch work (task logs, transcripts, drafts) out of the knowledge base; distil the reusable facts and file them by topic instead of moving whole logs. | KCS, Zettelkasten | medium; high with `S-AFTER` or `S-CAPTURE` |
| R-DIST-4 | D | Eligibility test: would another task need this in three months? Does it hold for another person or time? Is there enough context? | ISO 30401, KCS | medium |
| R-DIST-5 | D | Structure for reuse: state scope and preconditions; in troubleshooting entries keep the exact error text, the cause, and the fix. | KCS (structure for reuse) | medium; high with `S-STEPS` or `S-REPEAT` |
| R-DIST-6 | D | Keep external originals unchanged in the source archive, with their URL, version or date, and license; write summaries as separate documents that link back. | — | n/a; high with `S-EXTERNAL` |
| R-DIST-7 | D | One idea per note. Split when a part can be cited on its own; do not split what must be read together. | Zettelkasten (atomicity) | low; high with `S-FRAGMENTS`, medium with `S-LARGE` |

## Write

| ID | Layer | Rule | Source | Priority |
| --- | --- | --- | --- | --- |
| R-WRITE-1 | D | One topic per document; link to the document that owns another topic instead of copying it. | Zettelkasten, Diátaxis | core |
| R-WRITE-2 | D | Give each document one type — tutorial, how-to, reference, explanation — chosen with two questions: doing or knowing? learning or working? | Diátaxis (compass) | medium; high with `S-HUMANS`, `S-STEPS`, or `S-CONCEPTS` |
| R-WRITE-3 | D | Record a decision as its own document: context, options considered, the choice, and its consequences, including the downsides accepted. | ADR / MADR | low; high with `S-DECISIONS` |
| R-WRITE-4 | D | Organise knowledge in topic folders, each with a `README.md` index; link every new document from an index so none is orphaned. | Zettelkasten (structure notes) | medium; high with `S-LARGE` or `S-MANY-WRITERS` |
| R-WRITE-5 | D | Start each document with frontmatter: `title`, `domain`, `doc_type`, `status`; optional `aliases`, `related`, `references`. | — | low; high with `S-STRICT`, `S-LARGE`, or `S-AGENT-ONLY` |
| R-WRITE-6 | D | Name files in lowercase kebab-case after their topic; decisions as `NNNN-title.md`. | — | low; medium with `S-MANY-WRITERS` |
| R-WRITE-7 | D | Open with a paragraph that says what the document covers and what it does not. Be specific and verifiable; put commands, paths, and values in code formatting; state limits and defaults; keep a neutral tone. | Google style | medium; high with `S-HUMANS` |
| R-WRITE-8 | D | For content that depends on versions or environments, add an "Applicability" section with the verified range. | Google style | low; high with `S-REFERENCE` or `S-EXTERNAL` |
| R-WRITE-9 | D | Start new documents from the template for their type. | The Good Docs Project | low; high with `S-STRICT`, medium with `S-MANY-WRITERS` |
| R-WRITE-10 | D | Define terms once in a glossary, with aliases for synonyms and old names; use the same term everywhere. | — | low; high with `S-LARGE` or `S-MANY-WRITERS` |
| R-WRITE-11 | D | Knowledge links only to other knowledge. Procedures, agent instructions, and scratch areas may link to knowledge, never the reverse. | — | low; medium with `S-LARGE` |
| R-WRITE-12 | D | Use relative links with descriptive text, and list the main ones in `related`. | Zettelkasten (links) | medium |

## Update

| ID | Layer | Rule | Source | Priority |
| --- | --- | --- | --- | --- |
| R-UPD-1 | D | When two documents say the same thing, keep it in the one that owns the topic and link from the other. | KCS, Zettelkasten | high |
| R-UPD-2 | D | Reuse is review: when using a document reveals an error or a gap, fix it on the spot or flag it. | KCS (reuse is review) | medium; high with `S-REPEAT` |
| R-UPD-3 | D | Track status: `draft`, `verified`, `deprecated`, `superseded`, `archived` (meanings in the example). | ISO 30401 (lifecycle) | medium; high with `S-LARGE`, `S-MANY-WRITERS`, or `S-STRICT` |
| R-UPD-4 | D | Outdated content is updated now or marked deprecated with the reason; replaced content is marked superseded and links to its replacement. Never leave known-outdated content unmarked. | ISO 30401 | core |
| R-UPD-5 | D | A decision that changes gets a new record; mark the old one superseded instead of rewriting it. | ADR | n/a; high with `S-DECISIONS` |
| R-UPD-6 | D | Review is triggered by events: a source changes, a reader finds a problem, a task reports an error. | ISO 30401 (continual improvement) | medium; high with `S-MANY-WRITERS` |
| R-UPD-7 | D | Periodic review: on the chosen cadence, list stale drafts, orphans, and duplicates, and fix or retire them. | ISO 30401 (performance evaluation) | n/a; high with `S-PERIODIC` |
| R-UPD-8 | D | Shared ownership: whoever finds a problem fixes or flags it; there is no per-file owner. | ISO 30401, KCS (collective ownership) | low; medium with `S-MANY-WRITERS` |
| R-UPD-9 | D | Dates and authors come from version control; do not maintain them in frontmatter. | — | low; medium with `S-STRICT` |
| R-UPD-10 | D | Run the quality checklist before committing knowledge changes. | — | low; high with `S-MANY-WRITERS` or `S-STRICT` |
