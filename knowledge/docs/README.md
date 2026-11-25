# Knowledge management source archive

This directory keeps the external sources behind the knowledge-management methodology used by the
[`create-knowledge-rules`](../../.agents/skills/create-knowledge-rules/SKILL.md) skill: verbatim upstream
snapshots and study notes that summarise them.

Most notes are written in Traditional Chinese and are unofficial summaries, not translations endorsed by
the upstream authors. When a note and its source disagree, the source wins. Each folder names its sources
at the top of its `README.md`.

## Contents

| Folder | What it holds | Form | Upstream license |
| --- | --- | --- | --- |
| [`AgentSkills/`](AgentSkills/README.md) | The Agent Skills open standard (agentskills.io): format, authoring, evaluation, client behaviour | Notes (zh-TW) + verbatim English snapshot in `source/` | CC BY 4.0 |
| [`ADR/`](ADR/README.md) | Architectural Decision Records: concepts, templates, practices, tooling | Notes (zh-TW) | MIT (adr.github.io), MIT or CC0-1.0 (MADR) |
| [`Diataxis/`](Diataxis/README.md) | Diátaxis: the four documentation types and how to apply them | Notes (zh-TW) | CC BY-SA 4.0 — see [`Diataxis/LICENSE.md`](Diataxis/LICENSE.md) |
| [`GoogleDeveloperStyle/`](GoogleDeveloperStyle/README.md) | Google developer documentation style guide: reference hierarchy and editorial resources | Notes (zh-TW) | CC BY 4.0 |
| [`ISO30401/`](ISO30401/README.md) | ISO 30401:2018 knowledge management systems | Notes (zh-TW) written from public summaries; no text of the standard | Not applicable (the standard itself is not included) |
| [`TheGoodDocsProject/`](TheGoodDocsProject/README.md) | The Good Docs Project core documentation template pack | Verbatim templates | MIT-0 — see [`TheGoodDocsProject/LICENSE`](TheGoodDocsProject/LICENSE) |
| [`Zettelkasten/`](Zettelkasten/README.md) | The Zettelkasten method: atomic notes, links, archive structure | Notes (zh-TW) | CC BY-SA 4.0 — see [`Zettelkasten/LICENSE.md`](Zettelkasten/LICENSE.md) |

Attribution, licenses, and the changes made to each source are listed in
[`THIRD_PARTY_NOTICES.md`](../../THIRD_PARTY_NOTICES.md).

## Not included

Knowledge-Centered Service (KCS) is part of the methodology, but its
[Principles and Core Concepts](https://library.serviceinnovation.org/KCS/Principles_and_Core_Concepts) are
licensed CC BY-NC-ND 4.0, which does not allow sharing summaries or translations. Read the original pages
instead. The skill describes the capture-and-reuse ideas it takes from KCS in its own words and links back
to the source.

## Using this archive

- Treat it as background reading, not as rules. Rules for your own knowledge base come from the
  `create-knowledge-rules` skill.
- Keep upstream files as they are. Update a snapshot by replacing it with a newer upstream copy and
  recording the source commit or date in the folder `README.md`.
- Exclude this folder from routine knowledge searches; open it only when you need the original reasoning.
