### Copy & Paste

#### Bash

Clearest and most simplest case, just use the function provided by your OS or terminal clients.

- Windows Terminal
  - copy: mouse select, ^c
  - paste: ^v, or shift+mouse-rightclick

- WezTerm
  - copy: mouse select for copy
  - paste: mouse middle-click


#### Tmux

Tmux has built-in copy mode. It also supports `OSC 52` for copy [out of box](https://github.com/tmux/tmux/wiki/Clipboard). Copy function even works over SSH. Plugins like [tmux-yank](https://github.com/tmux-plugins/tmux-yank) seem to be not necessary these days. They use external tools like `xsel` and `xclip` to sync with host clipboard, and do not work over SSH.

A compatible client is **required**. Windows Terminal and [WezTerm](https://github.com/wezterm/wezterm), [Alacritty](https://github.com/alacritty/alacritty) and even `xterm` support `OSC 52`, while [VTE-based](https://wiki.archlinux.org/title/List_of_applications/Utilities#VTE-based) terminals including Gnome Terminal, Xfce Terminal [do not](https://gitlab.gnome.org/GNOME/vte/-/issues/2495). Remote copy does not work in these terminals. In this case, `tmux-yank` may help.

To check whether your terminal client supports `OSC 52`, run following command in remote shell, and check your local clipboard:

```bash
printf $'\e]52;c;%s\a' "$(base64 <<<'hello world')"
```

#### Neovim

Neovim is the most confusing case. It is under heavy development. Neovim also has built-in `OSC 52` support starting [v0.10](https://github.com/neovim/neovim/pull/26064). But some customization required. Add [lines](https://github.com/neovim/neovim/discussions/28010#discussioncomment-9877494) to your `~/.config/nvim/init.lua`:

```lua
vim.o.clipboard = "unnamedplus"

-- if vim.env.SSH_TTY ~= nil and vim.env.TMUX == nil then
if vim.env.TMUX == nil then
  local function paste()
    return {
      vim.fn.split(vim.fn.getreg(""), "\n"),
      vim.fn.getregtype(""),
    }
  end
  vim.g.clipboard = {
    name = "OSC 52",
    copy = {
      ["+"] = require("vim.ui.clipboard.osc52").copy("+"),
      ["*"] = require("vim.ui.clipboard.osc52").copy("*"),
    },
    paste = {
      ["+"] = paste,
      ["*"] = paste,
    },
  }
end
```

If you are using [LazyVim](https://github.com/LazyVim/LazyVim), add lines to `~/.config/nvim/lua/config/options.lua`. Otherwise, clipboard options are [overwritten](https://github.com/LazyVim/LazyVim/blob/75297733710951e81b505d88b2d728a5b0a9b6ab/lua/lazyvim/config/options.lua#L57). The result is: If Neovim is running inside Tmux, it uses Tmux as [clipboard provider](https://neovim.io/doc/user/provider.html#provider-clipboard). If Neovim is running out of Tmux(raw SSH), it uses `OSC 52` as clipboard provider. You can verify the config using `:set clipboard?` and `:checkhealth clipboard` in Neovim.

It seems Neovim will fallback to use `OSC 52`  as clipboard provider in [v0.11](https://github.com/neovim/neovim/pull/31730).

Now, the Neovim clipboard should sync with your local clipboard.

#### Paste

Windows Terminal does not support `OSC 52` paste for security reasons. Other terminal clients may have same concerns. Just use the shortcut provided in the first section.

