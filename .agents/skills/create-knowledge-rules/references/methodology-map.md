# Methodology map

Read this when the user's situation is not covered by the signals in [`rule-catalog.md`](rule-catalog.md), or
when you need to explain why a rule is there. Each methodology is summarised in our own words with the parts
this skill uses; follow the link for the original. Do not name these methodologies to the user during the
interview — use them to choose rules, and cite them in the generated document for readers who want depth.

## Situation → elements

| The user's situation | Use | Rules |
| --- | --- | --- |
| Agents and people follow steps, fix incidents, look up facts | Diátaxis document types; The Good Docs Project templates for how-to and troubleshooting | R-WRITE-2, R-WRITE-9, R-DIST-5 |
| The same problems keep coming back | KCS capture and reuse | R-DIST-1, R-DIST-2, R-UPD-2, R-READ-9 |
| People ask "why did we do it this way?" | Architectural decision records | R-WRITE-3, R-UPD-5 |
| Knowledge arrives as many small pieces | Zettelkasten atomic notes, links, and index notes | R-DIST-7, R-WRITE-4, R-WRITE-12 |
| The team wants regular clean-ups, or many people maintain it | ISO 30401 lifecycle and continual improvement | R-UPD-3, R-UPD-6, R-UPD-7, R-UPD-8 |
| People read the documents, not only agents | Google developer documentation style | R-WRITE-7, R-WRITE-8 |
| Summaries of vendor docs or standards | Source archive kept apart from own knowledge | R-DIST-6, R-READ-10 |

## The methodologies

### Diátaxis — <https://diataxis.fr/>

Documentation serves four different needs, and a document that tries to serve two serves neither well.
Two questions sort content: does it help the reader *do* something or *know* something, and is the reader
*learning* or *working*? Doing + learning is a tutorial, doing + working a how-to guide, knowing + working a
reference, knowing + learning an explanation. The skill uses the four types as `doc_type` values and the two
questions as the test for which one a document is (the "compass").

### Knowledge-Centered Service (KCS) — <https://library.serviceinnovation.org/KCS/Principles_and_Core_Concepts>

From support organisations: knowledge is a by-product of solving problems, so it is captured while the
problem is being solved, in the words of the person who hit it. An entry only needs to be good enough to help
the next person; it improves each time it is reused, because every reuse is also a review. The whole team owns
the knowledge base, and demand — what people actually look for — decides what gets improved. The skill uses
these ideas in its capture, reuse, and shared-ownership rules. The KCS texts are licensed CC BY-NC-ND 4.0; do
not copy or translate them into the outputs.

### Architectural Decision Records (ADR) — <https://adr.github.io/>

A decision record captures one significant decision: the context and problem, the options considered, the
option chosen and why, and the consequences — including the downsides accepted. Records are not rewritten
when a decision changes; a new record supersedes the old one, so the history of reasoning survives. MADR
(<https://adr.github.io/madr/>) is a lean Markdown template for such records.

### Zettelkasten — <https://zettelkasten.de/overview/>

A note archive built for thinking and writing over years. Each note holds one idea, stated in your own words,
so it can be linked precisely. Links carry meaning ("supports", "contradicts", "extends"), and structure notes
act as entry points into a topic. Collecting without processing ("collector's fallacy") creates a pile, not
knowledge. The skill uses atomicity, meaningful links, and index documents.

### ISO 30401 knowledge management systems — <https://www.iso.org/standard/68683.html>

A management-system standard for organisational knowledge. The parts used here are general: knowledge has a
lifecycle (created, validated, used, retired), the organisation reviews whether its knowledge is still
correct and useful, and the system improves continually through plan–do–check–act. The skill turns this into
status values, event-driven and optional periodic review, and handling of outdated content. The standard's
text is copyrighted; use the official standard for any conformity work.

### Google developer documentation style guide — <https://developers.google.com/style>

Editorial guidance for technical writing: project conventions first, then the guide, then general references.
The parts used here: say what a document covers up front, be specific and verifiable, put code-like text in
code formatting, state limits and defaults, keep a neutral tone, and break a rule only with a reason and
consistently.

### The Good Docs Project — <https://gitlab.com/tgdp/templates>

Fill-in templates with writing guides for common document types (concept, how-to, reference, tutorial,
troubleshooting, README, release notes). The skill points new documents at a template for their type.

### Progressive disclosure — <https://agentskills.io/specification#progressive-disclosure>

From the Agent Skills standard: load a short description first and the full content only when it is needed.
The reading rules apply the same idea to knowledge — search and skim headings before reading whole files —
and the two-layer output keeps only a short section always loaded.
