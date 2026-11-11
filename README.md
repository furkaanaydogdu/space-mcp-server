# Space MCP server

The MCP server of "Discover the Cosmos", a space-themed AI agent. It offers the agent three tools: archive search, near-Earth asteroids this week, and the position of the International Space Station. Tools return data only; the agent writes the words.

**Status:** under construction. The step-by-step progress is tracked in the application repository.

## Documentation

See [`docs/README.md`](docs/README.md). The main architecture overview lives in the application repository.

## Prerequisites

- [uv](https://docs.astral.sh/uv/) (installs Python 3.13 and the packages)

## Run the checks locally

```
uv sync
uv run ruff check .
uv run ruff format --check .
uv run mypy
uv run pytest
```

## Configuration

Copy `.env.example` to `.env`. It currently has no variables.
