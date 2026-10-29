Title: Official SolidCraft connector in the AI apps' directories (Claude, ChatGPT, Microsoft Copilot, Gemini, Mistral, Grok and others)

Template: Feature request

### What are you trying to do

Let anyone use SolidCraft from the AI app they already pay for, by finding "SolidCraft" in that app's connector directory and clicking Connect, with no install, no config file and no terminal.

Today the MCP server (`crates/mcp`, `solidcraft-cli mcp`, `docs/mcp.md`) is very good but **local only**: JSON-RPC over stdio, launched by a desktop client from a config entry that `solidcraft-cli mcp setup` writes. That reaches Claude Desktop, Claude Code, Cursor, Codex, Gemini CLI, Copilot CLI, VS Code, Zed and the others in the setup table, but not the web and mobile apps where most people meet an AI, and it never shows up in any directory. Every one of the big directories (Anthropic's Connectors Directory, the ChatGPT app directory, Microsoft's certified MCP servers, Mistral's Le Chat directory) only lists **remote MCP servers over HTTPS with OAuth**. So the step from "custom connector a power user sets up" to "official listed connector a novice finds" is one piece of work, done once and then submitted to each company.

This fits the project's aim of making novices produce parts like professionals: "connect SolidCraft in ChatGPT and ask for a bracket" is the shortest path a beginner will ever have to a real parametric part. It's also a large, self-contained task that someone outside the core team can pick up.

### How you imagine it working

**One remote MCP server, many listings.** Build a hosted, standards-conformant MCP endpoint once, then follow each company's listing process. No vendor-specific tool code: every tool keeps running the same engine commands (the "everything is a command" rule), so all clients see the same SolidCraft.

#### Part 1: the remote server (the work someone can pick up)

1. **Streamable HTTP transport** for the existing `Server` (`crates/mcp/src/server.rs`), at MCP protocol revision 2025-11-25 alongside the three revisions supported today. Keep it in its own crate or binary so `crates/mcp` stays dependency-light and the layering check (`cargo xtask layers`) stays green; a `solidcraft-cli mcp serve --http` flag or a small `apps/` server are both fine, maintainers' call.
2. **OAuth 2.1 per the MCP authorization spec**, which is what all the directories check for: return `401` with a `WWW-Authenticate` header pointing at protected-resource metadata (RFC 9728), serve authorization-server metadata (RFC 8414 or OIDC discovery), PKCE S256, form-encoded token endpoint, rotating refresh tokens, `invalid_grant` on a dead refresh token. Support **CIMD** (Client ID Metadata Documents) and **DCR** (RFC 7591), and also a plain registered client, because Microsoft doesn't support DCR and Amazon Quick doesn't support CIMD. Accept each vendor's redirect URIs (Claude's hosted apps use `https://claude.ai/api/mcp/auth_callback`; Claude Code uses a localhost redirect on any port; take the rest from each vendor's docs). Tools that need no account (`describe_command`, `skill_reference`, a scratch design) can work before sign-in.
3. **Hosted sessions instead of local folders.** Remote users have no local file system, so the path-based tools (`open`, `save`, `export`, `screenshot` with `path`) become per-account design storage: open and save by design name, exports and screenshots returned inline (image content, or a short-lived download link). The local folder rules in `docs/mcp.md` don't apply and must never be widened for this; there is simply no host path to reach. Where designs are stored (the existing cloud open/save behind CollabCraft, or a new store) is the maintainers' decision.
4. **Hostile input and fair use.** Per-session limits on CPU time, memory, design size, number of sessions and calls per minute; a long rebuild uses the existing job pattern (`skills/solidcraft-performance/SKILL.md`) instead of holding the request open. Nothing panics (the repo's crash rules), and `hostile_params_never_panic` style tests cover the HTTP layer too.
5. **Tool metadata the reviewers read.** Every tool already has a `title` and `readOnlyHint`/`destructiveHint` (`crates/mcp/src/tools.rs`), which Anthropic and OpenAI require. Re-check that they're honest for the remote tools, and that tool descriptions are short and don't instruct the model beyond the tool's own job (directories reject prompt-injection-style descriptions).
6. **Self-hostable.** The same binary runs inside a company's own network with its own identity provider (Entra ID, Okta, Keycloak). That's how enterprises and defense shops will use it (Part 3), and it lets reviewers and contributors test without the project's hosting.
7. **Optional later phase: interactive 3D view.** Where a client supports MCP's UI extension (MCP Apps in Claude; ChatGPT's apps also render UI from an MCP server), return a live viewer of the part instead of a still screenshot. Not needed for a first listing.

#### Part 2: what each company asks for (checked 29 Oct 2026; these change often, so re-check before submitting)

Listings are submitted by whoever **owns the endpoint and the domain**, so the code can come from any contributor, but the submissions themselves need a maintainer (or the organisation behind solidcraft.io) to hold the accounts.

| Company | Where a listed connector appears | How to get listed | Notable requirements |
| --- | --- | --- | --- |
| **Anthropic** (Claude) | Connectors Directory in claude.ai, Claude Desktop, mobile, Claude Code, Cowork | Developer portal at claude.ai/directory/manage, "MCP connector"; any paid plan can submit | Remote HTTPS URL; OAuth 2.0 (DCR, CIMD, or Anthropic-held client); every tool titled and annotated; docs URL, privacy policy URL, support contact, icon; fully populated reviewer test account; listed as "Community" after an automated scan, "Verified" after review. A local server can only be listed inside a Claude plugin, so the existing stdio server plus the bundled skills could also be submitted as a plugin. |
| **OpenAI** (ChatGPT, Codex) | ChatGPT app directory | Upload a plugin package on the OpenAI dashboard after individual or business verification | One remote MCP server per plugin over streamable HTTP; OAuth (registered client, DCR or CIMD; client secret entered in the portal, never in the package); domain verification file under `/.well-known/`; website, support, privacy policy and terms URLs; 5 positive and 3 negative test cases, a video walkthrough, and a reviewer account without MFA; the server is rescanned daily and new tools wait for approval. |
| **Microsoft** (Microsoft 365 Copilot, Copilot Studio, Azure AI Foundry) | Certified MCP servers in those products | Partner Center offer of type "Apps and Agents for M365 and Copilot" (preview) | Verified publisher who owns the endpoint; a Teams-style manifest naming the remote MCP URL plus a tool-definition file; OAuth secrets held in Azure Key Vault; **no DCR**; ASCII-only header names; functional, security and responsible-AI review; resubmit when tools change. |
| **Google** (Gemini) | No public self-serve directory found | Gemini app connectors are partnerships; Gemini Enterprise customers add a custom MCP server in their own tenant (streamable HTTP only); Gemini CLI extensions are a public Git repo with a `gemini-extension.json`, unreviewed | Streamable HTTP and OAuth cover all three. Gemini CLI was replaced by Antigravity CLI for consumer plans in June 2026 (unverified). |
| **Mistral** (Le Chat) | Le Chat connectors directory | No public submission process found; any user or admin can add a custom connector by URL | OAuth 2.0 is the usual sign-in. Worth asking Mistral directly once the server is live. |
| **xAI** (Grok) | Grok connectors catalogue (about 30 first-party integrations) | No public submission process found; "bring your own MCP" takes any public HTTPS server (unverified, from secondary sources) | Public HTTPS endpoint. |
| **Amazon** (Amazon Quick) | No third-party directory found; teams add MCP connectors by URL | Connectors > Model Context Protocol in Quick | Remote only, streamable HTTP preferred; OAuth with discovery, DCR or manual client, PKCE S256, resource indicators (RFC 8707); no CIMD. |
| **Meta** | No MCP connector directory for Meta AI found | — | Revisit later. |
| **MCP Registry** (vendor-neutral, run by the MCP project) | Clients and directories that read the registry | `server.json` published with `mcp-publisher`; namespace proved by GitHub org (`io.github.tangency-labs/...`) or DNS (`io.solidcraft/...`) | Lists both the remote URL and the existing stdio package in one entry. Cheapest listing; do it first. |

#### Part 3: export control and high-security users (applies to every vendor)

A hosted connector sends design data to the AI company's servers and to the project's host. Many SolidCraft users make ITAR/EAR-controlled or CUI parts, so the connector has to be safe by default for them:

- **Say it plainly.** The listing, the server's `instructions` and the docs state that the hosted service is not for export-controlled or CUI designs. A design carrying a control marking (once markings exist, see the high-security profile proposal) is refused by the hosted server with a clear message, never uploaded.
- **Announce the marking on every connection.** In the MCP `initialize` result (`instructions` and `_meta`), the server announces the session's marking, and design data only flows after the client confirms it accepts that marking. A client that doesn't confirm gets the marking and refusals, not geometry. Until markings land, ship the hook and a "not for controlled data" notice.
- **Controlled work stays inside the boundary.** For defense shops the supported path is the self-hosted server (Part 1, item 6) inside their own boundary (for example AWS GovCloud or Azure Government) with their own identity provider and an AI endpoint their policy trusts. Their admin decides which clients may connect.
- **Audit.** Every remote call is logged (who, which design, which command, when) from the command dispatcher, so MCP and UI share one audit trail.
- **Privacy policy facts the directories ask for:** what is stored and where (region), for how long, deletion on request and on account closure, no training on users' designs, no sale or sharing, and crash reports that never contain a design.

#### Suggested order (each a separate PR)

1. Streamable HTTP transport for the existing tools, unauthenticated, localhost only, with tests. No user-visible change.
2. OAuth 2.1 (CIMD, DCR, registered client) and hosted session storage replacing paths.
3. Limits, audit log, marking announcement hook, self-host guide in `docs/mcp.md` and the book.
4. Hosting at a project-owned HTTPS domain, privacy policy, terms, support page, icon (original, with an ATTRIBUTION row), reviewer account. Maintainers.
5. MCP Registry entry, then Anthropic, OpenAI and Microsoft submissions; ask Google, Mistral and xAI about listing. Maintainers.

#### Done when

- The server passes MCP Inspector and works as a custom connector in Claude, ChatGPT (developer mode) and at least one more client, with sign-in, open, model, measure, export and screenshot.
- `cargo xtask ci` is green, nothing panics on hostile requests, and the local stdio server behaves exactly as before.
- Docs cover connecting, self-hosting and data handling; the privacy policy is live.
- At least the MCP Registry and Anthropic listings are submitted, with the rest tracked as checkboxes here.

Related: #3380 (agent MCP replies and long-running work), the [AI agents & MCP milestone](https://github.com/tangency-labs/solidcraft/milestone/18). For the export-control side: [security-research-nsnlookupjn-high-security.md](https://github.com/nsnlookupjn/solidcraft/blob/claude/project-thread-2fa4g1/research/security-research-nsnlookupjn-high-security.md) (MIT OR Apache-2.0), section 4 "Announce the marking to every connected system".

Background: I've worked with CAD/CAM since the 1980s and run companies that make flight-safety and defense parts, which is where the export-control section comes from. I don't write code for this project, so this is up for grabs.
