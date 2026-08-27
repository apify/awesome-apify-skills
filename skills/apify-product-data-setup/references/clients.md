# Connecting each client to the Apify MCP server

All clients point at the same endpoint. Narrowing to one Actor is optional but makes
tool selection more reliable when product data is the only job:

```
https://mcp.apify.com?tools=apify/e-commerce-scraping-tool
```

Dropping `?tools=` lets the agent search all of Apify Store at runtime. That is a
trade rather than an upgrade: it buys a fallback for retailers this Actor cannot read,
and it costs tool-selection reliability, because the model then chooses from tens of
thousands of Actors instead of one. Narrow it when product data is the whole job, and
open it up only if you actually need the fallback.

Either way, a retailer missing from the Actor's `marketplaces` list is not a reason to
reach for another Actor: generic extraction is on by default, so unlisted sites often
work. Try this Actor first.

## Contents

- [Claude Desktop](#claude-desktop)
- [Claude Code](#claude-code)
- [Cursor](#cursor)
- [n8n](#n8n)
- [Any other MCP client](#any-other-mcp-client)
- [Python, with the official SDK](#python-with-the-official-sdk)
- [Verifying the connection](#verifying-the-connection)

## Claude Desktop

Edit `claude_desktop_config.json`:

- macOS: `~/Library/Application Support/Claude/`
- Windows: `%APPDATA%\Claude\`

```json
{
  "mcpServers": {
    "apify-ecommerce": {
      "url": "https://mcp.apify.com?tools=apify/e-commerce-scraping-tool"
    }
  }
}
```

Restart Claude. Authentication is OAuth on first use, so no token goes in the file.

## Claude Code

Same block, same shape. Either add it to the user-level config, or scope it to a
project so only that project sees the tool.

## Cursor

`.cursor/mcp.json` in the project, or `~/.cursor/mcp.json` for every project:

```json
{
  "mcpServers": {
    "apify-ecommerce": {
      "url": "https://mcp.apify.com?tools=apify/e-commerce-scraping-tool"
    }
  }
}
```

## n8n

There is no JSON to copy, because MCP is a node rather than a config file.

1. Add an **AI Agent** node.
2. Attach an **MCP Client Tool** to it.
3. Set the endpoint to the URL above.
4. Authenticate with a bearer token from Apify Console, not OAuth, because the node
   runs unattended with nobody present to approve a browser prompt.

## Any other MCP client

Anything speaking **Streamable HTTP** works against the same URL. SSE was removed on
April 1, 2026, so a client that only speaks SSE will fail to connect.

## Python, with the official SDK

Requires Python 3.10 or newer. The SDK does not take headers directly, so pass an
authenticated HTTP client:

```python
import httpx
from mcp import ClientSession
from mcp.client.streamable_http import streamable_http_client

URL = "https://mcp.apify.com?tools=apify/e-commerce-scraping-tool"

async with httpx.AsyncClient(
    headers={"Authorization": f"Bearer {token}"},
    timeout=300,
    follow_redirects=True,
) as http:
    async with streamable_http_client(URL, http_client=http) as streams:
        async with ClientSession(streams[0], streams[1]) as session:
            await session.initialize()

            run = await session.call_tool(
                "apify--e-commerce-scraping-tool",
                {
                    "detailsUrls": [{"url": "https://www.amazon.com/dp/B09XS7JWHH"}],
                    "maxProductResults": 1,
                    "additionalProperties": True,
                    "scrapeMode": "AUTO",
                },
            )
            # First call returns metadata and a datasetId, not products.
            meta = json.loads(run.content[0].text)
            dataset_id = meta["storages"]["datasets"]["default"]["id"]

            items = await session.call_tool(
                "get-dataset-items", {"datasetId": dataset_id, "limit": 1}
            )
```

The `mcp` 2.x API is snake_case: `server_info`, `is_error`. Attribute names from 1.x
guides such as `serverInfo` and `isError` raise `AttributeError`.

## Verifying the connection

Ask the client to list its tools. A working connection shows five:

```
apify--e-commerce-scraping-tool
get-actor-run
get-dataset-items
get-key-value-store-record
abort-actor-run
```

The helper tools arrive even when `?tools=` narrows the list, because the Actor tool
alone cannot deliver products: `get-actor-run` waits for the run to finish and
`get-dataset-items` reads the output.

### Gotcha

If only the helper tools appear and the Actor is missing, the `?tools=` value is
probably wrong. It takes the Actor id with a slash (`apify/e-commerce-scraping-tool`),
while the tool it produces is named with two hyphens
(`apify--e-commerce-scraping-tool`). Mixing the two forms silently yields a connection
with no Actor on it.
