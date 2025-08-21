# Table of Contents

- [Table of Contents](#table-of-contents)
- [Config](#config)
- [Output Style](#output-style)
  - [Type](#type)
  - [Output Styles vs. CLAUDE.md vs. —append-system-prompt](#output-styles-vs-claudemd-vs-append-system-prompt)
  - [Output Styles vs. Custom Slash Commands](#output-styles-vs-custom-slash-commands)
  - [Output Styles vs. Agents](#output-styles-vs-agents)
- [Hooks](#hooks)
  - [Benefits](#benefits)
  - [Example use cases for hooks include:](#example-use-cases-for-hooks-include)
  - [Hook Events](#hook-events)
  - [Configuration](#configuration)
- [MCP](#mcp)
  - [MCP Installation Scopes](#mcp-installation-scopes)
    - [Local scope](#local-scope)
    - [Project scope](#project-scope)
    - [User scope](#user-scope)
  - [Scope Precedence](#scope-precedence)
  - [3 Ways to Add/Install MCP Server](#3-ways-to-addinstall-mcp-server)
    - [Option 1: Add a local stdio server](#option-1-add-a-local-stdio-server)
    - [Option 2: Add a remote SSE server](#option-2-add-a-remote-sse-server)
    - [Option 3: Add a remote HTTP server](#option-3-add-a-remote-http-server)
  - [Manage Servers](#manage-servers)
    - [Important Tips on Add \& Manage MCP Servers](#important-tips-on-add--manage-mcp-servers)
  - [Note](#note)
  - [Authenticate MCP servers](#authenticate-mcp-servers)
  - [Environment variable expansion in .mcp.json](#environment-variable-expansion-in-mcpjson)
  - [Other ways to add MCP Servers](#other-ways-to-add-mcp-servers)
    - [Add MCP servers from JSON configuration](#add-mcp-servers-from-json-configuration)
    - [Import MCP servers from Claude Desktop](#import-mcp-servers-from-claude-desktop)
  - [Use Claude Code as an MCP server](#use-claude-code-as-an-mcp-server)
  - [Reference MCP resources](#reference-mcp-resources)
  - [Use MCP prompts as slash commands](#use-mcp-prompts-as-slash-commands)
- [Command](#command)
  - [`/init`](#init)
  - [`/add-dir`](#add-dir)
  - [`/hooks`](#hooks-1)
  - [`/mcp`](#mcp-1)
  - [`/output-style`](#output-style-1)
  - [`/config`](#config-1)


# Config
- User level: `~/.claude/output-styles`
  - Used across projects
- Project level: `.claude/output-styles`

# Output Style
## Type
- __Default__: System prompt (used for software engineering)
- __Explanatory__: Provides educational “Insights” in between helping you complete software engineering tasks. Helps you understand implementation choices and codebase patterns.
- __Learning__: Collaborative, learn-by-doing mode where Claude will not only share “Insights” while coding, but also ask you to contribute small, strategic pieces of code yourself. Claude Code will add `TODO(human)` markers in your code for you to implement.

## Output Styles vs. CLAUDE.md vs. —append-system-prompt
- Output styles completely “turn off” the parts of Claude Code’s default system prompt specific to software engineering.
- Neither CLAUDE.md nor --append-system-prompt edit Claude Code’s default system prompt.
- CLAUDE.md adds the contents as a user message following Claude Code’s default system prompt.
- `--append-system-prompt` appends the content to the system prompt.

## Output Styles vs. Custom Slash Commands
- Think of output styles as “stored system prompts”
- and custom slash commands as “stored prompts”.

## Output Styles vs. Agents
- Output styles directly affect the main agent loop and only affect the system prompt.
- Agents are invoked to handle specific tasks and can include additional settings like the model to use, the tools they have available, and some context about when to use the agent.

# Hooks
## Benefits
Claude Code hooks are user-defined shell commands that execute at various points in Claude Code’s lifecycle. Hooks provide deterministic control over Claude Code’s behavior, ensuring certain actions always happen rather than relying on the LLM to choose to run them

By encoding these rules as hooks rather than prompting instructions, you turn suggestions into app-level code that executes every time it is expected to run.

## Example use cases for hooks include:
- __Notifications__: Customize how you get notified when Claude Code is awaiting your input or permission to run something.
- __Automatic formatting__: Run prettier on .ts files, gofmt on .go files, etc. after every file edit.
- __Logging__: Track and count all executed commands for compliance or debugging.
- __Feedback__: Provide automated feedback when Claude Code produces code that does not follow your codebase conventions.
- __Custom permissions__: Block modifications to production files or sensitive directories.

## Hook Events
 hook events that run at different points in the workflow:

- __PreToolUse__: Runs before tool calls (can block them)
- __PostToolUse__: Runs after tool calls complete
- __UserPromptSubmit__: Runs when the user submits a prompt, before Claude processes it
- __Notification__: Runs when Claude Code sends notifications
- __Stop__: Runs when Claude Code finishes responding
- __Subagent Stop__: Runs when subagent tasks complete
- __PreCompact__: Runs before Claude Code is about to run a compact operation
- __SessionStart__: Runs when Claude Code starts a new session or resumes an existing session

Each event receives different data and can control Claude’s behavior in different ways.

## Configuration
Claude Code hooks are configured in your settings files:

~/.claude/settings.json - User settings
.claude/settings.json - Project settings
.claude/settings.local.json - Local project settings (not committed)
Enterprise managed policy settings



# MCP

## MCP Installation Scopes
MCP servers can be configured at three different scope levels, each serving distinct purposes for managing server accessibility and sharing.

### Local scope
Local-scoped servers represent the default configuration level and are stored in your project-specific user settings. These servers remain private to you and are only accessible when working within the current project directory. This scope is ideal for personal development servers, experimental configurations, or servers containing sensitive credentials that shouldn’t be shared.

```shell
# Add a local-scoped server (default)
claude mcp add my-private-server /path/to/server

# Explicitly specify local scope
claude mcp add my-private-server --scope local /path/to/server
```


### Project scope
Project-scoped servers enable team collaboration by storing configurations in a .mcp.json file at your project’s root directory. This file is designed to be checked into version control, ensuring all team members have access to the same MCP tools and services. When you add a project-scoped server, Claude Code automatically creates or updates this file with the appropriate configuration structure.

```shell
# Add a project-scoped server
claude mcp add shared-server --scope project /path/to/server
```

The resulting .mcp.json file follows a standardized format:

```shell
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

For security reasons, Claude Code prompts for approval before using project-scoped servers from .mcp.json files. If you need to reset these approval choices, use the `claude mcp reset-project-choices` command.

### User scope
User-scoped servers provide cross-project accessibility, making them available across all projects on your machine while remaining private to your user account. This scope works well for personal utility servers, development tools, or services you frequently use across different projects.

```shell
# Add a user server
claude mcp add my-user-server --scope user /path/to/server
```

## Scope Precedence

When servers with the same name exist at multiple scopes, the system resolves conflicts by prioritizing local-scoped servers first, followed by project-scoped servers, and finally user-scoped servers.

```text
Local > Project > User
```



If a required environment variable is not set and has no default value, Claude Code will fail to parse the config.



## 3 Ways to Add/Install MCP Server
### Option 1: Add a local stdio server
Stdio servers run as local processes on your machine. They’re ideal for tools that need direct system access or custom scripts.

``` shell
# Basic syntax
claude mcp add <name> <command> [args...]

# Real example: Add Airtable server
claude mcp add airtable --env AIRTABLE_API_KEY=YOUR_KEY \
  -- npx -y airtable-mcp-server
```

Understanding the ”—” parameter: The -- (double dash) separates Claude’s own CLI flags from the command and arguments that get passed to the MCP server. Everything before -- are options for Claude (like --env, --scope), and everything after -- is the actual command to run the MCP server.

For example:

- `claude mcp add myserver -- npx server` → runs npx server
- `claude mcp add myserver --env KEY=value -- python server.py --port 8080` → runs python server.py --port 8080 with KEY=value in environment

This prevents conflicts between Claude’s flags and the server’s flags.

### Option 2: Add a remote SSE server
SSE (Server-Sent Events) servers provide real-time streaming connections. Many cloud services use this for live updates.

```shell
# Basic syntax
claude mcp add --transport sse <name> <url>

# Real example: Connect to Linear
claude mcp add --transport sse linear https://mcp.linear.app/sse

# Example with authentication header
claude mcp add --transport sse private-api https://api.company.com/mcp \
  --header "X-API-Key: your-key-here"
```
### Option 3: Add a remote HTTP server
HTTP servers use standard request/response patterns. Most REST APIs and web services use this transport.

```shell
# Basic syntax
claude mcp add --transport http <name> <url>

# Real example: Connect to Notion
claude mcp add --transport http notion https://mcp.notion.com/mcp

# Example with Bearer token
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

## Manage Servers
```shell
# List all configured servers
claude mcp list

# Get details for a specific server
claude mcp get github

# Remove a server
claude mcp remove github

# (within Claude Code) Check server status
/mcp
```

### Important Tips on Add & Manage MCP Servers
- Use the --scope flag to specify where the configuration is stored:
  - local (default): Available only to you in the current project (was called project in older versions)
  - project: Shared with everyone in the project via .mcp.json file
  - user: Available to you across all projects (was called global in older versions)
- Set environment variables with `--env` flags (e.g., --env KEY=value)
- Configure MCP server startup timeout using the `MCP_TIMEOUT` environment variable (e.g., `MCP_TIMEOUT=10000` claude sets a 10-second timeout)
- Use /mcp to authenticate with remote servers that require OAuth 2.0 authentication

## Note
Windows Users: On native Windows (not WSL), local MCP servers that use npx require the cmd /c wrapper to ensure proper execution.

```shell
# This creates command="cmd" which Windows can execute
claude mcp add my-server -- cmd /c npx -y @some/package
```

Without the `cmd /c` wrapper, you’ll encounter “Connection closed” errors because Windows cannot directly execute npx. (See the note above for an explanation of the -- parameter.)

## Authenticate MCP servers

1. Add the server that requires authentication

For example:

```shell
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```
2. Use the /mcp command within Claude Code
In Claude code, use the command:

```shell
> /mcp
```
3. Then follow the steps in your browser to login.


## Environment variable expansion in .mcp.json
Claude Code supports environment variable expansion in .mcp.json files, allowing teams to share configurations while maintaining flexibility for machine-specific paths and sensitive values like API keys.

Supported syntax:

```text
${VAR} - Expands to the value of environment variable VAR
${VAR:-default} - Expands to VAR if set, otherwise uses default
```

Expansion locations: Environment variables can be expanded in:

- `command` - The server executable path
- `args` - Command-line arguments
- `env` - Environment variables passed to the server
- `url` - For SSE/HTTP server types
- `headers` - For SSE/HTTP server authentication

Example with variable expansion:

```JSON
{
  "mcpServers": {
    "api-server": {
      "type": "sse",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

## Other ways to add MCP Servers
### Add MCP servers from JSON configuration
If you have a JSON configuration for an MCP server, you can add it directly:

1. Add an MCP server from JSON

``` shell
# Basic syntax
claude mcp add-json <name> '<json>'

# Example: Adding a stdio server with JSON configuration
claude mcp add-json weather-api '{"type":"stdio","command":"/path/to/weather-cli","args":["--api-key","abc123"],"env":{"CACHE_DIR":"/tmp"}}'
```

2. Verify the server was added

``` shell
claude mcp get weather-api
```

> Tips:
>- Make sure the JSON is properly escaped in your shell
>- he JSON must conform to the MCP server configuration schema
>- Use `--scope` user to add the server to your user configuration instead of the project-specific one

### Import MCP servers from Claude Desktop
If you've already configured MCP servers in Claude Desktop, you can import them:
[Learn more about importing MCP servers from Claude Desktop](https://docs.anthropic.com/en/docs/claude-code/mcp#import-mcp-servers-from-claude-desktop)

## Use Claude Code as an MCP server
You can use Claude Code itself as an MCP server that other applications can connect to:

``` shell
# Start Claude as a stdio MCP server
claude mcp serve
```
You can use this in Claude Desktop by adding this configuration to claude_desktop_config.json:

``` JSON
{
  "mcpServers": {
    "claude-code": {
      "command": "claude",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
```

> **Tips:**
>
> - The server provides access to Claude's tools like View, Edit, LS, etc.
> - In Claude Desktop, try asking Claude to read files in a directory, make edits, and more.
> - Note that this MCP server is simply exposing Claude Code's tools to your MCP client, so your own client is responsible for implementing user confirmation for individual tool calls.

## Reference MCP resources
1. List available resources

    Type `@` in your prompt to see available resources from all connected MCP servers. Resources appear alongside files in the autocomplete menu.

2. Reference a specific resource

    Use the format `@server:protocol://resource/path` to reference a resource:


    ``` text
    > Can you analyze @github:issue://123 and suggest a fix?


    > Please review the API documentation at @docs:file://api/authentication
    ```

3. Multiple resource references

    You can reference multiple resources in a single prompt:

    ``` text
    > Compare @postgres:schema://users with @docs:file://database/user-model
    ```

> Tips:
> - Resources are automatically fetched and included as attachments when referenced
> - Resource paths are fuzzy-searchable in the @ mention autocomplete
> - Claude Code automatically provides tools to list and read MCP resources when servers support them
> - Resources can contain any type of content that the MCP server provides (text, JSON, structured data, etc.)

## Use MCP prompts as slash commands
MCP servers can expose prompts that become available as slash commands in Claude Code.

​
Execute MCP prompts
1. Discover available prompts

    Type `/` to see all available commands, including those from MCP servers. MCP prompts appear with the format `/mcp__servername__promptname.`

2. Execute a prompt without arguments

    ``` text
    > /mcp__github__list_prs
    ```

3. Execute a prompt with arguments

    Many prompts accept arguments. Pass them space-separated after the command:


    ```text
    > /mcp__github__pr_review 456

    > /mcp__jira__create_issue "Bug in login flow" high
    ```

> Tips:
> - MCP prompts are dynamically discovered from connected servers
> - Arguments are parsed based on the prompt’s defined parameters
> - Prompt results are injected directly into the conversation
> - Server and prompt names are normalized (spaces become underscores)





# Command
## `/init`
## `/add-dir`
## `/hooks`
## `/mcp`
  - Usage:
    1. `/mcp list`: list all installed MCP Servers
    2. `/mcp add-json`
    3. `claude mcp add-from-claude-desktop`: Only work on MacOS and Windows WSL.
## `/output-style`
  - to access the menu and select your output style (this can also be accessed from the /config menu)
  - Usage:
    1.  `/output-style [style]`: example: `/output-style explanatory`, to directly switch to a style
    2.  `/output-style:new`: to setup a new output style
        - By default, output styles created through /output-style:new are saved as markdown files at the user level in `~/.claude/output-styles` and can be used across projects
        - can also create your own output style Markdown files and save them either at the user level (~/.claude/output-styles) or the project level (.claude/output-styles).
    3. `/output-style:new I want an output style that ...`: set up a new output style with Claude’s help
## `/config`
