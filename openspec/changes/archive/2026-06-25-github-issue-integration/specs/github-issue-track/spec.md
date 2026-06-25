## ADDED Requirements

### Requirement: Sync task progress to issue checklist
The system SHALL update the linked GitHub issue body to reflect the current task completion state from tasks.md.

#### Scenario: Tasks synced from tasks.md to issue body
- **WHEN** the apply stage completes a task and marks it `[x]` in tasks.md
- **THEN** the system SHALL regenerate the issue body's `## Tasks` section to match the current tasks.md checklist
- **AND** update the issue via `gh issue edit <N> --body "<updated body>"`

#### Scenario: New tasks added after issue creation
- **WHEN** new tasks are added to tasks.md that did not exist in the original issue body
- **THEN** the system SHALL append the new tasks to the issue body's `## Tasks` section

#### Scenario: Issue body preserves non-checklist sections
- **WHEN** updating the issue body to sync tasks
- **THEN** the system SHALL preserve all other sections (`## Why`, `## What Changes`, etc.)
- **AND** only modify the `## Tasks` section

### Requirement: Update issue on milestone events
The system SHALL add a comment to the linked issue when significant milestones are reached.

#### Scenario: Comment on implementation start
- **WHEN** the apply stage begins implementation
- **THEN** the system SHALL comment on the linked issue: "🚧 Implementation started — [PR #X](link)"

#### Scenario: Comment on implementation completion
- **WHEN** all tasks in tasks.md are marked complete
- **THEN** the system SHALL comment on the linked issue: "✅ Implementation complete — see linked PR"

#### Scenario: No comment for individual task completion
- **WHEN** a single task is completed (but others remain)
- **THEN** the system SHALL NOT add a comment (only the body checklist is updated)
