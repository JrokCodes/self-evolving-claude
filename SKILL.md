---
name: self-evolving-setup
description: Set up a self-evolving Claude Code environment with expertise tracking, safety hooks, and crash protection
triggers:
  - set up self-evolving
  - self-evolving setup
  - set up mulch
  - expertise tracking setup
  - make claude remember
  - self-improving claude
user_invocable: true
---

# Self-Evolving Claude Code Setup

## What This Does (Read This to the User First)

Before doing anything, explain this to the user in plain language:

> **Here's what I'm about to set up for you:**
>
> Right now, every time you start a new Claude Code session, I start completely fresh. I don't remember what bugs we fixed, what patterns worked, or what broke last time. Every session, you're working with a version of me that has amnesia.
>
> This setup gives me a memory system. Whenever we discover something during a build — a bug workaround, a deploy command that works, an API quirk — it gets saved to a small file in your project. Next time you open Claude Code, all of those learnings are automatically loaded before you even type anything.
>
> Over time, I get faster and make fewer mistakes because I never re-learn the same lesson twice. After a few weeks of building, the difference is massive.
>
> **I'm also going to add:**
> - **Safety rails** — I'll block myself from running destructive commands (like `rm -rf /` or `DROP DATABASE`) so I can't accidentally nuke your stuff
> - **Crash protection** — if our conversation gets too long and needs to compress, I'll save a checkpoint of what we were doing so I can pick up where we left off
> - **Issue tracking** — a lightweight task tracker that persists between sessions (unlike my built-in tasks which vanish when the session ends)
>
> **What it won't do:**
> - Won't change how you talk to me — everything works the same, just smarter
> - Won't slow anything down — the hooks run in milliseconds
> - Won't touch your existing code or projects
> - Won't send data anywhere — everything stays local in your repo
>
> **Takes about 3 minutes. I do everything.**

After explaining, ask if they want to proceed.

---

## Prerequisites Check

Before starting, verify these requirements:

```bash
# Check 1: Claude Code version (needs 2.0+ for hooks)
claude --version

# Check 2: Git repo exists in current directory
git rev-parse --is-inside-work-tree

# Check 3: bash is available
which bash

# Check 4: python3 is available (safety hook needs it)
which python3

# Check 5: Check if Bun is already installed
which bun

# Check 6: Check if Mulch/Seeds already installed
which mulch
which sd

# Check 7: Detect OS/platform
uname -s
```

If not in a git repo, tell the user they need to `cd` into their project repo first (or `git init` one). Mulch and Seeds are git-native — they need a repo.

If python3 is missing, warn that the safety hook won't work without it and offer to skip it.

---

## Phase 1: Ask the User

Use AskUserQuestion to gather setup info. Ask these:

**Question 1: What do you mainly build?**
Options:
- Frontend (React, Vue, Svelte, etc.)
- Backend / APIs (Python, Node, Go, etc.)
- Full-stack (both frontend and backend)
- Data / ML / AI
- DevOps / Infrastructure
- AI agents / automation
- Mobile apps
- Other (let them describe)

**Question 2: What's your primary project directory?**
Default to the current working directory. Let them type a path if different.

**Question 3: Setup level?**
- Full setup (Mulch + Seeds + all hooks) — Recommended
- Minimal (just Mulch + expertise hooks, no Seeds or safety hook)

---

## Phase 2: Install Dependencies

### Step 1: Install Bun (if not already installed)

```bash
# Check if bun exists
if ! command -v bun &> /dev/null; then
    curl -fsSL https://bun.sh/install | bash
    export BUN_INSTALL="$HOME/.bun"
    export PATH="$BUN_INSTALL/bin:$PATH"
fi
```

**IMPORTANT**: After installing Bun, the user may need to restart their shell or source their profile for PATH to update. Check with `bun --version` after install.

### Step 2: Install Mulch CLI

```bash
bun add -g @os-eco/mulch-cli@0.6.3
```

Verify: `mulch --version` should output `0.6.3`

If `mulch` command not found after install, the user needs to add Bun to PATH:
```bash
echo 'export BUN_INSTALL="$HOME/.bun"' >> ~/.bashrc
echo 'export PATH="$BUN_INSTALL/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

For zsh users, replace `.bashrc` with `.zshrc`.

### Step 3: Install Seeds CLI (if full setup)

```bash
bun add -g @os-eco/seeds-cli@0.2.4
```

Verify: `sd --version` should output `0.2.4`

---

## Phase 3: Initialize Project

### Initialize Mulch in the user's project repo

```bash
cd <USER_REPO>
mulch init
```

This creates:
```
.mulch/
  mulch.config.yaml    # Config file
  README.md            # Quick reference
  expertise/           # Where records are stored (empty)
```

### Create initial domains based on user's answers

Domains are just organizational labels. They auto-create when you record to them, but seeding a few helps Claude know where to categorize things.

Based on Question 1 answers, record a starter entry in each domain:

```bash
# Example for a full-stack builder:
cd <USER_REPO>
mulch record frontend --type convention --name "initial setup" --description "Frontend expertise domain initialized. Record patterns, failures, and conventions discovered during frontend builds here."
mulch record backend --type convention --name "initial setup" --description "Backend expertise domain initialized. Record patterns, failures, and conventions discovered during backend builds here."
mulch record deployment --type convention --name "initial setup" --description "Deployment expertise domain initialized. Record deploy patterns, port mappings, and infrastructure conventions here."
mulch record tools --type convention --name "initial setup" --description "Tools expertise domain initialized. Record tool quirks, CLI bugs, and workarounds here."
```

**Domain recommendations by user profile:**

| Profile | Domains to Create |
|---------|------------------|
| Frontend | frontend, backend, deployment, tools |
| Backend / APIs | backend, deployment, database, tools |
| Full-stack | frontend, backend, deployment, database, tools |
| Data / ML | data, models, pipelines, tools |
| DevOps / Infra | deployment, infrastructure, monitoring, tools |
| AI agents | agents, backend, deployment, tools |
| Mobile | mobile, backend, deployment, tools |

### Initialize Seeds (if full setup)

```bash
cd <USER_REPO>
sd init
```

This creates:
```
.seeds/
  config.yaml       # Project name
  issues.jsonl      # Issue storage (empty)
  templates.jsonl   # Templates (empty)
  .gitignore        # Excludes lock files
```

---

## Phase 4: Create Hook Scripts

### Create the hooks directory

```bash
mkdir -p ~/.claude/hooks
```

### Hook 1: safety-check.sh

Write this file to `~/.claude/hooks/safety-check.sh`:

```bash
#!/bin/bash
# Safety check hook — blocks destructive commands
# Exit 0 = allow, Exit 2 = block (stderr sent back to Claude)

INPUT=$(cat)

COMMAND=$(echo "$INPUT" | python3 -c "
import sys, json
try:
    data = json.load(sys.stdin)
    cmd = data.get('tool_input', {}).get('command', '')
    print(cmd)
except:
    print('')
" 2>/dev/null)

if [ -z "$COMMAND" ]; then
    exit 0
fi

BLOCKED_PATTERNS=(
    "rm -rf /"
    "rm -rf ~"
    "rm -rf \$HOME"
    "git push --force main"
    "git push --force master"
    "git push -f main"
    "git push -f master"
    "git push origin main --force"
    "git push origin master --force"
    "git reset --hard origin"
    "git clean -fdx"
    "DROP DATABASE"
    "DROP TABLE"
    "TRUNCATE TABLE"
    "DROP SCHEMA"
    "> /dev/sda"
    "mkfs"
    "dd if="
)

PIPE_SHELL_PATTERNS=(
    "curl.*\|.*\bbash\b"
    "curl.*\|.*\bsh\b"
    "wget.*\|.*\bbash\b"
    "wget.*\|.*\bsh\b"
    "curl.*\|.*\bpython\b"
    "wget.*\|.*\bpython\b"
)

for pattern in "${BLOCKED_PATTERNS[@]}"; do
    if echo "$COMMAND" | grep -qi "$pattern"; then
        echo "{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"BLOCKED: Destructive command detected - '$pattern'. Ask the user first.\"}}" >&2
        exit 2
    fi
done

for pattern in "${PIPE_SHELL_PATTERNS[@]}"; do
    if echo "$COMMAND" | grep -qiE "$pattern"; then
        echo "{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"BLOCKED: Pipe-to-shell detected. This can execute arbitrary remote code. Ask the user first.\"}}" >&2
        exit 2
    fi
done

exit 0
```

Make executable: `chmod +x ~/.claude/hooks/safety-check.sh`

### Hook 2: pre-compact.sh

Write this file to `~/.claude/hooks/pre-compact.sh`.

**IMPORTANT**: Replace `<USER_REPO>` with the user's actual project directory path.

```bash
#!/bin/bash
# Pre-compaction hook — saves session state before context compression

CHECKPOINT_FILE="$HOME/.claude/checkpoint.md"
TASKS_DIR="$HOME/.claude/tasks"
PROJECT_DIR="<USER_REPO>"

{
  echo "# Session Checkpoint"
  echo ""
  echo "**Saved at**: $(date -u '+%Y-%m-%dT%H:%M:%SZ')"
  echo ""

  if [ -d "$TASKS_DIR" ]; then
    for team_dir in "$TASKS_DIR"/*/; do
      if [ -d "$team_dir" ]; then
        team_name=$(basename "$team_dir")
        echo "## Team: $team_name"
        echo ""
        for task_file in "$team_dir"/*.json; do
          if [ -f "$task_file" ]; then
            python3 -c "
import json, sys
try:
    with open('$task_file') as f:
        t = json.load(f)
    status = t.get('status', 'unknown')
    subject = t.get('subject', 'unknown')
    owner = t.get('owner', 'unassigned')
    tid = t.get('id', '?')
    print(f'- [{status}] #{tid}: {subject} (owner: {owner})')
except:
    pass
" 2>/dev/null
          fi
        done
        echo ""
      fi
    done
  fi

  echo "## Recently Modified Files (last 30 min)"
  echo ""
  if [ -d "$PROJECT_DIR" ]; then
    find "$PROJECT_DIR" \
      -not -path "*/node_modules/*" \
      -not -path "*/.git/*" \
      \( -name "*.md" -o -name "*.ts" -o -name "*.tsx" -o -name "*.py" -o -name "*.json" -o -name "*.js" -o -name "*.jsx" \) \
      2>/dev/null | while read f; do
      if [ "$(find "$f" -maxdepth 0 -mmin -30 2>/dev/null)" ]; then
        echo "- $f"
      fi
    done
  fi

  echo ""
  echo "## Instructions"
  echo ""
  echo "You are resuming after context compaction. Read this checkpoint to recover your state."
  echo "Check the task list and read any relevant files to continue where you left off."

} > "$CHECKPOINT_FILE" 2>/dev/null

exit 0
```

Make executable: `chmod +x ~/.claude/hooks/pre-compact.sh`

### Hook 3: session-recovery.sh

Write this file to `~/.claude/hooks/session-recovery.sh`:

```bash
#!/bin/bash
# Session recovery — re-injects checkpoint after compaction or resume

CHECKPOINT_FILE="$HOME/.claude/checkpoint.md"

if [ -f "$CHECKPOINT_FILE" ]; then
  cat "$CHECKPOINT_FILE"
else
  echo "No checkpoint file found. Starting fresh session."
fi
```

Make executable: `chmod +x ~/.claude/hooks/session-recovery.sh`

---

## Phase 5: Configure settings.json

Read the user's existing `~/.claude/settings.json` and MERGE the following configuration. Do NOT overwrite existing settings — add to them.

**IMPORTANT**: Replace `<USER_REPO>` with the user's actual project directory (absolute path).

### What to merge into settings.json:

**env section** — add `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`:
```json
"env": {
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "70"
}
```

**hooks section** — add all hook configurations. If the user already has hooks, append to the existing arrays:

```json
"hooks": {
    "PreToolUse": [
        {
            "matcher": "Bash",
            "hooks": [
                {
                    "type": "command",
                    "command": "bash ~/.claude/hooks/safety-check.sh",
                    "timeout": 5
                }
            ]
        }
    ],
    "PreCompact": [
        {
            "matcher": "",
            "hooks": [
                {
                    "type": "command",
                    "command": "bash ~/.claude/hooks/pre-compact.sh",
                    "timeout": 10
                },
                {
                    "type": "command",
                    "command": "cd <USER_REPO> && mulch prime --budget 3000 2>/dev/null || true",
                    "timeout": 10
                }
            ]
        }
    ],
    "Stop": [
        {
            "matcher": "",
            "hooks": [
                {
                    "type": "command",
                    "command": "cd <USER_REPO> && LEARN_OUTPUT=$(mulch learn 2>/dev/null) && CHANGED=$(echo \"$LEARN_OUTPUT\" | grep -c 'Changed files' || true) && if [ \"$CHANGED\" -gt 0 ]; then echo '{\"systemMessage\":\"SESSION CLOSE: mulch detected changed files. Run: 1) mulch learn 2) mulch record <domain> --type <type> --name \\\"name\\\" --description \\\"what you learned\\\" 3) mulch sync\"}'; fi",
                    "timeout": 10
                }
            ]
        }
    ],
    "SessionStart": [
        {
            "matcher": "",
            "hooks": [
                {
                    "type": "command",
                    "command": "cd <USER_REPO> && mulch prime --budget 3000 2>/dev/null || true",
                    "timeout": 10
                },
                {
                    "type": "command",
                    "command": "cd <USER_REPO> && mulch prime --context 2>/dev/null || true",
                    "timeout": 10
                }
            ]
        },
        {
            "matcher": "compact",
            "hooks": [
                {
                    "type": "command",
                    "command": "bash ~/.claude/hooks/session-recovery.sh",
                    "timeout": 5
                }
            ]
        },
        {
            "matcher": "resume",
            "hooks": [
                {
                    "type": "command",
                    "command": "bash ~/.claude/hooks/session-recovery.sh",
                    "timeout": 5
                }
            ]
        }
    ]
}
```

---

## Phase 6: Update CLAUDE.md

Read the user's existing `~/.claude/CLAUDE.md`. If it doesn't exist, create it. Append these two sections:

### Section 1: Record Learnings In Real-Time

```markdown
### Record Learnings In Real-Time (Mulch)
When you discover any of these during a session, IMMEDIATELY run `mulch record` right then — do NOT wait until session end:
- **A workaround** for a bug or platform quirk (type: failure, requires --resolution flag)
- **An API field mapping** that wasn't obvious (type: pattern)
- **A deploy/infra command sequence** that works (type: pattern)
- **A tool/library that works well** for a specific task (type: pattern)
- **A convention** you established or discovered (type: convention)
- **An actor/service/endpoint that is broken or deprecated** (type: failure)

Format: `cd <USER_REPO> && mulch record <domain> --type <type> --name "short name" --description "specific problem + specific solution"`
For failure type, also add: `--resolution "what to do instead"`

Records must be specific enough that a future Claude session can act on them without context.
Bad: "Fixed the SSH thing." Good: "SSH heredocs mangle Python f-strings. Write locally then SCP to server."
```

### Section 2: Expertise & Issue Tracking Commands

```markdown
### Expertise & Issue Tracking (Mulch + Seeds)

**Mulch — Expertise tracking:**
- `mulch prime` — Load all expertise (happens automatically on session start)
- `mulch search "query"` — Search expertise records
- `mulch status` — Show domains and record counts
- `mulch record <domain> --type <type> --name "..." --description "..."` — Save a learning
  - Types: convention, pattern, failure (needs --resolution), decision, reference, guide
- `mulch learn` — See what files changed (suggests what to record)
- `mulch sync` — Validate + commit .mulch/ to git

**Seeds — Issue tracking:**
- `sd create --title "..." --type task` — Create an issue
- `sd list` — View all issues
- `sd ready` — Find unblocked work items
- `sd close <id>` — Mark done
- `sd sync` — Commit .seeds/ to git
```

Replace `<USER_REPO>` with the user's actual project directory.

---

## Phase 7: Verify & Explain

### Verification commands

```bash
# Verify Mulch is working
cd <USER_REPO> && mulch status

# Verify Seeds is working (if installed)
cd <USER_REPO> && sd list

# Verify hooks exist and are executable
ls -la ~/.claude/hooks/

# Verify settings.json has hooks configured
cat ~/.claude/settings.json | python3 -c "import sys,json; d=json.load(sys.stdin); print('Hooks configured:', list(d.get('hooks',{}).keys()))"
```

### What to tell the user after setup

> **Setup complete. Here's what happens now:**
>
> 1. **Restart Claude Code** — the settings.json changes need a restart to take effect
>
> 2. **Next session**: When you open Claude Code, you'll see expertise records loaded automatically at the top. Right now there's just the starter records, but this grows over time.
>
> 3. **During builds**: When I discover something useful — a bug fix, a working command, a tool quirk — I'll save it immediately using `mulch record`. You don't need to do anything.
>
> 4. **Session end**: A hook checks if files changed and reminds me to record anything I learned.
>
> 5. **Over time**: Every session starts smarter. After a few weeks, I'll know your deploy process, your API quirks, your naming conventions, your common bugs — all before you type a word.
>
> **To see your expertise anytime:** `mulch status`
> **To search for something specific:** `mulch search "deploy"`
> **To see all records:** `mulch prime`

---

## How It Works (The Session Lifecycle)

```
SESSION START
  │
  ├─→ mulch prime fires automatically
  │   All past expertise loaded into context
  │   "I already know SSH heredocs break f-strings"
  │
  ├─→ Checkpoint recovered (if resuming from compaction)
  │
  ▼
BUILD / WORK
  │
  ├─→ Claude discovers a bug workaround
  │   → Immediately runs: mulch record backend --type failure ...
  │
  ├─→ Claude finds a working deploy command
  │   → Immediately runs: mulch record deployment --type pattern ...
  │
  ▼
SESSION END
  │
  ├─→ Stop hook fires: mulch learn
  │   "These files changed. Did you record what you learned?"
  │
  ├─→ mulch sync → commits expertise to git
  │
  ▼
NEXT SESSION
  │
  └─→ mulch prime fires → ALL learnings available
      Claude never re-learns the same lesson
```

---

## Platform-Specific Notes

### Windows MINGW / Git Bash
- Lock file creation can fail. Before running `mulch record`, you may need: `touch .mulch/expertise/<domain>.jsonl.lock`
- Hook scripts should work but test with `bash ~/.claude/hooks/safety-check.sh < /dev/null` to verify

### macOS
- If using zsh (default), add Bun to `~/.zshrc` instead of `~/.bashrc`
- Everything else works natively

### WSL2
- Full support. This setup was built and tested on WSL2.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `mulch: command not found` | Add Bun to PATH: `export PATH="$HOME/.bun/bin:$PATH"` (add to ~/.bashrc) |
| `sd: command not found` | Same fix as above |
| Hooks not firing | Restart Claude Code (settings.json changes need restart) |
| `mulch record` fails with lock error | Run `touch .mulch/expertise/<domain>.jsonl.lock` first (Windows only) |
| Safety hook blocking valid commands | Edit `~/.claude/hooks/safety-check.sh` — remove the pattern from BLOCKED_PATTERNS |
| Checkpoint not recovering after compaction | Verify `chmod +x ~/.claude/hooks/session-recovery.sh` and check SessionStart matcher is "compact" |
| `mulch init` fails | Must be inside a git repository. Run `git init` first if needed. |

---

*Setup by @jayydoesai — Self-Evolving Claude Code System*
*Tools: Mulch (os-eco) by Nick Arellano*
