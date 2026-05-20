---
name: naming-things
description: Choose precise, consistent names for variables, functions, types, and files based on Ousterhout's "A Philosophy of Software Design" Chapter 14. Use when naming new entities, reviewing/renaming existing ones, when a name feels vague or generic, when reviewing code for clarity, or when the user says "name this", "rename", "better name", "what should I call this", or "review these names".
---

# Naming Things

Good names are a form of documentation. They reduce the need for comments, make bugs easier to spot, and lower the cognitive load of every reader who follows you. Bad names quietly add complexity to every line they appear in.

Two properties matter: **precision** and **consistency**.

## Quick start

When asked to name (or rename) something, run this loop:

1. **Describe the entity in one sentence** — what it IS and what it is NOT.
2. **Draft 3 candidate names**, each ≤3 words.
3. **Test each against the checklist** below. Pick the strongest, or iterate.
4. **If no candidate is precise without being long** → red flag: the entity itself is probably poorly defined. Stop and reshape the design before naming.

## The image test

A good name creates a clear image in the reader's mind of the underlying entity.

Ask: *"If a teammate saw only this name with no surrounding code, would they picture the right thing?"*

- **Too vague?** The image is fuzzy — multiple entities could match.
- **Too long (>3 words)?** The name has stopped being an abstraction and is acting as a sentence. Trim it, or move the detail into the type.
- **Misleading?** The image points at something this entity is *not* (e.g. `list` for a `Set`, `count` for a `bool`).

## Precision checklist

A name is precise when it:

- [ ] Describes the entity's specific role, not its generic category (`block`, `data`, `info`, `value`, `item`, `manager`, `handler`, `helper`, `process` are usually warning signs).
- [ ] Distinguishes this entity from siblings nearby (if two variables could swap names without breaking meaning, at least one is imprecise).
- [ ] Encodes units / shape / state when relevant: `timeoutMs` over `timeout`, `userIds` over `users`, `isLoaded` over `loaded`.
- [ ] Names what it *is*, not *how it's used* by one caller (avoid `tempForLoop`, `resultForCaller`).
- [ ] Booleans answer a yes/no question (`isReady`, `hasParent`, `canRetry`) — never ambiguous nouns.
- [ ] Functions name the action and what they return: `getUserById`, `parseConfig`, `formatBytes` — not `process`, `doWork`, `handle`.

**Red flag**: if a precise, intuitive, short name is hard to find, the entity itself may lack a clear definition. Use naming as a design probe — split the variable, or narrow its purpose, then naming gets easy.

## Consistency rules

Consistency lets readers reuse what they learned from one part of the codebase in another. Three rules:

1. **Always use the common name for a given purpose.** If the codebase uses `userId` for the primary key of users, don't suddenly write `uid`, `user`, or `accountId` for the same thing.
2. **Never reuse the common name for something else.** If `block` already means "filesystem block" project-wide, do not name a UI element `block`.
3. **Make the purpose narrow enough that all variables sharing the name behave the same.** If `node` sometimes means a DOM node and sometimes a tree node, the name is doing two jobs — split it (`domNode`, `treeNode`).

Before introducing a new name, grep the codebase for it. Match the existing convention or pick a clearly distinct word.

## Workflow: reviewing code for naming

When asked to review names in a file or diff:

1. List every identifier the author introduced (variables, params, functions, types, files).
2. For each, mark: **precise?** **consistent with codebase?** **right length?**
3. For failures, propose a concrete replacement — never just "this is bad". Include the *why* (vague / wrong image / collides with `X` elsewhere / encodes units missing).
4. Group findings: blockers (misleading / colliding names) vs. nice-to-have (slightly vague but workable).

## Workflow: renaming an existing entity

1. Grep for current name across the codebase to gauge blast radius.
2. Check rules 1–3 above against the *new* name before committing.
3. Rename via tooling (IDE refactor, `replace_all` only when the name is unambiguous in context).
4. Re-run type checks / tests — silent shadowing is the main risk.

## Common antipatterns

| Smell | Why it's bad | Try instead |
|---|---|---|
| `data`, `info`, `value`, `result` | Tells reader nothing | Name the *kind* of data: `parsedConfig`, `rowCount` |
| `manager`, `handler`, `helper`, `util` | Generic; entity probably does too much | Name the actual responsibility, or split the class |
| `tmp`, `temp`, `x`, `foo` (outside 2-line scope) | No image | A real name, however boring |
| `processData()` | Action verb without object | `validateRows()`, `compressPayload()` |
| `flag`, `status` | Boolean meaning hidden | `isPublished`, `paymentStatus: "paid" \| "pending"` |
| `list`, `array` in name | Encodes type, not meaning; lies if type changes | `users`, `pendingOrders` (plural = collection) |
| Same word, two meanings | Breaks consistency rule 2 | Disambiguate with a qualifier |
| `userUserId` | Repeats context | `userId`, or just `id` if context is unambiguous |

## When to push back on the user

If the user proposes a name that fails the checklist, say so directly with the specific reason and a concrete alternative. Don't accept a vague name just because they suggested it — readability is determined by future readers, not the writer in the moment.

## More thoughts

- **Naming is incremental complexity.** One mediocre name barely matters; thousands compound into an unreadable system. Spend the extra 30 seconds.
- **The reader decides.** A name that's "obvious to me" is not the bar. Picture a teammate joining next month.
- **Skill compounds.** Naming feels slow at first. Keep doing it deliberately — the loop above will run in your head in seconds within a few weeks.
