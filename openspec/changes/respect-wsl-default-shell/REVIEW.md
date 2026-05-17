# Review

Escopo revisado:
- `src-tauri/src/modules/workspace.rs`
- `src-tauri/src/modules/pty/shell_init.rs`

## Findings

### Warning

1. `resolve_wsl_shell()` ainda pode abortar abertura do terminal em vez de cair para shell seguro.

Arquivos:
- `src-tauri/src/modules/workspace.rs:208`
- `src-tauri/src/modules/pty/shell_init.rs:351`

Problema:
- `build_wsl()` faz `resolve_wsl_shell(distro.clone())?`.
- Se qualquer uma das duas probes via `sh -lc` falhar (`wsl.exe` transitório, `/bin/sh` quebrado/ausente, erro de bootstrap do distro), erro sobe direto e PTY nem abre.
- Spec da própria change diz que falha de resolução deve cair para shell seguro com log, não falhar hard.

Impacto:
- Distros em estado parcialmente quebrado perdem terminal inteiro, mesmo quando `/bin/bash` ainda existe e poderia abrir sessão utilizável.

Correção esperada:
- Tratar erro de resolução como fallback explícito para `/bin/bash`, com `warn!`, em vez de propagar `Err`.

2. Fluxo WSL de `zsh` perde configs de usuários com `ZDOTDIR` custom.

Arquivos:
- `src-tauri/src/modules/pty/shell_init.rs:428`
- comparação útil: `src-tauri/src/modules/pty/shell_init.rs:175`
- scripts afetados: `src-tauri/src/modules/pty/scripts/zshenv.zsh:7`, `src-tauri/src/modules/pty/scripts/zprofile.zsh:5`

Problema:
- Branch Unix preserva `ZDOTDIR` antigo via `TERAX_USER_ZDOTDIR` antes de sobrescrever `ZDOTDIR`.
- Branch WSL só faz `cmd.env("ZDOTDIR", zdotdir)` e nunca exporta `TERAX_USER_ZDOTDIR`.
- Scripts gerados procuram configs do usuário em `${TERAX_USER_ZDOTDIR:-$HOME}`.
- Resultado: setups válidos com dotfiles fora de `$HOME` (`ZDOTDIR=$HOME/.config/zsh`, por exemplo) deixam de carregar `.zshenv/.zprofile/.zshrc/.zlogin`.

Impacto:
- Regressão direta contra requisito "user's normal zsh startup flow".
- Usuário entra em `zsh` sem aliases, plugins, prompt, PATH custom ou outros init hooks.

Correção esperada:
- Resolver/preservar `ZDOTDIR` real do usuário no WSL antes de aplicar integração, ou ajustar estratégia para que shell integrado continue sourcing dotfiles do dotdir configurado.

## Verification

- `cargo check --quiet` ✅
- `cargo test resolve_wsl_shell --quiet` ✅
- `cargo test wsl_ --quiet` ✅
