## Why

When a user switches Terax to a WSL workspace, the terminal path hardcodes `bash --rcfile ... -i` instead of launching the distro's configured default shell. This bypasses the user's normal shell startup flow, so common WSL setups that default to `zsh` or `fish` lose their login and interactive initialization and behave differently from a native terminal.

## What Changes

- Make WSL terminal sessions launch the distro user's configured default shell instead of forcing `bash`.
- Preserve Terax shell integration for supported WSL shells so cwd tracking and prompt boundary markers still work.
- Keep a safe fallback path for unsupported or unresolvable WSL shells.
- Align WSL terminal startup behavior with user expectations from launching the same distro outside Terax.

## Capabilities

### New Capabilities
- `wsl-shell-initialization`: Defines how Terax detects and launches the correct shell inside a WSL distro while preserving shell integration and fallbacks.

### Modified Capabilities

## Impact

- Affected Rust PTY startup code in `src-tauri/src/modules/pty/shell_init.rs`.
- Affected WSL helper logic in `src-tauri/src/modules/workspace.rs`.
- Likely affects WSL-specific shell integration assets under `src-tauri/src/modules/pty/scripts/`.
- May require follow-up alignment for agent/one-shot shell execution paths if WSL shell behavior should be consistent across terminal and AI tooling.
