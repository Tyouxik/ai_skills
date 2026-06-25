## ADDED Requirements

### Requirement: Verify PR merge status before archiving
The system SHALL check whether the associated PR has been merged before closing the issue and archiving the change.

#### Scenario: PR merged — proceed with archive
- **WHEN** the archive stage runs and `gh pr view --json state` returns `"MERGED"`
- **THEN** the system SHALL close the linked issue via `gh issue close <N>`
- **AND** add a closing comment: "✅ Merged in PR #X. Change archived."
- **AND** proceed with the standard OpenSpec archive flow

#### Scenario: PR not merged — warn user
- **WHEN** the archive stage runs and the PR state is `"OPEN"` or `"CLOSED"` (not merged)
- **THEN** the system SHALL display a warning: "PR #X has not been merged. Close the issue and archive anyway?"
- **AND** await user confirmation before proceeding

#### Scenario: No PR found
- **WHEN** the archive stage runs and no PR is associated with the issue
- **THEN** the system SHALL display a warning: "No PR found for issue #N. Close the issue and archive anyway?"
- **AND** await user confirmation before proceeding

#### Scenario: No issue linked
- **WHEN** the archive stage runs and no `github.issue` is set in the change metadata
- **THEN** the system SHALL proceed with the standard OpenSpec archive flow without any GitHub operations

### Requirement: Close issue with merge reference
The system SHALL close the linked issue with a comment referencing the merged PR.

#### Scenario: Issue closed with PR reference
- **WHEN** the PR is merged and archive proceeds
- **THEN** the system SHALL close the issue and add a comment: "✅ Complete. Implementation merged in PR #X and change archived."
- **AND** GitHub SHALL auto-close the issue if `Closes #<N>` was in the PR body
