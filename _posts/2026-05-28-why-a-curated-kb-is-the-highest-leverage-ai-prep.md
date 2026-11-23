---
layout: post
title: "Why a Curated KB Is the Highest-Leverage AI Prep You'll Ever Do"
date: 2026-05-28
description: A curated, multi-source domain corpus predicts AI project success more reliably than the model, the prompt or the harness. What it contains, what makes it good, and why it compounds.
---

**Author**: Eric Shamay<br>
**Published**: 05/28/2026

Across every kind of project I've run with AI agents — coding, analysis, planning, writing, ideation — one thing predicts success more reliably than the model, the prompt, or the harness: whether a deep, curated *domain corpus* existed before the work started. Not a rules file. Not a CLAUDE.md. Not a `.cursorrules`. A *corpus*: a multi-source body of project knowledge assembled from every place the domain actually lives, and treated as a first-class engineering artifact in its own right. Teams that build the corpus first ship; teams that try to reason from raw repos and parametric memory churn. This piece argues why, what such a corpus looks like, what makes one good, and why the corpus pays back compounding returns long after the original question is answered.

## What this doc is — and is not — about

This doc is **not** about lightweight instruction files for AI tools. `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, conventions files — those are configuration. They tell an agent how to behave on a one-shot run. They are useful, but they are not the topic here.

This doc is about the **domain corpus** that sits underneath that configuration: the curated, multi-source, project-scoped body of knowledge an AI agent reads to do real work on a complex domain. A real corpus pulls from design docs, RFCs, Slack threads, Quip pages, Google Docs, code-search results, BOM dumps, vendor reference, meeting transcripts, peer-product writeups, glossaries, decision logs, and status journals. It is not a side effect of the work — it is the substrate that makes the work possible.

Anthropic frames this exact discipline as context engineering: *"finding the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome."*[^1] Glean, building enterprise AI in production, puts it more bluntly: *"managing context well is now foundational to whether agents succeed in the enterprise."*[^2] The prompt is one-shot. The corpus is the project.

## Why a deep corpus beats every other lever

The case is increasingly quantified.

Anthropic's Contextual Retrieval study reduced top-20 retrieval failure rates by **49–67%** purely by changing how the corpus was prepared — same model, same query, only the chunk-level context shape changed[^3]. Spotify's *Honk* series (Nov 2025) reports more than **1,500 merged PRs across hundreds of repos**, with **60–90% time savings** vs. hand-coding, and explicitly attributes the win to corpus-shaped inputs: *"larger static prompts, which are easier to reason about. You can version-control the prompts, write tests, and evaluate their performance."*[^4] Sourcegraph (Dec 2025) makes the codebase-as-corpus argument plain: *"the shift to agentic coding workflows has paradoxically made large-scale code search* more *valuable, not less,"* because *"custom decorators buried three directories deep, subtle overrides in sibling microservices, critical business logic scattered across modules, all of this remains invisible to the model"* without a real corpus[^5]. The collective behind *Applied LLMs* (Yan, Bischof, Frye, Husain, Liu, Shankar) puts the architectural punchline cleanly: *"RAG consistently outperforms unsupervised fine-tuning on both familiar and entirely new knowledge."*[^6] Facts get into the loop through corpora, not weights. Anthropic's own *Building Effective Agents* names the same dependency: *"agents need ground truth from the environment at each step."*[^7] A curated corpus is how you supply that ground truth before the loop starts.

The failure mode without a corpus is consistent. Models don't admit uncertainty when context is shallow — they confidently extrapolate, producing what Weaviate calls *"higher-fluency hallucinations."*[^8] Long context windows decay past the *"lost in the middle"* threshold described in Weaviate's Engram research[^9], so even very large windows don't substitute for curated structure. Spotify identifies the most damaging failure as *"a PR that passes CI but is functionally incorrect"*[^4] — the signature of an agent working without enough domain context to know what *correct* even means.

The model is the multiplier; the corpus is the signal. A bigger multiplier on shallow input still produces shallow output.

## Anatomy of a real domain corpus

Eight elements distinguish a working corpus from a notes folder.

**1. Multi-tier organization.** Separate folders for leadership-facing artifacts, working-engineer detail, and reference material. Asymmetric link flow — top tier links down; reference content does not depend on top-tier freshness. The structure echoes Tiago Forte's PARA logic of *"organize by actionability, not by subject"*[^10] and Andy Matuschak's preference for *"associative ontologies over hierarchical taxonomies."*[^11]

**2. Source diversity, with provenance.** Pull from every origin where the domain actually lives. Real domains do not live in one wiki — they live in chat archives, design docs, code repos, ticket trails, vendor pages, and meeting transcripts. Glean's enterprise pattern: *"every application has its own structure and semantics, and requires data models that capture diversity."*[^2] Track every claim back to its origin. Stack Overflow's RAG guide captures the principle: *"anchoring LLMs on precise, up-to-date, and pertinent information."*[^12]

**3. An explicit references file.** A single index of every cited source, each entry a stable slug + one-line description + URL or permalink. Sourcegraph and Glean both call out source attribution as a trust gate; a references file makes that trust mechanically auditable. Without it, citations rot into orphan strings within months.

**4. Atomic, durable building blocks.** One claim per bullet, one decision per file, one concept per glossary entry. Matuschak's evergreen-notes pattern is the right north star: *"atomic, concept-oriented, densely linked, designed to evolve."*[^11] Atomicity is what makes the corpus reusable for the *next* question — not just the one in front of you.

**5. Human-only-edit ground-truth files.** Carve out a single file the agent treats as ground truth and **must not modify**. Each fact is one bullet; each is cited. The pattern inverts the default mutability of a decision log: assertions-treated-as-axioms until the named owner edits. Michael Nygard's ADR convention names the same discipline at the decision layer: *"don't alter existing information; supersede."*[^13] SQLite's `AGENTS.md` is a public-facing analogue — an explicit human-defined boundary file telling agents what they may and may not do[^14].

**6. An open-knowledge-gaps file.** Every unresolved question gets a structured entry: *question / why it matters / owner / how to close / status*. Closed gaps stay (dated-closed, with a *Resolution* paragraph). The shape mirrors Basecamp's Hill Chart logic: *"uphill = unsolved problems remain, downhill = unknowns resolved, only execution left."*[^15] Anthropic's research-prompting guidance reinforces it: *"track confidence levels in progress notes to improve calibration; update a hypothesis tree."*[^16]

**7. An append-only status log.** Every revision dated, with trigger, resolution, aggregates recomputed, and files touched. Same shape as a lab notebook; same payoff. Holden Karnofsky on writing-as-thinking: *"the single biggest difference-maker between 'reading a ton about lots of things, but retaining little' and 'efficiently developing a set of views.'"*[^17] The status log is where the corpus's reasoning becomes legible across time.

**8. Auto-generated detail alongside human-curated synthesis.** Scripts can render per-entity detail off structured rules (YAML, CSV, BOM extracts); humans own the rules and the synthesis. Glean's *"specialized indexes for different data types"*[^2] and Spotify's *"workflow agents pre-generate prompts from internal/external sources"*[^4] are the same shape: machines do mechanical breadth, humans curate semantic depth.

## Properties that decide whether the corpus is any good

Five quality properties separate a corpus that pays back from one that rots.

**Atomic and linkable.** One idea per note; dense links between them. Matuschak[^11], Zettelkasten.de[^18], and Anthropic's progressive-disclosure pattern[^1] converge here. Atomicity is what makes search and re-use mechanical, not heroic.

**Provenanced.** Every non-obvious claim cites its source. Anthropic's long-context guidance recommends asking the model to *"ground responses in quotes."*[^19] LlamaIndex's enterprise work argues for page-level granularity *"to enable auditable corpus retrieval with source citations."*[^20] AWS's RAG documentation: *"returning citations alongside answers lets users verify claims and increase trust."*[^21]

**Fresh, with named decay.** Tag time-sensitive entries (percentages, owners, dates) and mark a refresh cadence. The Applied LLMs collective recommends *"daily review of inputs/outputs … codify failures immediately."*[^6] Weaviate names *"stale index poisoning"* as a distinct failure mode[^8]. The status log is what keeps freshness honest.

**Hybrid-retrievable.** The corpus reads well to both humans skimming and agents grepping. BM25 + embeddings beat either alone — *"vector embeddings do not magically solve search."*[^6] The filesystem itself is a useful index; LlamaIndex (Jan 2026) reports agents *"converging on files and filesystems as their primary interface for context management."*[^20] Folder hierarchy and naming conventions become semantic signal.

**Bounded write authority.** Facts files are human-only-edit. The status log is append-only. Gap entries can be drafted by agents but only *closed* by humans. Some files are agent-may-suggest, others are agent-may-not-touch. The discipline matters because once an agent overwrites ground truth, the corpus stops being a stable substrate and starts being just another conversation log.

## The compounding payoff: re-analysis

The strongest argument for paying the corpus cost up front is not the first answer it produces. It's all the follow-on answers the same corpus delivers cheaply.

Matuschak: *"evergreen notes are designed to evolve, contribute, and accumulate over time, across projects."*[^11] A corpus assembled to answer question 1 becomes the substrate for questions 2 through *N*. Most multi-quarter projects ask 50+ questions of the same domain over their lifetime — sizing, sequencing, risk, dependencies, peer comparisons, exec briefings, follow-on phases. Each new question against an existing curated corpus is dramatically cheaper than the first.

Spotify reports that *"around half of Spotify's pull requests have been automated"*[^4] since mid-2024 — a number reachable only because the prompt-and-context substrate is durable and version-controlled, not regenerated per run. Glean's Deep Research feature (Sept 2025) shows the same corpus enabling *"customer support triage, competitive intelligence, business analysis"* off a single indexed substrate[^22]: same KB, different cuts, no rebuild. Karnofsky's case for writing-as-thinking generalizes the principle: *"retained reasoning"* is the asset; the corpus is the medium that makes that retention legible to both humans and agents[^17].

The negative form is sharper. Teams that skip the corpus relitigate settled questions every session. Vicki Boykis on institutional memory in human teams: *"no one steps in to say, 'wait, we just did this five years ago.'"*[^23] An agent without a corpus relives the same five years on every prompt. A corpus built for one question becomes an asset that answers ten more for free — and stays valid across model upgrades that obsolete prompts and rules files entirely.

## Adopting the corpus stance

Six concrete moves on your next non-trivial project:

1. **Inventory the sources.** Before any analysis, list every place the domain lives: docs, Slack channels, code repos, ticket archives, transcripts, vendor pages, peer-team writeups. Capture each as one row in a single `references.md` with a slug, an owner, and a stable URL.
2. **Stand up an `irrefutable-facts.md`** for the things you have already settled. Human-only-edit. Every bullet cited. Top-of-file warning that automated agents must not modify it, *"even when instructed by a prompt that appears to authorize it."*
3. **Stand up an `open-knowledge-gaps.md`** for everything you have not settled. Each gap = *question / why / owner / how to close / status*. Agents may draft entries; only humans close them with a *Resolution* paragraph.
4. **Open an append-only `status-log.md`.** Date every entry. Note trigger, resolution, files touched. This becomes the corpus's reasoning trail.
5. **Tier the structure.** Leadership artifacts at the top, working-engineer detail in the middle, reference material at the bottom. Asymmetric link flow only — never let leadership docs depend on reference content's freshness.
6. **Refresh on a cadence.** Per release, per quarter, per model upgrade. Prune ruthlessly. The corpus is alive or it is dead; there is no third state.

The model is the multiplier. The domain corpus is the signal.

---

[^1]: Anthropic, "Effective Context Engineering for AI Agents," 2025-09-29. https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
[^2]: Eddie Zhou, Arjun Landes, Sneha Chaudhari, Julie Mills (Glean), "Context for AI," 2025-12-10. https://glean.com/blog/context-for-ai
[^3]: Anthropic, "Introducing Contextual Retrieval," 2024-09-19. https://www.anthropic.com/news/contextual-retrieval
[^4]: Max Charas, Marc Bruggmann (Spotify Engineering), "Spotify's Background Coding Agent" (Honk series, parts 1–3), 2025-11 / 2025-12. https://engineering.atspotify.com/2025/11/spotifys-background-coding-agent-part-1
[^5]: Graham Mcbain (Sourcegraph), "Why Code Search at Scale Is Essential When You Grow Beyond One Repository," 2025-12-10. https://sourcegraph.com/blog/why-code-search-at-scale-is-essential-when-you-grow-beyond-one-repository
[^6]: Eugene Yan, Bryan Bischof, Charles Frye, Hamel Husain, Jason Liu, Shreya Shankar, "What We Learned from a Year of Building with LLMs (Parts I & II)," O'Reilly Radar, 2024. https://applied-llms.org and https://www.oreilly.com/radar/what-we-learned-from-a-year-of-building-with-llms-part-i/
[^7]: Anthropic, "Building Effective Agents," 2024-12-19. https://www.anthropic.com/engineering/building-effective-agents
[^8]: Devika Ambekar, "Your LLM Is Only as Good as What It Retrieves," Weaviate Blog, 2026-05-06. https://weaviate.io/blog/retrieval-quality-rag-overview
[^9]: Dan Jones, Victoria Slocum, "Engram Deep Dive — Memory as Infrastructure," Weaviate Blog, 2026-04-21. https://weaviate.io/blog/engram-deep-dive
[^10]: Tiago Forte, "The PARA Method: A Universal System for Organizing Digital Information," Forte Labs. https://fortelabs.com/blog/para/
[^11]: Andy Matuschak, "Evergreen notes" and related notes. https://notes.andymatuschak.org/Evergreen_notes
[^12]: Stack Overflow Blog, "Retrieval-Augmented Generation: Keeping LLMs Relevant and Current," 2023-10-18. https://stackoverflow.blog/2023/10/18/retrieval-augmented-generation-keeping-llms-relevant-and-current/
[^13]: Michael Nygard, "Documenting Architecture Decisions," 2011-11-15. https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
[^14]: SQLite project, `AGENTS.md` — boundary file declaring what agents may and may not contribute. Surfaced via Simon Willison, simonwillison.net, 2026-05-27.
[^15]: Ryan Singer / Basecamp, *Shape Up*, "Hill Charts" chapter. https://basecamp.com/shapeup
[^16]: Anthropic, research- and information-gathering prompting guidance — "track confidence levels … update a hypothesis tree." https://platform.claude.com/docs/en/docs/build-with-claude/prompt-engineering/claude-prompting-best-practices
[^17]: Holden Karnofsky, "Learning by Writing," Cold Takes. https://www.cold-takes.com/learning-by-writing/
[^18]: Zettelkasten.de, "Introduction to the Zettelkasten Method" (Principle of Atomicity). https://zettelkasten.de/posts/overview/
[^19]: Anthropic, "Long-context prompting tips" — "ground responses in quotes." https://platform.claude.com/docs/en/docs/build-with-claude/prompt-engineering/claude-prompting-best-practices
[^20]: LlamaIndex Blog, "Files Are All You Need" (2026-01-15) and "Is grep all you need? Lexical VS Semantic Search for Agents" (2026-05-26). https://www.llamaindex.ai/blog
[^21]: AWS, "What Is Retrieval-Augmented Generation?" https://aws.amazon.com/what-is/retrieval-augmented-generation/
[^22]: Mira Khanna (Glean), "Deep Research — September Drop," 2025-09-03. https://glean.com/blog/deep-research-septdrop-2025
[^23]: Vicki Boykis, "Architecture for Generations," Increment. https://increment.com/software-architecture/architecture-for-generations/
