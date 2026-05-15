# MCP Server Lifecycle

## Lifecycle Phases

┌─────────────────────────────────────────┐
│ 1. INITIALIZATION                       │
│ - Client launches server process        │
│ - Server starts, waits for messages     │
└─────────────────────────────────────────┘
↓
┌─────────────────────────────────────────┐
│ 2. CAPABILITY NEGOTIATION               │
│ Client: {"method": "initialize"}        │
│ Server: Returns capabilities            │
│   - Tools: ✅                           │
│   - Resources: ✅                       │
│   - Prompts: ✅                         │
└─────────────────────────────────────────┘
↓
┌─────────────────────────────────────────┐
│ 3. DISCOVERY                            │
│ Client: {"method": "tools/list"}        │
│ Server: Returns available tools         │
│ Client: {"method": "resources/list"}    │
│ Server: Returns available resources     │
└─────────────────────────────────────────┘
↓
┌─────────────────────────────────────────┐
│ 4. OPERATION (Active Phase)             │
│ Client: {"method": "tools/call"}        │
│ Server: Executes tool, returns result   │
│ [Repeat as needed]                      │
└─────────────────────────────────────────┘
↓
┌─────────────────────────────────────────┐
│ 5. SHUTDOWN                             │
│ Client closes connection                │
│ Server cleanup & exit                   │
└─────────────────────────────────────────┘

## Detailed Flow

### Phase 1: Initialization (stdio transport)

**Claude Desktop starts your server:**

```bash
# What Claude Desktop executes
python /path/to/server.py
```

**Your server code:**

```python
if __name__ == "__main__":
    async with stdio_server() as (read_stream, write_stream):
        await server.run(read_stream, write_stream)
```

**Server state:** Listening on stdin, writing to stdout

---

### Phase 2: Capability Negotiation

**Client sends initialize:**

```json
{
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-11-25",
    "capabilities": {},
    "clientInfo": {
      "name": "claude-ai",
      "version": "0.1.0"
    }
  }
}
```

**Server responds:**

```json
{
  "result": {
    "protocolVersion": "2025-11-25",
    "capabilities": {
      "tools": {"listChanged": true},
      "resources": {"listChanged": true},
      "prompts": {"listChanged": true}
    },
    "serverInfo": {
      "name": "weather-demo",
      "version": "1.0.0"
    }
  }
}
```

---

### Phase 3: Discovery

**Client asks what tools exist:**

```python
# Client sends
{"method": "tools/list"}

# Server responds
{
  "result": {
    "tools": [
      {"name": "get_weather", "description": "...", "inputSchema": {...}},
      {"name": "get_forecast", "description": "...", "inputSchema": {...}}
    ]
  }
}
```

**Client caches:** Now Claude knows `get_weather` and `get_forecast` are available

---

### Phase 4: Operation (The Active Loop)

User: "What's the weather in Chennai?"
↓
Claude AI decides: "I should use get_weather tool"
↓
Client sends: {"method": "tools/call", "params": {"name": "get_weather", "arguments": {"city": "Chennai"}}}
↓
Server executes: get_weather("Chennai")
↓
Server responds: {"result": {"content": [{"type": "text", "text": "32°C, humid"}]}}
↓
Claude formats: "The current weather in Chennai is 32°C and humid."

---

### Phase 5: Shutdown

**Normal shutdown:**
- Client closes stdin
- Server detects EOF
- Server cleanup (close DB connections, flush logs)
- Server exits with code 0

**Error shutdown:**
- Server crashes
- Client detects broken pipe
- Client logs error
- Client may retry or mark server as unavailable

## Server State Machine
[STOPPED]
↓ start command
[STARTING]
↓ stdio connected
[INITIALIZING]
↓ initialize message received
[READY]
↓ tool call received
[PROCESSING]
↓ result sent
[READY]  ← loops here
↓ shutdown signal
[STOPPING]
↓ cleanup complete
[STOPPED]

## Common Lifecycle Issues

### Issue 1: Server exits immediately

**Symptom:** Server starts, then exits (no errors)

**Cause:** `await server.run()` not called properly

**Fix:**
```python
# ❌ Wrong
if __name__ == "__main__":
    server = create_server()
    # Missing run() call

# ✅ Correct
if __name__ == "__main__":
    asyncio.run(main())

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await server.run(read_stream, write_stream)
```

### Issue 2: Tools not appearing in client

**Symptom:** Server starts, but tools don't show in Claude

**Cause:** `tools/list` not implemented or returns empty array

**Fix:** Verify `@server.list_tools()` decorator is present

### Issue 3: Tool calls timing out

**Symptom:** Claude says "Tool execution timed out"

**Cause:** Tool takes too long (>30s default)

**Fix:** Add async operations, don't block:
```python
# ❌ Wrong - blocks
def slow_tool():
    time.sleep(60)  # Blocks entire server

# ✅ Correct - async
async def fast_tool():
    await asyncio.sleep(5)  # Doesn't block
```
