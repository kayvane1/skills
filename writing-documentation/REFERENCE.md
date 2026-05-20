# Documentation — Principles and Rationale

This file expands on the rules in [SKILL.md](SKILL.md). Read it once to internalise the *why*; refer back when a rule feels wrong for a specific case.

## Why these three principles

**Documentation is an interface to human memory.** Words on a screen are how knowledge moves from one head to another. The principles below minimise the friction of that transfer:

- **Easy to skim** respects the reader's time. Few readers read linearly.
- **Well-written** respects the reader's attention. Bad prose burns it.
- **Broadly helpful** respects the reader's starting point. They may not be you.

## Make it skimmable — extended notes

**Section titles act as signposts.** A reader scanning a 500-line doc decides in milliseconds whether to read a section based on its title. "Results" forces them to dive in. "Streaming reduced time to first token by 50%" tells them the answer up front — they can keep skimming, or dive in for the *how*.

**Tables of contents are hash maps.** They turn an O(n) linear scan into an O(1) jump. Their second job is signalling what the doc *is* — a reader sometimes decides not to read at all based on the ToC. Both are wins.

**Topic sentences must stand alone.** When skimming, readers look disproportionately at the first word, first line, and first sentence of a section. If that sentence depends on prior context ("Building on the above…"), it becomes meaningless to the skimmer. Write it so a reader landing here from a search engine gets the point.

**Topic words come first** because skimmers read one or two words before deciding to move on. Lead with the noun the paragraph is about.

**Bold sparingly.** Bolding everything is the same as bolding nothing. Pick the load-bearing phrase per long paragraph.

## Write well — extended notes

**Ambiguous parsing wastes a few milliseconds per sentence, but they compound.** "Title sections with sentences" — is "Title" a verb (imperative: "give titles to sections") or a noun ("the title sections, the ones with sentences")? The reader's brain commits to one parse, sometimes wrong, then has to backtrack. "Write section titles as sentences" parses unambiguously even though it's two words longer.

**Left-branching trees overload working memory.** "You need flour, eggs, milk, butter, and a dash of salt to make pancakes" — the reader doesn't know what the list is *for* until the end. They hold the whole list in memory while parsing the predicate. Flip it: "To make pancakes, you need flour, eggs, milk, butter, and a dash of salt." Now the predicate frames the list as it streams in.

**Cross-sentence "this" forces lookups.** "Building on this, now let's discuss function calling." Building on *what*? The reader has to scan upward. Spell it out: "Building on message formatting, let's discuss function calling." Often you can cut the demonstrative entirely: "Let's discuss function calling."

**Consistency is pattern-matching cooperation.** The reader's brain is an excellent pattern matcher. If you use Title Case for some headings and sentence case for others, the reader notices and wonders why — burning attention on formatting instead of content.

**Don't presume the reader's mental state.** "Now you probably want to…" presumes. Some readers don't. The phrasing risks annoyance or loss of credibility. Use task-framed alternatives: "To call a function, …".

## Be broadly helpful — extended notes

**Write for the worst-case reader.** Many readers don't speak English as a first language. Many are experts in adjacent fields (great C++ engineer, beginner in Python). Even within experts, fatigue, interruptions, or unfamiliar terminology make basic prose easier to absorb than dense prose. Simplification is rarely a loss to experts — they skim past it — but it's a major gain to everyone else.

**Abbreviations cost beginners more than they save experts.** Spell out "instruction following" instead of "IF"; "retrieval-augmented generation" instead of "RAG"; "time to first token" instead of "TTFT". The cost of writing the long form is one extra second. The cost of decoding to a beginner is a context switch and possibly a search.

**Jargon optimises for the writer.** Prefer self-evident terms. "Input" beats "prompt". "Max token limit" beats "context limit". Field jargon may be familiar to you and locks out everyone else.

**Cover common problems even if the answer feels obvious.** Documentation is read by a long tail. If 95% of your readers already know how to install a Python package, the cost of including a one-line install hint is near zero — experts skim past. The cost of *excluding* it for the 5% is they get stuck and may abandon you. Asymmetric cost-benefit.

**Self-contained examples respect the reader's setup.** Examples that require installing three libraries, configuring environment variables, and reading a referenced section halfway across the doc are barriers. Inline everything that's small enough to inline.

**Never demo bad habits.** A reader will copy-paste your example into production. If you put an API key in code "for illustration", that's the pattern they'll follow. Show the right way the first time.

**Prioritise by reader value, not by writer ease.** Common problems get top placement. Rare problems go in appendices or are linked from a "see also" section. Order content by how often it's needed, not by how interesting it was to write.

## Antipatterns gallery

| Wrong | Right | Why |
|---|---|---|
| `## Overview` followed by 4 paragraphs of setup before the point | State the point in sentence 1; justify after | No Socratic build-up |
| `## Results` | `## Streaming cut TTFT by 50%` | Titles state conclusions |
| "Embeddings search can be sped up by vector databases." | "Vector databases speed up embeddings search." | Topic word first |
| "Title sections with sentences." | "Write section titles as sentences." | Parses unambiguously |
| "You need flour, eggs, milk to make pancakes." | "To make pancakes, you need flour, eggs, milk." | Right-branching |
| "Building on this, …" | "Building on message formatting, …" | No cross-sentence demonstrative |
| "Now you probably want to call a function." | "To call a function, …" | No presuming the reader |
| `IF`, `RAG`, `TTFT` without expansion | "instruction following (IF)" on first use | Abbreviations cost beginners |
| `OPENAI_API_KEY = "sk-…"` in an example | `os.environ["OPENAI_API_KEY"]` | No bad habits |
| 10-line install snippet inside a 3-line example | Link to install docs; keep example minimal | Self-contained but not bloated |
| Inconsistent terminology (`prompt` here, `input` there) | Pick one, use it everywhere | Consistency |
| 500-line doc with no ToC | Add ToC | Hash map for the reader |

## When to break the rules

These are guidelines optimised for the median case. Reasons to break them:

- **A name has industry-standard jargon** — using "REST" instead of expanding is fine when your audience is definitely API engineers.
- **A specific structure is mandated** — RFC, ADR, or internal templates may dictate ordering you can't change.
- **The conclusion needs evidence first** — sometimes the surprise lands harder if you walk through the data. But default to up-front, and break only deliberately.

Documentation is empathy. Picture the reader; do what helps them most.
