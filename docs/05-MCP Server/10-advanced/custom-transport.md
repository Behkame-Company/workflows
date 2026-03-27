# Custom Transport Layers

> Building custom MCP transport implementations for specialized environments.

---

## When You Need Custom Transport

The standard transports (stdio, Streamable HTTP) cover most cases. Custom transport is needed for:

- **WebSocket-based environments** (real-time applications)
- **Message queue integration** (RabbitMQ, Kafka, SQS)
- **IPC channels** (shared memory, named pipes)
- **Embedded systems** (serial ports, Bluetooth)
- **Custom security requirements** (mTLS with custom CAs)

---

## Transport Interface

Every MCP transport must implement two capabilities:

```
1. Send a JSON-RPC message (Request, Response, or Notification)
2. Receive a JSON-RPC message

That's it. The protocol layer handles everything else.
```

### Python Interface
```python
from mcp.transport import Transport

class CustomTransport(Transport):
    async def send(self, message: dict) -> None:
        """Send a JSON-RPC message."""
        serialized = json.dumps(message)
        await self._write(serialized)
    
    async def receive(self) -> dict:
        """Receive a JSON-RPC message."""
        data = await self._read()
        return json.loads(data)
    
    async def close(self) -> None:
        """Clean up resources."""
        await self._cleanup()
```

### TypeScript Interface
```typescript
interface Transport {
  send(message: JSONRPCMessage): Promise<void>;
  receive(): AsyncIterableIterator<JSONRPCMessage>;
  close(): Promise<void>;
}
```

---

## Example: WebSocket Transport

```python
import websockets
import json

class WebSocketTransport:
    def __init__(self, url: str):
        self.url = url
        self.ws = None
    
    async def connect(self):
        self.ws = await websockets.connect(self.url)
    
    async def send(self, message: dict):
        await self.ws.send(json.dumps(message))
    
    async def receive(self) -> dict:
        data = await self.ws.recv()
        return json.loads(data)
    
    async def close(self):
        if self.ws:
            await self.ws.close()
```

---

## Message Framing

Whatever transport you use, you need reliable message framing:

| Transport | Framing Method |
|-----------|---------------|
| **stdio** | Newline-delimited JSON |
| **HTTP** | HTTP request/response boundaries |
| **WebSocket** | WebSocket message frames |
| **Message Queue** | One message per queue message |
| **Named Pipe** | Length-prefixed messages |

### Length-Prefix Example
```
[4 bytes: message length][N bytes: JSON-RPC message]
\x00\x00\x00\x2a{"jsonrpc":"2.0","method":"ping","id":1}
```

---

## Testing Custom Transports

1. **Unit test**: Can send and receive messages?
2. **Protocol test**: Run MCP Inspector against your transport
3. **Stress test**: Handle many concurrent messages
4. **Error test**: Graceful handling of disconnects, malformed messages
5. **Integration test**: Connect a real client to your transport

---

## Key Takeaways

1. **Transport is just send/receive** — Two methods to implement
2. **Message framing is critical** — Reliable message boundaries
3. **Test thoroughly** — Use MCP Inspector for protocol compliance
4. **Prefer standard transports** — Only build custom when necessary
5. **stdio and HTTP cover 99% of cases** — Custom is for edge cases

---

## Next Steps

- 🔗 [MCP + Meta-Prompting](mcp-plus-meta-prompting.md) — Framework integration
- 🔗 [Transport Overview](../04-transport/transport-overview.md) — Standard transports
