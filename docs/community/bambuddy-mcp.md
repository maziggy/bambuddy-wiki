---
title: Bambuddy MCP Server
description: Model Context Protocol server that exposes Bambuddy's REST API as tools for AI assistants
---

# Bambuddy MCP Server

An [MCP](https://modelcontextprotocol.io) server that exposes the full Bambuddy REST API as tools for AI assistants like Claude Desktop, Claude Code, and other MCP-compatible clients. Once configured, you can talk to your printers in plain language &mdash; "what's printing right now?", "add the benchy to the queue", "show me a camera snapshot of printer 3", "list my recent archives".

The server fetches Bambuddy's OpenAPI spec at startup and dynamically generates tools for **430+ endpoints** &mdash; without flooding the assistant's context window. By default it exposes a small set of meta-tools (`list_categories`, `search_tools`, `execute_tool`) so the AI can discover and call endpoints on demand. Binary responses like camera snapshots come back as native MCP `ImageContent` so the assistant can actually see and process them.

**Author:** [MrMebelMan](https://github.com/MrMebelMan)
**Repository:** [github.com/MrMebelMan/bambuddy-mcp](https://github.com/MrMebelMan/bambuddy-mcp)
**License:** GPL-3.0
**Status:** Independent community project

---

## What you need

- Python 3.10+ and [`uv`](https://docs.astral.sh/uv/)
- A running Bambuddy instance reachable from wherever you run the MCP server.
- A Bambuddy [API key](../features/api-keys.md) for authentication.
- An MCP-compatible client &mdash; Claude Desktop, Claude Code, or any other host that supports the Model Context Protocol.

---

## Quick example

Add the server to your Claude Desktop or Claude Code config:

```json
{
  "mcpServers": {
    "bambuddy": {
      "command": "uvx",
      "args": ["bambuddy-mcp"],
      "env": {
        "BAMBUDDY_URL": "http://localhost:8000",
        "BAMBUDDY_API_KEY": "your-api-key"
      }
    }
  }
}
```

Then ask your assistant questions like:

- *"What printers are connected?"*
- *"Show me the status of my A1 Mini."*
- *"Add the benchy to the print queue."*
- *"Turn on the chamber light on printer 2."*
- *"Show me a camera snapshot from printer X."*

Full installation, NixOS notes, and the complete environment-variable reference are in the [project README](https://github.com/MrMebelMan/bambuddy-mcp#readme).

---

## Privacy controls

Because AI assistants surface tool responses back into the conversation, the MCP server includes opt-in censoring for sensitive fields. By default it masks `access_code` and partially masks `serial_number` in API responses. Filename masking is also available via `BAMBUDDY_CENSOR_MODEL_FILENAME` if you'd rather not have model names ending up in chat logs. See the README for the full list of toggles.

---

## How it talks to Bambuddy

Bambuddy MCP authenticates via a standard Bambuddy [API key](../features/api-keys.md) and uses only the public REST API documented at `/openapi.json`. No internal endpoints are touched, so the integration stays stable across Bambuddy releases &mdash; new endpoints added in Bambuddy automatically become available as tools the next time the MCP server starts.

---

## Support

Issues and feature requests for the MCP server itself belong on the [project's GitHub repository](https://github.com/MrMebelMan/bambuddy-mcp/issues). The Bambuddy team doesn't maintain bambuddy-mcp.

For Bambuddy-side questions (API key setup, finding printer IDs, etc.), use the [Discord](https://discord.gg/aFS3ZfScHM).
