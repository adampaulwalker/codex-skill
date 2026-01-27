# Codex Review Skill for Claude Code

A Claude Code skill that integrates OpenAI's Codex CLI for independent code reviews. Perfect for "proposer-checker-maker-checker" workflows where Claude Code implements changes and Codex provides a second opinion.

## Installation

### Prerequisites

1. **Install Codex CLI:**
   ```bash
   brew install codex-cli
   # OR download from https://codex.storage.googleapis.com
   ```

2. **Configure your OpenAI API key:**
   ```bash
   codex login
   ```

### Install the Skill

**Option 1: Direct Download**
```bash
mkdir -p ~/.claude/skills/codex
curl -o ~/.claude/skills/codex/SKILL.md https://raw.githubusercontent.com/YOUR-USERNAME/YOUR-REPO/main/SKILL.md
```

**Option 2: Manual Installation**
1. Create directory: `mkdir -p ~/.claude/skills/codex`
2. Copy `SKILL.md` to `~/.claude/skills/codex/SKILL.md`
3. Restart Claude Code

## Usage

Once installed, invoke the skill in any Claude Code session:

```
/codex
```

Or ask Claude to use it naturally:
```
Can you have Codex review my changes?
```

## Workflow Example

```
You: I just implemented user authentication
Claude: [implements auth code]
You: /codex
Claude: [runs Codex review, shows results]
Claude: Codex found 2 issues: ...
You: Fix those
Claude: [fixes issues]
You: /codex again
Claude: Codex approves!
```

## Features

- ✅ Review uncommitted changes
- ✅ Review specific commits
- ✅ Review specific files or directories
- ✅ Review implementation plans before coding
- ✅ Seamless integration - stay in one terminal tab
- ✅ No copy/paste between tools

## Requirements

- Claude Code CLI
- Codex CLI (installed and configured)
- Git repository (for most review operations)

## Contributing

Found a bug or have a suggestion? Please open an issue or submit a pull request!

## License

MIT License - Feel free to use, modify, and share!

## Version

1.0.0

---

**Tip:** This skill works great in combination with Claude's code implementation capabilities. Let Claude write the code, then use `/codex` for an independent review before committing.
