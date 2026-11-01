## ADDED Requirements

### Requirement: Select a translation backend
The plugin SHALL offer `api` and `codex-cli` backends, defaulting to `api`, and a configurable Codex executable path defaulting to `codex`.

#### Scenario: Existing installation
- **WHEN** the backend is not configured
- **THEN** translation continues through the existing API backend

### Requirement: Configure during installation or later
The plugin SHALL expose its backend, target language, and backend-specific settings through native `userConfig`, usable in normal installation configuration and later through `/plugin configure` or `/config`, without requiring a custom launch command or source edits.

#### Scenario: Choose Codex during installation
- **WHEN** the user installs with `--config backend=codex-cli` on a host supporting installation config
- **THEN** the backend choice is saved and no translation API key is required

#### Scenario: Change the backend later
- **WHEN** the user changes the backend through the native configuration interface
- **THEN** subsequent activation uses that backend and previously saved fields of the other backend remain available

### Requirement: Translate through local Codex
The CLI backend SHALL invoke the executable without a shell, send protected text through stdin, consume only final-message stdout, and reuse saved CLI authentication without reading the API backend's key or making an HTTP translation request.

#### Scenario: Successful translation
- **WHEN** Codex exits successfully with a nonempty complete translation
- **THEN** the plugin restores protected fragments, preserves submission origin, and submits the translation with target-language reply context

#### Scenario: Custom executable
- **WHEN** a custom executable path contains spaces
- **THEN** it is passed as a single argv entry and input text is not interpolated into a command

### Requirement: Stop on CLI errors
The CLI backend SHALL stop the submission on startup failure, timeout, nonzero exit, cancellation, empty output, truncated stdout, or invalid protected markers, without falling back to the API or original text.

#### Scenario: Process failure
- **WHEN** Codex cannot complete translation
- **THEN** the prompt is dropped with an actionable error and no API request is made

#### Scenario: Invalid markers
- **WHEN** Codex loses or duplicates a protected marker
- **THEN** the prompt is not submitted

### Requirement: Preserve command bypass and feedback
The CLI backend SHALL preserve existing command/empty-input bypass and display Codex CLI as the selected translator in interactive progress feedback.

#### Scenario: Command submission
- **WHEN** the user submits a bypassed command
- **THEN** neither Codex nor the API translator is invoked
