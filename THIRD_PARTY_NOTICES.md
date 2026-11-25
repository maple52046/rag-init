# Third-party notices

The repository's own content — skills, rule templates, the default rules, the knowledge-base skeleton,
examples, and documentation — is licensed under the MIT license in [`LICENSE`](LICENSE). The folders below contain or adapt third-party material and keep the
upstream license. Where a folder's license differs from MIT, that license applies to the folder instead.

| Path | Upstream | License | Form and changes |
| --- | --- | --- | --- |
| `knowledge/sources/AgentSkills/source/` | [Agent Skills](https://agentskills.io/) ([agentskills/agentskills](https://github.com/agentskills/agentskills) @ `83021e2`) | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), full text in [`source/LICENSE`](knowledge/sources/AgentSkills/source/LICENSE) | Verbatim snapshot of the documentation site (`docs/`), including the `clients.jsx` data file |
| `knowledge/sources/AgentSkills/*.md` | Same as above | CC BY 4.0 | Summaries and Traditional Chinese translation; changed from the original |
| `knowledge/sources/ADR/` | [adr.github.io](https://adr.github.io/) and [MADR](https://adr.github.io/madr/) | adr.github.io: MIT (notice below); MADR: MIT or CC0-1.0 | Summaries and Traditional Chinese translation |
| `knowledge/sources/Diataxis/` | [Diátaxis](https://diataxis.fr/), © Daniele Procida | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/); the adaptation is CC BY-SA 4.0 too, see [`LICENSE.md`](knowledge/sources/Diataxis/LICENSE.md) | Summaries and Traditional Chinese translation; examples added; sections specific to the author's private repository removed |
| `knowledge/sources/GoogleDeveloperStyle/` | [Google developer documentation style guide](https://developers.google.com/style) | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) | Summaries and Traditional Chinese translation; sections specific to the author's private repository removed |
| `knowledge/sources/ISO30401/` | [ISO 30401:2018](https://www.iso.org/standard/68683.html) public pages and the BSI summary | No upstream text included | Original notes written from public summaries; they contain no text of the standard. ISO 30401 is © ISO; get the official standard for conformity work |
| `knowledge/sources/TheGoodDocsProject/` | [The Good Docs Project templates](https://gitlab.com/tgdp/templates) @ `41645d9` | [MIT-0](knowledge/sources/TheGoodDocsProject/LICENSE) | Verbatim Core documentation template pack; folder `README.md` is an added index |
| `knowledge/sources/Zettelkasten/` | [Zettelkasten.de overview](https://zettelkasten.de/overview/), Christian Tietze and Sascha Fast | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/); the adaptation is CC BY-SA 4.0 too, see [`LICENSE.md`](knowledge/sources/Zettelkasten/LICENSE.md) | Summaries and Traditional Chinese translation, split into topic notes |
| `.agents/skills/git-commit/references/conventional-commits-1.0.0.md` | [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) | [CC BY 3.0](https://creativecommons.org/licenses/by/3.0/) | Summary, Examples, and Specification sections reproduced verbatim from upstream commit `42b627e`; the rest of the page is omitted |

Not included: the [KCS Principles and Core Concepts](https://library.serviceinnovation.org/KCS/Principles_and_Core_Concepts)
by the Consortium for Service Innovation are licensed CC BY-NC-ND 4.0, which does not allow sharing
adaptations. The `create-knowledge-rules` skill describes the ideas it takes from KCS in its own words and
links to the original.

The methodology references in `.agents/skills/create-knowledge-rules/references/` are original summaries
written for this repository; they link to each source instead of reproducing it.

## adr.github.io — MIT License

```text
MIT License

Copyright (c) 2017-2023 adr.github.io

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
