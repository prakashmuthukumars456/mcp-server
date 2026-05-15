# MCP Protocol Fundamentals

## What is MCP?

Model Context Protocol (MCP) is an open protocol that standardizes how AI applications connect to data sources and tools.

## Core Concepts

### The 3-Layer Architecture

┌────────────────────────────────────────┐
│ Layer 1: MCP Client (Claude/Cursor)    │
│ - Understands user intent              │
│ - Decides which tools to call          │
│ - Formats natural language responses   │
└────────────────────────────────────────┘
↕ MCP Protocol (JSON-RPC 2.0)
┌────────────────────────────────────────┐
│ Layer 2: MCP Server (Your Code)        │
│ - Exposes tools, resources, prompts    │
│ - Implements business logic            │
│ - Handles auth, rate limiting          │
└────────────────────────────────────────┘
↕ HTTPS / SDK Calls
┌────────────────────────────────────────┐
│ Layer 3: External Service              │
│ - Datadog, JIRA, New Relic, etc.       │
│ - Actual data and functionality        │
└────────────────────────────────────────┘

## Protocol Messages

MCP uses JSON-RPC 2.0 format:

### Client → Server: List Tools

```json
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "params": {},
  "id": 1
}
```

### Server → Client: Tool Definitions

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "get_weather",
        "description": "Get current weather for a city",
        "inputSchema": {
          "type": "object",
          "properties": {
            "city": {"type": "string"}
          },
          "required": ["city"]
        }
      }
    ]
  }
}
```

### Client → Server: Call Tool

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "get_weather",
    "arguments": {"city": "Chennai"}
  },
  "id": 2
}
```

### Server → Client: Tool Result

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "32°C, humid, partly cloudy"
      }
    ]
  }
}
```

## The 5 Capabilities

| Capability | Purpose | Example |
|------------|---------|---------|
| **Tools** | Actions AI can execute | `create_issue()`, `query_metrics()` |
| **Prompts** | Workflow templates | "Investigate performance issue" |
| **Resources** | Read-only data | `nr://alert-policies` |
| **Roots** | Access boundaries | `file:///home/user/projects` |
| **Elicitation** | Server asks user | "Which environment?" |

## Why MCP?

### Before MCP

Every tool built custom integrations:

Claude Desktop → Custom GitHub API integration
Cursor → Different GitHub API integration
VSCode → Yet another GitHub API integration

### With MCP

Build once, use everywhere:
GitHub MCP Server ← Claude Desktop
← Cursor
← VSCode
← Any MCP client

## Key Principles

1. **Tool-agnostic** - Works with any MCP client
2. **Language-agnostic** - Servers in Python, Node, Go, etc.
3. **Transport-flexible** - stdio, SSE, or HTTP
4. **Security-first** - Credentials stay with server
5. **Composable** - Multiple servers can work together
