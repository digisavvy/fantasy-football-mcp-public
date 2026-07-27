# Local setup on macOS

This repository's Claude integration uses the stdio server in
`fantasy_football_multi_league.py`. The separate `fastmcp_server.py` entrypoint
runs an HTTP server and is not needed for Claude Code or Claude Desktop.

## Install

The commands below assume the repository is located at the requested absolute
path. Use Python 3.13 (`brew install python@3.13`) — the pinned
`pydantic==2.11.9` has no wheels for Python 3.14 and fails to build:

```bash
cd /Users/alexvasquez/Repositories/yahoo-fantasy
python3.13 --version
python3.13 -m venv /Users/alexvasquez/Repositories/yahoo-fantasy/.venv
/Users/alexvasquez/Repositories/yahoo-fantasy/.venv/bin/python -m pip install --upgrade pip
/Users/alexvasquez/Repositories/yahoo-fantasy/.venv/bin/python -m pip install -r /Users/alexvasquez/Repositories/yahoo-fantasy/requirements.txt
cp /Users/alexvasquez/Repositories/yahoo-fantasy/.env.example /Users/alexvasquez/Repositories/yahoo-fantasy/.env
```

Add the Yahoo client ID and client secret to `.env` before completing OAuth.
The `.venv` and `.env` paths are ignored by Git.

## Register with Claude Code

Register the stdio server at user scope. The `--` separates Claude CLI options
from the server command:

```bash
claude mcp add --transport stdio --scope user yahoo-fantasy -- /Users/alexvasquez/Repositories/yahoo-fantasy/.venv/bin/python /Users/alexvasquez/Repositories/yahoo-fantasy/fantasy_football_multi_league.py
```

## Configure Claude Desktop

Merge this entry into the `mcpServers` object in
`~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "yahoo-fantasy": {
      "command": "/Users/alexvasquez/Repositories/yahoo-fantasy/.venv/bin/python",
      "args": [
        "/Users/alexvasquez/Repositories/yahoo-fantasy/fantasy_football_multi_league.py"
      ]
    }
  }
}
```

The server loads `/Users/alexvasquez/Repositories/yahoo-fantasy/.env` itself,
so credentials do not need to be duplicated in either Claude configuration.
Restart Claude Desktop after changing its configuration.

## Human OAuth checklist

1. Create a Yahoo developer application and select the out-of-band (`oob`)
   redirect URI.
2. Put its consumer key in `YAHOO_CLIENT_ID` and consumer secret in
   `YAHOO_CLIENT_SECRET` in `.env`.
3. Run:

   ```bash
   cd /Users/alexvasquez/Repositories/yahoo-fantasy
   /Users/alexvasquez/Repositories/yahoo-fantasy/.venv/bin/python utils/setup_yahoo_auth.py
   ```

4. Complete Yahoo's browser authorization and confirm that the script writes
   `YAHOO_ACCESS_TOKEN`, `YAHOO_REFRESH_TOKEN`, and `YAHOO_GUID` to `.env`.
5. Restart Claude Code or Claude Desktop, then ask Claude to call
   `ff_get_leagues`. A successful response should list the authenticated
   account's fantasy football leagues.
