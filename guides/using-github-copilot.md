# Using GitHub Copilot with Orbit Framework

This guide explains how to effectively use GitHub Copilot (both in your editor and the CLI) to enhance your development experience with the Orbit Framework.

## GitHub Copilot Integration

Orbit Framework has been designed with AI-assisted development in mind. GitHub Copilot can be a powerful tool to help you write code faster and more efficiently when working with Orbit components, CLI tools, and the core framework.

## Setting Up GitHub Copilot

### In VS Code

1. Install the [GitHub Copilot extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot)
2. Sign in with your GitHub account that has Copilot access
3. Open an Orbit Framework project and start coding with Copilot suggestions

### In Command Line (CLI)

1. Install GitHub CLI: `brew install gh` (macOS) or see [installation instructions](https://github.com/cli/cli#installation)
2. Login to GitHub: `gh auth login`
3. Install the Copilot extension: `gh extension install github/gh-copilot`
4. Use the setup script: `./scripts/setup-copilot-cli.sh`

For detailed CLI setup, see [GitHub Copilot CLI Guide](tooling/github-copilot-cli.md).

## Best Practices for Using Copilot with Orbit

### Component Development

- Use Copilot to generate boilerplate for new Orbit components
- Ask Copilot to help with implementing event handlers and state management
- Get suggestions for component styling and layout

Example prompt:
```
Create an Orbit component for a user profile card that displays an avatar, name, and bio.
```

### Using with orbiton CLI

Copilot CLI can help you remember the correct orbiton commands:

```bash
ghcs "Create a new Orbit project"  # Will suggest: orbiton new my-project
ghcs "Run the development server"   # Will suggest: orbiton dev
```

### Static Analysis and Linting

Get help with fixing issues detected by orlint:

```bash
ghcs "Fix component lifecycle issues detected by orlint"
```

## Coding Patterns

GitHub Copilot has been trained on common Orbit Framework patterns. When writing code, try prompts like:

```
// Generate an Orbit component that manages a counter with increment and decrement buttons
```

```
// Implement a reactive state handler for a form submission in Orbit
```

```
// Create a layout for a responsive dashboard using Orbit's flexbox-like system
```

## Troubleshooting with Copilot

When you encounter errors or issues, use Copilot to help diagnose and fix them:

```bash
gh copilot explain "Error: Component lifecycle hook not called in the correct order"
```

```bash
gh copilot suggest "Debug why my Orbit component isn't rendering"
```

## Creating Issues with GitHub Copilot

GitHub Copilot can help you create well-structured issues faster. Instead of manually filling out every field, you can describe the issue in natural language or even upload a screenshot.

### How to Create Issues with Copilot

1. Go to the immersive view of Copilot Chat: [https://github.com/copilot](https://github.com/copilot)
2. In the "Ask Copilot" box at the bottom of the page, describe the issue you want to create. Specify the repository in `org/repository` format.

**Example prompts:**
```
Create a feature request for orbitrs/orbit to add dynamic component loading
```

```
Log a bug for orlint showing that linting fails on Windows paths
```

3. Copilot will draft an issue with a title, formatted body (using your repository's template), and suggest metadata like labels and assignees.
4. Review the draft and make any necessary edits.
5. Click "Create" when ready.

### Tips for Issue Creation

- Be specific about the type of issue (bug, feature request, etc.)
- Mention which component or part of the codebase is affected
- Include error messages or expected behavior in your description
- You can add screenshots by pasting them into the prompt box

## Finding Public Code that Matches Copilot Suggestions

GitHub Copilot can show references to public code that matches its suggestions. This helps you:

1. Verify the source and licensing of suggested code
2. Ensure proper attribution when required
3. Make informed decisions about using the suggested code

### How Code Referencing Works

When Copilot provides code that matches public repositories on GitHub:

1. **For code completion:** Information about matching code is logged in your editor
2. **For Copilot Chat:** A link appears at the end of the response to view matching code

### Viewing Code References

#### In VS Code:

1. Open the Output window (View > Output)
2. Select "GitHub Copilot Log (Code References)" from the dropdown
3. Accept a suggestion to see if any references appear in the log

#### In Copilot Chat:

When a response includes matching code, you'll see a link like:
```
Public code references from n repositories
```

Click this link to see details about the matching code and its license.

## Further Resources

- [GitHub Copilot Documentation](https://docs.github.com/en/copilot)
- [GitHub Copilot CLI Guide](tooling/github-copilot-cli.md)
- [Using GitHub Copilot to Create Issues](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-to-create-issues)
- [Finding Public Code that Matches GitHub Copilot Suggestions](https://docs.github.com/en/copilot/using-github-copilot/finding-public-code-that-matches-github-copilot-suggestions)
- [Orbit Framework Documentation](https://orbitrs.github.io/docs/)
