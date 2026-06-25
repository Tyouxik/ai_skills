## ADDED Requirements

### Requirement: Always create git branch on apply
The system SHALL create a git branch from the change name before any implementation work begins during the apply stage.

#### Scenario: Branch created from change name
- **WHEN** the apply stage starts for change `pantry-ingredient-catalog`
- **THEN** the system creates a branch named `pantry-ingredient-catalog` via `git checkout -b pantry-ingredient-catalog`
- **AND** pushes the branch to origin

#### Scenario: Branch name collision
- **WHEN** a branch with the change name already exists locally
- **THEN** the system SHALL append a numeric suffix (e.g., `-2`) to create a unique branch name
- **AND** report the modified name to the user

### Requirement: Always create pull request on apply
The system SHALL create a draft GitHub pull request linked to the associated issue every time the apply stage completes initial implementation.

#### Scenario: PR created with issue link
- **WHEN** implementation tasks are completed or the apply stage finishes
- **THEN** the system creates a PR via `gh pr create` with:
  - Title matching the change name (human-readable form)
  - Body containing `Closes #<N>` referencing the linked issue
  - The branch created for this change
- **AND** the PR URL is reported to the user

#### Scenario: No linked issue exists
- **WHEN** no issue is linked to the change (no `github.issue` set)
- **THEN** the system SHALL still create the PR
- **AND** omit the `Closes #<N>` reference from the PR body

#### Scenario: gh CLI not authenticated
- **WHEN** `gh auth status` fails
- **THEN** the system SHALL report the authentication error
- **AND** instruct the user to run `gh auth login` before retrying

### Requirement: Link PR to issue automatically
The system SHALL include the issue reference in the PR body so GitHub automatically links them.

#### Scenario: GitHub auto-links PR to issue
- **WHEN** a PR body contains `Closes #42`
- **THEN** GitHub SHALL automatically link the PR to issue #42
- **AND** closing the PR will close the issue
