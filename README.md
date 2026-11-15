# Calci MCP (as part of Wizori SDP)

A compact MCP server that solves two word-problem types:

- Apple addition
- Tree distance from angle of elevation

It exposes one MCP tool, `solve_math_problem`, and uses a Groq-hosted model through LangChain to choose and run the right calculator logic.

> Built with the help of the Wizori platform during SDC 7th semester.

## What Is MCP?

MCP (Model Context Protocol) is a standard way for AI assistants (like ChatGPT) to call external tools safely and in a structured format.

In this project:

1. ChatGPT sends a natural-language math problem.
2. MCP calls `solve_math_problem` on this server.
3. The server routes the task to the right internal calculator tool.
4. The result is returned to ChatGPT as a plain response.

## Features

- Single MCP entry tool: `solve_math_problem(problem: str)`
- Input validation for invalid geometry values
- Deterministic model setup for stable results (`temperature=0`)
- Streamable HTTP transport for easy MCP tunnel integration

## Quick Start

### 1) Install dependencies

```bash
python -m venv env
# Windows: env\Scripts\activate
# macOS/Linux: source env/bin/activate
pip install -r requirements.txt
```

### 2) Set your Groq API key

Option A (recommended): create a `.env` file with:

```env
GROQ_API_KEY=your_key_here
```

Option B: if key is not in environment, the app asks for it at startup.

### 3) Run the MCP server

```bash
python mcp_server.py
```

Server endpoint:

```text
http://127.0.0.1:8000/mcp
```

## Connect To ChatGPT With A Secure MCP Tunnel

1. Create a tunnel at https://platform.openai.com/settings/organization/tunnels and associate it with your ChatGPT workspace. Copy your `tunnel_id`.
2. Create a Platform runtime API key with Tunnels Read + Use permissions.
3. Download the full `tunnel-client` (not the runtime-only build):
   https://github.com/openai/tunnel-client/releases/latest
4. In a second terminal (Windows cmd example):

```bat
set CONTROL_PLANE_API_KEY=<your_runtime_key>
tunnel-client.exe init --sample sample_mcp_remote_no_auth --profile calculator --tunnel-id <tunnel_id> --mcp-server-url http://127.0.0.1:8000/mcp
tunnel-client.exe doctor --profile calculator --explain
tunnel-client.exe run --profile calculator
```

5. In ChatGPT:

- Plugins -> + -> Add custom MCP server
- Connection: Tunnel
- Select your tunnel
- Authentication: No authentication

Keep both processes running while testing:

- `python mcp_server.py`
- `tunnel-client.exe run --profile calculator`

## Example Prompts

Try prompts like these:

- One person brings five apples and another brings ten. How many apples do they have together?
- A tree is 12 m above eye level at a 30 degree angle of elevation. How far away is it?
- A tree is 10 m above eye level at a 95 degree angle.

Expected behavior:

- Valid addition gives a numeric sum
- Valid tree-distance gives a rounded meter value
- Invalid angles/heights request correction

## Security Notes

- Never commit `.env` or API keys.
- Keep tokens and runtime keys private.
