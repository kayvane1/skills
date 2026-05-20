# skills

Personal AI coding agent skills. Format-compatible with anything that consumes a `SKILL.md` with frontmatter (Claude Code, Codex, etc.).

## Installing

Clone the repo, then symlink individual skills into your agent's skills directory:

```bash
git clone git@github.com:kayvane1/skills.git ~/code/skills

# Claude Code
ln -s ~/code/skills/naming-things ~/.claude/skills/naming-things

# Codex
ln -s ~/code/skills/naming-things ~/.codex/skills/naming-things
```

Adjust paths to match your setup.

## Skills

| Skill | Description |
|---|---|
| [building-modal-apps](./building-modal-apps) | Scaffold Modal.com Python apps using the thin-wrapper pattern: thin Modal endpoints around pure-Python services, TDD with smoke tests on a dev environment. |
| [interactive-explainers](./interactive-explainers) | Build static diagrams, auto-playing phase machines, and full interactive simulators inside MDX post bundles on TanStack Start + Tailwind v4 blogs. Captures the Tailwind v4 traps, animation patterns, and post-bundle architecture in one place. |
| [naming-things](./naming-things) | Choose precise, consistent names for variables, functions, types, and files. Based on Ousterhout's *A Philosophy of Software Design*, Chapter 14. |
| [writing-documentation](./writing-documentation) | Write skimmable, well-written, broadly helpful technical docs. Includes templates for README, runbook, ADR, API reference, troubleshooting, deployment guide. |
