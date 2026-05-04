# Installing OpenCode on Linux

> Step-by-step guide to getting OpenCode running on a Linux machine — from installation to your first session.

**Official website:** [opencode.ai](https://opencode.ai/)  
**GitHub repository:** [sst/opencode](https://github.com/sst/opencode)  
**Estimated time:** 5–10 minutes

---

## Table of Contents

1. [What Is OpenCode?](#1-what-is-opencode)
2. [Prerequisites](#2-prerequisites)
3. [Installation Methods](#3-installation-methods)
4. [Verifying the Installation](#4-verifying-the-installation)
5. [First Launch and AI Provider Setup](#5-first-launch-and-ai-provider-setup)
6. [Troubleshooting](#6-troubleshooting)

---

## 1. What Is OpenCode?

OpenCode is an AI-powered coding assistant that runs directly in your terminal. Unlike chat-based tools, it has access to your filesystem and can read, edit, and reason about your actual code — no copy-pasting required.

It supports multiple AI providers (Anthropic Claude, OpenAI, and others) and works on any Linux system with a modern terminal emulator.

---

## 2. Prerequisites

**Required:**

- A modern terminal emulator with Unicode and true color support (see recommendations below)
- `bash` and `curl` (pre-installed on virtually all Linux distributions)

**Recommended terminal emulators:**

| Terminal | Notes |
|---|---|
| [WezTerm](https://wezfurlong.org/wezterm/) | Best overall support, recommended |
| [Alacritty](https://alacritty.org/) | Fast, GPU-accelerated |
| [Ghostty](https://ghostty.org/) | Modern, built-in ligature support |
| GNOME Terminal / Konsole | Works, but may lack some rendering features |

**Optional (depending on install method):**

- Node.js 18+ — required for the `npm` install method → [How to install Node.js on Linux](./install-nodejs-linux.md)
- Bun — required for the `bun` install method

---

## 3. Installation Methods

Choose the method that best fits your environment.

### Method 1 — Curl script (recommended)

The fastest and most universal option. Requires only `bash` and `curl`.

```bash
curl -fsSL https://opencode.ai/install | bash
```

The script auto-detects your CPU architecture (x64, ARM64, and musl variants) and installs the correct binary. It also updates your shell's `PATH` automatically.

**Install to a custom directory:**

```bash
OPENCODE_INSTALL_DIR=/usr/local/bin curl -fsSL https://opencode.ai/install | bash
```

**Install a specific version:**

```bash
curl -fsSL https://opencode.ai/install | bash -s -- --version 1.0.180
```

**Skip PATH modification (advanced setups):**

```bash
curl -fsSL https://opencode.ai/install | bash -s -- --no-modify-path
```

The binary is installed to the first writable path found in this priority order:

1. `$OPENCODE_INSTALL_DIR` (if set)
2. `$XDG_BIN_DIR` (if set)
3. `$HOME/bin` (if it exists)
4. `$HOME/.opencode/bin` (default fallback)

---

### Method 2 — npm

```bash
npm install -g opencode-ai
```

Requires Node.js 18 or later. To check your version: `node --version`.  
Don't have Node.js yet? See [How to install Node.js on Linux](./install-nodejs-linux.md).

---

### Method 3 — Bun

```bash
bun add -g opencode-ai
```

---

### Method 4 — Arch Linux (pacman / AUR)

**Stable release (official repo):**

```bash
sudo pacman -S opencode
```

**Latest build (AUR, using paru):**

```bash
paru -S opencode-bin
```

---

### Method 5 — Nix

```bash
nix run nixpkgs#opencode
```

To install persistently in your profile:

```bash
nix profile install nixpkgs#opencode
```

---

### Method 6 — Desktop application (GUI wrapper)

Download a pre-built package from the [GitHub Releases page](https://github.com/sst/opencode/releases):

| Distribution | Package format |
|---|---|
| Debian / Ubuntu | `.deb` |
| Red Hat / Fedora / CentOS | `.rpm` |
| Any Linux (universal) | `.AppImage` |

Install the `.deb` package:

```bash
sudo dpkg -i opencode_*.deb
```

Install the `.rpm` package:

```bash
sudo rpm -i opencode_*.rpm
```

Run the AppImage directly (no install needed):

```bash
chmod +x opencode_*.AppImage
./opencode_*.AppImage
```

---

## 4. Verifying the Installation

Open a new terminal (so the updated PATH is loaded) and run:

```bash
opencode --version
```

You should see the installed version number printed. If the command is not found:

1. Close and reopen your terminal to reload your shell profile.
2. Check that the install directory is in your `PATH`:
   ```bash
   echo $PATH
   ```
3. If using the curl method with default install path, add this to your `~/.bashrc` or `~/.zshrc`:
   ```bash
   export PATH="$HOME/.opencode/bin:$PATH"
   ```
   Then run: `source ~/.bashrc`

---

## 5. First Launch and AI Provider Setup

### Start OpenCode

Navigate to any project directory and launch it:

```bash
cd ~/your-project
opencode
```

### Connect an AI provider

On first launch, OpenCode will prompt you to connect an AI provider. You can also trigger this manually with the `/connect` command inside OpenCode, or visit [opencode.ai/auth](https://opencode.ai/auth) in your browser.

**Alternatively, set your API key as an environment variable:**

For Anthropic Claude:

```bash
export ANTHROPIC_API_KEY="sk-ant-your-key-here"
```

For OpenAI:

```bash
export OPENAI_API_KEY="sk-your-key-here"
```

Add the export line to your `~/.bashrc` or `~/.zshrc` to persist it across sessions.

### Initialize your project

Inside OpenCode, run the `/init` command. It analyzes your codebase and generates an `AGENTS.md` file, which gives the AI context about your project.

```
/init
```

You are now ready to use OpenCode.

---

## 6. Troubleshooting

**`command not found: opencode` after installation**

Your shell has not yet loaded the updated PATH. Run `source ~/.bashrc` (or `~/.zshrc`) or open a new terminal.

**Rendering issues (broken characters, missing colors)**

Your terminal emulator may not support true color or Unicode. Switch to WezTerm or Alacritty and ensure your `TERM` variable is set correctly:

```bash
echo $TERM
# Should output: xterm-256color or similar
```

**Permission denied when installing to `/usr/local/bin`**

Prefix with `sudo`, or install to a user-writable directory instead:

```bash
OPENCODE_INSTALL_DIR="$HOME/.local/bin" curl -fsSL https://opencode.ai/install | bash
```

**Updating OpenCode**

Re-run the install script to get the latest version:

```bash
curl -fsSL https://opencode.ai/install | bash
```

Or if using npm:

```bash
npm update -g opencode-ai
```

---

*Documentation based on OpenCode as of May 2026. For the latest information, see the [official docs](https://opencode.ai/docs/).*
