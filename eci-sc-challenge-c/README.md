# Summer Camp ECI cluster challenge

This repo is meant to be used as a baseline for your summer challenge.
The idea is to implement a `knwoledge-base` for your Agents to build and use to reduce token usage and streamline knowledge accesability regarding your project.

The knowledge base has to follow these aspects:

- an "artifacts" directory to where the LLM will ingest information from
- a "wiki" directory to where the LLM will write all the ingested information
- a "schema" definition (`CLAUDE.md` and `.claude/`) where you define the rules for the LLM to follow
- 3 specific `commands` for the `LLM <---> Wiki` interaction
  - `ingest`, for ingesting informaiton into the wiki
  - `lint`, for linting/checking the wiki to check if nothing is incorrectly formatted
  - `query`, to ask a question and get an answer baed on the existing wiki

> These names are only placeholders, use the ones you think best accomodate your needs.
>
> (With the exception of CLAUDE.md and .claude/, you are stuck with these ones. ¯\\\_(ツ)_/¯)
>
> The existing `CLAUDE.md` and `.claude/commands/*` are empty, these are meant to be filled by you.

## The challenge, in two phases

The challenge is split into two phases, and every variant below shares the same goal:

1. **Implement the llm-wiki.** Stand up the structure above (artifacts → wiki → schema) and the three core operations from Karpathy's pattern: **ingest**, **query**, and **lint**.
2. **Ingest the corpus.** Feed the wiki with **multiple PDF documents from the electrical-energy domain** (standards, regulations, technical datasheets, grid/market reports, whitepapers, etc.). These PDFs are the `artifacts` — the immutable raw source. The LLM converts them into interlinked wiki pages so that answers *compound* over the domain instead of being re-retrieved from raw PDFs on every query.

> The source material is **energy-related PDFs, not a codebase.** This matters for the variants below: the knowledge being captured is domain knowledge extracted from documents, not knowledge derived from source code. A PDF-to-Markdown ingestion step (see references) is a prerequisite for every variant.

## Variants

Depending on the office you are at, some aspects of the implementation will diverge.
This is meant to introduce some friendly banter/rivalry between offices.

The theme variant distribution is the following:

### Porto -> Challenge A

    Markdown + ADRs (in-repo) Capture the ingested energy knowledge as plain
    Markdown wiki pages committed to the repo — one topic/entity page per concept
    distilled from the source PDFs — plus a series of Architecture Decision Records
    in /docs/adr/ that document the decisions about the knowledge base itself
    (ingestion pipeline, page schema, naming/linking conventions, chosen PDF-to-
    Markdown tool). The team evaluates whether the simplest, most portable,
    version-controlled approach — no tools, no external service — is "good enough"
    as an ingestion target for the energy PDFs and for both humans and AI agents.

### Viseu -> Challenge B

    Obsidian Second Brain Build the same energy knowledge in an Obsidian vault — a
    linked, graph-based team second brain where each concept extracted from the
    source PDFs becomes a note, wired together with backlinks and Maps of Content
    (e.g. a MoC per PDF, per standard, or per topic such as "grid balancing" or
    "power quality"), and an MCP plugin that exposes the vault to an AI agent. The
    team evaluates whether rich linking and graph structure over the ingested energy
    corpus produce better human navigation and AI retrieval than flat Markdown.

### Coimbra -> Challenge C

    QMD (Quarto, Markdown-mode) Author the knowledge distilled from the energy PDFs
    as Quarto .qmd files (Markdown-mode only — no executable code) and render them
    into a navigable HTML documentation site. The team evaluates the split between a
    polished published artefact for humans and the source quality for AI, and probes
    Quarto's built-in llms.txt / .llms.md generation as the AI-consumption index over
    the ingested energy corpus.

### Lisbon -> Challenge D (equipa maior)

    Merged auto-generated + AI-First KB Combine two complementary layers at one team.
    First, an auto-generated layer derived directly from the source PDFs — the
    structure a DeepWiki-style tool produces automatically: extracted sections,
    tables, figures, entities and a glossary of energy terms, one page per document
    with links back to the origin PDF. Second, a hand-authored AI-context pack for
    the knowledge that is *not* stated in any single PDF — cross-document synthesis,
    domain rationale, regulatory/business rules, a unified glossary, and the tribal
    knowledge that ties the corpus together. The team evaluates a realistic hybrid
    that mirrors how real teams assemble domain knowledge from many documents: a
    machine-generated backbone enriched by human-curated context.

    (Note: DeepWiki itself targets code repositories, so it isn't a drop-in here —
    the source is PDFs, not code. Use it as the reference for the "auto-generated
    backbone" idea and reproduce that layer over the energy corpus with a PDF-to-
    Markdown pipeline; see references.)

## References

### The core pattern

- [LLM-Wiki — Andrej Karpathy (original gist)](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Karpathy's LLM Wiki: a knowledge base that compounds (walkthrough)](https://www.aibuilderclub.com/blog/karpathy-llm-wiki)
- [How to build Karpathy's LLM Wiki — complete guide](https://blog.starmorph.com/blog/karpathy-llm-wiki-knowledge-base-guide)
- [LLM Wiki tutorial — Data Science Dojo](https://datasciencedojo.com/blog/llm-wiki-tutorial/)
- [LLM Wiki v2 — extending the pattern](https://gist.github.com/rohitg00/2067ab416f7bbe447c1977edaaa681e2)

### Phase 2 — PDF → Markdown ingestion (used by every variant)

- [Docling (IBM Research / LF AI & Data) — advanced PDF understanding: layout, reading order, tables, formulas](https://github.com/docling-project/docling)
- [Docling MCP — expose Docling to an AI agent over MCP](https://github.com/docling-project/docling-mcp)
- [MarkItDown (Microsoft) — lightweight, fast document-to-Markdown for LLMs](https://github.com/microsoft/markitdown)
- [MarkItDown — Real Python guide](https://realpython.com/python-markitdown/)
- [Marker (Datalab) — high-fidelity, deep-learning PDF-to-Markdown](https://github.com/datalab-to/marker)
- [MinerU (OpenDataLab) — PDF extraction to Markdown/JSON](https://github.com/opendatalab/MinerU)
- [Best open-source PDF-to-Markdown tools compared (Marker vs MinerU vs MarkItDown)](https://jimmysong.io/blog/pdf-to-markdown-open-source-deep-dive/)
- [MarkItDown vs Docling vs Marker — trade-offs](https://www.danilchenko.dev/posts/markitdown-vs-docling-vs-marker/)

### Challenge A (Coimbra) — Markdown + ADRs

- [CommonMark — Markdown specification](https://commonmark.org/)
- [Documenting Architecture Decisions — Michael Nygard (original ADR essay)](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [adr.github.io — Architectural Decision Records homepage & templates](https://adr.github.io/)
- [adr-tools — Nat Pryce (command-line ADR tooling)](https://github.com/npryce/adr-tools)
- [architecture-decision-record — templates & examples (Joel Parker Henderson)](https://github.com/joelparkerhenderson/architecture-decision-record)

### Challenge B (Viseu) — Obsidian Second Brain

- [Obsidian](https://obsidian.md/)
- [Obsidian MCP plugin — vault access for AI agents over MCP](https://github.com/aaronsb/obsidian-mcp-plugin)
- [Semantic Notes Vault MCP — community plugin](https://community.obsidian.md/plugins/semantic-vault-mcp)
- [Obsidian MCP server guide (2026)](https://www.morphllm.com/obsidian-mcp-server)
- [Maps of Content & Linking-Your-Thinking (Nick Milo)](https://www.linkingyourthinking.com/)

### Challenge C (Porto) — Quarto (QMD)

- [Quarto](https://quarto.org/)
- [Quarto — Output for LLMs (llms.txt / .llms.md generation)](https://quarto.org/docs/websites/website-llms.html)
- [llms.txt specification — llmstxt.org (Jeremy Howard, Answer.AI)](https://llmstxt.org/)
- [llms.txt explained — Towards Data Science](https://towardsdatascience.com/llms-txt-414d5121bcb3/)

### Challenge D (Lisbon) — Auto-generated + AI-First KB

- [DeepWiki — deepwiki.com (Cognition AI)](https://deepwiki.com/)
- [CognitionAI/deepwiki — Devin-generated docs for any public repo](https://github.com/CognitionAI/deepwiki)
- [DeepWiki — Devin documentation](https://docs.devin.ai/work-with-devin/deepwiki)
- [llms.txt / llms-full.txt spec (basis for the AI-context pack)](https://llmstxt.org/)
- [AGENTS.md — conventions for AI-context files](https://agents.md/)
