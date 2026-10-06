# WHY

Purpose-built zellij plugin that owns the tmux-parity tab-bar and status-bar behavior in one WebAssembly file.

# Facts

- `muxline` is a Rust zellij plugin compiled to `wasm32-wasip1`.
- `.cargo/config.toml` sets the build target, so `cargo build --release` writes `target/wasm32-wasip1/release/muxline.wasm`.
- Homebrew's `rust` formula ships no `wasm32-wasip1` target. `rustup run stable cargo build --release` builds with rustup's toolchain.
- One `muxline.wasm` binary selects behavior through the `mode` plugin configuration key.
- The plugin modes are Indicator, RenameTab, RenameSession, and ReloadConfig.
- Indicator mode normalizes tab names and owns waiting and completed attention icons.
- RenameTab and RenameSession modes share the same line editor.
- ReloadConfig mode reads `~/.config/zellij/config.kdl` and hands the contents to `reconfigure`.
- `Cargo.toml` depends on `zellij-tile` version `0.45.1`.
- Indicator mode sends `zjstatus::pipe::<name>::<content>` for `pipe_cwd`, `pipe_cmd`, and `pipe_zoom` to every zjstatus instance.
- Indicator mode reads `muxline::cwd::<pane>`, `muxline::cmd::<pane>`, `muxline::waiting::<pane>`, and `muxline::completed::<pane>` pipe messages. `README.md` shows the wiring.
