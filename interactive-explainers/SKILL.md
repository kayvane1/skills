---
name: interactive-explainers
description: Build interactive and animated explainer figures inside MDX post bundles for TanStack Start + Tailwind v4 + MDX blogs. Three archetypes — static diagrams, auto-playing phase machines, and full interactive simulators. Use when adding a diagram, animation, simulator, or any embedded JSX figure to a blog post; when creating a new post bundle; when the user says "add an interactive thing", "animate this", "build a diagram", "visualise X" for a blog post; or when debugging visual regressions in an existing post component (jittery animation, invisible text, layout shifts after a Tailwind/MDX migration).
---

# Interactive Explainers

Build embedded React figures that live inside a blog post bundle: static diagrams, auto-playing phase machines, full interactive simulators. The architecture is the same across all three.

## Architecture

```
src/content/posts/<slug>/
├── index.mdx           Post prose + JSX imports
├── MyDiagram.tsx       Post-specific component (Tailwind inline)
└── ...                 More components per archetype
```

A post bundle is self-contained. Deleting the directory removes the prose, the components, and their styling atomically. Components that look reusable but only one post uses go inside the bundle. Components imported by two or more posts move out to your project's shared components directory and become **shared primitives**.

## Three archetypes

| Archetype | When to use | Skeleton |
|---|---|---|
| **Static diagram** | Architecture overview, layered concept, no time element. | [EXAMPLES.md#static-diagram](EXAMPLES.md#static-diagram) |
| **Phase machine** | Sequence that auto-plays — race conditions, protocol handshakes, anything with discrete steps. Includes play/pause/restart + clickable timeline. | [EXAMPLES.md#phase-machine](EXAMPLES.md#phase-machine) |
| **Interactive simulator** | User-driven state machine — fire events, observe state evolve. Includes event log, stats, conditional controls. | [EXAMPLES.md#interactive-simulator](EXAMPLES.md#interactive-simulator) |

## Workflow

When adding an interactive figure to a post:

1. **Pick the archetype.** Cheapest to write, easiest to read. Don't reach for a simulator when a static diagram tells the same story.
2. **Co-locate.** Create the component file inside the post's bundle directory.
3. **Import in MDX.**
   ```mdx
   import { MyDiagram } from "./MyDiagram";
   <MyDiagram />
   ```
4. **Style with Tailwind v4** — inline classes only, no new entries in the global stylesheet. See [REFERENCE.md#tailwind-v4-traps](REFERENCE.md#tailwind-v4-traps) for the gotchas that bite hardest.
5. **Animate with care.** Per-property transitions via inline `style`; asymmetric easing so "return" motions are hidden by fades. See [REFERENCE.md#animation-patterns](REFERENCE.md#animation-patterns).
6. **Verify in the browser.** SSR can render fine while client hydration fails silently. Open the page, fire every action, check the dev console.

## Core gotchas (read before writing)

These each cost an hour the first time they bit me — read them once, recognise them next time:

- **Tailwind v4 important is suffix, not prefix.** `text-white!` not `!text-white`. The prefix form generates nothing in v4 and your override silently does nothing.
- **Arbitrary-value utilities don't always beat base utilities.** Adding `bg-[color:var(--ink)]` after `bg-white` in your class string is *not* guaranteed to win — Tailwind v4 may emit them in different cascade order. When toggling colors on an "active" state, use `!` suffix: `bg-[color:var(--ink)]!`.
- **`.post-content` cascades into your component.** `<ul>`, `<li>`, `<code>`, `<p>` inside your figure inherit the post-content element rules (`list-disc`, `mt-4`, code-block styling). Reset them inline: `list-none m-0 p-0`, and add `!` overrides on `<code>` text colors when on dark backgrounds.
- **Don't return React components from TanStack Start route loaders.** seroval can't serialise functions; SSR throws. Strip the `Component` from the loader payload and re-look it up at render time.
- **YAML frontmatter requires quotes for values containing `:`.** Titles like `Doing Things: A Guide` break parsing unless quoted.
- **`remark-gfm` is required for markdown tables.** Default MDX doesn't enable GFM.

Full list of gotchas, palette tokens, and animation recipes in [REFERENCE.md](REFERENCE.md).

## Component skeletons

See [EXAMPLES.md](EXAMPLES.md) for copy-pasteable starting points covering all three archetypes, plus the recurring building blocks (`Token`, `Slab`, `Caption`, `Lane`, `Timeline`).
