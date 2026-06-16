---
name: rayu-mcp-builder
description: Build a Model Context Protocol (MCP) server that exposes tools, resources, and prompts to AI agents. Use when creating an MCP server or integration — covers server anatomy, transports, input schemas, error handling, testing, and safe tool exposure.
---

# Rayu MCP Builder

The Model Context Protocol (MCP) is a standard way to expose capabilities to AI clients. A server offers three primitives — **tools** (actions the model can call), **resources** (readable data the model can pull in), and **prompts** (reusable templates) — over a JSON-RPC transport.

## 1. Pick a transport

- **stdio** — the server is launched as a subprocess; it speaks JSON-RPC over stdin/stdout. Best for local tools a client spawns. (Most common.)
- **HTTP + SSE / streamable HTTP** — the server runs as a network service. Use for remote/shared servers.

Start with stdio unless you need a networked server.

## 2. Minimal server anatomy (TypeScript SDK)

```ts
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { z } from 'zod'

const server = new McpServer({ name: 'my-server', version: '1.0.0' })

server.tool(
  'get_weather',
  'Get current weather for a city',
  { city: z.string().describe('City name') },     // input schema (zod)
  async ({ city }) => {
    const data = await fetchWeather(city)
    return { content: [{ type: 'text', text: JSON.stringify(data) }] }
  },
)

await server.connect(new StdioServerTransport())
```

(Python SDK mirrors this with `FastMCP` and `@mcp.tool()` decorators.)

## 3. Design the tools well — the schema IS the UX

The model only sees names, descriptions, and schemas. So:

- **Name** tools by action (`create_invoice`, not `invoices`).
- **Describe** what the tool does AND when to use it — the description drives selection.
- **Schema**: every parameter typed, with a `.describe()`. Mark optionals; give safe defaults. Validate strictly (zod/pydantic) and reject bad input early.
- Keep each tool **single-purpose**. Many small, clear tools beat one mega-tool with a mode flag.
- Return concise, structured results; large blobs belong in **resources**, not tool output.

## 4. Resources and prompts

- **Resources** expose readable content by URI (`file://…`, `db://orders/123`). Provide a list endpoint + a read endpoint; support templates (`db://orders/{id}`) when enumerable.
- **Prompts** are named, parameterized templates the user can invoke. Use them for repeatable workflows.

## 5. Error handling

- Validation failures → return a clear tool error the model can recover from (message says what was wrong), not an unhandled crash.
- Distinguish *user/input* errors (fixable by changing args) from *system* errors (retryable/fatal).
- Never let an exception kill the process on stdio — catch, return a structured error, keep serving.

## 6. Security of tool exposure (critical)

A tool is an action you're handing to a model. Treat every input as untrusted:

- **Least privilege** — expose the narrowest capability that does the job. Don't expose `run_shell(cmd)` when `list_files(dir)` suffices.
- **Validate + sanitize** all inputs; constrain paths (no traversal), allow-list commands, parameterize queries.
- **Guard side effects** — destructive tools (delete, transfer, deploy) should require explicit args and ideally a confirmation/dry-run mode.
- **Secrets** come from the environment, never from tool args or hard-coded values; never echo them in results.
- **Rate/size limits** on anything that hits the network or returns large data.

## 7. Test it

- Unit-test each tool's handler directly (input → output, incl. error paths).
- Integration-test over the transport with the MCP Inspector or a scripted client: list tools, call each with valid + invalid input, assert the structured result.
- Verify the server starts, advertises its tools/resources, and shuts down cleanly.

## 8. Ship checklist

- [ ] Clear `name`/`version`; tools have action names + when-to-use descriptions.
- [ ] Strict input schemas with descriptions and validation.
- [ ] Structured errors; process never dies on bad input.
- [ ] Least-privilege tools; destructive ops guarded; secrets from env.
- [ ] Tested over the transport; documented setup (command, env vars).

## Working in the rayu swarm

The `backend` collaborator uses this when a task needs an MCP server/integration. Pair with `rayu-api-design` for the underlying data/contract conventions. Record the exposed tool surface in `.rayu/swarm/` so the rest of the swarm knows the integration's capabilities.
