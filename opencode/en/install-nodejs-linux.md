# Installing Node.js on Linux

> How to install Node.js on a Linux machine. Covers the three most common approaches so you can pick the one that fits your setup.

**Recommended version:** Node.js 18 or later (LTS)  
**Official website:** [nodejs.org](https://nodejs.org/)

---

## Table of Contents

1. [Which Method Should I Choose?](#1-which-method-should-i-choose)
2. [Method 1 — nvm (recommended)](#2-method-1--nvm-recommended)
3. [Method 2 — NodeSource (system-wide, apt/rpm)](#3-method-2--nodesource-system-wide-aptrpm)
4. [Method 3 — Snap](#4-method-3--snap)
5. [Verifying the Installation](#5-verifying-the-installation)

---

## 1. Which Method Should I Choose?

| Situation | Recommended method |
|---|---|
| You want to switch between Node versions | nvm |
| You want a system-wide install, managed by apt | NodeSource |
| You're on Ubuntu and want the simplest one-liner | Snap |

---

## 2. Method 1 — nvm (recommended)

[nvm](https://github.com/nvm-sh/nvm) (Node Version Manager) installs Node.js in your home directory without requiring `sudo`. It also lets you switch between Node versions per project — useful if you work on multiple codebases.

### Install nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

Reload your shell:

```bash
source ~/.bashrc
# or, if you use zsh:
source ~/.zshrc
```

Verify nvm is available:

```bash
nvm --version
```

### Install Node.js

Install the latest LTS version (recommended for most users):

```bash
nvm install --lts
```

Or install a specific version:

```bash
nvm install 22
```

Set it as the default for new shell sessions:

```bash
nvm alias default 22
```

### Switching versions

```bash
nvm use 20    # switch to Node 20 in the current session
nvm ls        # list all installed versions
```

---

## 3. Method 2 — NodeSource (system-wide, apt/rpm)

NodeSource maintains official repositories for Debian/Ubuntu and Red Hat/Fedora families. This installs Node.js system-wide via your package manager.

### Debian / Ubuntu

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
sudo apt-get install -y nodejs
```

Replace `22.x` with `20.x` or `18.x` if you need a specific LTS version.

### Red Hat / Fedora / CentOS

```bash
curl -fsSL https://rpm.nodesource.com/setup_22.x | sudo bash -
sudo dnf install -y nodejs
```

---

## 4. Method 3 — Snap

Available on Ubuntu and any distribution with `snapd` installed.

```bash
sudo snap install node --classic
```

To install a specific major version:

```bash
sudo snap install node --classic --channel=22
```

---

## 5. Verifying the Installation

Check that Node.js and npm are available:

```bash
node --version
npm --version
```

Both commands should print a version number. Node.js 18 or later is required for tools like OpenCode that depend on it.

---

*For more details, see the [official Node.js installation guide](https://nodejs.org/en/download/package-manager).*
