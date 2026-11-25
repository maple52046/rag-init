# Knowledge management rules

<!--
Example output of the create-knowledge-rules skill at the Full tier, for a fictional team: several engineers
and their agents maintain a knowledge base about their payment platform. They record procedures,
troubleshooting, reference facts, design decisions, and summaries of vendor documentation; knowledge is
captured while working, reviewed when problems are found, and both people and agents read it.
Use it as a model for structure and wording; generate real rules from the user's own situation.
-->

The knowledge base in `knowledge/` keeps long-lived knowledge that later work can reuse: how to operate and
fix the payment platform, the facts it depends on, and why it is built the way it is. Engineers and their
agents both read it — to look up an answer in the middle of a task, to follow a procedure, or to understand
background. It is written in English.

Tier: Full. The always-on rules live in the `knowledge-rules` section of [`AGENTS.md`](../AGENTS.md); this
document holds the rest. Read it before creating, moving, or restructuring knowledge.

## Layout

| Path | Holds | Rules |
| --- | --- | --- |
| `knowledge/<domain>/` | Active knowledge, one folder per domain (for example `knowledge/payments/`, `knowledge/database/`, `knowledge/deployment/`) | Everything in this document |
| `knowledge/<domain>/README.md` | The domain index: scope, reading order, external references | R-WRITE-4 |
| `knowledge/<domain>/decisions/` | Decision records for that domain | R-WRITE-3, R-UPD-5 |
| `knowledge/glossary.md`, `knowledge/taxonomy.md` | Term definitions; the list of domains and their boundaries | R-WRITE-10 |
| `knowledge/sources/` | External originals and snapshots, unchanged | R-DIST-6 |
| `AGENTS.md`, `playbooks/` | Agent instructions and operational procedures that use knowledge | Link down to knowledge; knowledge does not link up (R-WRITE-11) |
| `scratch/` | One-off work: task logs, transcripts, experiments | Distil, do not move (R-DIST-3) |

This team chose to let people and agents share one corpus. A team whose knowledge base serves agents only can
drop the human-facing writing rules (R-WRITE-7, the tutorial type) and keep the classification and retrieval
rules.

## 1. Reading

The core reading rules (where to start, how to search, how to judge status) are in `AGENTS.md`. In addition:

- **R-READ-4** Follow `related` links only when the current document lacks the context you need. Read
  `knowledge/taxonomy.md` only when the domain is unclear and `knowledge/glossary.md` only when a term is.
  *(progressive disclosure, Zettelkasten)*
- **R-READ-10** Versions, prices, limits, and vendor APIs in knowledge are a baseline. When one decides the
  outcome of a task, confirm it against the live source and update the document if it changed.
  *(ISO 30401)*

A minimal lookup:

```sh
rg -l -i '<terms>' knowledge -g '*.md' -g '!sources/**'
sed -n '1,24p' <candidate>
rg -n '^#{1,3} ' <candidate>
```

## 2. Distilling

- **R-DIST-1** Capture knowledge while the work happens: when a task ends, after an incident review, during
  research, and when a decision is made. Details such as rejected options and exact error messages are lost
  within days. *(KCS)*
- **R-DIST-2** Record a usable draft now rather than a perfect document later. A draft that lets the next
  reader find, understand, and apply the fix is enough; reuse will improve it. Mark it `status: draft`. *(KCS)*
- **R-DIST-3** One-off output stays in `scratch/`. Move only the reusable facts into `knowledge/`, filed under
  the domain they belong to and linked across domains, instead of copying a whole log into one file.
  *(KCS, Zettelkasten)*
- **R-DIST-4** Before keeping something, ask: would another task need this in three months? Does it still
  hold for another person, another time, another task? Is there enough context to understand it without the
  original conversation? If most answers are no, leave it in `scratch/`. *(ISO 30401, KCS)*
- **R-DIST-5** Structure for reuse when capturing: state the scope and preconditions; in troubleshooting
  entries keep the exact error text, the cause, and the fix, so a search for the error finds the entry.
  *(KCS)*
- **R-DIST-6** External material goes to `knowledge/sources/<source>/` unchanged, with its URL, version or
  retrieval date, and license in that folder's `README.md`. Write summaries and team-specific notes as
  separate documents in the domain folder that link to the original. Do not store material whose license
  forbids redistribution; link to it instead.
- **R-DIST-7** One idea per document, with a clear boundary so it can be cited precisely. Split a document when
  a part is cited on its own from elsewhere or the title cannot state its subject in one sentence. Do not
  split what must be read together, and do not produce one-line fragments without context.
  *(Zettelkasten)*

Content that does not belong in `knowledge/`: one-off task records, unverified guesses, personal preferences,
fragments without context, and anything already known to be outdated but not marked `deprecated`.

## 3. Writing

### Topics and domains

- **R-WRITE-1** Each document covers one topic. Content that belongs to another topic is linked, not copied:
  every fact has one home. When a passage would hold true for any component, it is general knowledge — move it
  to the document that owns that topic and keep only what is specific to the current one. *(Zettelkasten,
  Diátaxis)*
- **R-WRITE-4** Every domain folder has a `README.md` index that states what the domain covers and excludes,
  lists its documents in reading order, and names external references. Link every new document from its index;
  a document that nothing links to is an orphan and must be linked or merged. *(Zettelkasten)*
- **R-WRITE-11** Knowledge is the most stable layer. Links from `knowledge/` go only to other knowledge;
  `AGENTS.md`, `playbooks/`, and `scratch/` may link to knowledge, never the reverse. A link from stable
  knowledge to a short-lived procedure or scratch file breaks when that file goes away.

### Document types

- **R-WRITE-2** Give each document one `doc_type`. Decide with two questions: does it help the reader *do*
  something or *know* something? Is the reader *learning* or *working*? *(Diátaxis)*

| | Learning | Working |
| --- | --- | --- |
| **Doing** | `tutorial` — a guided first success | `how-to` — steps to complete a real task |
| **Knowing** | `explanation` — why it works this way, trade-offs | `reference` — neutral facts, parameters, limits |

Other types: `decision` (one choice with its reasoning, R-WRITE-3), `glossary`, `taxonomy`, and `index` (a
domain `README.md`). A document that teaches, gives steps, lists facts, and explains reasons at once is split
into one document per type, linked together.

- **R-WRITE-3** Record a decision in `knowledge/<domain>/decisions/NNNN-title.md` when several reasonable
  options existed, the choice affects quality attributes or future maintenance, and someone will later ask
  "why not the other option?". Record the rejected options and the downsides accepted, not just the choice.
  Routine choices visible in the code do not need a record. *(ADR, MADR)*

### Metadata and names

- **R-WRITE-5** Every document starts with frontmatter. The first four fields are required:

```yaml
---
title: ""          # states the topic (and the fix, for troubleshooting)
domain: ""         # a domain listed in knowledge/taxonomy.md
doc_type: ""       # tutorial | how-to | reference | explanation | decision | glossary | taxonomy | index
status: "draft"    # see R-UPD-3
aliases: []        # optional: synonyms, abbreviations, old names
related: []        # optional: relative paths of the main related documents
references: []     # optional: external sources
---
```

- **R-WRITE-6** File names are lowercase kebab-case after the topic (`connection-pool-limits.md`); domain
  entries are `README.md`; decisions are `NNNN-title-with-dashes.md`, numbered per domain.
- **R-WRITE-10** Define each term once, in `knowledge/glossary.md`, with `aliases` for synonyms and old names;
  use the same term and capitalisation everywhere. Add a domain to `knowledge/taxonomy.md` before creating its
  folder.
- **R-WRITE-12** Link with relative paths and descriptive link text, and list the main links in `related`.
  When linking to another domain, say why the reader needs it there.

### Style

- **R-WRITE-7** Open with a paragraph that says what the document covers, what it does not, and where it
  applies. Be specific and verifiable — no "should probably" or "soon". Put commands, paths, file names,
  parameters, and values in code formatting. State preconditions, limits, defaults, and known issues. Keep a
  neutral, non-promotional tone; break a style rule only with a reason, and consistently. Match the tone to
  the type: tutorials encourage, how-to guides are direct, references are neutral, explanations discuss.
  *(Google developer documentation style)*
- **R-WRITE-8** When a conclusion depends on versions, operating systems, or external services, add an
  `## Applicability` section with the verified range and the limits it must not be extended beyond.
  *(Google developer documentation style)*
- **R-WRITE-9** Start a new document from the template for its type below. Fuller templates with writing
  guides: [The Good Docs Project](https://gitlab.com/tgdp/templates). *(The Good Docs Project)*

How-to:

```markdown
# How to <complete a task>
## Goal
<what the reader achieves; preconditions and scope>
## Before you begin
<access, tools, versions>
## Steps
1. <action; branch with "if …, then …" where needed>
## Verify
<how to confirm success>
## Related
```

Troubleshooting (a how-to variant):

```markdown
# <exact error or symptom>
## Symptom
<the exact error text and where it appears>
## Cause
## Fix
1. <action>
## Verify
## Related
```

Reference:

```markdown
# <component, command, or setting> reference
## Overview
<what is described and where it applies>
## Reference
<parameters, options, or fields — a table where possible>
## Constraints and defaults
## Examples
```

Explanation:

```markdown
# About <topic>
## Overview
## Background and context
## Why it works this way
## Alternatives and trade-offs
## Related
```

Decision:

```markdown
# <decision title: the problem and the chosen option>
- Status: proposed | accepted | deprecated | superseded
- Date: YYYY-MM-DD
## Context and problem statement
## Decision drivers
## Considered options
## Decision outcome
Chosen option: "<option>", because <reason>.
### Consequences
- Good, because <…>
- Bad, because <…>
## Confirmation
<how to check the implementation follows the decision (optional)>
```

Tutorial:

```markdown
# <build something> tutorial
## What we will build
## Prerequisites
## Steps
1. <action and the result to expect>
## What we did
<recap; links to how-to and reference documents>
```

## 4. Updating

- **R-UPD-1** When several documents state the same thing, keep it in the document that owns the topic and
  replace the others with a link. A fact is maintained in one place only. *(KCS, Zettelkasten)*
- **R-UPD-2** Reuse is review. When using a document shows it is wrong, outdated, unclear, or incomplete, fix
  it on the spot; if you cannot, flag it (status and a note at the top) instead of opening a new document.
  *(KCS)*
- **R-UPD-3** Every document has a lifecycle status: *(ISO 30401)*

| Status | Meaning |
| --- | --- |
| `draft` | Not yet verified; usable, but readers must know it is provisional |
| `verified` | Confirmed by running, testing, or checking against the source |
| `accepted` | Adopted as the stable reference (common for decisions and stable references) |
| `deprecated` | Outdated or no longer recommended; kept for history, with the reason |
| `superseded` | Replaced by another document, which it links to |
| `archived` | No longer maintained; kept only as a record |

- **R-UPD-4** Outdated content is updated now, or marked `deprecated` with the reason and a link to the closest
  current document. Replaced content is marked `superseded` and links to its replacement. Known-outdated
  content is never left unmarked. *(ISO 30401)*
- **R-UPD-5** When a decision changes, add a new decision record and mark the old one `superseded` with a link
  to the new one. Do not rewrite the old record; its reasoning is part of the history. *(ADR)*
- **R-UPD-6** Review is triggered by events, not by a calendar: a source changes, a reader finds a problem,
  or a task reports an error. A review checks that the content is still correct and that its scope still
  holds, not just the wording. High-change content (versions, parameters, external dependencies) is checked
  every time it is reused. *(ISO 30401)*
- **R-UPD-8** The knowledge base is maintained collectively; there is no per-file owner. Whoever finds a
  problem fixes or flags it without waiting for an owner. *(ISO 30401, KCS)*
- **R-UPD-9** Creation dates, update dates, and authors come from version control. Do not maintain them in
  frontmatter: hand-written dates drift and then mislead.

When substantially updating a document: adjust `status` (for example `draft` → `verified`), check that its
`related` links still hold, and mark any document it replaces.

## Quality checklist

- **R-UPD-10** Before committing a knowledge change, check:

- [ ] Searched for existing documents in the domain, with synonyms and aliases (R-PRE-1)
- [ ] A new document is really needed rather than an update (R-UPD-1)
- [ ] `doc_type`, `domain`, and the required frontmatter fields are correct (R-WRITE-2, R-WRITE-5)
- [ ] The first paragraph states purpose and scope (R-WRITE-7)
- [ ] Links are relative, resolve, and the domain index links the document (R-WRITE-4, R-WRITE-12)
- [ ] No fact is duplicated from another document (R-WRITE-1, R-UPD-1)
- [ ] Sources are named in `references` or the text
- [ ] Outdated content is updated or marked (R-UPD-4)
- [ ] No credentials, tokens, or private keys (R-PRE-3)

## Examples

- **Reuse before create.** A task needs "how to rotate the database password". Searching `knowledge/database/`
  and the aliases `credential rotation` and `password change` finds an existing how-to → extend it with the new
  case instead of writing a second guide.
- **Choosing the type.** "We chose logical replication over a dump-and-restore for the migration because it
  keeps downtime under a minute, at the cost of a more complex cut-over" → a `decision` in
  `knowledge/database/decisions/`. The replication settings are a `reference`; why replication lag happens is
  an `explanation`.
- **Not mixing types.** A reference starts explaining the design and listing steps → move the reasoning to an
  explanation and the steps to a how-to, and link both from the reference.
- **Outdated content.** A reference lists an API version the vendor has retired, and there is no time to
  update it → mark it `deprecated`, note the reason at the top, and link the vendor's current page.
- **Secrets.** A how-to contains a real API key → replace it with `<PAYMENT_API_KEY>` and say where the
  operator gets the value.
- **Only what is specific.** A deployment guide needs to start a container → link the general container
  how-to and keep only what this service needs in addition, such as its environment variables.

## Sources

The rules above adapt ideas from these sources; read them for the reasoning behind a rule.

- Diátaxis — <https://diataxis.fr/>
- Knowledge-Centered Service (KCS) — <https://library.serviceinnovation.org/KCS/Principles_and_Core_Concepts>
- Architectural Decision Records and MADR — <https://adr.github.io/>
- Zettelkasten — <https://zettelkasten.de/overview/>
- ISO 30401 knowledge management systems — <https://www.iso.org/standard/68683.html>
- Google developer documentation style guide — <https://developers.google.com/style>
- The Good Docs Project templates — <https://gitlab.com/tgdp/templates>
- Progressive disclosure (Agent Skills) — <https://agentskills.io/specification#progressive-disclosure>

## Excluded rules

These rules were left out to stay within the Full budget or because the situation does not need them yet.

| Rule | Why it was left out | Add it back when |
| --- | --- | --- |
| R-UPD-7 Periodic review | The team reviews when problems surface, and scheduled reviews were abandoned before | Agents start answering from stale drafts or duplicates, or the corpus doubles in size |
