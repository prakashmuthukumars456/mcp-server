# MCP Server Implementation Guide

## Building Your First Server

### Step 1: Choose Your Stack

**Python (Recommended for beginners)**
```bash
pip install mcp anthropic
```

**Node.js/TypeScript**
```bash
npm install @modelcontextprotocol/sdk
```

**Go** (community SDK)

---

### Step 2: Define Your Server

```python
# server.py
import asyncio
from mcp.server import Server
from mcp.server.stdio import stdio_server

# Create server instance
server = Server("my-server")

# Lifecycle hook (optional)
@server.on_startup()
async def startup():
    print("Server starting...", file=sys.stderr)
    # Initialize DB connections, load config, etc.

@server.on_shutdown()
async def shutdown():
    print("Server shutting down...", file=sys.stderr)
    # Cleanup resources
```

---

### Step 3: Register Tools

```python
@server.list_tools()
async def list_tools():
    return [
        {
            "name": "create_ticket",
            "description": "Create a JIRA ticket",
            "inputSchema": {
                "type": "object",
                "properties": {
                    "title": {"type": "string"},
                    "description": {"type": "string"},
                    "priority": {
                        "type": "string",
                        "enum": ["low", "medium", "high"]
                    }
                },
                "required": ["title", "description"]
            }
        }
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "create_ticket":
        # Your implementation
        ticket_id = await jira_api.create(
            title=arguments["title"],
            description=arguments["description"],
            priority=arguments.get("priority", "medium")
        )
        return [{"type": "text", "text": f"Created ticket {ticket_id}"}]
```

---

### Step 4: Add Resources (Optional)

```python
@server.list_resources()
async def list_resources():
    return [
        {
            "uri": "jira://projects",
            "name": "projects",
            "mimeType": "application/json",
            "description": "List of JIRA projects"
        }
    ]

@server.read_resource()
async def read_resource(uri: str):
    if uri == "jira://projects":
        projects = await jira_api.list_projects()
        return {
            "contents": [{
                "uri": uri,
                "mimeType": "application/json",
                "text": json.dumps(projects)
            }]
        }
```

---

### Step 5: Run Server

```python
async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            server.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Error Handling

### Tool Errors

```python
@server.call_tool()
async def call_tool(name: str, arguments: dict):
    try:
        result = await execute_tool(name, arguments)
        return [{"type": "text", "text": result}]
    except ToolNotFoundException:
        raise ValueError(f"Unknown tool: {name}")
    except RateLimitError as e:
        return [{
            "type": "text",
            "text": f"Rate limit exceeded. Retry after {e.retry_after}s",
            "isError": true
        }]
    except Exception as e:
        # Log error
        logging.error(f"Tool {name} failed: {e}")
        return [{
            "type": "text",
            "text": f"Error executing tool: {str(e)}",
            "isError": true
        }]
```

---

## Testing Your Server

### Manual Test (stdio)

```bash
# Run server
python server.py

# In another terminal, send JSON-RPC messages
echo '{"jsonrpc":"2.0","method":"tools/list","id":1}' | python server.py
```

### Automated Test

```python
# test_server.py
import pytest
from mcp.client import Client

@pytest.mark.asyncio
async def test_list_tools():
    client = Client()
    tools = await client.list_tools()
    assert "create_ticket" in [t["name"] for t in tools]

@pytest.mark.asyncio
async def test_create_ticket():
    client = Client()
    result = await client.call_tool("create_ticket", {
        "title": "Test ticket",
        "description": "Test description"
    })
    assert "Created ticket" in result[0]["text"]
```

---

## Configuration

### Environment Variables

```python
# server.py
import os

JIRA_URL = os.getenv("JIRA_URL", "https://company.atlassian.net")
JIRA_API_TOKEN = os.getenv("JIRA_API_TOKEN")

if not JIRA_API_TOKEN:
    raise ValueError("JIRA_API_TOKEN environment variable required")
```

### Config File

```python
# config.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    jira_url: str
    jira_api_token: str
    jira_email: str
    
    class Config:
        env_file = ".env"

settings = Settings()
```

---

## Deployment

### stdio (Local Development)

```json
{
  "command": "python",
  "args": ["/path/to/server.py"],
  "env": {
    "JIRA_API_TOKEN": "your-token"
  }
}
```

### SSE (Production - AWS Fargate)

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY server.py .
CMD ["uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8080"]
```

```bash
# Deploy
docker build -t jira-mcp .
aws ecr get-login-password | docker login --username AWS --password-stdin 123456.dkr.ecr.us-east-1.amazonaws.com
docker tag jira-mcp:latest 123456.dkr.ecr.us-east-1.amazonaws.com/jira-mcp:latest
docker push 123456.dkr.ecr.us-east-1.amazonaws.com/jira-mcp:latest
```