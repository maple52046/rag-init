# Interview guide

Ask the dimensions in this order, one per turn, and skip any that the repository or an earlier answer already
settles. Every question carries a suggested answer and a short reason; adapt the suggestion to what step 1
found. Each answer turns on **situation signals** (`S-...`), which
[`rule-catalog.md`](rule-catalog.md) uses to rank rules.

Ask in the user's language. Use the example situations; never name a methodology in a question.

## 1. What goes into the knowledge base

**Ask**: "What do you want agents (and people) to find here later? Pick any that apply."

| Option | Example situation | Signal |
| --- | --- | --- |
| Step-by-step procedures | "Restart the payment worker without losing jobs" | `S-STEPS` |
| Troubleshooting | "The build fails with `ENOSPC` — what fixed it last time?" | `S-STEPS`, `S-REPEAT` |
| Design decisions | "Why did we pick PostgreSQL over MongoDB?" | `S-DECISIONS` |
| Concept explanations | "How does our caching layer fit together?" | `S-CONCEPTS` |
| Reference facts | "Which ports, flags, and limits does the service use?" | `S-REFERENCE` |
| Summaries of external documents | "Notes on the vendor API we depend on" | `S-EXTERNAL` |

**Suggest**: what step 1 found in the existing files. With nothing to go on, suggest procedures,
troubleshooting, and reference facts — the content agents most often need in the middle of a task.

## 2. Who reads it, and how

**Ask**: "Who reads this knowledge, and what are they usually doing at that moment?"

- Readers: agents only (`S-AGENT-ONLY`), or agents and people (`S-HUMANS`).
- Use: looking up an answer, following steps (`S-STEPS`), or understanding background (`S-CONCEPTS`).
- New people need it to get started ("a new hire's first week") → `S-HUMANS`, `S-CONCEPTS`.

**Suggest**: agents and people. Reason: notes written only for agents tend to lose the context a person needs
when an agent's answer looks wrong; the extra cost is small.

## 3. Where knowledge comes from

**Ask**: "When does new knowledge usually get written down?"

| Option | Example | Signal |
| --- | --- | --- |
| Written up after the fact | "We write a summary when the project ends" | `S-AFTER` |
| Captured while working | "Whoever fixes it adds a note right away" | `S-CAPTURE` |
| Excerpted from external documents | "We summarise vendor docs and standards" | `S-EXTERNAL` |
| Small pieces that pile up | "Lots of short notes, organised later" | `S-FRAGMENTS` |

**Suggest**: captured while working. Reason: details such as rejected options and the exact error message are
gone a week later; rules that make capture cheap keep them.

## 4. Writing and maintenance habits

Ask the three parts together; they are one dimension (how much process the team accepts).

- **Who writes**: mostly agents, mostly people, or both. Several regular writers → `S-MANY-WRITERS`.
- **Review**: when something is found wrong or outdated (event-driven), or on a schedule (`S-PERIODIC`).
- **Format strictness**: light (a title and a few fields), or strict (required metadata, fixed templates,
  `S-STRICT`).

**Suggest**: both write; event-driven review; light format. Reason: scheduled reviews are the first thing teams
stop doing, and strict formats slow down capture; tighten later if the corpus grows (`S-LARGE` when it already
has more than about 200 files).

## 5. Language and location

**Ask**: "Which language should the knowledge and the rules use, and where should they live?"

**Suggest**: the language of the existing documents (or of this conversation when there are none);
knowledge in `knowledge/`; the rules document at `docs/knowledge-governance.md`; the always-on section in the
root `AGENTS.md`. Mention a source archive folder for external material (for example `knowledge/sources/`)
only when `S-EXTERNAL` is on.

## 6. Token budget

**Ask**: explain the trade-off with a situation, then offer the three tiers from `SKILL.md`. For example:
"Agents read the always-on rules on every task. Fuller rules catch more mistakes but cost tokens on every
request — like a longer checklist that everyone has to read before each job."

**Suggest**: Standard. Suggest Lean for a single person with a small corpus, and Full with `S-MANY-WRITERS`
plus `S-LARGE` or `S-STRICT`.

## 7. Reading rules

Do not ask an open question. Present this default and ask only about the adjustable points below it.

Default proposal:

1. When the task or a skill already names a knowledge file, start from that file.
2. Otherwise search file names, frontmatter, headings, and exact error strings; leave the source archive out of
   routine searches.
3. Read the frontmatter, first paragraph, and headings of the candidates; then read only the matching section.
4. Follow related links only when context is missing; look terms up in the glossary only when they are unclear.
5. Prefer verified content. Say so when using draft, outdated, or conflicting content; do not adopt it silently.

Adjustable points, each with a suggestion:

| Point | Options | Suggest |
| --- | --- | --- |
| Cite sources in answers | always / only when the answer relies on knowledge / never | only when it relies on knowledge (`S-CITE` when "always") |
| When to consult knowledge first | every task / tasks in a matching area / only when asked | tasks in a matching area — every task wastes tokens on unrelated work |
| Retrieval tool | none / a vector index or search tool exists | none unless step 1 found one; with one, use it first and then open the files it returns (`S-RAG`) |
| Gap found while reading | note it in the answer / propose a new entry / write it right away | propose a new entry — the user stays in control of what gets written |

## Closing

After the last dimension, go to step 5 of `SKILL.md` (confirm a summary) before writing anything.
