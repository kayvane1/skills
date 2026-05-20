# skills

Personal [Claude Code](https://docs.claude.com/en/docs/claude-code) skills.

## Installing

Clone into `~/.claude/skills/` (or symlink individual skills):

```bash
git clone git@github.com:kayvane1/skills.git ~/.claude/skills-repo
ln -s ~/.claude/skills-repo/naming-things ~/.claude/skills/naming-things
```

Skills are picked up automatically by Claude Code on next session.

## Skills

| Skill | Description |
|---|---|
| [naming-things](./naming-things) | Choose precise, consistent names for variables, functions, types, and files. Based on Ousterhout's *A Philosophy of Software Design*, Chapter 14. |
