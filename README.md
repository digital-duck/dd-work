# Wen Gong's Work

[Google Scholar](https://scholar.google.com/citations?view_op=list_works&hl=en&user=P-E_YsMAAAAJ)

## Tools

| Name|  Source | CLI | PyPI-pkg | Notes |
|---|---|---|---|---|
| ZiNet |  [zinet](https://github.com/digital-duck/zinet) | `zinet` | zinet | The shared graph core: the `graph.yaml` format with its schema, validator and transforms, the graph algorithms (build order, cycles, reducibility, learning paths) and the drawing engines (cytoscape, graphviz, mermaid, ascii). GraphLang, Work-Graph and App-Graph all build on it. |
| Graph-Lang | [graflang](https://github.com/digital-duck/graflang) | `graf` | graflang | Evolved from [SPL.py](https://github.com/digital-duck/SPL.py): a language for LLM workflows written as graphs ("recipes") rather than SPL scripts. Model calls, programs, loops, Cedar-guarded risky steps and reusable parts are nodes. A recipe can be validated, drawn, capped, checkpointed and replayed. Also has a visual editor (`graf vis --edit`) and a Python API (`graphlang.recipes`). |
| App-Graph | [app-graph](https://github.com/digital-duck/app-graph) | `appgraph` | app-graph | An app builder: describe an app as a blueprint graph and it generates tested code with an LLM. It runs on GraphLang recipes, with the tests in Docker. It also works in reverse (`appgraph extract`, an existing repo to a blueprint) and measures drift between two blueprints. Successor to graph-code. |
| Work-Graph|  [work-graph](https://github.com/digital-duck/work-graph) | `workgraph` | workgraph | A knowledge layer beside Claude Code: one searchable graph of your projects, their lifecycle (requirements → design → tests), products, teams and processes. Includes gap reports, `workgraph ask` with checked citations, a Streamlit UI and private git sync. |
| Data-Graph | [data-graph](https://github.com/digital-duck/data-graph) | `datagraph` | data-graph | Lineage, checks and profiling for a semantic layer written in [Apache Ossie](https://ossie.apache.org/) (datasets, fields, metrics, joins). It reads both spec versions into a lineage graph (source columns → fields → metrics), answers `lineage` and `impact`, checks references, joins and gaps, and exports to ZiNet. It also drafts a model from a SQLite catalogue and profiles the data, read-only through dd-db. Third in a line after data-copilot (vanna.ai, 2023) and anadata (dd-* stack, 2026); next: GraphLang recipes to draft models and for text-to-SQL checked against the model, benchmarked on Chinook. |
|Concept-Book|[concept-book](https://github.com/digital-duck/concept-book)||-| Generates a concept-book from a concept graph: an LLM writes, and a verifier checks, an explanation for each concept along its learning path. Path A goes from an authored graph to a book; Path B ingests an existing textbook into a graph and generates a companion book. Comes with a web portal (Vite frontend, FastAPI backend) and a series of `cb-*` books (OpenStax and others). It still runs on SPL.py `.spl` workflows (`build_concept_book`, `answer_on_demand`, `compare_concepts`), so a GraphLang refactor is planned. |
| CVE-Coder | [cve-coder](https://github.com/digital-duck/cve-coder) | `cvecoder` | cve-coder | An agentic CVE pipeline. `hunt` finds new vulnerabilities (a semgrep scan, LLM triage, validation gates, a disclosure draft), and `pipeline` remediates known CVEs end to end (NVD spec, triage, detect, patch on an isolated clone, regression test on both checkouts). Also has a Snyk scan, a Streamlit workbench and a SQLite job queue. Each stage writes a `.yaml` artifact, which makes a run auditable and replayable; it also runs as a vulnerability-remediation playbook on Devin. Next: refactor onto GraphLang recipes and add a vulnerability knowledge graph over NVD with lineage across CVEs. |

## Papers 

| # | arXiv ID | Category | Title | Comment | Repo |
|---|----------|----------|-------|-------|-------|
| 12 | submit/8350460 | cs.CL | [ConceptBook: A Graph-First Framework for AI-Generated Curricula](./docs/spl4ed-paper-arxiv.pdf) | under [TMLR](./docs/spl4ed-tmlr-v2.1.pdf) review  | https://github.com/digital-duck/concept-book |
| 11 | submit/8214143 | cs.CL | [Data-Driven Typesetting of Chinese Character Decompositions with zitex](./docs/zitex-arxiv.pdf) | | https://github.com/digital-duck/zinets-latex |
| 10 | [2607.07727](https://arxiv.org/abs/2607.07727) | cs.CL, cs.PL | [SPL: Orchestrating Workflows with Declarative Deterministic–Probabilistic Composition](./docs/spl-dual-mode.pdf) |  | https://github.com/digital-duck/SPL.py |
| 9 | submit/7556205 | cs.CL | [Beyond Vibe Coding: Intent Invariance and Structured Prompt Language](./docs/beyond-vibe-coding-arxiv-submitted.pdf) |  | https://github.com/digital-duck/SPL.py |
| 8 | submit/7420152 | cs.CL, cs.PL | [SPL 2.0: A Declarative Language for Agentic Workflow Orchestration](./docs/SPL20-arxiv.pdf) |   | https://github.com/digital-duck/SPL20 |
| 7 | submit/7373026 | cs.CL, cs.PL, cs.DC | [Momagrid: A Decentralized Inference Runtime with Linear Complexity via Semantic Chunking](./docs/Momagrid-arxiv.pdf) |   | https://github.com/digital-duck/momagrid | 
| 6 | [2602.21257](https://arxiv.org/abs/2602.21257) | cs.CL, cs.PL, cs.DB | [Structured Prompt Language (SPL): Declarative Context Management for LLMs](./docs/SPL-2602.21257v1.pdf) | | https://github.com/digital-duck/SPL |
| 5 | [2602.15860](https://arxiv.org/abs/2602.15860) | cs.CL | [Reranker Optimization via Geodesic Distances on k-NN Manifolds](./docs/Reranker_Optimization-tmlr.pdf) | published at [TMLR 2026-05](https://openreview.net/pdf?id=HvzgEt51f2) |  https://github.com/digital-duck/maniscope |
| 4 | [2601.09732](https://arxiv.org/abs/2601.09732) | cs.CL | [Benchmarking Cross-Lingual Semantic Alignment in Multilingual Embeddings](./docs/SA-ICML-2601.09732v1.pdf) |  | https://github.com/digital-duck/semanscope | 
| 3 | [2601.09731](https://arxiv.org/abs/2601.09731) | cs.CL | [Geometric Patterns of Meaning: A PHATE Manifold Analysis of Multi-lingual Embeddings](./docs/PHATE-2601.09731v1.pdf) | | |
| 2 | [2510.01230](https://arxiv.org/abs/2510.01230) | cs.CL | [Geometric Structures and Patterns of Meaning: A PHATE Manifold Analysis of Chinese Character Embeddings](./docs/PHATE-2510.01230v1.pdf) | | |
| 1 | [2502.19428](https://arxiv.org/abs/2502.19428) | physics.soc-ph | [A New Exploration into Chinese Characters: from Simplification to Deeper Understanding](./docs/Elemental-Zi-2502.19428v1.pdf) | | |


## Blogs

| Title |    URL | Comment  | Repo |
|-------|--------|----------|--------|
| SPL: Orchestrating Workflows with Declarative Deterministic-Probabilistic Composition | https://gist.science/paper/2607.07727 | explanation by gist.science | |
| Compression is Understanding | https://medium.com/@wen.g.gong/compression-is-understanding-409b9a7d6779 | | |
| NeuroSymbolic Math in a Structured Prompt Language | https://medium.com/@wen.g.gong/neurosymbolic-math-in-a-structured-prompt-language-6b920d44d11b | | |
| AI@Home with Gemma | https://medium.com/@wen.g.gong/ai-home-with-gemma-369fa8c27e2b?postPublishedType=repub | | |
| The Momahub Moment | https://medium.com/@wen.g.gong/the-momahub-moment-df852c42f3da | | https://github.com/digital-duck/momahub.py |


## Books

- [A Cookbook of SPL Recipes](docs/SPL-Cookbook-DRAFT-v0.5.pdf)


## Work in progress


| # | Title | Comment | Repo  |
|---|-------|---------|-------|

