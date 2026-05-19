---
name: codex
description: Review code changes, implementation plans, or specific files using OpenAI Codex CLI. Triggers on "review", "check", "validate", "codex". Perfect for maker-checker workflows where you want a second AI agent to review Claude's work before committing. Requires the Codex CLI to be installed (brew install codex-cli or from https://codex.storage.googleapis.com).
allowed-tools:
  - Bash
  - Read
---

# Codex Code Review Skill

**Version:** 1.1.0
**Author:** Community-contributed
**Requires:** [Codex CLI](https://codex.storage.googleapis.com) installed and configured

## Overview

Use the Codex CLI to perform independent code reviews on changes made during your Claude Code session. This enables a "proposer-checker-maker-checker" workflow where Claude Code proposes/implements changes and Codex provides an independent review.

**Why use this skill instead of the MCP?** The `mcp__multi-llm__code_review` tool has hard timeout limitations that fail on large codebases. Running via Bash allows:
- No timeout constraints (configurable up to 10 min per call)
- Background execution with `run_in_background: true` for very large reviews
- Progress checking via `TaskOutput`

## Pair-Programming Flow (Codex thinks first, every time)

Per global CLAUDE.md the canonical loop is: **(1) Codex analyzes → (2) you implement the plan → (3) Codex reviews the diff → (4) fix until approved**. This applies to every code change regardless of size.

### Step 1 input bundle (NON-NEGOTIABLE - before any prompt reaches Codex)

For any bug-fix or AC-driven change, the input bundle sent to Codex MUST contain BOTH the bug ticket AND the parent user story, plus a labeled discrepancy framing paragraph. Codex without the parent user story will frequently propose a symptom fix; with the user story it proposes a contract-conformance fix. The 30-second cost of the bundle saves a rework cycle.

Before sending a problem to Codex:

1. **Pull the bug ticket** from ADO (or the equivalent source-system record) using `ADO_PAT_NEW` from `devops/.env.local` with `?$expand=relations`. Capture `System.Description`, `Microsoft.VSTS.TCM.ReproSteps`, comments, attachments.
2. **Pull the parent user story.** From the bug ticket's `relations` array, find `System.LinkTypes.Hierarchy-Reverse`, parse the work-item id off the URL, fetch that work item. Read `System.Description`, `Microsoft.VSTS.Common.AcceptanceCriteria` (Acceptance Criteria), `Microsoft.VSTS.TCM.ReproSteps`, attachments IN FULL.
3. **Write a 4-line discrepancy framing block** and paste it at the top of the Codex prompt:

   ```
   USER STORY: <verbatim AcceptanceCriteria>
   BUG: <verbatim repro / symptom>
   CURRENT CODE/DB: <observed behavior, with file:line citations>
   DISCREPANCY: <where the gap is - what the user story requires that current code does not deliver>
   ```

4. **Codex receives all four** (user story, bug, current state, discrepancy) as input, not just the bug ticket alone.

If the bug has no `Hierarchy-Reverse` parent, STOP and surface to the user. Do not invent a parent. Do not send Codex a bug-ticket-only prompt and hope. See memory [[feedback_pull_parent_user_story_first]] for the rule and incident pattern (it is the bug-ticket-to-user-story extension of [[feedback_source_of_truth_before_replication_brief]] and [[feedback_access_hub_source_of_truth_hierarchy]]).

### Step 3 review input

When the diff is ready, paste `git diff` PLUS the same 4-line discrepancy framing block from Step 1 into the Codex review prompt. Codex's review checks the diff against the user story AC, not just the symptom in the bug ticket.

### CLI fallback (when MCP is unavailable)

The same 4-line framing mandate applies to the CLI fallback (`codex exec --sandbox read-only --full-auto "<prompt>"`). The prompt the CLI receives MUST include the bug + parent user story + 4-line USER STORY / BUG / CURRENT CODE / DISCREPANCY block. A CLI-fallback prompt missing the parent user story is the same shallow-context failure mode as an MCP prompt missing it - the surface is different, the failure is identical.

**Acceptance check before sending anything to Codex (MCP or CLI):**

- [ ] Bug ticket pulled with `$expand=relations`?
- [ ] Parent user story fetched via `System.LinkTypes.Hierarchy-Reverse`?
- [ ] Verbatim `AcceptanceCriteria` text in the prompt?
- [ ] 4-line USER STORY / BUG / CURRENT CODE / DISCREPANCY block at the top of the prompt?
- [ ] Patch acceptance bar stated as "satisfies the parent AC", not "closes the bug symptom"?

If any check fails, regenerate the prompt before sending.

## Prerequisites

Install the Codex CLI:
```bash
brew install codex-cli
# OR download from https://codex.storage.googleapis.com
```

Configure your OpenAI API key:
```bash
codex login
```

## Use the Codex CLI to review recent changes, implementation plans, or specific files.

## For Large Codebases

When reviewing large changes (50+ files, major refactors), run in background mode:

```bash
# Use run_in_background: true in the Bash tool
codex review "Comprehensive review of all changes since v2.0"
```

Then check progress with `TaskOutput` tool using the returned task_id.

## Review Recent Git Changes

```bash
# Review uncommitted changes
codex review "Review the uncommitted changes in this repo"

# Review specific commit
codex review "Review the changes in commit <hash>"

# Review changes between branches
codex review "Review the diff between main and current branch"
```

## Review Specific Files

```bash
# Review a specific file
codex review "Review the code in <file_path>"

# Review multiple files
codex review "Review the implementation in <file1> and <file2>"
```

## Review Implementation Plan

```bash
# Review a plan before implementation
codex review "Review this implementation plan: <plan_content>"
```

## Usage Pattern

1. Identify what needs review (recent changes, specific files, or a plan)
2. Run `codex review` with appropriate prompt
3. Present the review results to the user

The `codex review` command runs non-interactively and returns feedback that can be incorporated into the Claude Code session.

## Workflow Example

```
User: I just implemented user authentication
Claude: [implements auth.ts, auth-middleware.ts, etc.]
User: /codex
Claude: [runs: codex review --uncommitted]
Claude: Codex found 2 issues:
  1. Missing error handling in auth.ts:45
  2. Password hash strength could be improved
User: Fix those issues
Claude: [fixes issues]
User: /codex
Claude: [runs review again]
Claude: Codex approves! No issues found.
```

## Notes

- Works best in git repositories
- Review results are shown inline in the Claude Code session
- Combines well with Claude's implementation capabilities for a robust development workflow
