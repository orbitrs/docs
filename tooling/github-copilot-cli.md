# Using GitHub Copilot CLI with Orbit Framework

This guide explains how to set up and use GitHub Copilot in the command line to enhance your development experience with the Orbit Framework.

## Prerequisites

1. Access to GitHub Copilot. See [What is GitHub Copilot?](https://docs.github.com/en/copilot/about-github-copilot/what-is-github-copilot#getting-access-to-copilot).
2. GitHub CLI installed. For installation instructions, see the [GitHub CLI repository](https://github.com/cli/cli#installation).
3. Copilot in the CLI extension installed. See [Installing GitHub Copilot in the CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli/installing-github-copilot-in-the-cli).

## Installation

### 1. Install GitHub CLI

**macOS:**
```bash
brew install gh
```

**Linux:**
```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```

**Windows:**
```powershell
winget install --id GitHub.cli
```

### 2. Login to GitHub CLI

```bash
gh auth login
```

### 3. Install GitHub Copilot CLI Extension

```bash
gh extension install github/gh-copilot
```

### 4. Set up aliases (optional, but recommended)

To simplify using GitHub Copilot CLI, add these aliases to your shell configuration:

**For Bash/ZSH:**

Add to your `~/.bashrc` or `~/.zshrc`:
```bash
# Copilot CLI aliases
alias '??'='gh copilot explain'
alias ghcs='gh copilot suggest'
```

Then reload your shell configuration:
```bash
source ~/.zshrc  # or ~/.bashrc
```

## Using GitHub Copilot CLI with Orbit Framework

### Getting command explanations

To ask Copilot to explain a command, run:

```bash
gh copilot explain "cargo test -p orbit --features desktop"
```

Or with the alias:
```bash
?? "cargo test -p orbit --features desktop"
```

### Getting command suggestions

To ask Copilot to suggest a command, run:

```bash
gh copilot suggest "Run tests for the orlint package"
```

Or with the alias:
```bash
ghcs "Run tests for the orlint package"
```

## Common Orbit Framework Tasks with Copilot CLI

Here are some examples of how to use Copilot CLI with the Orbit Framework:

### Development Tasks

```bash
ghcs "Create a new Orbit component"
ghcs "Run the dev server for my Orbit project"
ghcs "Build the orbit framework for WebAssembly"
ghcs "Check for linting errors in my Orbit component"
```

### Testing Tasks

```bash
ghcs "Run all tests for the orbit package"
ghcs "Run only tests with 'component' in their name"
ghcs "Run tests with the desktop feature enabled"
```

### Git Workflow

```bash
ghcs "Create a new branch for fixing a bug in orlint"
ghcs "Commit changes with a conventional commit message"
ghcs "Update my branch with the latest from main"
```

## Sharing Feedback

To send feedback about GitHub Copilot CLI suggestions, select the "Rate response" option in the interactive CLI session.

You can also open an issue in the [Copilot in the CLI extension repository](https://github.com/github/gh-copilot) if you encounter any problems or have suggestions for improvement.

## Further Reading

- [Copilot in the CLI extension README](https://github.com/github/gh-copilot?tab=readme-ov-file)
- [Configuring GitHub Copilot in the CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli/configuring-github-copilot-in-the-cli)
- [Using GitHub Copilot in the command line](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-in-the-command-line)
