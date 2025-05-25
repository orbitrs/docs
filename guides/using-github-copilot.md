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

## Further Resources

- [GitHub Copilot Documentation](https://docs.github.com/en/copilot)
- [GitHub Copilot CLI Guide](tooling/github-copilot-cli.md)
- [Orbit Framework Documentation](https://orbitrs.github.io/docs/)
