## 1. WSL shell resolution

- [x] 1.1 Add a WSL helper in `src-tauri/src/modules/workspace.rs` that resolves the selected distro user's default shell with layered fallbacks.
- [x] 1.2 Cover the helper's parsing and fallback behavior with targeted tests or equivalent validation hooks.

## 2. PTY startup integration

- [x] 2.1 Refactor the WSL branch in `src-tauri/src/modules/pty/shell_init.rs` to match the resolved shell and launch supported shells with the correct integration mode.
- [x] 2.2 Add WSL-side integration asset preparation for supported shells so `zsh`, `bash`, and `fish` can load user init plus Terax hooks from the distro home.
- [x] 2.3 Implement fallback behavior for unsupported or unresolvable WSL shells without regressing cwd handling.

## 3. Validation

- [x] 3.1 Verify WSL terminal startup for supported shells preserves expected user init and Terax cwd/prompt markers.
- [x] 3.2 Verify unsupported-shell and shell-resolution-failure paths still open usable terminals and emit useful logs.
- [x] 3.3 Run the relevant Rust checks for the touched backend modules and document any remaining follow-up around WSL AI/one-shot shell consistency.
