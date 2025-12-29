# Switch-Omo-Config

Interactive CLI tool to switch between [oh-my-opencode](https://github.com/code-yeongyu/oh-my-opencode) configuration profiles.

![Demo](https://img.shields.io/badge/macOS-compatible-brightgreen) ![Shell](https://img.shields.io/badge/shell-bash-blue)

## What it does

Quickly switch between different `oh-my-opencode` configurations without manually copying files. Useful when you have multiple profiles for different AI providers (ChatGPT, Google, Copilot, etc.).

## Prerequisites

- macOS (uses `md5` for file comparison)
- [oh-my-opencode](https://github.com/code-yeongyu/oh-my-opencode) installed
- At least one profile named `oh-my-opencode-*.json` in either:
  - `~/.config/opencode/` (global profiles)
  - `.opencode/` in your project (project-local profiles)

## Installation

### Option 1: Direct download

```bash
curl -o ~/.config/opencode/switch-omo-config.sh \
  https://raw.githubusercontent.com/AnPod/Switch-Omo-Config/main/switch-omo-config.sh
chmod +x ~/.config/opencode/switch-omo-config.sh
```

### Option 2: Manual

```bash
# Copy the script to your opencode config directory
cp switch-omo-config.sh ~/.config/opencode/
chmod +x ~/.config/opencode/switch-omo-config.sh
```

### Add shell alias (recommended)

```bash
# For zsh
echo 'alias omo-switch="~/.config/opencode/switch-omo-config.sh"' >> ~/.zshrc
source ~/.zshrc

# For bash
echo 'alias omo-switch="~/.config/opencode/switch-omo-config.sh"' >> ~/.bashrc
source ~/.bashrc
```

## Usage

```bash
# Run directly
~/.config/opencode/switch-omo-config.sh

# Or with alias
omo-switch
```

### Controls

| Key | Action |
|-----|--------|
| `↑` or `k` | Move selection up |
| `↓` or `j` | Move selection down |
| `Enter` | Apply selected config |
| `q` | Quit without changes |

### Example output

```
Switch oh-my-opencode Configuration
Use arrow keys to navigate, Enter to select, q to quit

> oh-my-opencode-ChatGPT.json
  oh-my-opencode-baseline.json
  oh-my-opencode-copilot.json
  oh-my-opencode-google.json (active)
  oh-my-opencode-minimax.json
```

## How it works

1. Detects whether you’re in a project that has a `.opencode/` directory.
2. Chooses the working config directory:
   - If `.opencode/` exists in the current directory: uses `.opencode/`
   - Otherwise: uses `~/.config/opencode/`
3. (Project-local only, first run) Prompts whether to copy global profiles `~/.config/opencode/oh-my-opencode-*.json` into `.opencode/`.
   - Your answer is persisted to `.opencode/.switch-omo-config.copy-profiles` (`y` or `n`) so you won’t be asked again in that project.
4. Scans the chosen directory for `oh-my-opencode-*.json`, compares hashes to find the active profile, and copies the selected profile to `oh-my-opencode.json` in the same directory.

> **Note**: After switching configs, you must exit and reopen OpenCode for the changes to take effect. To continue from your previous session, use `/session` and select the last session.

## Reset the project prompt

To be asked again whether to copy global profiles into a specific project’s `.opencode/`, delete this file in that project:

- `.opencode/.switch-omo-config.copy-profiles`

## Config file structure

### Global mode (no `.opencode/` in current directory)

```
~/.config/opencode/
├── opencode.json              # Main OpenCode config
├── oh-my-opencode.json        # Active oh-my-opencode config (managed by this tool)
├── oh-my-opencode-ChatGPT.json
├── oh-my-opencode-google.json
├── oh-my-opencode-copilot.json
└── oh-my-opencode-baseline.json
```

### Project-local mode (when `.opencode/` exists)

```
./.opencode/
├── oh-my-opencode.json        # Active oh-my-opencode config (managed by this tool)
├── oh-my-opencode-ChatGPT.json
├── oh-my-opencode-google.json
├── oh-my-opencode-copilot.json
├── oh-my-opencode-baseline.json
└── .switch-omo-config.copy-profiles  # Persists copy prompt answer (y/n)
```

## Known Issues

### Anthropic "Invalid signature in thinking block" error

After switching back to Anthropic models within the same session, you may experience:

```
messages.1.content.0: Invalid 'signature' in 'thinking' block
```

**Solution**: Disable "thinking" for Anthropic models in your OpenCode config.

Add this to `~/.config/opencode/opencode.json` (or project `opencode.json`) and restart OpenCode:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "anthropic": {
      "models": {
        "claude-sonnet-4-5": {
          "options": {
            "thinking": { "type": "disabled" }
          }
        }
      }
    }
  }
}
```

## License

MIT
