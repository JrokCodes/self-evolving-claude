# Self-Evolving Claude Code

> One file. 3 minutes. Your Claude Code never forgets again.

Normal Claude Code starts every session from scratch — it doesn't remember what bugs you fixed, what commands worked, or what broke last time. After 50 sessions, it's learned and forgotten the same lessons 50 times.

This setup gives Claude a memory that compounds. Every bug workaround, deploy pattern, API quirk, and convention gets saved and auto-loaded into every future session. Over time, Claude becomes a senior engineer on your project instead of a day-1 hire every single time.

---

## What It Does

**Expertise Tracking** — Discovered that SSH heredocs mangle Python f-strings? Saved. Next session, Claude already knows to SCP the file instead. Never re-learn the same lesson.

**Safety Hooks** — Blocks destructive commands (`rm -rf /`, `DROP DATABASE`, `git push --force main`) before they execute. Claude can't accidentally nuke your stuff.

**Crash Protection** — Long conversation hits the context limit? Your work state is checkpointed and auto-recovered. No more "what were we doing?"

**Issue Tracking** — Persistent task list that survives session restarts. Unlike Claude's built-in tasks which vanish when the session ends.

Everything is local. Nothing phones home. All data lives in your git repo.

---

## Setup (30 seconds of work, Claude does the rest)

### Step 1: Copy the skill file

```bash
mkdir -p ~/.claude/skills/self-evolving-setup
curl -o ~/.claude/skills/self-evolving-setup/SKILL.md https://raw.githubusercontent.com/JrokCodes/self-evolving-claude/main/SKILL.md
```

Or clone and copy:
```bash
git clone https://github.com/JrokCodes/self-evolving-claude.git
cp self-evolving-claude/SKILL.md ~/.claude/skills/self-evolving-setup/SKILL.md
```

### Step 2: Open Claude Code in your project and say

```
/self-evolving-setup
```

### Step 3: Answer 3 questions

Claude asks what you build, your project directory, and full vs minimal setup. Then it installs and configures everything automatically.

### Step 4: Restart Claude Code

Settings changes need a restart. After that, you're live.

---

## What Gets Installed

| Tool | What It Does |
|------|-------------|
| **Mulch** | Records expertise (patterns, failures, conventions) as typed, searchable entries. Auto-loaded every session. |
| **Seeds** | Git-native issue tracker. Tasks persist across sessions, committed alongside your code. |
| **3 hook scripts** | Safety rails, crash protection, session recovery. Total ~3KB. |

**Dependencies**: [Bun](https://bun.sh) (installed automatically if missing)

---

## How It Works

```
You start Claude Code
  |
  +--> mulch prime fires automatically
  |    All past expertise loaded into context
  |    Claude already knows your bugs, patterns, conventions
  |
  +--> You build something
  |    Claude discovers a workaround
  |    Immediately saves it (mulch record)
  |
  +--> Session ends
  |    Hook checks for unsaved learnings
  |
  +--> Next session
       That learning is already loaded
       Claude never re-learns the same lesson
       Repeat. Compound. Get faster.
```

After 50 sessions, your Claude knows every platform quirk, every deploy sequence, every API field mapping in your stack. A vanilla Claude Code instance is still guessing.

---

## Requirements

- Claude Code v2.0+
- A git repository (your project)
- bash + python3 in PATH
- Linux, macOS, or WSL2 (Windows MINGW partial support)

---

## The Difference

| | Vanilla Claude Code | With Self-Evolving Setup |
|---|---|---|
| Session start knowledge | Zero | Everything from every past session |
| Bugs re-discovered | Every time | Never (recorded as failures) |
| Deploy commands | Guesses, reads files | Knows the exact sequence |
| API field names | Trial and error | Already mapped |
| Destructive command protection | None | Blocked before execution |
| Context crash recovery | Lost everything | Checkpoint + auto-restore |
| Task persistence | Gone when session ends | Git-native, permanent |

---

## Credit

- Setup by [@jayydoesai](https://instagram.com/jayydoesai) — [SaaS Killers Community](https://discord.gg/your-invite)
- [Mulch + Seeds (os-eco)](https://github.com/nickarellano/os-eco) by Nick Arellano
- Inspired by the Pied Piper of AI coding setups
