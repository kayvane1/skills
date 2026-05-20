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
| [naming-things](./naming-things) | Choose precise, consistent names for variables, functions, types, and files. Based on Ousterhout's *A Philosophy of Software Design*, Chapter 14. |
