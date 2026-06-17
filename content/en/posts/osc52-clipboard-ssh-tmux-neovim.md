+++
title = "SSH + tmux + Neovim + OSC 52 Clipboard Guide"
date = 2026-06-17T22:23:00+08:00
draft = false

tags = ["neovim", "tmux", "ssh", "clipboard", "terminal"]
categories = ["tech"]
+++

## Problem

When working remotely via SSH inside tmux with Neovim, the system clipboard doesn't work.
You yank text in Neovim but can't paste it on your local machine with Ctrl+V.

## How OSC 52 Works

OSC 52 is a terminal escape sequence that tells your **local terminal emulator** to put
text into the system clipboard. The data flow is:

```
Neovim yank → OSC 52 escape sequence → tmux → SSH → local terminal → system clipboard
```

Each layer must allow the sequence to pass through untouched.

---

## Prerequisites

- A local terminal emulator that supports OSC 52 (see Compatibility below)
- tmux on the remote server
- Neovim on the remote server

---

## Step 1: Verify Your Local Terminal Supports OSC 52

SSH into your remote server and run:

```bash
printf '\033]52;c;%s\033\\' "$(printf 'OSC52_TEST' | base64)"
```

Then Ctrl+V locally. If you get `OSC52_TEST`, your terminal supports OSC 52.

### Terminal Compatibility

| Terminal        | OSC 52 Support | Notes                                        |
|-----------------|---------------|----------------------------------------------|
| Alacritty       | ✅ Yes        | No size limit                                |
| Kitty           | ✅ Yes        | No size limit                                |
| WezTerm         | ✅ Yes        | No size limit                                |
| foot            | ✅ Yes        | No size limit                                |
| iTerm2          | ✅ Yes        | Enable in Settings → General → Selection     |
| Windows Terminal | ✅ Yes       | Works out of the box                         |
| xfce4-terminal  | ⚠️ Partial   | VTE ≥0.70 required; needs OSC 52 patch (see below) |
| GNOME Terminal  | ⚠️ Partial   | VTE ≥0.70 required                           |
| PuTTY/KiTTY    | ❌ No         | Very limited support                         |
| MobaXterm       | ❌ No         | ~4KB hard limit, cannot be changed           |

### VTE-based terminals (xfce4-terminal, GNOME Terminal, etc.)

VTE (≤0.84) has two issues with OSC 52:

1. **No OSC 52 handler** — the `VTE_OSC_XTERM_SET_XSELECTION` case is a no-op (falls through to ignore)
2. **Parser string size limit** — `VTE_SEQ_STRING_MAX_CAPACITY` in `src/parser-string.hh` is `(1 << 12)` = 4096 unicode codepoints. Any OSC sequence exceeding this is silently dropped.

If large yanks (>100 lines) don't arrive in your clipboard but small ones do, you're
hitting the parser limit.

#### Fix: Rebuild VTE with OSC 52 patch

The patch does two things:
- Increases `VTE_SEQ_STRING_MAX_CAPACITY` from `(1 << 12)` to `(1 << 21)` (~2M codepoints, supports 1MB clipboard content after base64 encoding)
- Implements the actual OSC 52 clipboard handler in `src/vteseq.cc` (decodes base64, sets clipboard/primary selection)

**Arch Linux PKGBUILD + patch:** https://github.com/user/vte-osc52-patch

```bash
git clone https://github.com/user/vte-osc52-patch.git
cd vte-osc52-patch
makepkg -sf --nocheck
sudo pacman -U vte-common-*.pkg.tar.zst vte3-*.pkg.tar.zst
```

Then **fully restart** the terminal (kill the process, don't just open a new tab).

#### Files changed by the patch

| File | Change |
|------|--------|
| `src/parser-string.hh` | `VTE_SEQ_STRING_MAX_CAPACITY`: `(1 << 12)` → `(1 << 21)` |
| `src/vtegtk.cc` | Updated documentation comment |
| `src/vteseq.cc` | Added OSC 52 handler (clipboard `c` and primary `p` selection) |

---

## Step 2: Configure tmux

Add to `~/.tmux.conf` (or `~/.tmux.conf.local` if using oh-my-tmux):

```tmux
# Pass OSC 52 escape sequences through to the terminal
set -g allow-passthrough on

# Forward clipboard sequences to the outer terminal (not tmux's buffer)
set -g set-clipboard external
```

### Understanding `set-clipboard`

| Value      | Behavior                                              |
|------------|-------------------------------------------------------|
| `on`       | tmux intercepts OSC 52 into its own paste buffer      |
| `external` | tmux forwards OSC 52 to the outer terminal            |
| `off`      | tmux strips OSC 52 entirely                           |

**Use `external`** for OSC 52 from applications (nvim) to reach your local clipboard.

Note: tmux 3.3+ with `on` may do both (buffer + forward), but on tmux 3.2 and earlier,
`on` blocks forwarding. Use `external` for reliable behavior.

### prefix+y to push tmux buffer to local clipboard

Create `~/.local/bin/tmux-osc52-copy`:

```bash
#!/bin/sh
TTY=$(tmux display -p '#{client_tty}')
buf=$(tmux save-buffer -)
encoded=$(printf '%s' "$buf" | base64 | tr -d '\n')
printf '\033]52;c;%s\033\\' "$encoded" > "$TTY"
```

```bash
chmod +x ~/.local/bin/tmux-osc52-copy
```

Add binding:

```tmux
bind y run-shell "~/.local/bin/tmux-osc52-copy"
```

Now `prefix + y` sends the tmux paste buffer to your local clipboard.

### Key Detail: `client_tty` vs `pane_tty`

- `#{pane_tty}` — the pseudo-terminal for a specific pane (e.g., `/dev/pts/110`)
- `#{client_tty}` — the terminal that the tmux client is attached to (e.g., `/dev/pts/118`)

**Always use `#{client_tty}`** when writing OSC 52 — it's the path to your actual
terminal emulator. Writing to `pane_tty` may not reach the outer terminal.

---

## Step 3: Configure Neovim

The built-in `vim.ui.clipboard.osc52` module (Neovim 0.10+) uses `nvim_chan_send(2, ...)`
to write the OSC 52 sequence to stderr. Inside tmux's TUI mode, this doesn't reliably
reach the pane pty. The fix is to write the sequence **directly to the client tty** instead.

Add this to `~/.config/nvim/init.lua`:

```lua
vim.schedule(function()
  vim.opt.clipboard = 'unnamedplus'
end)

local function osc52_copy(lines, _)
  local text = table.concat(lines, '\n')
  local encoded = vim.base64.encode(text)
  local tty = vim.fn.system('tmux display -p "#{client_tty}"'):gsub('%s+', '')
  if tty ~= '' then
    local fd = io.open(tty, 'w')
    if fd then
      fd:write(string.format('\027]52;c;%s\027\\', encoded))
      fd:close()
    end
  end
end

vim.g.clipboard = {
  name = 'OSC 52',
  copy = {
    ['+'] = osc52_copy,
    ['*'] = osc52_copy,
  },
  paste = {
    ['+'] = function()
      return { vim.fn.split(vim.fn.getreg '', '\n'), vim.fn.getregtype '' }
    end,
    ['*'] = function()
      return { vim.fn.split(vim.fn.getreg '', '\n'), vim.fn.getregtype '' }
    end,
  },
}
```

### How it works

1. `vim.opt.clipboard = 'unnamedplus'` — makes all yanks use the `+` register
2. `vim.g.clipboard` — defines the provider that handles the `+` register
3. On yank: text → base64 encode (via `vim.base64.encode`) → wrap in OSC 52 → write to `client_tty` via `io.open`
4. On paste: reads the unnamed register directly (local paste only)

### Why not the built-in module?

Neovim's `vim.ui.clipboard.osc52` uses `nvim_chan_send(2, ...)` which writes to stderr.
In TUI mode inside tmux, this doesn't consistently reach the pane pty, so tmux's
`set-clipboard external` never sees the sequence. Writing to `client_tty` via `io.open`
is equivalent to the manual `printf > "$CLIENT_TTY"` test — it writes directly to the
terminal device that tmux is connected to, bypassing any fd-layer issues.

### Note on paste

OSC 52 paste (reading clipboard from terminal) requires the terminal to respond to a
query. This is unreliable across SSH/tmux. The practical workflow is:

- **Yank in nvim** → Ctrl+V locally (works via OSC 52)
- **Paste into nvim** → use terminal's bracketed paste (Ctrl+Shift+V in most terminals)

---

## Step 4: Reload Configuration

```bash
# Reload tmux
tmux source-file ~/.tmux.conf

# Restart nvim
:qa
nvim <file>
```

---

## Troubleshooting

### "X lines yanked" but Ctrl+V has nothing

1. **Check tmux settings:**
   ```bash
   tmux show -g allow-passthrough   # must be: on
   tmux show -g set-clipboard       # must be: external
   ```

2. **Verify the copy function writes to `client_tty`:**
   Ensure your clipboard config matches Step 3 — it must open `client_tty`
   directly rather than relying on stderr passthrough.

3. **Test OSC 52 directly:**
   ```bash
   CLIENT_TTY=$(tmux display -p '#{client_tty}')
   printf '\033]52;c;%s\033\\' "$(printf 'TEST' | base64)" > "$CLIENT_TTY"
   ```
   Then Ctrl+V. If nothing: terminal doesn't support OSC 52.

### Small yanks work, large yanks don't

Your terminal has a payload size limit. Test the threshold:

```bash
CLIENT_TTY=$(tmux display -p '#{client_tty}')
# Try 5KB
TEXT=$(seq 1 200 | sed 's/^/line /')
printf '\033]52;c;%s\033\\' "$(printf '%s' "$TEXT" | base64 | tr -d '\n')" > "$CLIENT_TTY"
```

If Ctrl+V is empty, your terminal caps around 4KB base64. Solutions:
- Upgrade to a terminal without limits (Alacritty, Kitty, WezTerm)
- Rebuild VTE with larger limit (see Step 1)

### tmux paste buffer works but local clipboard doesn't

You have `set-clipboard on` instead of `external`. Fix:
```bash
tmux set -g set-clipboard external
```

### OSC 52 works outside tmux but not inside

1. **Check `allow-passthrough`:**
   ```bash
   tmux set -g allow-passthrough on
   ```

2. **Check nvim's copy function:** The built-in `vim.ui.clipboard.osc52` uses
   `nvim_chan_send(2, ...)` which writes to stderr and may not reach the pane pty
   in TUI mode. Replace it with the custom `client_tty` implementation from Step 3.

---

## Complete Configuration Reference

### ~/.tmux.conf (or .local)
```tmux
set -g allow-passthrough on
set -g set-clipboard external
bind y run-shell "~/.local/bin/tmux-osc52-copy"
```

### ~/.local/bin/tmux-osc52-copy
```bash
#!/bin/sh
TTY=$(tmux display -p '#{client_tty}')
buf=$(tmux save-buffer -)
encoded=$(printf '%s' "$buf" | base64 | tr -d '\n')
printf '\033]52;c;%s\033\\' "$encoded" > "$TTY"
```

### ~/.config/nvim/init.lua (clipboard section)
```lua
vim.schedule(function()
  vim.opt.clipboard = 'unnamedplus'
end)

local function osc52_copy(lines, _)
  local text = table.concat(lines, '\n')
  local encoded = vim.base64.encode(text)
  local tty = vim.fn.system('tmux display -p "#{client_tty}"'):gsub('%s+', '')
  if tty ~= '' then
    local fd = io.open(tty, 'w')
    if fd then
      fd:write(string.format('\027]52;c;%s\027\\', encoded))
      fd:close()
    end
  end
end

vim.g.clipboard = {
  name = 'OSC 52',
  copy = { ['+'] = osc52_copy, ['*'] = osc52_copy },
  paste = {
    ['+'] = function()
      return { vim.fn.split(vim.fn.getreg '', '\n'), vim.fn.getregtype '' }
    end,
    ['*'] = function()
      return { vim.fn.split(vim.fn.getreg '', '\n'), vim.fn.getregtype '' }
    end,
  },
}
```
