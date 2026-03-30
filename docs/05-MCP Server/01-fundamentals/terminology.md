# MCP Key Terminology

> A comprehensive glossary of Model Context Protocol concepts. Reference this when reading any MCP documentation.

---

## Core Architecture Terms

| Term | Definition |
|------|-----------|
| **Host** | The application environment (IDE, desktop app, agent framework) that manages MCP connections. Examples: VS Code, Claude Desktop, Cursor. The host creates clients, enforces permissions, and mediates AI-tool interactions. |
| **Client** | A protocol handler inside the host that maintains a 1:1 connection with a single MCP server. Each client handles capability negotiation, message routing, and session management for its connected server. |
| **Server** | An external program that exposes tools, resources, and prompts to clients via the MCP protocol. Each server is typically focused on a single domain (e.g., GitHub, Postgres, filesystem). |
| **Session** | A stateful connection between a client and server. Sessions begin with initialization and capability negotiation, and persist until explicitly closed. |

---

## Primitive Types

| Term | Definition | Analogy |
|------|-----------|---------|
| **Tool** | An executable function the AI can invoke. Tools perform actions (create issue, query database, send email). They are the "verbs" of MCP. | API endpoint |
| **Resource** | Read-only data exposed by a server (files, database schemas, config). Resources are the "nouns" of MCP. Identified by URIs. | GET endpoint |
| **Prompt** | A reusable template for structuring AI interactions. Prompts can be parameterized and are the "recipes" of MCP. | Template |
| **Sampling** | A mechanism for servers to request LLM completions from the client. Flips the normal direction — the server asks the AI to think. | Callback |

---

## Additional Primitives

| Term | Definition |
|------|-----------|
| **Root** | A workspace or file system location exposed by the client to servers. Roots define what parts of the filesystem are accessible. |
| **Elicitation** | A structured request from the server for user input during a workflow. Like a form or dialog that the server presents through the client. |

---

## Protocol Terms

| Term | Definition |
|------|-----------|
| **JSON-RPC 2.0** | The message format MCP uses. Every message is a JSON object following the JSON-RPC specification (request, response, or notification). |
| **Capability** | A feature that a client or server declares it supports during initialization. Capabilities enable progressive enhancement without breaking backward compatibility. |
| **Capability Negotiation** | The handshake during session initialization where client and server declare what features they support. |
| **Request** | A JSON-RPC message that expects a response. Contains a method name, parameters, and a unique ID. |
| **Response** | A JSON-RPC message sent in reply to a request. Contains either a result or an error. |
| **Notification** | A one-way JSON-RPC message that does not expect a response. Used for events, progress updates, and streaming. |

---

## Transport Terms

| Term | Definition |
|------|-----------|
| **Transport** | The communication mechanism between client and server. Transport is independent of the protocol content. |
| **stdio** | Standard Input/Output transport. The server runs as a subprocess; messages flow through stdin/stdout. Best for local use. |
| **Streamable HTTP** | HTTP-based transport using a single endpoint. Supports both request-response and streaming via SSE. The current standard for remote servers. |
| **SSE (Server-Sent Events)** | A one-way streaming mechanism over HTTP. Previously used as a transport; now deprecated in favor of Streamable HTTP. |

---

## Security Terms

| Term | Definition |
|------|-----------|
| **Trust Boundary** | The security perimeter between components. In MCP: host→client→server each have distinct trust boundaries. |
| **Host Mediation** | The principle that all AI-tool interactions must pass through the host, which enforces permissions and user consent. |
| **OAuth 2.1** | The authentication standard used by remote MCP servers. Supports PKCE and dynamic client registration. |
| **Consent** | Explicit user approval before an AI agent executes a tool action. Required for destructive or sensitive operations. |
| **Tool Poisoning** | An attack where a malicious tool description tricks the AI into harmful actions. |
| **Prompt Injection** | An attack where malicious content in tool inputs/outputs manipulates the AI's behavior. |
| **Confused Deputy** | An attack where a legitimate client is tricked into acting on behalf of a malicious one. |

---

## Ecosystem Terms

| Term | Definition |
|------|-----------|
| **MCP Registry** | A directory of available MCP servers. The official registry is at `github.com/modelcontextprotocol/servers`. |
| **SDK** | Software Development Kit for building MCP clients or servers. Official SDKs exist for Python and TypeScript. |
| **Gateway** | An enterprise proxy that sits between MCP clients and servers, providing centralized auth, routing, rate limiting, and logging. |
| **Connector** | Informal term for an MCP server that bridges to a specific service (e.g., "GitHub connector"). |

---

## Common Abbreviations

| Abbreviation | Full Form |
|-------------|-----------|
| **MCP** | Model Context Protocol |
| **LLM** | Large Language Model |
| **PKCE** | Proof Key for Code Exchange (OAuth security extension) |
| **mTLS** | Mutual Transport Layer Security |
| **URI** | Uniform Resource Identifier |

---

*Bookmark this page — you'll reference it often as you learn MCP.*
