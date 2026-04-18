# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Setup (uv recommended)
uv venv && .venv/Scripts/activate && uv pip install -e .

# Run MCP server
python main.py

# Run tests
pytest

# Run a single test file
pytest tests/test_document.py
```

## Architecture

This is a Python **MCP (Model Context Protocol) server** that exposes document processing tools to AI assistants.

**Entry point:** `main.py` — creates a `FastMCP` server named `"docs"`, registers tools via `mcp.tool()(fn)`, and starts with `mcp.run()`.

**Tools** live in `tools/` as plain Python functions. Each tool registered with `mcp.tool()` becomes callable by any MCP-compatible client (e.g., Claude Desktop). Current tools:
- `tools/math.py` — `add()` (demo tool)
- `tools/document.py` — `binary_document_to_markdown()` converts DOCX/PDF binary data to markdown using `markitdown`

**Adding a new tool:**
1. Implement a Python function in `tools/` using `pydantic.Field` for parameter descriptions
2. Import and register it in `main.py` with `mcp.tool()(my_function)`
3. Add tests in `tests/` with real file fixtures in `tests/fixtures/`

**Tool docstring convention** (required for MCP clients to understand the tool):
```python
def my_tool(
    param: str = Field(description="What this param does")
) -> str:
    """One-line summary.

    Detailed explanation of functionality.

    When to use (and not use) this tool.

    Examples:
        Input: ...
        Output: ...
    """
```

## MCP Integration Notes

- The server uses `FastMCP` from the `mcp` package — a lightweight wrapper over the MCP protocol
- `mcp.run()` defaults to **stdio transport**, making it compatible with Claude Desktop and other MCP hosts that launch servers as subprocesses
- To add the server to Claude Desktop, configure it in Claude Desktop's settings pointing to `python main.py` in this project directory
- The `mcp[cli]` dependency also provides `mcp dev` for inspecting tools interactively during development
