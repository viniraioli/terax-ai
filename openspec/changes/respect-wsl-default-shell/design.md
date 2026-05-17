## Context

Terax already has shell-specific integration for Unix shells: `zsh` uses a generated `ZDOTDIR`, `bash` uses a generated `--rcfile`, and `fish` uses `--init-command`. On Windows local shells, Terax uses a separate PowerShell path. WSL currently bypasses the Unix shell selection flow and always launches `wsl.exe -d <distro> --cd <cwd> --exec bash --rcfile ... -i`, which means the selected distro shell is ignored and users lose their expected login and interactive startup behavior.

The affected path is cross-cutting enough to justify a design artifact:

- WSL shell selection lives in `workspace.rs`.
- PTY startup lives in `pty/shell_init.rs`.
- Shell integration assets live in `pty/scripts/`.
- The change must preserve prompt/cwd markers for supported shells while avoiding regressions for unsupported ones.

## Goals / Non-Goals

**Goals:**

- Detect the configured default shell for the selected WSL distro before starting a PTY session.
- Launch supported WSL shells (`zsh`, `bash`, `fish`) with the same shell-specific integration strategy Terax already uses on Unix.
- Preserve WSL `cwd` behavior and keep a safe fallback when shell detection or integration is not possible.
- Keep the behavior change narrow enough to fix the interactive terminal bug without destabilizing unrelated shell features.

**Non-Goals:**

- Do not redesign the general shell integration architecture for all platforms.
- Do not add a user-facing shell picker or override UI for WSL in this change.
- Do not change one-shot or agent shell execution in `src-tauri/src/modules/shell/` as part of this fix; that can be handled in a follow-up if consistency is desired.

## Decisions

### Detect the WSL user's default shell from the distro itself

Add a WSL helper that resolves the current user's login shell inside the selected distro, preferably via the passwd database, with fallbacks to `$SHELL` and finally `/bin/bash`.

- Why: the PTY launcher must know which shell it is starting in order to choose the right integration mode and flags.
- Alternative considered: drop `--exec bash` and let `wsl.exe` choose implicitly. Rejected because Terax still needs shell-aware setup for `zsh`, `bash`, and `fish`, and implicit launch gives the app no place to prepare the matching integration assets.

### Reuse Terax's existing shell-specific integration model for WSL

For supported shells, keep the current integration approach but materialize files inside the distro's home directory via UNC paths:

- `zsh`: create a generated WSL `ZDOTDIR` and launch the detected shell with `-l`.
- `bash`: create a generated WSL rcfile and launch with `--rcfile <path> -i`.
- `fish`: create a generated WSL init file and launch with `--init-command "source <path>" -i`.

The launch target should be the detected shell path, not a hardcoded binary name.

- Why: this preserves cwd tracking and prompt-boundary markers while respecting the distro's configured shell.
- Alternative considered: inject a generic wrapper shell script for all WSL shells. Rejected because existing integration already differs by shell semantics, and forcing a generic wrapper would be more brittle than extending the proven per-shell paths.

### Keep unsupported shells usable without forcing a shell swap

If the resolved WSL shell is not one of the supported integrated shells, launch that shell directly without Terax-specific shell integration. If resolution fails entirely, log the reason and fall back to `/bin/bash`.

- Why: respecting the user's configured shell is better than silently replacing it. Losing prompt markers is acceptable for unsupported shells; replacing the shell is not.
- Alternative considered: always fall back to `bash` for any non-supported shell. Rejected because it recreates the user-facing bug in a slightly different form.

### Limit scope to interactive PTY sessions

This change should update the PTY path used by the visible terminal tabs. WSL one-shot commands and persistent agent shell sessions can stay on their current `sh -lc` path for now.

- Why: the reported bug is in the interactive terminal opened from the workspace environment menu. Fixing that path first reduces risk and keeps the change reviewable.
- Alternative considered: unify PTY and agent shell behavior in one pass. Rejected for now because it broadens the blast radius and introduces extra product questions about whether AI tooling must mirror the user's interactive shell exactly.

## Risks / Trade-offs

- [WSL shell lookup differs across distros] -> Use a layered fallback path (`passwd` -> `$SHELL` -> `/bin/bash`) and log failures.
- [Shell integration assets may drift between Unix and WSL code paths] -> Keep WSL using the same script contents and shell matching rules as Unix, only changing where files are written and how they are launched.
- [Unsupported shells lose prompt markers] -> Accept as explicit fallback behavior and keep the shell usable rather than forcing `bash`.
- [Terminal and AI shell behavior remain inconsistent in WSL] -> Document as non-goal and consider a follow-up change if user expectations or tests show the inconsistency matters.

## Migration Plan

No data migration is required. Rollout is a runtime behavior change in PTY startup only.

- Update WSL shell resolution and PTY launch logic.
- Validate supported shells (`zsh`, `bash`, `fish`) in a WSL distro with custom startup files.
- Validate unsupported-shell fallback and unresolvable-shell fallback.
- Roll back by reverting the PTY WSL launch changes if a release regression appears.

## Open Questions

- Should a follow-up change align WSL one-shot and agent shell execution with the interactive terminal once this bug is fixed?
- Do we want telemetry or debug logs specific enough to confirm which WSL shell path was selected during support incidents?
