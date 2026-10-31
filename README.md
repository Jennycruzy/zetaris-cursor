# Zetaris for Cursor

Connect Cursor's agent to your Zetaris instance, so it gets data through Zetaris instead of going around it.

[![Add Zetaris to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=zetaris&config=eyJjb21tYW5kIjoidXZ4IiwiYXJncyI6WyItLWZyb20iLCJnaXQraHR0cHM6Ly9naXRodWIuY29tL0plbm55Y3J1enkvemV0YXJpcy1jdXJzb3IiLCJ6ZXRhcmlzLW1jcCJdLCJlbnYiOnsiWkVUQVJJU19CQVNFX1VSTCI6Imh0dHA6Ly9sb2NhbGhvc3Q6MzAwMCIsIlpFVEFSSVNfT1JHX0lEIjoiMSIsIlpFVEFSSVNfVVNFUk5BTUUiOiJ5b3VAZXhhbXBsZS5jb20iLCJaRVRBUklTX1BBU1NXT1JEIjoiY2hhbmdlLW1lIiwiWkVUQVJJU19USU1FT1VUIjoiNTAifX0%3D)

After clicking, replace the sign-in values in Cursor's MCP settings with your own. The button runs the server with [uv](https://docs.astral.sh/uv/) (`uvx`).

## What you get

- **`zetaris_mcp.py`**: a small MCP server (Python standard library only). It gives the agent five tools:
  - `list_databases`: see what data Zetaris already has.
  - `list_tables`: see the tables in one database, plus any ready-made views you describe.
  - `describe_table`: see a table's columns.
  - `run_query`: run read-only SQL (`SELECT`, `WITH`, `SHOW`, `DESCRIBE`, `EXPLAIN`).
  - `register_source`: add a new source (`CREATE LIGHTNING ...`) or cache a table (`CACHE TABLE`).
- **`.cursor/rules/zetaris.mdc`**: a project rule Cursor reads before every task. It tells the agent to get data through Zetaris, register a missing source instead of downloading it, join sources inside one Zetaris query, and show the SQL behind every figure.
- **`examples/montgomery/`**: the rule and view list used for Montgomery, Alabama (City of Montgomery, US Census, CDC and Ookla data), as an example of describing your own ready-made views.

## Set up by hand (no uv needed)

1. Copy `zetaris_mcp.py` and the `.cursor/` folder into your project.
2. Create `.env.local` next to them (it is in `.gitignore`):

   ```
   ZETARIS_BASE_URL=http://localhost:3000
   ZETARIS_ORG_ID=1
   ZETARIS_USERNAME=you@example.com
   ZETARIS_PASSWORD=...
   ```

   Or set `ZETARIS_API_KEY` instead of the username and password.
3. Open the project in Cursor. In **Settings → MCP**, turn on `zetaris`. With the Cursor CLI: `cursor-agent mcp enable zetaris`.
4. Ask a question in plain English. The agent finds the data in Zetaris, writes the SQL and shows it with the answer.

Each tool call is written as one JSON line to `zetaris-mcp.log`, so you can see exactly what the agent ran in Zetaris.

## Settings

| Variable | Default | Meaning |
|---|---|---|
| `ZETARIS_BASE_URL` | `http://localhost:3000` | Your Zetaris UI address |
| `ZETARIS_ORG_ID` | none | Numeric organisation ID |
| `ZETARIS_API_KEY` or `ZETARIS_USERNAME` + `ZETARIS_PASSWORD` | none | How to sign in |
| `ZETARIS_ENV_FILE` | none | File to read these from (the bundled `.cursor/mcp.json` uses `.env.local`) |
| `ZETARIS_TIMEOUT` | `50` | Seconds each call may take before the agent is told to make the query lighter |
| `ZETARIS_SPARK_UI` | none | Spark status address (for example `http://127.0.0.1:4040`); turns on the busy check below |
| `ZETARIS_COOLDOWN` | `180` | Longest wait, in seconds, before a query that ran past the time limit is assumed finished |
| `ZETARIS_CATALOG` | none | JSON file describing ready-made views, shown by `list_tables` |
| `ZETARIS_MAX_ROWS` | `200` | Most rows returned per query |
| `ZETARIS_MCP_LOG` | none | Where tool calls are logged |

## Safety

- `run_query` refuses anything that is not read-only. `register_source` only accepts `CREATE LIGHTNING` and `CACHE TABLE`. Neither tool can drop or change existing data.
- Each call has a time limit, so the agent gets a clear answer instead of hanging.
- With `ZETARIS_SPARK_UI` set, a query that ran past the limit blocks new queries until Spark shows it has finished. An agent cannot pile retries onto a busy engine and slow down other users of the same Zetaris.

## Licence

MIT
