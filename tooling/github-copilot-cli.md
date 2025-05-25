# Using GitHub Copilot with Orbit Framework

This guide explains how to set up and use GitHub Copilot in the command line and web interface to enhance your development experience with the Orbit Framework.

## GitHub Copilot Features for Orbit Framework

GitHub Copilot provides several powerful features that can improve your development workflow:

1. **Command Line Assistance**: Get suggestions and explanations for terminal commands
2. **Code Completion**: Smart code suggestions in your editor
3. **Issue Creation**: Create well-structured issues using natural language
4. **Code Reference Tracking**: Find public code that matches Copilot suggestions
5. **Chat Interface**: Ask questions about code and get help with development tasks

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

## Creating Issues with GitHub Copilot

You can use GitHub Copilot to create well-structured issues without manually filling out every field.

1. Go to the immersive view of Copilot Chat: [https://github.com/copilot](https://github.com/copilot)
2. Describe the issue you want to create in natural language, specifying the repository:

```
Create a feature request for orbitrs/orbit to improve component hot-reloading
```

3. Copilot will draft an issue with title, body, labels, and assignees based on your repository's templates
4. Review, edit if needed, and click "Create" when ready

**Pro Tip**: You can include screenshots by pasting them directly into the Copilot prompt box for visual bugs.

For more details, see [Using GitHub Copilot to create issues](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-to-create-issues).

## Finding Public Code that Matches Suggestions

GitHub Copilot can show references to public code that matches its suggestions, helping you ensure proper attribution and understand the sources influencing the generated code.

### In VS Code:

1. Open the Output window (View > Output)
2. Select "GitHub Copilot Log (Code References)" from the dropdown
3. When you accept a suggestion that matches public code, details will appear in this log

### In Copilot Chat:

When a response includes matching code, you'll see a link like "Public code references from n repositories." Click this to view matching code details and licensing information.

For more information, see [Finding public code that matches GitHub Copilot suggestions](https://docs.github.com/en/copilot/using-github-copilot/finding-public-code-that-matches-github-copilot-suggestions).

## Sharing Feedback

To send feedback about GitHub Copilot CLI suggestions, select the "Rate response" option in the interactive CLI session.

You can also open an issue in the [Copilot in the CLI extension repository](https://github.com/github/gh-copilot) if you encounter any problems or have suggestions for improvement.

## Further Reading

- [Copilot in the CLI extension README](https://github.com/github/gh-copilot?tab=readme-ov-file)
- [Configuring GitHub Copilot in the CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli/configuring-github-copilot-in-the-cli)
- [Using GitHub Copilot in the command line](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-in-the-command-line)
- [Using GitHub Copilot to create issues](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-to-create-issues)
- [Finding public code that matches GitHub Copilot suggestions](https://docs.github.com/en/copilot/using-github-copilot/finding-public-code-that-matches-github-copilot-suggestions)
