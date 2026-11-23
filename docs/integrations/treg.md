---
catalog_title: Treg
catalog_description: Search and call 4,200+ live data, action, media, and team tools
catalog_icon: /integrations/assets/treg.png
catalog_tags: ["data", "mcp"]
---

# Treg MCP tool for ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span><span class="lst-typescript">TypeScript</span>
</div>

The remote MCP server at `https://treg.to/mcp/` connects your ADK agent to
[Treg](https://treg.to/) ("OpenRouter for agent tools"), providing one
connection to 4,200+ tools across 140 providers in the
[Treg catalog](https://treg.to/catalog) and the tools your team already uses.
Your agent searches by task, compares providers by inputs, price, speed, and
reliability, and calls the tool it chooses to look up live data, take actions in
connected accounts, or generate images, video, speech, and music.

Treg holds provider credentials on the server. When Treg supplies the provider
account, calls are billed per call from your team's prepaid balance at the
provider's rate. Teams can also connect their own API keys, OAuth accounts,
CLIs, and skills, whose costs and permissions follow those connections.

## Use cases

- **Company, SEO, and market research**: Start from a company domain or keyword
  to pull search volume, backlinks, SERP rankings, public ad libraries, web
  scraping results, or people and company enrichment. Routed `treg.*` tools can
  fall back across providers for supported jobs.
- **Connected account actions**: Publish content, update records, or call
  internal APIs, OAuth accounts, CLIs, and skills your team registered in Treg,
  through the same MCP server.
- **Media generation**: Generate images, video, speech, or music without signing
  up for separate vendor accounts or storing provider keys on the agent's
  machine.

## Prerequisites

- A [Treg](https://treg.to/) account and an org-scoped token (created with
  `treg org agent-new <name>`). See the [agent setup guide](https://treg.to/llms.txt)
  and [integration and billing guide](https://treg.to/integrate.md) for setup
  and token details.
- For Python: Python 3.11+ with `pip install "google-adk[mcp]"`.
- For TypeScript: Node.js with
  `npm install @google/adk @modelcontextprotocol/sdk`.
- A model API key such as `GOOGLE_API_KEY` for Gemini.

## Use with agent

=== "Python"

    === "Remote MCP Server"

        ```python
        import os
        from google.adk.agents import Agent
        from google.adk.tools.mcp_tool import McpToolset
        from google.adk.tools.mcp_tool.mcp_session_manager import (
            StreamableHTTPConnectionParams,
        )

        TREG_TOKEN = os.environ["TREG_TOKEN"]

        root_agent = Agent(
            model="gemini-flash-latest",
            name="treg_agent",
            instruction=(
                "Help users research companies, run connected team tools, and "
                "generate media with Treg. Call catalog_search, then "
                "catalog_get, before calling a catalog tool so you know its "
                "parameters and price."
            ),
            tools=[
                McpToolset(
                    connection_params=StreamableHTTPConnectionParams(
                        url="https://treg.to/mcp/",
                        headers={
                            "Authorization": f"Bearer {TREG_TOKEN}",
                        },
                        timeout=120,
                    ),
                )
            ],
        )
        ```

        !!! note

            ADK's default Streamable HTTP timeout is 5 seconds. Catalog
            lookups, routed fallbacks, and media generation can take longer, so
            this example sets `timeout=120`.

=== "TypeScript"

    === "Remote MCP Server"

        ```typescript
        import { LlmAgent, MCPToolset } from "@google/adk";

        const TREG_TOKEN = process.env.TREG_TOKEN ?? "YOUR_TREG_TOKEN";

        const rootAgent = new LlmAgent({
            model: "gemini-flash-latest",
            name: "treg_agent",
            instruction:
                "Help users research companies, run connected team tools, and " +
                "generate media with Treg. Call catalog_search, then " +
                "catalog_get, before calling a catalog tool so you know its " +
                "parameters and price.",
            tools: [
                new MCPToolset({
                    type: "StreamableHTTPConnectionParams",
                    url: "https://treg.to/mcp/",
                    transportOptions: {
                        requestInit: {
                            headers: {
                                Authorization: `Bearer ${TREG_TOKEN}`,
                            },
                        },
                    },
                }),
            ],
        });

        export { rootAgent };
        ```

## Available tools

Tool | Description
---- | -----------
`catalog_search` | Search the catalog by the job you want done, with prices
`catalog_get` | Inspect one endpoint's parameters, price, and provider reliability
`call` | Call a catalog or connected team tool through Treg
`call_media` | Same as `call`, returning audio results as MCP audio
`call_result` | Recover a stored call result that did not reach the client
`balance` | Show the team's prepaid balance and spend in flight
`my_tools` | List tools your team registered and can call
`resources_list` | List durable resources created for the current team
`catalog_request` | File a request when the catalog is missing a tool
`feedback` | Report a problem or suggestion about Treg
`review` | Rate a catalog call after using the result
`mark` | Mark rows in a result as wrong or confirmed right
`compare` | Judge two anonymous answers when a result invites a comparison

Ordinary HTTP catalog lookups (such as inspecting the public
[`treg.web.search` endpoint](https://treg.to/catalog/endpoints/treg.web.search))
require no authentication, while connecting to `https://treg.to/mcp/` and
executing provider calls require a Treg token or OAuth grant. Confirm the
current price with `catalog_get` before calling a new tool, and prefer routed
`treg.*` tools when available.

## Additional resources

- [Treg catalog](https://treg.to/catalog)
- [Treg agent setup guide](https://treg.to/llms.txt)
- [Treg integration and billing guide](https://treg.to/integrate.md)
- [Treg team tools guide](https://treg.to/tutorial.md)
- [Treg operating guide](https://treg.to/skill.md)
- [Treg on GitHub](https://github.com/superdesigndev/treg)
