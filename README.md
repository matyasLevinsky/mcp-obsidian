# MCP server for Obsidian

MCP server to interact with Obsidian via the Local REST API community plugin.

<a href="https://glama.ai/mcp/servers/3wko1bhuek"><img width="380" height="200" src="https://glama.ai/mcp/servers/3wko1bhuek/badge" alt="server for Obsidian MCP server" /></a>

## Components

### Tools

The server implements multiple tools to interact with Obsidian:

- list_files_in_vault: Lists all files and directories in the root directory of your Obsidian vault
- list_files_in_dir: Lists all files and directories in a specific Obsidian directory
- get_file_contents: Return the content of a single file in your vault.
- search: Simple text search that returns file paths and basic context snippets (fast, lightweight)
- fulltext_search: Advanced fulltext search with regex support, file filtering, path restrictions, and detailed context extraction
- patch_content: Insert content into an existing note relative to a heading, block reference, or frontmatter field.
- append_content: Append content to a new or existing file in the vault.
- delete_file: Delete a file or directory from your vault.

### Search Tool Comparison

**Simple Search (`search`):**
- **Purpose**: Quick text search across all files
- **Returns**: File paths with basic context snippets and match positions
- **Best for**: Finding files that contain specific terms
- **Performance**: Fast, suitable for large vaults
- **Limitations**: No regex, no file filtering, basic context

**Advanced Fulltext Search (`fulltext_search`):**
- **Purpose**: Comprehensive search with advanced filtering and context
- **Returns**: Detailed matches with precise positioning, line numbers, and rich context
- **Best for**: Complex searches, pattern matching, detailed analysis
- **Performance**: More intensive - may be slow on vaults with 1M+ words
- **Features**: Regex support, file filtering, path restrictions, configurable context

⚠️ **Performance Warning**: The `fulltext_search` tool processes file contents in detail and may be slow on large vaults (1,000,000+ words). For simple text searches, use the basic `search` tool instead.

**Quick Decision Guide:**
- Need file paths? → Use `search`
- Need regex patterns? → Use `fulltext_search` 
- Need to search specific folders/file types? → Use `fulltext_search`
- Large vault (1M+ words) and simple query? → Use `search`
- Need detailed context and match positioning? → Use `fulltext_search`

#### Advanced Fulltext Search Details

The `fulltext_search` tool provides powerful search capabilities beyond the basic search:

**Key Features:**
- **Regex Pattern Matching**: Search for complex patterns like email addresses, phone numbers, URLs
- **File Type Filtering**: Limit search to specific extensions (.md, .txt, .pdf) or search all files
- **Path Restrictions**: Search only within specific vault folders (e.g., "work/", "notes/meetings/")
- **Context Control**: Configurable snippet size around matches (50-1000 characters)
- **Case Sensitivity**: Optional case-sensitive or case-insensitive matching
- **Precise Match Positioning**: Exact character positions within snippets

**Parameters:**
- `query` (required): Search text or regex pattern
- `context_window`: Characters around matches (default: 200)
- `use_regex`: Enable regex pattern matching (default: false)
- `path`: Restrict to vault subfolder (optional)
- `file_extension`: File type filter (default: ".md")
- `case_sensitive`: Case matching control (default: false)

**Example Use Cases:**
```
Find all email addresses:
- Query: "\w+@\w+\.\w+"
- use_regex: true

Search in specific folder:
- Query: "meeting agenda"
- path: "work/meetings/"

Find TODOs in text files:
- Query: "TODO"
- file_extension: ".txt"
- case_sensitive: true

Get more context:
- Query: "important decision"
- context_window: 300
```

### Example prompts

Its good to first instruct Claude to use Obsidian. Then it will always call the tool.

**For Simple Search (fast, basic results):**
- "Search for files mentioning Azure CosmosDb and list them"
- "Find all notes that contain 'meeting agenda' and show me the file paths"
- "Quick search for 'project deadline' across my vault"

**For Advanced Fulltext Search (detailed, feature-rich):**
- "Use fulltext search to find all email addresses in my notes using regex pattern"
- "Search for 'API documentation' only in the work/ folder with more context around matches"
- "Find all TODO items in .txt files across my entire vault"
- "Use regex to find all phone numbers in my contacts folder"
- "Search for 'important decision' with 300 characters of context"

**For General Tasks:**
- Get the contents of the last architecture call note and summarize them
- Summarize the last meeting notes and put them into a new note 'summary meeting.md'. Add an introduction so that I can send it via email.

## Configuration

### Obsidian REST API Key

There are two ways to configure the environment with the Obsidian REST API Key. 

1. Add to server config (preferred)

```json
{
  "mcp-obsidian": {
    "command": "uvx",
    "args": [
      "mcp-obsidian"
    ],
    "env": {
      "OBSIDIAN_API_KEY": "<your_api_key_here>",
      "OBSIDIAN_HOST": "<your_obsidian_host>",
      "OBSIDIAN_PORT": "<your_obsidian_port>"
    }
  }
}
```
Sometimes Claude has issues detecting the location of uv / uvx. You can use `which uvx` to find and paste the full path in above config in such cases.

2. Create a `.env` file in the working directory with the following required variables:

```
OBSIDIAN_API_KEY=your_api_key_here
OBSIDIAN_HOST=your_obsidian_host
OBSIDIAN_PORT=your_obsidian_port
```

Note:
- You can find the API key in the Obsidian plugin config
- Default port is 27124 if not specified
- Default host is 127.0.0.1 if not specified
- **For fulltext search functionality**: Requires Obsidian Local REST API plugin with `/search/fulltext/` endpoint support

## Quickstart

### Install

#### Obsidian REST API

You need the Obsidian REST API community plugin running: https://github.com/coddingtonbear/obsidian-local-rest-api

Install and enable it in the settings and copy the api key.

#### Claude Desktop

On MacOS: `~/Library/Application\ Support/Claude/claude_desktop_config.json`

On Windows: `%APPDATA%/Claude/claude_desktop_config.json`

<details>
  <summary>Development/Unpublished Servers Configuration</summary>
  
```json
{
  "mcpServers": {
    "mcp-obsidian": {
      "command": "uv",
      "args": [
        "--directory",
        "<dir_to>/mcp-obsidian",
        "run",
        "mcp-obsidian"
      ],
      "env": {
        "OBSIDIAN_API_KEY": "<your_api_key_here>",
        "OBSIDIAN_HOST": "<your_obsidian_host>",
        "OBSIDIAN_PORT": "<your_obsidian_port>"
      }
    }
  }
}
```
</details>

<details>
  <summary>Published Servers Configuration</summary>
  
```json
{
  "mcpServers": {
    "mcp-obsidian": {
      "command": "uvx",
      "args": [
        "mcp-obsidian"
      ],
      "env": {
        "OBSIDIAN_API_KEY": "<YOUR_OBSIDIAN_API_KEY>",
        "OBSIDIAN_HOST": "<your_obsidian_host>",
        "OBSIDIAN_PORT": "<your_obsidian_port>"
      }
    }
  }
}
```
</details>

## Development

### Building

To prepare the package for distribution:

1. Sync dependencies and update lockfile:
```bash
uv sync
```

### Debugging

Since MCP servers run over stdio, debugging can be challenging. For the best debugging
experience, we strongly recommend using the [MCP Inspector](https://github.com/modelcontextprotocol/inspector).

You can launch the MCP Inspector via [`npm`](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) with this command:

```bash
npx @modelcontextprotocol/inspector uv --directory /path/to/mcp-obsidian run mcp-obsidian
```

Upon launching, the Inspector will display a URL that you can access in your browser to begin debugging.

You can also watch the server logs with this command:

```bash
tail -n 20 -f ~/Library/Logs/Claude/mcp-server-mcp-obsidian.log
```
