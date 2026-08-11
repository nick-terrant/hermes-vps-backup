---
name: coding-agents
description: "Orchestrate autonomous AI coding agents: Claude Code, Codex CLI, OpenCode, Hermes Agent."
version: 1.0.0
author: Hermes Agent (consolidated from claude-code, hermes-agent)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [Coding Agents, Claude Code, Codex, OpenCode, Hermes Agent, Autonomous AI, AI Pair Programming]
    supersedes: [claude-code, hermes-agent]
---

# Coding Agent Orchestration

Delegate coding tasks to autonomous AI agents. This skill covers spawning, configuring, and working with external coding agents (Claude Code, Codex CLI, OpenCode) and internal Hermes Agent instances.

**Pick the section for your agent:**
- [Section I: Claude Code](#section-i-claude-code) — Anthropic's autonomous coding agent CLI
- [Section II: Codex CLI](#section-ii-codex-cli-openai) — OpenAI's terminal-based coding agent
- [Section III: OpenCode](#section-iii-opencode) — Open-source coding agent alternative
- [Section IV: Hermes Agent](#section-iv-hermes-agent) — Spawn and configure Hermes Agent instances

---

# Section I: Claude Code

Anthropic's autonomous coding agent. Works in project directories, reads files, runs commands, edits code.

## Setup

```bash
# Install (requires Node.js 18+)
npm install -g @anthropic-ai/claude-code

# Verify
claude --version
```

## Common Usage Patterns

```bash
# Interactive session (starts in current directory)
claude

# Non-interactive with prompt
claude -p "Add error handling to the API routes"

# Read specific files for context
claude -p "Review src/api.ts for security issues" --files src/api.ts src/auth.ts

# Use specific model
claude --model claude-sonnet-4-20250514

# Auto-accept file edits (for trusted repos)
claude --dangerously-skip-permissions -p "Refactor the database layer"
```

## Key Features

| Feature | Description |
|---------|-------------|
| **File awareness** | Automatically reads project files, understands structure |
| **Command execution** | Runs shell commands, tests, builds |
| **Git integration** | Understands diffs, can commit, create PRs |
| **Multi-file edits** | Can edit multiple files in a single task |
| **Conversation** | Interactive multi-turn sessions |

## Best Practices

1. **Give clear, specific tasks** — "Fix the null pointer in UserService.getUser()" not "fix bugs"
2. **Provide context** — Use `--files` to include relevant source files
3. **Verify changes** — Always review Claude Code's edits before committing
4. **Use `--dangerously-skip-permissions` only in trusted sandboxes**
5. **Break large tasks into smaller steps** — Claude Code works best with focused tasks

## When to Use Claude Code

- Refactoring, bug fixes, feature implementation
- Code review and analysis
- Writing tests
- Documentation generation
- Any task that requires reading/editing code in a project

## When NOT to Use Claude Code

- Simple single-command operations (just use `terminal`)
- Tasks requiring browser interaction (use Hermes browser tools directly)
- When you need fine-grained control over each edit

---

# Section II: Codex CLI (OpenAI)

OpenAI's terminal-based autonomous coding agent.

## Setup

```bash
# Install (requires Node.js 18+)
npm install -g @openai/codex

# Authenticate
codex auth login
```

## Common Usage

```bash
# Interactive session
codex

# Non-interactive
codex -q "Implement user authentication with JWT"

# Use specific model
codex --model gpt-4.1
```

## Key Differences from Claude Code

| Aspect | Claude Code | Codex CLI |
|--------|------------|-----------|
| Provider | Anthropic | OpenAI |
| Auth | `claude` CLI auth | `codex auth login` |
| Permissions | Granular file-level | Project-level |
| Best for | Deep reasoning, careful edits | Quick implementation, broad tasks |

---

# Section III: OpenCode

Open-source terminal coding agent alternative. Good when you want a self-hosted or customizable option.

## Setup

```bash
# Install via Go
go install github.com/opencode-ai/opencode@latest

# Or via npm
npm install -g opencode

# Configure (uses OpenAI-compatible API by default)
opencode config set api-key $OPENAI_API_KEY
```

## Usage

```bash
# Interactive
opencode

# With custom provider
opencode --provider anthropic --api-key $ANTHROPIC_API_KEY

# Non-interactive
opencode -q "Add unit tests for the calculator module"
```

---

# Section IV: Hermes Agent

Spawn and configure additional Hermes Agent instances for parallel or delegated work.

## Spawning Hermes Instances

Use Hermes' built-in agent spawning to delegate subtasks to isolated contexts:

- **`delegate_task`** — Spawn subagents for parallel work. Each gets its own terminal, conversation, and toolset. Results summarized back to parent.
- **`hermes spawn`** — CLI command to spawn standalone Hermes instances.
- **Worktree isolation** — Each spawned agent can work in an isolated git worktree.

## Configuration

Hermes Agent is configured via `~/.hermes/config.yaml`:

```yaml
agent:
  model: anthropic/claude-sonnet-4
  system_prompt: "You are an expert Python developer."

# MCP servers for external tool integration
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_TOKEN: "***

# Profiles for different configurations
profiles:
  python-dev:
    model: anthropic/claude-sonnet-4
    system_prompt: "Python expert"
```

## Common CLI Commands

```bash
# Check configuration
hermes config show

# Set a config value
hermes config set agent.model anthropic/claude-sonnet-4

# List available tools
hermes tools

# Setup wizard
hermes setup
```

## Profiles

Profiles allow multiple Hermes configurations for different use cases:

```bash
# Create a profile
hermes profile create python-dev

# Use a profile
hermes --profile python-dev

# List profiles
hermes profile list
```

## MCP Server Integration

Connect external tools via MCP (Model Context Protocol):

```yaml
mcp_servers:
  filesystem:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_TOKEN: "***  # Use env var reference
```

See `references/mcp-server-setup.md` for detailed MCP configuration guide.

## Skill Authoring

Create reusable skills for recurring task patterns:

```bash
# Create a skill
hermes skills create my-skill

# Skills live in ~/.hermes/skills/
# Each skill has SKILL.md with YAML frontmatter + markdown body
```

See `references/skill-authoring.md` for skill conventions and best practices.

## Gateway and Webhooks

Hermes can be configured as a gateway for external integrations:

- **Webhook subscriptions** for receiving external events
- **Container supervision** for managing long-running tasks
- **Multi-platform messaging** (Telegram, Discord, etc.)

See `references/` for detailed guides on these features.

## References

| File | Contents |
|------|----------|
| `references/mcp-server-setup.md` | MCP server configuration patterns |
| `references/skill-authoring.md` | Skill conventions, SKILL.md format, check_fn |
| `references/webhook-subscriptions.md` | Webhook setup and patterns |
| `references/container-supervision.md` | Long-running task management |
