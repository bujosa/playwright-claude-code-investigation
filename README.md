# Playwright CLI + Claude Code: UI Testing Investigation

This repository documents the setup and usage of [Playwright CLI](https://github.com/bujosa/playwright-cli) integrated with [Claude Code](https://claude.ai/claude-code) for automated UI review and testing.

## Overview

Playwright CLI is a lightweight command-line wrapper around Playwright's browser automation capabilities, designed to be **token-efficient for AI coding agents**. Instead of running as an MCP server with large tool schemas, it exposes browser automation as simple shell commands — keeping context usage minimal.

### Why Playwright CLI over MCP?

| Aspect | Playwright CLI | MCP Browser Tools |
|--------|---------------|-------------------|
| Token usage | Low (shell commands) | High (tool schemas + accessibility trees) |
| Integration | Claude Code Skills | MCP Server |
| Setup | `npm install` + `install --skills` | Server configuration |
| Flexibility | Direct CLI commands | Structured tool calls |

## Prerequisites

- **Node.js** >= 18
- **Claude Code** CLI installed
- **Google Chrome** (or another supported browser)
- **GitHub CLI** (`gh`) for repo management (optional)

## Installation

### 1. Install Playwright CLI

```bash
npm install -g @playwright/cli@latest
```

Verify the installation:

```bash
playwright-cli --version
```

### 2. Register Skills in Claude Code

Navigate to your project root and run:

```bash
playwright-cli install --skills
```

This will:
- Initialize a `.playwright` workspace directory
- Install skill definitions to `.claude/skills/playwright-cli/`
- Auto-detect your default browser

Expected output:
```
Workspace initialized at `/path/to/project`.
Skills installed to `.claude/skills/playwright-cli`.
Found chrome, will use it as the default browser.
```

### 3. Verify Setup

Check that the skill file was created:

```bash
ls .claude/skills/playwright-cli/SKILL.md
```

## Usage

### Basic Workflow

```bash
# Open a browser and navigate to your app
playwright-cli open http://localhost:3000

# Take a snapshot (YAML representation of the page with element refs)
playwright-cli snapshot

# Interact with elements using refs from the snapshot
playwright-cli click e3
playwright-cli fill e5 "test@example.com"

# Take a screenshot
playwright-cli screenshot --filename=homepage.png

# Close the browser
playwright-cli close
```

### UI Review Workflow

When reviewing UI changes in a project:

```bash
# 1. Open the app
playwright-cli open http://localhost:3000

# 2. Take a screenshot of the current state
playwright-cli screenshot --filename=before.png

# 3. Navigate to the changed page/component
playwright-cli goto http://localhost:3000/changed-page

# 4. Take a snapshot to inspect the DOM structure
playwright-cli snapshot --filename=changed-page.yaml

# 5. Take a screenshot for visual comparison
playwright-cli screenshot --filename=after.png

# 6. Test interactions
playwright-cli click e7
playwright-cli snapshot

# 7. Close
playwright-cli close
```

### Multi-Tab Testing

```bash
playwright-cli open http://localhost:3000
playwright-cli tab-new http://localhost:3000/dashboard
playwright-cli tab-list
playwright-cli tab-select 0
playwright-cli screenshot --filename=home.png
playwright-cli tab-select 1
playwright-cli screenshot --filename=dashboard.png
playwright-cli close
```

### Form Testing

```bash
playwright-cli open http://localhost:3000/login
playwright-cli snapshot
playwright-cli fill e1 "user@example.com"
playwright-cli fill e2 "password123"
playwright-cli click e3
playwright-cli snapshot
playwright-cli screenshot --filename=after-login.png
playwright-cli close
```

### Network Mocking

```bash
playwright-cli open http://localhost:3000
playwright-cli route "https://api.example.com/**" --body='{"mock": true}'
playwright-cli reload
playwright-cli snapshot
playwright-cli unroute
playwright-cli close
```

## Claude Code Integration

Once the skills are installed, Claude Code can use `playwright-cli` commands directly within a conversation. The skill definition in `.claude/skills/playwright-cli/SKILL.md` tells Claude Code:

- What commands are available
- How to interpret snapshots (YAML with element refs like `e15`)
- How to chain commands for complex workflows

### Example: Asking Claude Code to Review UI

You can ask Claude Code (via terminal or Telegram):

> "Open my app at localhost:3000, navigate to the settings page, and take a screenshot. Check if the form fields are properly aligned."

Claude Code will then execute the appropriate `playwright-cli` commands to fulfill the request.

## Command Reference

| Category | Commands |
|----------|----------|
| **Navigation** | `open`, `goto`, `go-back`, `go-forward`, `reload`, `close` |
| **Interaction** | `click`, `dblclick`, `type`, `fill`, `drag`, `hover`, `select`, `upload`, `check`, `uncheck` |
| **Inspection** | `snapshot`, `eval`, `console`, `network` |
| **Capture** | `screenshot`, `pdf`, `tracing-start`, `tracing-stop`, `video-start`, `video-stop` |
| **Keyboard** | `press`, `keydown`, `keyup` |
| **Mouse** | `mousemove`, `mousedown`, `mouseup`, `mousewheel` |
| **Tabs** | `tab-list`, `tab-new`, `tab-close`, `tab-select` |
| **Storage** | `cookie-*`, `localstorage-*`, `sessionstorage-*`, `state-save`, `state-load` |
| **Sessions** | `list`, `close-all`, `kill-all` |
| **Network** | `route`, `route-list`, `unroute` |

## File Structure

```
project/
├── .claude/
│   └── skills/
│       └── playwright-cli/
│           ├── SKILL.md          # Skill definition (auto-generated)
│           └── references/       # Additional command docs
├── .playwright/
│   └── cli.config.json           # Optional configuration
└── ...
```

## Configuration

Create `.playwright/cli.config.json` for custom settings:

```json
{
  "browser": "chrome",
  "timeout": 30000,
  "viewport": {
    "width": 1920,
    "height": 1080
  }
}
```

Environment variables prefixed with `PLAYWRIGHT_MCP_` override config file values.

## Resources

- [Playwright CLI Repository](https://github.com/bujosa/playwright-cli)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Playwright Documentation](https://playwright.dev)

## License

MIT
