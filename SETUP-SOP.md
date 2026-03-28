---
title: Self-Evolving Claude Code Setup
prerequisites:
  - Claude Code CLI v2.0+
  - A git repository
  - bash + python3 in PATH
stack: Bun, Mulch CLI, Seeds CLI, Claude Code hooks
difficulty: beginner
estimated_time: 3 minutes
---

# Self-Evolving Claude Code Setup

## What This Does

Turns vanilla Claude Code into a self-improving system that remembers what worked, what broke, and what you learned — across every session. Adds safety hooks that block destructive commands and crash protection that saves your state.

## Prerequisites

- [ ] Claude Code CLI installed (v2.0+)
- [ ] Inside a git repository (cd into your project)
- [ ] bash available (Linux/Mac/WSL2)
- [ ] python3 available in PATH
- [ ] Internet connection

## Step 1: Download the skill file

```bash
mkdir -p ~/.claude/skills/self-evolving-setup
curl -o ~/.claude/skills/self-evolving-setup/SKILL.md https://raw.githubusercontent.com/JrokCodes/self-evolving-claude/main/SKILL.md
```

## Step 2: Open Claude Code in your project

```bash
cd ~/your-project
claude
```

## Step 3: Trigger the setup

Type this in Claude Code:

```
/self-evolving-setup
```

## Step 4: Answer the questions

Claude will ask:
1. What you mainly build (frontend, backend, full-stack, etc.)
2. Your primary project directory
3. Full or minimal setup

Claude handles everything else automatically.

## Step 5: Restart Claude Code

```bash
# Exit Claude Code (Ctrl+C or type /exit)
# Re-open it
claude
```

Settings changes need a restart to take effect.

## How to Verify

```bash
# Check expertise system
mulch status

# Check issue tracker
sd list

# Check hooks exist
ls ~/.claude/hooks/

# You should see expertise records loaded at the top of your next session
```

## Common Issues

| Problem | Fix |
|---------|-----|
| `mulch: command not found` | Run: `echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc` |
| `sd: command not found` | Same as above |
| Hooks not firing after restart | Check `~/.claude/settings.json` has a `hooks` section |
| `mulch init` fails | Must be inside a git repo — run `git init` first |
| Lock file error on Windows | Run `touch .mulch/expertise/<domain>.jsonl.lock` before recording |

## What Happens After Setup

- **Every session start**: All past expertise auto-loaded (you'll see it at the top)
- **During builds**: Claude saves learnings in real-time as it discovers them
- **Session end**: Hook reminds to save anything missed
- **Destructive commands**: Blocked before execution (rm -rf, DROP TABLE, etc.)
- **Context compression**: Work state checkpointed and auto-recovered
