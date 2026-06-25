---
name: openspec-github
description: Bridges OpenSpec changes and GitHub Issues throughout the spec-driven workflow. Provides GitHub operations at each OpenSpec stage — explore (pull issue content), propose (create/link issues with labels), apply (branch + PR + progress sync), archive (verify merge + close issue). Use whenever the user mentions GitHub issues in connection with OpenSpec changes, wants to create/link/sync issues, or needs branch/PR creation during implementation. Triggers on phrases like "create a GitHub issue", "link to issue", "pull issue #N", "sync the issue", "update issue progress", "create a PR", or any workflow that combines GitHub with OpenSpec.
compatibility: Requires gh CLI, git, and openspec CLI.
metadata:
  author: openspec
  version: "1.0"
---

# OpenSpec GitHub Integration

Bridges OpenSpec changes with GitHub Issues and Pull Requests across the full spec-driven workflow. This skill augments each OpenSpec stage with GitHub operations.

## Prerequisites

Before any GitHub operation, verify the environment:

```bash
# Check gh CLI is installed and authenticated
gh auth status

# Detect the GitHub repository from git remote
gh repo set-default $(git remote get-url origin | sed 's|.*[:/]\([^/]*/[^/]*\)\.git|\1|')
```

If `gh` is not authenticated, instruct the user to run `gh auth login` and stop.

If no git remote is found, prompt: "No GitHub remote detected. Which repository should I use? (owner/repo)"

## Setup: Link Storage

The change↔issue relationship is stored in the change's `.openspec.yaml` via:

```bash
openspec set github.issue <N> --change "<change-name>"
openspec set github.repo <owner/repo> --change "<change-name>"
```

Read the link at any time:
```bash
openspec status --change "<change-name>" --json | jq -r '.metadata.github'
```

This is the SOURCE OF TRUTH for the issue→change mapping. Always read from here, never assume.

---

## Stage: Explore

**When to use:** The explore stage needs context from a GitHub issue. Pull issue content for investigation — read-only, no artifacts created.

### Workflow

1. **Get the issue number** from the user or conversation context. If ambiguous, ask.

2. **Fetch the issue:**
   ```bash
   gh issue view <N> --json title,body,labels,comments
   ```
   Parse the JSON output to extract:
   - `title` — issue title
   - `body` — issue description (markdown)
   - `labels` — array of label names
   - `comments` — array of comment objects (body, author, createdAt)

3. **Parse the issue body** for structured sections:
   - Look for `## Why`, `## What Changes`, `## Tasks`, `## Impact Areas`, `## Design Notes`
   - Each `##` section maps to a potential OpenSpec artifact area
   - Extract checklist items from `## Tasks` section (lines matching `- [ ]` or `- [x]`)

4. **Fallback for unstructured issues:** If no `##` sections are detected, treat the entire body as exploration context. Don't force structure — present it as-is.

5. **Interpret labels:** Labels matching `spec:<name>` indicate capabilities. Note them for later use if the issue becomes a change.

6. **Present findings** as exploration context — synthesized understanding, not a decision. Do NOT create any files, changes, or artifacts.

### Error Handling

| Condition | Action |
|---|---|
| Issue not found (404) | Report: "Issue #N not found in [repo]. Check the issue number?" |
| No repo detected | Prompt for `owner/repo` |
| `gh` not authenticated | Instruct: "Run `gh auth login` first" |
| Empty issue body | Report: "Issue #N has no description. Working with title only." |

---

## Stage: Propose

Two scenarios: **Export** (create/update an issue from a change) and **Import** (create a change from an issue).

### Propose A: Export (Change → Issue)

**When to use:** An OpenSpec change exists (or is being created) and needs a GitHub issue.

#### Workflow

1. **Select the change.** If creating a new change, complete the standard OpenSpec propose flow first so artifacts exist. If working with an existing change, confirm which one.

2. **Check for existing issue link:**
   ```bash
   openspec status --change "<name>" --json
   ```
   If `github.issue` is already set, this is an UPDATE scenario (see step 5).

3. **Read the change artifacts:**
   - `proposal.md` — "Why" and "What Changes" sections
   - `design.md` — "Decisions" section
   - `tasks.md` — checklist items (`- [ ]` and `- [x]` lines)
   - `specs/**/*.md` — capability names from directory names and requirements

4. **Build the structured issue body** using this exact template:

   ```markdown
   ## Why
   [From proposal.md "Why" section]

   ## What Changes
   [From proposal.md "What Changes" section, as bullet list]

   ## Tasks
   [From tasks.md, preserving checkbox format]

   ## Impact Areas
   [From proposal.md "Impact" section, as bullet list]

   ## Design Notes
   [From design.md "Decisions" section, summarized — include key rationale]
   ```

   Keep the body concise but complete. Each section maps to the corresponding artifact.

5. **Determine labels** from capabilities:
   - Read capabilities from `proposal.md` "Capabilities" section
   - Convert each capability name to label: `spec:<capability-name>`
   - Example: `github-issue-export` → `spec:github-issue-export`

6. **Create or update the issue:**

   **New issue:**
   ```bash
   gh issue create \
     --title "<human-readable title from proposal Why>" \
     --body "$(cat /tmp/issue-body.md)" \
     --label "spec:capability-1,spec:capability-2" \
     --repo <owner/repo>
   ```

   **Update existing issue** (if `github.issue` already set):
   ```bash
   gh issue edit <N> --body "$(cat /tmp/issue-body.md)" --repo <owner/repo>
   ```

7. **Store the link:**
   ```bash
   openspec set github.issue <N> --change "<change-name>"
   openspec set github.repo <owner/repo> --change "<change-name>"
   ```

8. **Report:** "Created issue #N: [URL]. Change `change-name` is now linked."

### Propose B: Import (Issue → Change)

**When to use:** A GitHub issue exists and should become an OpenSpec change.

#### Workflow

1. **Get the issue number** from the user.

2. **Fetch the issue:**
   ```bash
   gh issue view <N> --json title,body,labels
   ```

3. **Derive a change name** from the issue title:
   - Lowercase the title
   - Replace spaces and special characters with hyphens
   - Remove leading/trailing hyphens
   - Example: "Add pantry ingredient catalog" → `pantry-ingredient-catalog`

4. **Check for name collision:**
   ```bash
   openspec list --json
   ```
   If a change with that name exists, append `-2` (incrementing until unique). Report the disambiguation to the user.

5. **Create the OpenSpec change:**
   ```bash
   openspec new change "<derived-name>"
   ```

6. **Parse the issue body** into artifacts:

   **proposal.md** — Map issue sections:
   - `## Why` → `## Why` section
   - `## What Changes` → `## What Changes` section
   - `## Impact Areas` → `## Impact` section
   - Labels matching `spec:<name>` → `## Capabilities` section

   **Fallback for unstructured issues:** If no sections found, write the entire body as the "Why" section and add:
   ```
   ## Capabilities
   <!-- Derived from issue #N — refine as needed -->
   - `<derived-from-title>`: [Issue description]
   ```

   **tasks.md** — Extract from `## Tasks` checklist. If no checklist exists:
   ```markdown
   ## 1. Implementation
   - [ ] 1.1 Implement as described in issue #<N>
   ```

   **design.md** — If `## Design Notes` section exists, create with that content.

7. **Store the link:**
   ```bash
   openspec set github.issue <N> --change "<derived-name>"
   openspec set github.repo <owner/repo> --change "<derived-name>"
   ```

8. **Update the issue** description to reference the change: append the change name so it's visible in the issue body (without disrupting the structured format).

9. **Report:** "Created OpenSpec change `derived-name` from issue #N. Artifacts populated. Review and refine proposal.md."

---

## Stage: Apply — Branch and PR

**When to use:** The apply stage begins implementation. ALWAYS create a branch and PR — no exceptions, no prompts, no skip option.

### Workflow

1. **Read the linked issue:**
   ```bash
   openspec status --change "<change-name>" --json
   ```
   Extract `github.issue` and `github.repo`. If no issue is linked, proceed with PR creation but omit the `Closes #N` reference.

2. **Create the branch:**
   ```bash
   git checkout -b "<change-name>"
   ```
   If the branch already exists locally, append a suffix:
   ```bash
   # Check for collision
   git branch --list "<change-name>"
   # If exists, use: <change-name>-2, -3, etc.
   git checkout -b "<change-name>-2"
   ```
   Report the final branch name.

3. **Push the branch:**
   ```bash
   git push -u origin "<branch-name>"
   ```

4. **Create the pull request:**
   ```bash
   gh pr create \
     --title "<human-readable title from proposal>" \
     --body "Closes #<N>" \
     --base main \
     --head "<branch-name>"
   ```
   If no linked issue, omit `Closes #<N>` from the body.

5. **Report:** "Branch `<branch-name>` created. PR #X: [URL]"

### Error Handling

| Condition | Action |
|---|---|
| Branch name collision | Append `-2`, `-3`; report disambiguation |
| Push rejected (no permission) | Report: "Cannot push to [repo]. Check your permissions." |
| `gh pr create` fails | Report the error; ask if user wants to create PR manually |
| No `main` branch (different default) | Detect default branch via `git remote show origin` |
| `gh` not authenticated | Instruct: "Run `gh auth login` first" |

---

## Stage: Apply — Progress Sync

**When to use:** During implementation, keep the linked issue in sync with task progress.

### Workflow

1. **Read the linked issue** from change metadata.

2. **Read current tasks.md** — extract all checkbox lines (`- [ ]` and `- [x]`).

3. **Fetch current issue body:**
   ```bash
   gh issue view <N> --json body
   ```

4. **Regenerate the `## Tasks` section** in the issue body:
   - Replace the existing `## Tasks` section with the current checklist from tasks.md
   - Preserve ALL other sections exactly as-is (`## Why`, `## What Changes`, etc.)
   - If a `## Tasks` section doesn't exist, append it after `## What Changes`

5. **Update the issue:**
   ```bash
   gh issue edit <N> --body "$(cat /tmp/updated-body.md)"
   ```

6. **Milestone comments** — add a comment ONLY for major transitions:
   - **Implementation start** (first task marked `in_progress` or `[x]`):
     ```
     🚧 Implementation started — PR #[N](link)
     ```
   - **Implementation complete** (all tasks `[x]`):
     ```
     ✅ Implementation complete — see linked PR
     ```
   - Do NOT comment for individual task completions — body checklist update is sufficient.

### Comment creation:
```bash
gh issue comment <N> --body "🚧 Implementation started — PR #[N](link)"
```

---

## Stage: Archive

**When to use:** The change is complete and ready to archive. Verify GitHub state before finalizing.

### Workflow

1. **Read the linked issue** from change metadata. If no `github.issue` is set, proceed with standard OpenSpec archive — no GitHub operations needed.

2. **Check PR status.** Find the PR associated with the issue:
   ```bash
   gh pr list --search "<issue-number>" --json number,state,mergedAt,url
   ```
   If multiple PRs are found, use the most recent one.

3. **Handle merge status:**

   **PR merged:**
   - Close the issue:
     ```bash
     gh issue close <N> --comment "✅ Complete. Implementation merged in PR #[X](link) and change archived."
     ```
   - Proceed with standard OpenSpec archive:
     ```bash
     openspec archive "<change-name>"
     ```

   **PR not merged** (state is `OPEN`):
   - Display warning: "PR #[X] is still open and has not been merged."
   - Ask: "Close the issue and archive anyway?"
   - If user confirms → close issue, archive change.
   - If user declines → stop. Do not archive.

   **PR closed without merge:**
   - Display warning: "PR #[X] was closed without merging."
   - Ask: "Close the linked issue and archive anyway?"
   - If user confirms → close issue, archive change.
   - If user declines → stop. Do not archive.

   **No PR found:**
   - Display warning: "No PR found associated with issue #N."
   - Ask: "Close the issue and archive anyway?"
   - If user confirms → close issue, archive change.
   - If user declines → stop. Do not archive.

### Guardrail

Never archive a change without verifying GitHub state when an issue is linked. The archive is the final step — it must be intentional and verified.

---

## Labels Convention

Map OpenSpec capabilities to GitHub labels using the `spec:<capability-name>` format:

| Direction | Mapping |
|---|---|
| Capability → Label | `github-issue-export` → `spec:github-issue-export` |
| Label → Capability | `spec:github-issue-export` → `github-issue-export` |

Labels are applied during Propose (Export) and read during Explore/Propose (Import). The `spec:` prefix avoids namespace collisions with other label systems.

---

## Error Message Templates

| Error | Message |
|---|---|
| No `gh` CLI | "GitHub CLI (`gh`) is not installed. Install it from https://cli.github.com and run `gh auth login`." |
| Not authenticated | "Not authenticated with GitHub. Run `gh auth login` and try again." |
| No git remote | "No GitHub remote detected in this repository. Which repo should I use? (owner/repo)" |
| Issue not found | "Issue #N not found in [repo]. Verify the issue number and repository." |
| No linked issue | "No GitHub issue is linked to this change. [Stage-specific guidance]." |
| `openspec set` failed | "Could not store link metadata. The skill requires openspec CLI v1.4+. Update: `npm update -g @fission-ai/openspec`." |
