## 1. Skill Skeleton and Setup

- [x] 1.1 Create `.opencode/skills/openspec-github/` directory structure with SKILL.md and `references/` subdirectory
- [x] 1.2 Write SKILL.md frontmatter (name: `openspec-github`, description covering all trigger contexts)
- [x] 1.3 Write SKILL.md Setup section: `gh` CLI check (`gh auth status`), repo detection from `git remote`, link storage via `openspec set`
- [x] 1.4 Write `references/issue-template.md` with the canonical structured issue format (`## Why`, `## What Changes`, `## Tasks`, `## Impact Areas`, `## Design Notes`)

## 2. Explore Stage — Issue Import (Read-Only)

- [x] 2.1 Write SKILL.md Explore section: fetch issue via `gh issue view <N> --json title,body,labels,comments`
- [x] 2.2 Implement issue body parsing logic: detect `##` sections, extract content for exploration context
- [x] 2.3 Handle unstructured issues: treat full body as "Why" context without forcing structure
- [x] 2.4 Handle error cases: issue not found, unauthenticated, no repo detected

## 3. Propose Stage — Export (Change → Issue)

- [x] 3.1 Write SKILL.md Propose/Export section: read proposal.md, design.md, tasks.md, specs/ from the change directory
- [x] 3.2 Build issue body from artifacts: map proposal sections to issue sections, tasks.md checklist to `## Tasks`
- [x] 3.3 Implement capability → label mapping: read capabilities from proposal, apply `spec:<name>` labels
- [x] 3.4 Create issue via `gh issue create` with title, body, labels; handle `--repo` flag
- [x] 3.5 Store link: `openspec set github.issue <N> --change "<name>"` and `openspec set github.repo <owner/repo> --change "<name>"`
- [x] 3.6 Handle existing issue case: update body via `gh issue edit <N> --body` instead of creating new

## 4. Propose Stage — Import (Issue → Change)

- [x] 4.1 Write SKILL.md Propose/Import section: fetch issue, parse body into sections
- [x] 4.2 Derive kebab-case change name from issue title (strip special chars, lowercase, hyphenate)
- [x] 4.3 Create OpenSpec change: `openspec new change "<name>"`
- [x] 4.4 Write proposal.md from parsed "Why" and "What Changes" sections; fallback for unstructured issues
- [x] 4.5 Write tasks.md from parsed `## Tasks` checklist; create default single task if no checklist found
- [x] 4.6 Update issue body to embed change name reference; store link via `openspec set`
- [x] 4.7 Handle name collision: append `-2`, `-3` as needed; report disambiguation

## 5. Apply Stage — Branch and PR

- [x] 5.1 Write SKILL.md Apply section: read linked issue from `openspec set` metadata
- [x] 5.2 Implement branch creation: `git checkout -b <change-name>`, handle collisions with `-2` suffix, push to origin
- [x] 5.3 Implement PR creation: `gh pr create --title "<human-readable title>" --body "Closes #<N>" --base main`
- [x] 5.4 Handle missing issue link: create PR without `Closes #<N>` reference
- [x] 5.5 Handle auth and error cases: `gh` not authenticated, branch push rejected, PR creation fails

## 6. Apply Stage — Progress Sync

- [x] 6.1 Write SKILL.md Track section: read tasks.md, diff against issue body's `## Tasks` section
- [x] 6.2 Implement issue body update: `gh issue edit <N> --body` with regenerated `## Tasks` checklist
- [x] 6.3 Preserve non-task sections (`## Why`, `## What Changes`, etc.) when updating body
- [x] 6.4 Add milestone comments: "🚧 Implementation started" on first apply, "✅ Implementation complete" on all tasks done
- [x] 6.5 Do NOT comment for individual task completions (only body checklist update)

## 7. Archive Stage — Verify and Close

- [x] 7.1 Write SKILL.md Archive section: read linked issue and PR info
- [x] 7.2 Check PR merge status: `gh pr view --json state,mergedAt`
- [x] 7.3 Merged path: close issue via `gh issue close <N>`, add closing comment with PR ref, proceed with archive
- [x] 7.4 Unmerged path: display warning with merge status, ask for user confirmation before proceeding
- [x] 7.5 No PR path: display warning, ask for user confirmation
- [x] 7.6 No issue path: skip GitHub operations entirely, proceed with standard archive

## 8. Reference Files and Polish

- [x] 8.1 Finalize `references/issue-template.md` with the canonical structured issue format
- [x] 8.2 Add error message templates for common failures (no gh CLI, unauthenticated, repo not detected, issue not found)
- [x] 8.3 Review SKILL.md against design decisions (PR creation always, labels convention, structured format)
- [x] 8.4 Verify skill triggers correctly by testing with sample prompts matching each stage
