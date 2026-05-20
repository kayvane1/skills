---
name: writing-documentation
description: Write clear, skimmable technical documentation — READMEs, runbooks, ADRs, API references, troubleshooting guides, specs, design docs. Enforces three principles: easy to skim, well-written, broadly helpful. Use when creating or revising any docs over ~30 lines, when reviewing docs before publishing, when a doc feels hard to read, or when the user asks to "write a README / runbook / guide / API reference", "review these docs", or "improve this documentation".
---

# Writing Documentation

Documentation puts useful information into other people's heads. Three principles govern everything:

1. **Easy to skim** — readers jump around. Optimise for partial reading.
2. **Well-written** — bad prose taxes the reader. Minimise the tax.
3. **Broadly helpful** — assume varying knowledge, patience, and English fluency.

## Workflow

**Plan:** audience, purpose, scope, doc type.
**Write:** apply the rules below.
**Review:** run the rules as a checklist before publishing.

See [TEMPLATES.md](TEMPLATES.md) for the right template per doc type. See [REFERENCE.md](REFERENCE.md) for the full essay and rationale.

## Make it skimmable

- **Section titles state the conclusion**, not the topic. "Streaming cut TTFT by 50%" > "Results".
- **Topic sentences open every section** and stand alone — a reader who only reads the first line should get the point.
- **Topic words come first** in those sentences. "Vector databases speed up embeddings search" > "Embeddings search can be sped up by vector databases".
- **Takeaways up front.** No Socratic build-up. State the conclusion, then justify.
- **Short paragraphs** (1–4 sentences). One-sentence paragraphs for critical points.
- **Bullets and tables** for parallel items.
- **Bold the load-bearing phrase** in any long paragraph.
- **Table of contents** for docs over ~200 lines.

## Write well

- **Short sentences.** Split long ones. Cut adverbs and filler. Imperative mood where it fits.
- **Unambiguous parsing.** "Write section titles as sentences" > "Title sections with sentences" (is "Title" a verb?).
- **Right-branching, not left-branching.** "To make pancakes, you need X, Y, Z" > "You need X, Y, Z to make pancakes" — don't make the reader hold the verb in memory.
- **Replace "this"/"that" across sentences** with the actual noun. "Building on message formatting…" > "Building on this…".
- **Consistency** in capitalisation, punctuation, naming, terminology. Inconsistencies snag the reader.
- **Don't presume the reader's state.** "To call a function…" > "Now you probably want to call a function…".

## Be broadly helpful

- **Write simply.** Many readers don't speak English as a first language. Don't oversimplify, but don't show off.
- **Avoid abbreviations.** "Instruction following" > "IF". Expand on first use if you must use them.
- **Specific over jargony.** "Input" > "prompt". "Max token limit" > "context limit".
- **Solve common problems proactively** — the 5% who get stuck cost more than the 95% who skim past.
- **Self-contained code examples** — no extra installs, no cross-page hopping.
- **Never demo bad habits** — no API keys in code, no untrusted shell pipes, no insecure defaults.
- **Prioritise by reader value.** Common problems first, rare ones last or in an appendix.

## Pre-publish checklist

- [ ] Section titles state conclusions (or specific topics), not abstract nouns ("Results", "Overview" alone)
- [ ] Each section opens with a standalone topic sentence
- [ ] No left-branching sentences burdening working memory
- [ ] No cross-sentence "this" / "that"
- [ ] Paragraphs 1–4 sentences
- [ ] Bullets/tables used where content is parallel
- [ ] Abbreviations expanded on first use
- [ ] Code examples are self-contained and demonstrate good habits
- [ ] Common problems addressed; topics ordered by reader value
- [ ] All links work; all commands run as shown

## Override clause

These are guidelines, not commandments. Break a rule when serving the reader requires it. Documentation is empathy in writing.
