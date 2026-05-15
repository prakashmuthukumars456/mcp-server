# MCP Servers Learning Repository

Documentation and implementations from learning Model Context Protocol (MCP) for AI-powered developer experience tools.

## 🎯 Purpose

This repo demonstrates:
- Building custom MCP servers from scratch
- Integrating production systems (New Relic, Azure DevOps)
- Understanding MCP protocol architecture
- System design for autonomous AI workflows

## 📚 Documentation

- [01 - MCP Protocol Fundamentals](docs/01-mcp-protocol.md)
- [02 - Server Lifecycle](docs/02-lifecycle.md)
- [03 - Implementation Guide](docs/03-implementation-guide.md)
- [04 - Transport Comparison](docs/04-transport-comparison.md)
- [05 - Best Practices](docs/05-best-practices.md)

## 🛠️ Servers Built

### 1. Weather Server (Custom)
**Purpose:** Learning MCP fundamentals  
**Stack:** Python, FastMCP  
**Transport:** stdio  
**Status:** ✅ Working

[View Code](servers/weather-server/) | [Setup Guide](servers/weather-server/README.md)

### 2. New Relic Integration (Production)
**Purpose:** Observability platform integration  
**Stack:** Node.js, NerdGraph API  
**Transport:** stdio  
**Status:** ✅ Working with production data

[View Config](servers/newrelic-integration/) | [Setup Guide](servers/newrelic-integration/README.md)

### 3. Azure DevOps Integration (Production)
**Purpose:** Work item management  
**Stack:** npx package, ADO REST API  
**Transport:** stdio  
**Status:** ✅ Working with production data

[View Config](servers/ado-integration/) | [Setup Guide](servers/ado-integration/README.md)

## 🏗️ Architecture Overview

Claude Desktop / Cursor
↓ MCP Protocol (stdio/SSE/HTTP)
MCP Server (Local or Remote)
↓ HTTPS REST API
External Service (Datadog, JIRA, etc.)

[See detailed architecture diagrams](diagrams/)

## 🚀 Quick Start

### Prerequisites
- Python 3.12+ (for custom servers)
- Node.js 18+ (for New Relic server)
- Claude Desktop or Cursor with MCP support

### Setup Weather Server

```bash
# Clone repo
git clone https://github.com/yourusername/mcp-servers-learning
cd mcp-servers-learning/servers/weather-server

# Install dependencies
python -m venv venv
source venv/bin/activate  # or: venv\Scripts\activate on Windows
pip install -r requirements.txt

# Test server
python server.py
```

### Configure in Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "weather": {
      "command": "C:\\path\\to\\venv\\Scripts\\python.exe",
      "args": ["C:\\path\\to\\server.py"]
    }
  }
}
```

Restart Claude Desktop → Ask: "What's the weather in Chennai?"

## 📖 Key Learnings

### MCP Protocol Capabilities
1. **Tools** - Functions the AI can execute
2. **Prompts** - Pre-built conversation templates
3. **Resources** - Structured data sources
4. **Roots** - URI/filesystem boundaries
5. **Elicitation** - Server-initiated user prompts

### Transport Layers
- **stdio** - Local execution (solo dev)
- **SSE** - Remote persistent connection (team)
- **HTTP** - Remote request/response (enterprise)

### Real-World Patterns
- Solo dev → stdio with local API keys
- Team of 10-50 → SSE with OAuth
- Enterprise 100+ → HTTP behind API Gateway

## 📝 References

- [MCP Official Docs](https://modelcontextprotocol.io)
- [Claude API Documentation](https://docs.anthropic.com)
- [Uber Freight Job Description](docs/job-description.md)
