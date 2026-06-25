## ADDED Requirements

### Requirement: Pull GitHub issue for exploration context
The system SHALL pull a GitHub issue's content for exploration without creating any OpenSpec artifacts.

#### Scenario: Issue content loaded for explore stage
- **WHEN** the explore stage is invoked with a GitHub issue number
- **THEN** the system fetches the issue via `gh issue view <N> --json title,body,labels,comments`
- **AND** presents the parsed content as exploration context
- **AND** does NOT create any files, changes, or OpenSpec artifacts

#### Scenario: Issue not found
- **WHEN** the specified issue number does not exist in the detected repository
- **THEN** the system SHALL report the error and prompt for a valid issue number or URL

### Requirement: Import GitHub issue into OpenSpec change
The system SHALL create an OpenSpec change from a GitHub issue by parsing its body into proposal and tasks artifacts.

#### Scenario: Change created from well-structured issue
- **WHEN** an issue body contains sections `## Why`, `## What Changes`, `## Tasks`
- **THEN** the system derives a kebab-case change name from the issue title
- **AND** creates the OpenSpec change via `openspec new change "<name>"`
- **AND** writes proposal.md with "Why" and "What Changes" from the issue body
- **AND** writes tasks.md with the checklist from the "Tasks" section
- **AND** stores the issue link via `openspec set github.issue <N>`

#### Scenario: Change created from unstructured issue
- **WHEN** an issue body does not contain OpenSpec-format sections
- **THEN** the system treats the entire issue body as the "Why" section of proposal.md
- **AND** creates a single task: "Implement as described in issue #<N>"
- **AND** marks the proposal for refinement

#### Scenario: Change name derived from issue title
- **WHEN** an issue is titled "Add pantry ingredient catalog and search"
- **THEN** the system SHALL derive the change name `pantry-ingredient-catalog-and-search`
- **AND** update the issue description to reference the change name

#### Scenario: Name collision with existing change
- **WHEN** a change with the derived name already exists
- **THEN** the system SHALL append `-2` (incrementing) until a unique name is found
- **AND** report the disambiguation to the user

### Requirement: Read labels as capability mapping
The system SHALL interpret `spec:<name>` labels on an imported issue as OpenSpec capabilities.

#### Scenario: Labels mapped to capabilities
- **WHEN** an issue has labels `spec:github-issue-export` and `spec:github-branch-pr`
- **THEN** the system SHALL list `github-issue-export` and `github-branch-pr` in the proposal's Capabilities section
