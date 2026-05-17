## ADDED Requirements

### Requirement: WSL terminal sessions use the distro user's configured shell
Terax SHALL start an interactive terminal for a WSL workspace using the selected distro user's configured default shell instead of forcing `bash`.

#### Scenario: Distro default shell is zsh
- **WHEN** a user selects a WSL distro whose configured shell is `zsh`
- **THEN** Terax starts the PTY session with `zsh` as the interactive shell

#### Scenario: Distro default shell is fish
- **WHEN** a user selects a WSL distro whose configured shell is `fish`
- **THEN** Terax starts the PTY session with `fish` as the interactive shell

### Requirement: Supported WSL shells preserve user initialization and Terax integration
For supported WSL shells, Terax SHALL start the shell in a mode that preserves the user's expected login and interactive initialization while also enabling Terax cwd and prompt-boundary integration.

#### Scenario: zsh startup files are honored
- **WHEN** a WSL user's configured shell is `zsh` and the user opens a terminal in Terax
- **THEN** the shell session loads the user's normal `zsh` startup flow
- **AND** Terax receives cwd and prompt-boundary markers for that session

#### Scenario: bash startup files are honored
- **WHEN** a WSL user's configured shell is `bash` and the user opens a terminal in Terax
- **THEN** the shell session loads the user's normal `bash` login and interactive startup flow
- **AND** Terax receives cwd and prompt-boundary markers for that session

#### Scenario: fish interactive startup is honored
- **WHEN** a WSL user's configured shell is `fish` and the user opens a terminal in Terax
- **THEN** the shell session loads the user's normal `fish` interactive startup flow
- **AND** Terax receives cwd and prompt-boundary markers for that session

### Requirement: WSL shell fallback remains usable
If Terax cannot apply shell-specific integration for the resolved WSL shell, it SHALL still open a usable terminal session without silently replacing a resolved user shell. If the user's shell cannot be resolved at all, Terax SHALL fall back to a safe default shell.

#### Scenario: Unsupported shell opens without integration
- **WHEN** the resolved WSL shell is not one of Terax's supported integrated shells
- **THEN** Terax opens the terminal using that resolved shell
- **AND** the session remains usable even if cwd or prompt-boundary integration is unavailable

#### Scenario: Shell resolution fails
- **WHEN** Terax cannot resolve the WSL user's configured shell
- **THEN** Terax falls back to a safe default shell
- **AND** Terax logs that fallback path for debugging

### Requirement: WSL terminal startup preserves requested working directory
Terax SHALL preserve the requested WSL working directory when opening a terminal session for a selected distro.

#### Scenario: Opening a terminal from a workspace path
- **WHEN** Terax opens a WSL terminal session with a specific initial cwd
- **THEN** the interactive shell starts in that cwd
