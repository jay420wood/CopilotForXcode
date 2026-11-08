# Connect Copilot for Xcode to UNCHAINED OS

UNCHAINED OS provides a local stdio MCP server in its `mcp_server.py` file. This lets MCP-capable clients call selected UNCHAINED OS tools. Copilot for Xcode documents support for configured MCP tools, but this repository does not currently include a dedicated native UNCHAINED OS panel or a Swift-side process launcher; this guide uses the MCP client configuration instead.

## 1. Start UNCHAINED OS

On the Mac or host running the node, start the API and make sure it is reachable from the Mac running Copilot for Xcode:

```sh
cd /path/to/unchained-os
python3 unchained_master_suite.py --run-server
```

The default local address is `http://localhost:8000`. For a remote node, use its HTTPS URL. Do not expose the dashboard token in a committed config file or share it with untrusted clients.

Get the dashboard token from the node's configured `UNCHAINED_DASHBOARD_TOKEN` or the generated `dashboard_token.txt` file. Keep the token private.

## 2. Install bridge dependency

The MCP bridge uses `httpx`. It is already in UNCHAINED OS `requirements.txt`; install project requirements in the environment you will use to run the bridge:

```sh
cd /path/to/unchained-os
python3 -m pip install -r requirements.txt
```

## 3. Add a local MCP server entry

In Copilot for Xcode's MCP/tool configuration, add a stdio server with these values (use absolute paths):

- **Name:** `unchained-os`
- **Command:** `/absolute/path/to/unchained-os/.venv/bin/python` (or the Python interpreter where `httpx` is installed)
- **Arguments:** `/absolute/path/to/unchained-os/mcp_server.py`
- **Environment:**
  - `UNCHAINED_MCP_BASE_URL`: `http://localhost:8000` (or your node's HTTPS URL)
  - `UNCHAINED_MCP_TOKEN`: your dashboard token

The exact UI/config-file format depends on the Copilot for Xcode version. If using a JSON config format compatible with VS Code MCP servers, this template can be adapted:

```json
{
  "servers": {
    "unchained-os": {
      "command": "/absolute/path/to/unchained-os/.venv/bin/python",
      "args": ["/absolute/path/to/unchained-os/mcp_server.py"],
      "env": {
        "UNCHAINED_MCP_BASE_URL": "http://localhost:8000",
        "UNCHAINED_MCP_TOKEN": "SET_LOCALLY_NEVER_COMMIT"
      }
    }
  }
}
```

Do not commit a real token. Do not auto-approve the tools unless you have reviewed what they return and are comfortable with the client invoking them.

## Available tools

- `get_status`: node and license status
- `get_aircraft`: aircraft near the configured home location
- `get_satellites`: satellite data
- `search_aircraft`: query the node's search endpoint
- `ask_chat`: send a question to UNCHAINED OS Ask

The bridge is a local integration, not an Xcode app launcher. It does not start the node service. Start the node separately, and ensure the dashboard token has only the intended access. Restart or reload MCP servers in Copilot for Xcode after changing the local settings.

## Troubleshooting

- **Missing token:** set `UNCHAINED_MCP_TOKEN` in the server configuration; do not put it in a repository file.
- **Connection refused:** verify the node is running and `UNCHAINED_MCP_BASE_URL` is reachable from the Mac.
- **401/403:** check the dashboard token and the active license/features on the node.
- **Python import error:** use the project virtual environment's Python and install `requirements.txt` there.
