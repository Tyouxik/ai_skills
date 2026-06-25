## ADDED Requirements

### Requirement: Export OpenSpec change to GitHub issue
The system SHALL create a structured GitHub issue from an OpenSpec change's proposal, design, and tasks artifacts using the `gh` CLI.

#### Scenario: New issue created from change
- **WHEN** the propose stage runs and no existing issue is linked to the change
- **THEN** the system creates a GitHub issue with:
  - Title derived from the proposal's "Why" section
  - Body containing sections: `## Why`, `## What Changes`, `## Tasks`, `## Impact Areas`, `## Design Notes`
  - Labels applied from the `spec:<capability-name>` convention for each capability listed in the proposal
  - And stores the issue number via `openspec set github.issue <N>`

#### Scenario: Existing issue updated from change
- **WHEN** the propose stage runs and an issue is already linked to the change
- **THEN** the system updates the existing issue body to reflect current proposal/design/tasks content
- **AND** preserves the existing issue number and does not create a duplicate

#### Scenario: Issue body populated from tasks.md
- **WHEN** tasks.md contains a markdown checklist with `- [ ]` items
- **THEN** the issue body SHALL include a `## Tasks` section with the same checklist format
- **AND** each task item is preserved as a GitHub-flavored markdown checkbox

#### Scenario: Labels derived from capabilities
- **WHEN** the proposal lists capabilities such as `github-issue-export` and `github-branch-pr`
- **THEN** the issue SHALL receive labels `spec:github-issue-export` and `spec:github-branch-pr`

### Requirement: Auto-detect the GitHub repository
The system SHALL detect the GitHub repository from `git remote get-url origin` with user confirmation before creating any issue.

#### Scenario: Repo auto-detected with confirmation
- **WHEN** the propose stage initiates issue creation
- **THEN** the system extracts `owner/repo` from the git remote URL
- **AND** confirms with the user: "Create issue in owner/repo?"
- **AND** proceeds only after confirmation

#### Scenario: No git remote found
- **WHEN** no `origin` remote is configured
- **THEN** the system SHALL prompt the user to specify `owner/repo` manually
