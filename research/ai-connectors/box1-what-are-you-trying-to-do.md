
Let anyone use SolidCraft from the AI app they already pay for, by finding "SolidCraft" in that app's connector directory and clicking Connect, with no install, no config file and no terminal.

Today the MCP server (`crates/mcp`, `solidcraft-cli mcp`, `docs/mcp.md`) is very good but **local only**: JSON-RPC over stdio, launched by a desktop client from a config entry that `solidcraft-cli mcp setup` writes. That reaches Claude Desktop, Claude Code, Cursor, Codex, Gemini CLI, Copilot CLI, VS Code, Zed and the others in the setup table, but not the web and mobile apps where most people meet an AI, and it never shows up in any directory. Every one of the big directories (Anthropic's Connectors Directory, the ChatGPT app directory, Microsoft's certified MCP servers, Mistral's Le Chat directory) only lists **remote MCP servers over HTTPS with OAuth**. So the step from "custom connector a power user sets up" to "official listed connector a novice finds" is one piece of work, done once and then submitted to each company.

This fits the project's aim of making novices produce parts like professionals: "connect SolidCraft in ChatGPT and ask for a bracket" is the shortest path a beginner will ever have to a real parametric part. It's also a large, self-contained task that someone outside the core team can pick up.

