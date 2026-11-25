<!--
Example always-on section produced together with knowledge-governance.example.md (Full tier). It is inserted
into the root AGENTS.md of the target repository; the marker lines let later runs replace it in place.
-->
<!-- knowledge-rules:start -->
## Knowledge base

Long-lived knowledge lives in `knowledge/`, in English. Read [the knowledge rules](docs/knowledge-governance.md) before creating, moving, or restructuring knowledge.

Reading:
- Consult `knowledge/` first for tasks about the payment platform, its database, or deployment; skip it for unrelated work.
- When the task or a playbook names a knowledge file, start from it. Otherwise search file names, frontmatter, headings, aliases, and exact error strings with `knowledge/sources/` excluded.
- Read the frontmatter, first paragraph, and headings of two to five candidates, then only the matching section.
- Prefer `verified` and `accepted` documents. Say so when you rely on `draft`, `deprecated`, or `superseded` content, and point out contradictions instead of picking one.
- Name the knowledge files your answer relies on.
- When knowledge is missing or wrong, propose a new or corrected entry at the end of your answer.

Before writing:
- Search the domain, synonyms, and aliases for an existing document, and extend it instead of adding a new one.
- Write only what later work can reuse, with enough context to stand alone; mark unverified content `draft`.
- Never write credentials, tokens, or private keys; use placeholders such as `<API_TOKEN>`.
<!-- knowledge-rules:end -->
