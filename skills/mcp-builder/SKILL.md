---
name: mcp-builder
description: Use when building a new MCP server, adding tools or resources to an existing MCP server, or wiring Claude to a custom MCP server for the first time
---

# Building MCP Servers

## Overview

MCP (Model Context Protocol) servers extend Claude with custom tools, resources, and prompts. Build one when Claude's built-in tools don't cover your domain.

**Core principle:** Design tools for Claude's reasoning loop, not general-purpose APIs.

## When to Use

**Build an MCP server when:**
- You need domain-specific tools (internal APIs, databases, proprietary services)
- The same tool set will be reused across sessions or projects

**Don't build when:**
- A built-in tool (Bash, Read, WebFetch) already does the job
- The tool is one-session-only — use in-process tools via the Agent SDK instead

## Setup

```bash
mkdir my-mcp-server && cd my-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk zod
```

## Minimal Server

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.tool(
  "get-item",
  "Fetch an item by ID",
  { id: z.string().describe("The item ID") },
  async ({ id }) => {
    const result = await fetchItem(id); // your implementation
    return { content: [{ type: "text", text: JSON.stringify(result) }] };
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

**Use stdio transport** — not HTTP. Claude CLI expects servers to communicate over stdin/stdout.

## Quick Reference

| Primitive | Registration |
|-----------|-------------|
| Tool (function Claude calls) | `server.tool(name, desc, zodSchema, handler)` |
| Resource (data Claude reads) | `server.resource(name, uriTemplate, handler)` |
| Prompt template | `server.prompt(name, desc, args, handler)` |

**Handler return format:**
```typescript
// Success
return { content: [{ type: "text", text: "result" }] };

// Error
return { content: [{ type: "text", text: "Error: reason" }], isError: true };
```

## Configure Claude Code

Add to `~/.claude/settings.json` — use the `update-config` skill to edit this safely:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["dist/index.js"],
      "cwd": "/absolute/path/to/my-mcp-server"
    }
  }
}
```

For an npx-distributed server:
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

Restart Claude Code after editing settings.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using HTTP transport instead of stdio | Replace with `StdioServerTransport` |
| Relative path in `cwd` | Use absolute paths — Claude starts the process from a different working directory |
| Editing settings.json manually without restarting | Always restart Claude Code after config changes |
| One giant tool that does everything | Split into focused tools with clear names — Claude selects tools by description |
| Returning raw objects instead of `content` array | Always return `{ content: [{ type: "text", text: string }] }` |
