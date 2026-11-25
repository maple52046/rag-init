# Agent instructions

This repository is a knowledge base that agents and people read and write.

<!-- knowledge-rules:start -->
## Knowledge base

Long-lived knowledge lives in `knowledge/`, in English. Read [the knowledge rules](docs/knowledge-governance.md) before creating, moving, or restructuring knowledge.

Reading:
- Consult `knowledge/` first for tasks in an area it covers; skip it for unrelated work.
- When the task or a skill names a knowledge file, start from it. Otherwise search file names, frontmatter, headings, aliases, and exact error strings, leaving `knowledge/sources/` out of routine searches.
- Read the frontmatter, first paragraph, and headings of a few candidates, then only the matching section.
- Prefer `verified` and `accepted` documents. Say so when you rely on `draft`, `deprecated`, or `superseded` content, and point out contradictions instead of picking one.
- Name the knowledge files an answer relies on.
- When knowledge is missing or wrong, propose a new or corrected entry.

Before writing:
- Search the domain, synonyms, and aliases for an existing document, and extend it instead of adding a new one.
- Write only what later work can reuse, with enough context to stand alone; mark unverified content `draft`.
- Never write credentials, tokens, or private keys; use placeholders such as `<API_TOKEN>`.
<!-- knowledge-rules:end -->
