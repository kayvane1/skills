# Reference

Deep notes for [SKILL.md](SKILL.md). Read sections as the gotcha bites you, not top-to-bottom.

## Post bundle structure

```
src/content/posts/<slug>/
├── index.mdx                    # prose + JSX imports
├── ArchitectureDiagram.tsx      # static
├── CoordinationDiagram.tsx      # phase machine
├── CacheSimulator.tsx           # interactive simulator
└── (any other co-located deps)
```

A bundle owns *everything* the post needs. The deletion test (Ousterhout): if you delete `src/content/posts/<slug>/`, the post and every supporting file go with it. That's the invariant to preserve.

**Shared primitives** (used across two or more bundles) move out of the bundle to your project's site-wide components directory (typically `src/components/`). A `<Tldr />` callout used by every post is the canonical example — pull it out the moment a second post wants it.

## MDX import paths

From a post at `src/content/posts/<slug>/index.mdx`:

```mdx
import { Tldr } from "../../../components/Tldr";       // shared primitive
import { MyDiagram } from "./MyDiagram";                // bundle-local component
```

## Frontmatter rules

```yaml
---
title: "Doing Things: A Guide"   # quote any value containing ':'
date: 2026-05-20
summary: A one-liner.            # also quote if it contains ':'
tags: [foo, bar]                 # tags should be validated by your post loader
github: https://github.com/...
readingTime: 8 min               # optional override
---
```

- Validate frontmatter with Zod at load time (e.g. a `FrontmatterSchema` in your post loader). Typos then surface as load-time errors, not silent missing fields.
- If you maintain a tag allow-list (recommended — prevents typos and stray tags), tags not in the allow-list will be silently dropped. Add new tags to the allow-list before referencing them in a post.
- Auto-estimate `readingTime` from JSX-stripped MDX source. Authors override it via the frontmatter field only when the estimate is wrong (long code blocks, lots of embedded components).

## Tailwind v4 traps

### Important modifier moved from prefix to suffix

```ts
// v3 (broken in v4 — generates nothing)
"!text-white"

// v4 (correct)
"text-white!"
```

This bit twice in the cache simulator — active chips kept dark text on dark backgrounds (invisible) because the prefix-`!` class generated no CSS.

### Arbitrary-value utilities and cascade order

When you toggle a base utility with an active-state arbitrary-value utility:

```tsx
const CHIP = "bg-white text-[color:var(--ink)] …";
const CHIP_ACTIVE = "bg-[color:var(--ink)] text-white …";
// Used as: className={`${CHIP} ${active ? CHIP_ACTIVE : ""}`}
```

The active classes are *later* in the className string, but Tailwind v4 may emit them *earlier* in the cascade. The base `bg-white` wins, the override silently does nothing.

**Fix:** add `!` to the active-state colors/bgs/borders so they win regardless of cascade order:

```ts
const CHIP_ACTIVE = "bg-[color:var(--ink)]! text-white! border-[color:var(--ink)]!";
```

### `.post-content` cascades into your figures

The blog wraps post content in `<div class="post-content">` with element rules like:

```css
.post-content code  { @apply rounded bg-black/5 px-2 py-0.5 …; }
.post-content ul    { @apply mt-4 space-y-2 pl-5; }
.post-content li    { @apply list-disc; }
.post-content p     { @apply mt-4; }
```

These cascade into your figure's children. Reset inline:

```tsx
<ul className="list-none m-0 p-0 …">
  <li className="list-none border-b border-dashed border-black/10 …">
    <code className="bg-transparent! p-0! text-[#f3f3f0]!">{key}</code>
    {/* `!` because .post-content code has higher specificity */}
  </li>
</ul>
```

### Type-hinted arbitrary values

`bg-[color:var(--ink)]` works in v4 but the `color:` type hint is occasionally unnecessary. If a class isn't applying, try `bg-[var(--ink)]` as a fallback. CSS variables defined in `@theme` blocks can also use the shorthand `bg-(--ink)`.

## Animation patterns

### Per-property transitions via inline style

Tailwind can't express per-property durations cleanly. When you need transform + opacity to have different timings, fall back to inline `style`:

```tsx
<span
  style={{
    transition:
      "transform 0.45s cubic-bezier(0.4,0,0.2,1), opacity 0.22s ease",
  }}
/>
```

The single-duration Tailwind class `duration-[450ms]` collapses everything into one timing function and one duration. That's usually wrong for transitions where you want fade-in to happen as a smooth slide-in, or for the next pattern below.

### Asymmetric ease for "return" motions

When an element fades in while sliding in, you want the inverse to fade out *before* sliding back. Otherwise the return motion is visible and ugly.

Conditionally swap the `transition` based on the active flag:

```tsx
style={{
  transform: active ? "translate(-50%, 26px)" : "translate(-50%, 0)",
  transition: active
    // active: slide down smoothly while fading in
    ? "transform 0.45s cubic-bezier(0.4,0,0.2,1), opacity 0.22s ease"
    // inactive: fade out fast, *delay* the position reset
    : "transform 0s linear 0.22s, opacity 0.18s ease",
}}
```

CSS applies the destination state's `transition` to the value change. Going down uses the smooth ease. Going back uses an instant transform delayed by 0.22s — by which point opacity has faded to 0, so the snap-back is invisible.

### Combined transforms via inline style

Stacked Tailwind transform utilities don't always compose cleanly. Instead of:

```tsx
className="translate-x-[-50%] translate-y-[26px]"   // can jitter
```

write the transform inline:

```tsx
style={{ transform: "translate(-50%, 26px)" }}
```

One transform string, no composition surprises.

### Keyframes for pulse / fanout effects

Keyframes go in your project's global stylesheet (Tailwind v4 picks them up globally):

```css
@keyframes my-pulse {
  0%, 100% { box-shadow: 0 0 0 0 rgba(46, 125, 50, 0.45); }
  50%      { box-shadow: 0 0 0 8px rgba(46, 125, 50, 0); }
}
```

Reference them from components via Tailwind arbitrary-value animations:

```tsx
<span className="animate-[my-pulse_1.4s_ease-in-out_infinite]" />
```

This is the only thing that stays in the global stylesheet — keyframes are not bundle-local. The trade-off is acceptable because keyframes are rarely deleted.

### will-change for GPU compositing

For elements with frequent transitions:

```tsx
className="will-change-transform"
```

Force-promotes the element to a compositor layer. Use sparingly; every `will-change` costs memory.

## Layout patterns

### Aspect-ratio stage with percentage positioning

For animated figures with absolutely-positioned actors:

```tsx
<div className="relative w-full aspect-[16/7] max-[640px]:aspect-[4/5]">
  <Actor style={{ left: "10%", top: "22%" }} />
  …
</div>
```

The stage scales with the article width. Children position by percentage. Transitions on `top`/`left` interpolate smoothly. Mobile flips to a taller aspect ratio (4:5) so the same content stays readable.

### Fixed token slots across phases

Don't override positions per phase if you can avoid it. Define one canonical slot per logical location:

```ts
const NODES = {
  containerA:  { x: 6, y: 22 },
  containerB:  { x: 6, y: 50 },
  containerC:  { x: 6, y: 78 },
  dictPending: { x: 59, y: 23 },
  dictResult:  { x: 59, y: 62 },
  queueB:      { x: 91, y: 28 },
  queueC:      { x: 91, y: 65 },
};
```

Then each phase references the slot:

```ts
TOKEN_TRACKS: {
  A: {
    idle:    NODES.containerA,
    request: NODES.dictPending,
    write:   NODES.dictResult,
    // …
  },
}
```

Tokens stay at the same pixel position across phases unless the phase intends to move them. Removes a class of "where is the token now?" bugs.

### Grid blowout prevention

CSS Grid columns can blow out if their content is wider than the column allows. Always use `minmax(0, ...)`:

```tsx
className="grid grid-cols-[minmax(0,1.35fr)_minmax(0,1fr)]"
```

Without `minmax(0, ...)`, long text inside a column will force the column wider than its `fr` share, breaking the layout.

## Color palette

Use the project's CSS variables (`var(--ink)`, `var(--ink-muted)`, `var(--paper-bright)`) for anything that should follow the site theme. Hard-code only the figure-specific accents.

| Token | Value | Use |
|---|---|---|
| `var(--ink)` | `#141413` | Primary text, dark fills |
| `var(--ink-muted)` | `#4c4a45` | Secondary text, dashed borders |
| `var(--paper-bright)` | `#fbfaf7` | Card backgrounds |
| Accent green | `#2e7d32` | Active states, "hit" tone |
| Accent soft | `rgba(46,125,50,0.16)` | Active glow, soft fills |
| Dark slab | `#0e0f11` | "Shared/durable" things (Dict, Queue, DB) |
| Dark slab text | `#f3f3f0` | Body text on dark slabs |

### Tonal palette for event logs

When a figure has a log feed, use a consistent four-tone palette:

| Tone | Hex | Used for |
|---|---|---|
| hit | `#b6f0a5` (log) / `#2e7d32` (legend dot) | Cache hits, success |
| miss | `#ffb454` (log) / `#c87a2f` (legend dot) | Cache misses, failures |
| lock | `#9bd6ff` (log) / `#3a6ea5` (legend dot) | Coordination events |
| compute | `#d9b3ff` (log) / `#6b4ea0` (legend dot) | Expensive work |

## TanStack Start gotchas

### Don't return React components from route loaders

The route loader payload is serialised by seroval for SSR hydration. Functions can't be serialised; a `Component: ComponentType` field will crash hydration.

```tsx
// ❌ Broken
loader: ({ params }) => getPostBySlug(params.slug);

// ✅ Strip the Component, re-look up at render time
loader: ({ params }) => {
  const post = getPostBySlug(params.slug);
  if (!post) throw notFound();
  const { Component: _Component, ...meta } = post;
  return meta;
}

function PostPage() {
  const meta = Route.useLoaderData();
  const full = getPostBySlug(meta.slug);
  const Body = full?.Component;
  return <article>{Body ? <Body /> : null}</article>;
}
```

The post map is module-level and identical on client + server, so the re-lookup is free.

### MDX plugin order in vite.config

The MDX plugin must run *before* `viteReact` so its JSX output is what React's plugin transforms:

```ts
plugins: [
  // …
  {
    enforce: "pre",
    ...mdx({
      remarkPlugins: [
        remarkFrontmatter,
        [remarkMdxFrontmatter, { name: "frontmatter" }],
        remarkGfm,    // <-- required for tables
      ],
    }),
  },
  tanstackStart({ … }),
  viteReact({ include: /\.(jsx|tsx|mdx)$/ }),
]
```

### Raw-source globs return different shapes

`import.meta.glob(..., { eager: true, query: "?raw" })` returns objects with a `.default` property in some Vite versions, raw strings in others. Defend against both:

```ts
function readSource(value: unknown): string {
  if (typeof value === "string") return value;
  if (value && typeof value === "object" && "default" in value) {
    const inner = (value as { default: unknown }).default;
    if (typeof inner === "string") return inner;
  }
  return "";
}
```

## Debugging recipe

When a figure renders but looks wrong:

1. **Page returns 200, content present?** `curl http://localhost:<port>/posts/<slug>` and `grep` for known text from your figure. If the text is there, it's a CSS / hydration issue, not a render error.
2. **Open the dev server log.** `Serialization error: Seroval Error` → loader returns a function (see TanStack Start gotchas). `YAMLParseError: Nested mappings...` → unquoted `:` in frontmatter.
3. **Inspect rendered class strings.** `grep -oE 'class="[^"]*<bem-name>[^"]*"' rendered.html`. Confirm the active classes are present. If they are present but visually doing nothing → Tailwind v4 important / cascade issue → add `!` suffix.
4. **Look at the deleted CSS via git.** If you migrated from BEM (or any other CSS) to Tailwind and the new version feels off, the old per-property `transition` / animation timings are in git history. `git show <commit-before-migration>:<path-to-stylesheet> | grep -A20 <selector>`.
