# muxline

A zellij plugin that gives zellij a tmux-style tab bar, rendered through [zjstatus](https://github.com/dj95/zjstatus).

- A default `Tab #N` shows as its number, and a renamed tab shows as `N: name`.
- ⏳ waiting and ✅ completed icons mark the tab that holds a pane. Focusing the tab clears them.
- The focused pane's directory, command, and zoom state go to zjstatus.
- Popups rename the tab or the session with a full line editor.
- A keybinding reloads `config.kdl`.

## Build

```bash
rustup target add wasm32-wasip1
cargo build --release
mkdir -p ~/.config/zellij/plugins
cp target/wasm32-wasip1/release/muxline.wasm ~/.config/zellij/plugins/
```

## Configure zellij

```kdl
plugins {
    muxline location="file:~/.config/zellij/plugins/muxline.wasm"
}
load_plugins {
    muxline
}
keybinds {
    tmux {
        bind "r" { LaunchOrFocusPlugin "muxline" { floating true; move_to_focused_tab true; skip_plugin_cache true; mode "rename_tab"; }; SwitchToMode "Normal"; }
        bind "R" { LaunchOrFocusPlugin "muxline" { floating true; move_to_focused_tab true; skip_plugin_cache true; mode "rename_session"; }; SwitchToMode "Normal"; }
        bind "C" { LaunchPlugin "muxline" { floating true; skip_plugin_cache true; mode "reload_config"; }; SwitchToMode "Normal"; }
    }
}
```

muxline asks for `ReadApplicationState`, `ChangeApplicationState`, `ReadCliPipes`, `RunCommands`, and `MessageAndLaunchOtherPlugins`.

## Configure zjstatus

```kdl
format_left          "{mode} {session} {pipe_cwd} {pipe_cmd} {pipe_zoom}"
tab_normal           " {index}{name} "
tab_active           " {index}{name} "
pipe_cwd_rendermode  "static"
pipe_cmd_rendermode  "static"
pipe_zoom_rendermode "dynamic"
```

## Shell integration (zsh)

```zsh
precmd() {
  [ -n "$ZELLIJ" ] || return
  zellij pipe --name "muxline::cwd::$ZELLIJ_PANE_ID" --payload "$PWD" >/dev/null 2>&1
  zellij pipe --name "muxline::cmd::$ZELLIJ_PANE_ID" --payload "zsh" >/dev/null 2>&1
  zellij pipe --name "muxline::completed::$ZELLIJ_PANE_ID" >/dev/null 2>&1
}
preexec() {
  [ -n "$ZELLIJ" ] && zellij pipe --name "muxline::cmd::$ZELLIJ_PANE_ID" --payload "${1%% *}" >/dev/null 2>&1
}
```

Any process can mark a pane as waiting with `zellij pipe --name "muxline::waiting::$ZELLIJ_PANE_ID"`.
