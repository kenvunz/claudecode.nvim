# claudecode.nvim

[![Tests](https://github.com/coder/claudecode.nvim/actions/workflows/test.yml/badge.svg)](https://github.com/coder/claudecode.nvim/actions/workflows/test.yml)
![Neovim version](https://img.shields.io/badge/Neovim-0.8%2B-green)
![Status](https://img.shields.io/badge/Status-beta-blue)

> ⚠️ **Important**: IDE integrations are currently broken in Claude Code releases newer than v1.0.27. Please use [Claude Code v1.0.27](https://www.npmjs.com/package/@anthropic-ai/claude-code/v/1.0.27) or older until these issues are resolved:
>
> - [Claude Code not detecting IDE integrations #2299](https://github.com/anthropics/claude-code/issues/2299)
> - [IDE integration broken after update #2295](https://github.com/anthropics/claude-code/issues/2295)

**The first Neovim IDE integration for Claude Code** — bringing Anthropic's AI coding assistant to your favorite editor with a pure Lua implementation.

> 🎯 **TL;DR:** When Anthropic released Claude Code with VS Code and JetBrains support, I reverse-engineered their extension and built this Neovim plugin. This plugin implements the same WebSocket-based MCP protocol, giving Neovim users the same AI-powered coding experience.

<https://github.com/user-attachments/assets/9c310fb5-5a23-482b-bedc-e21ae457a82d>

## What Makes This Special

When Anthropic released Claude Code, they only supported VS Code and JetBrains. As a Neovim user, I wanted the same experience — so I reverse-engineered their extension and built this.

- 🚀 **Pure Lua, Zero Dependencies** — Built entirely with `vim.loop` and Neovim built-ins
- 🔌 **100% Protocol Compatible** — Same WebSocket MCP implementation as official extensions
- 🎓 **Fully Documented Protocol** — Learn how to build your own integrations ([see PROTOCOL.md](./PROTOCOL.md))
- ⚡ **First to Market** — Beat Anthropic to releasing Neovim support
- 🛠️ **Built with AI** — Used Claude to reverse-engineer Claude's own protocol

## Installation

```lua
{
  "coder/claudecode.nvim",
  dependencies = { "folke/snacks.nvim" },
  config = true,
  keys = {
    { "<leader>a", nil, desc = "AI/Claude Code" },
    { "<leader>ac", "<cmd>ClaudeCode<cr>", desc = "Toggle Claude" },
    { "<leader>af", "<cmd>ClaudeCodeFocus<cr>", desc = "Focus Claude" },
    { "<leader>ar", "<cmd>ClaudeCode --resume<cr>", desc = "Resume Claude" },
    { "<leader>aC", "<cmd>ClaudeCode --continue<cr>", desc = "Continue Claude" },
    { "<leader>ab", "<cmd>ClaudeCodeAdd %<cr>", desc = "Add current buffer" },
    { "<leader>as", "<cmd>ClaudeCodeSend<cr>", mode = "v", desc = "Send to Claude" },
    {
      "<leader>as",
      "<cmd>ClaudeCodeTreeAdd<cr>",
      desc = "Add file",
      ft = { "NvimTree", "neo-tree", "oil" },
    },
    -- Diff management
    { "<leader>aa", "<cmd>ClaudeCodeDiffAccept<cr>", desc = "Accept diff" },
    { "<leader>ad", "<cmd>ClaudeCodeDiffDeny<cr>", desc = "Deny diff" },
  },
}
```

That's it! The plugin will auto-configure everything else.

## Requirements

- Neovim >= 0.8.0
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- [folke/snacks.nvim](https://github.com/folke/snacks.nvim) for enhanced terminal support

## Quick Demo

```vim
" Launch Claude Code in a split
:ClaudeCode

" Claude now sees your current file and selections in real-time!

" Send visual selection as context
:'<,'>ClaudeCodeSend

" Claude can open files, show diffs, and more
```

## Usage

1. **Launch Claude**: Run `:ClaudeCode` to open Claude in a split terminal
2. **Send context**:
   - Select text in visual mode and use `<leader>as` to send it to Claude
   - In `nvim-tree`/`neo-tree`/`oil.nvim`, press `<leader>as` on a file to add it to Claude's context
3. **Let Claude work**: Claude can now:
   - See your current file and selections in real-time
   - Open files in your editor
   - Show diffs with proposed changes
   - Access diagnostics and workspace info

## Commands

- `:ClaudeCode [arguments]` - Toggle the Claude Code terminal window (simple show/hide behavior)
- `:ClaudeCodeFocus [arguments]` - Smart focus/toggle Claude terminal (switches to terminal if not focused, hides if focused)
- `:ClaudeCodeTmux [arguments]` - Open Claude Code in a tmux pane (works regardless of terminal provider setting)
- `:ClaudeCode --resume` - Resume a previous Claude conversation
- `:ClaudeCode --continue` - Continue Claude conversation
- `:ClaudeCodeSend` - Send current visual selection to Claude, or add files from tree explorer
- `:ClaudeCodeTreeAdd` - Add selected file(s) from tree explorer to Claude context (also available via ClaudeCodeSend)
- `:ClaudeCodeAdd <file-path> [start-line] [end-line]` - Add a specific file or directory to Claude context by path with optional line range
- `:ClaudeCodeDiffAccept` - Accept the current diff changes (equivalent to `<leader>aa`)
- `:ClaudeCodeDiffDeny` - Deny/reject the current diff changes (equivalent to `<leader>ad`)

### Toggle Behavior

- **`:ClaudeCode`** - Simple toggle: Always show/hide terminal regardless of current focus
- **`:ClaudeCodeFocus`** - Smart focus: Focus terminal if not active, hide if currently focused

### Tree Integration

- `:ClaudeCode` - Toggle the Claude Code terminal window
- `:ClaudeCodeFocus` - Smart focus/toggle Claude terminal
- `:ClaudeCodeSend` - Send current visual selection to Claude
- `:ClaudeCodeAdd <file-path> [start-line] [end-line]` - Add specific file to Claude context with optional line range
- `:ClaudeCodeDiffAccept` - Accept diff changes
- `:ClaudeCodeDiffDeny` - Reject diff changes

## Working with Diffs

When Claude proposes changes, the plugin opens a native Neovim diff view:

- **Accept**: `:w` (save) or `<leader>aa`
- **Reject**: `:q` or `<leader>ad`

You can edit Claude's suggestions before accepting them.

## How It Works

This plugin creates a WebSocket server that Claude Code CLI connects to, implementing the same protocol as the official VS Code extension. When you launch Claude, it automatically detects Neovim and gains full access to your editor.

The protocol uses a WebSocket-based variant of MCP (Model Context Protocol) that:

1. Creates a WebSocket server on a random port
2. Writes a lock file to `~/.claude/ide/[port].lock` (or `$CLAUDE_CONFIG_DIR/ide/[port].lock` if `CLAUDE_CONFIG_DIR` is set) with connection info
3. Sets environment variables that tell Claude where to connect
4. Implements MCP tools that Claude can call

📖 **[Read the full reverse-engineering story →](./STORY.md)**
🔧 **[Complete protocol documentation →](./PROTOCOL.md)**

## Architecture

Built with pure Lua and zero external dependencies:

- **WebSocket Server** - RFC 6455 compliant implementation using `vim.loop`
- **MCP Protocol** - Full JSON-RPC 2.0 message handling
- **Lock File System** - Enables Claude CLI discovery
- **Selection Tracking** - Real-time context updates
- **Native Diff Support** - Seamless file comparison

For deep technical details, see [ARCHITECTURE.md](./ARCHITECTURE.md).

## Advanced Configuration

<details>
<summary>Complete configuration options</summary>

```lua
{
  "coder/claudecode.nvim",
  dependencies = { "folke/snacks.nvim" },
  opts = {
    -- Server Configuration
    port_range = { min = 10000, max = 65535 },
    auto_start = true,
    log_level = "info", -- "trace", "debug", "info", "warn", "error"
    terminal_cmd = nil, -- Custom terminal command (default: "claude")

    -- Selection Tracking
    track_selection = true,
    visual_demotion_delay_ms = 50,

    -- Terminal Configuration
    terminal = {
      split_side = "right", -- "left" or "right"
      split_width_percentage = 0.30,
      provider = "auto", -- "auto", "snacks", or "native"
      auto_close = true,
    },

    -- Diff Integration
    diff_opts = {
      auto_close_on_accept = true,              -- Close diff view after accepting changes
      show_diff_stats = true,                   -- Show diff statistics
      vertical_split = true,                    -- Use vertical split for diffs
      open_in_current_tab = true,               -- Open diffs in current tab vs new tab
    },
  },
}
```

</details>

### Configuration Options Explained

#### Server Options

- **`port_range`**: Port range for the WebSocket server that Claude connects to
- **`auto_start`**: Whether to automatically start the integration when Neovim starts
- **`terminal_cmd`**: Override the default "claude" command (useful for custom Claude installations)
- **`log_level`**: Controls verbosity of plugin logs

#### Selection Tracking

- **`track_selection`**: Enables real-time selection updates sent to Claude
- **`visual_demotion_delay_ms`**: Time to wait before switching from visual selection to cursor position tracking

#### Connection Management

- **`connection_wait_delay`**: Prevents overwhelming Claude with rapid @ mentions after connection
- **`connection_timeout`**: How long to wait for Claude to connect before giving up
- **`queue_timeout`**: How long to keep queued @ mentions before discarding them

#### Terminal Configuration

- **`split_side`**: Which side to open the terminal split (`"left"` or `"right"`)
- **`split_width_percentage`**: Terminal width as a fraction of screen width (0.1 = 10%, 0.5 = 50%)
- **`provider`**: Terminal implementation to use:
  - `"auto"`: Try tmux (if in tmux session), then snacks.nvim, fallback to native
  - `"snacks"`: Force snacks.nvim (requires folke/snacks.nvim)
  - `"native"`: Use built-in Neovim terminal
  - `"tmux"`: Use tmux panes (requires tmux session)
  - `"external"`: Use external terminal (e.g., separate terminal window)
- **`show_native_term_exit_tip`**: Show help text for exiting native terminal
- **`auto_close`**: Automatically close terminal when commands finish

#### Diff Options

- **`auto_close_on_accept`**: Close diff view after accepting changes with `:w` or `<leader>aa`
- **`show_diff_stats`**: Display diff statistics (lines added/removed)
- **`vertical_split`**: Use vertical split layout for diffs
- **`open_in_current_tab`**: Open diffs in current tab instead of creating new tabs

### Example Configurations

#### Minimal Configuration

```lua
{
  "coder/claudecode.nvim",
  keys = {
    { "<leader>a", nil, desc = "AI/Claude Code" },
    { "<leader>ac", "<cmd>ClaudeCode<cr>", desc = "Toggle Claude" },
    { "<leader>af", "<cmd>ClaudeCodeFocus<cr>", desc = "Focus Claude" },
    { "<leader>ar", "<cmd>ClaudeCode --resume<cr>", desc = "Resume Claude" },
    { "<leader>aC", "<cmd>ClaudeCode --continue<cr>", desc = "Continue Claude" },
    { "<leader>as", "<cmd>ClaudeCodeSend<cr>", mode = "v", desc = "Send to Claude" },
    {
      "<leader>as",
      "<cmd>ClaudeCodeTreeAdd<cr>",
      desc = "Add file",
      ft = { "NvimTree", "neo-tree", "oil" },
    },
  },
  keys = {
    -- Your keymaps here
  },
}
```

</details>

#### External Terminal Configuration

If you prefer to run Claude Code in an external terminal (e.g., tmux, separate terminal window), configure the plugin to use the external provider and load on startup:

```lua
{
  "coder/claudecode.nvim",
  event = "VeryLazy",  -- Load on startup for auto-start behavior
  opts = {
    terminal = {
      provider = "external",  -- Don't launch internal terminals
    },
  },
  keys = {
    { "<leader>a", nil, desc = "AI/Claude Code" },
    -- Add any keymaps you want (but they're not required for loading)
    { "<leader>as", "<cmd>ClaudeCodeSend<cr>", mode = "v", desc = "Send to Claude" },
    {
      "<leader>as",
      "<cmd>ClaudeCodeTreeAdd<cr>",
      desc = "Add file",
      ft = { "NvimTree", "neo-tree", "oil" },
    },
    -- Diff management
    { "<leader>aa", "<cmd>ClaudeCodeDiffAccept<cr>", desc = "Accept diff" },
    { "<leader>ad", "<cmd>ClaudeCodeDiffDeny<cr>", desc = "Deny diff" },
  },
}
```

With this configuration:

- The MCP server starts automatically when Neovim loads
- Run `claude` in your external terminal to connect
- Use `:ClaudeCodeStatus` to check connection status and get guidance

#### Tmux Integration

If you work with tmux sessions, claudecode.nvim can create tmux panes automatically:

```lua
{
  "coder/claudecode.nvim",
  keys = {
    { "<leader>a", nil, desc = "AI/Claude Code" },
    { "<leader>ac", "<cmd>ClaudeCode<cr>", desc = "Toggle Claude" },
    { "<leader>ct", "<cmd>ClaudeCodeTmux<cr>", desc = "Claude in tmux pane" },
    { "<leader>as", "<cmd>ClaudeCodeSend<cr>", mode = "v", desc = "Send to Claude" },
    {
      "<leader>as",
      "<cmd>ClaudeCodeTreeAdd<cr>",
      desc = "Add file",
      ft = { "NvimTree", "neo-tree", "oil" },
    },
    -- Diff management
    { "<leader>aa", "<cmd>ClaudeCodeDiffAccept<cr>", desc = "Accept diff" },
    { "<leader>ad", "<cmd>ClaudeCodeDiffDeny<cr>", desc = "Deny diff" },
  },
  opts = {
    terminal = {
      provider = "tmux",      -- Use tmux panes when available
      split_side = "right",   -- Create panes to the right
      split_width_percentage = 0.4,  -- 40% of terminal width
    },
  },
}
```

With tmux integration:

- **Auto-detection**: `provider = "auto"` automatically uses tmux when in tmux sessions
- **Manual command**: `:ClaudeCodeTmux` creates tmux panes regardless of provider setting
- **Pane control**: Supports `split_side` ("left"/"right") and `split_width_percentage`
- **Session persistence**: Tmux panes survive across Neovim restarts

#### Custom Claude Installation

```lua
{
  "coder/claudecode.nvim",
  keys = {
    { "<leader>a", nil, desc = "AI/Claude Code" },
    { "<leader>ac", "<cmd>ClaudeCode<cr>", desc = "Toggle Claude" },
    { "<leader>af", "<cmd>ClaudeCodeFocus<cr>", desc = "Focus Claude" },
    { "<leader>ar", "<cmd>ClaudeCode --resume<cr>", desc = "Resume Claude" },
    { "<leader>aC", "<cmd>ClaudeCode --continue<cr>", desc = "Continue Claude" },
    { "<leader>as", "<cmd>ClaudeCodeSend<cr>", mode = "v", desc = "Send to Claude" },
    {
      "<leader>as",
      "<cmd>ClaudeCodeTreeAdd<cr>",
      desc = "Add file",
      ft = { "NvimTree", "neo-tree", "oil" },
    },
    -- Diff management
    { "<leader>aa", "<cmd>ClaudeCodeDiffAccept<cr>", desc = "Accept diff" },
    { "<leader>ad", "<cmd>ClaudeCodeDiffDeny<cr>", desc = "Deny diff" },
  },
  opts = {
    terminal_cmd = "/opt/claude/bin/claude",  -- Custom Claude path
    port_range = { min = 20000, max = 25000 }, -- Different port range
  },
}
```

## Troubleshooting

- **Claude not connecting?** Check `:ClaudeCodeStatus` and verify lock file exists in `~/.claude/ide/` (or `$CLAUDE_CONFIG_DIR/ide/` if `CLAUDE_CONFIG_DIR` is set)
- **Need debug logs?** Set `log_level = "debug"` in opts
- **Terminal issues?** Try `provider = "native"` if using snacks.nvim
- **Auto-start not working?** If using external terminal provider, ensure you're using `event = "VeryLazy"` instead of `keys = {...}` only, as lazy loading prevents auto-start from running

## Contributing

See [DEVELOPMENT.md](./DEVELOPMENT.md) for build instructions and development guidelines. Tests can be run with `make test`.

## License

[MIT](LICENSE)

## Acknowledgements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) by Anthropic
- Inspired by analyzing the official VS Code extension
- Built with assistance from AI (how meta!)
