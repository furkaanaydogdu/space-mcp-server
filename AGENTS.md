# AGENTS.md: Space MCP server

The MCP server of "Discover the Cosmos". It offers the agent four data tools: archive search, archive lookup by id, near-Earth asteroids this week, and the position of the International Space Station. It runs as a private HTTP service. The app lives in `space-themed-ai-agent` (the main architecture overview is there); the search index is built by the private `space-index` repo. Start with `docs/README.md` for the decision records (ADRs).

## Commands

From the repo root:

```
uv sync
uv run ruff check .
uv run ruff format --check .
uv run mypy
uv run pytest
```

Run every check before saying a step is done, and report the real results. Do not invent commands; if one is missing, ask.

## How to work

- Work arrives as a step brief. Do only that step.
- Plan first. Before editing, list the files you will touch and the tests you will add, then wait for my approval.
- Keep each change small: at most about 3 files or 150 changed lines. If the step needs more, stop and propose how to split it.
- Do not add features, refactors, abstractions or dependencies that were not asked for. If you see something worth improving, mention it; do not do it.
- If a requirement is unclear, or a library API is not verified (the MCP SDK, LlamaIndex, the NASA and ISS services), ask or check the current docs. Do not guess from memory.
- Do not commit, push, create branches or edit `.github/` unless the step says so. I do the git work.
- When done, report: what changed and why; the one or two places most worth reading closely; any decision you made; anything you could not verify. I read every diff to learn from it, so prefer plain, idiomatic code over clever code, and explain an unfamiliar idiom in one line the first time you use it.

## Code rules

- Functions are short (under about 30 lines) and do one thing. If you need "and" to describe one, split it.
- Names say what a thing is or does. No abbreviations.
- Comments explain why, not what. No commented-out code. No TODO without a step reference.
- Strict types (mypy strict). No `Any`, `# type: ignore` or `noqa` without a one-line reason and my approval.
- Do not swallow errors: no bare `except`. Fail loudly or return an explicit result.
- Prefer plain functions and data over classes and inheritance unless state is needed. Do not abstract until the same code appears a third time.
- Tests are deterministic: no live network, no model calls. NASA and ISS services are tested against recorded responses, including the edge cases (zero asteroids, one asteroid, a path crossing the 180° line). One behaviour per test, named for it. When fixing a bug, write the failing test first.
- Versions are pinned exactly, the MCP SDK included. Add no dependency without asking. Change the lockfile only through `uv`, never by hand.
- No unbounded `while` loops. No regex on long or untrusted text without a length cap. Every call to an outside service (NASA, ISS) has a timeout.

## Architecture rules

- **Tools return data only** (ADR 0003): structured results, never prose, wording or layouts. The app writes the words and draws the pictures.
- Every tool publishes an output schema, so the app can read it from the tool listing. There is no shared schema package with the app, on purpose.
- **Archive search** (ADR 0004) takes the query text and the number of entries wanted (default and upper limit are configuration). It merges chunks of the same entry, applies the relevance cut-off, and returns up to that many entries in rank order; fewer if the cut-off removes the rest. There is no exclusion list and no extra result. The app decides how many entries to show and how many to hold in reserve.
- **Archive lookup** (`get_archive_entries`, ADR 0004) takes a list of entry ids, at most 3, and returns those entries in the requested order, in the same shape as search. An id that is not in the index is left out, not an error. A list longer than 3 or an id in the wrong format is rejected. Test both of those cases.
- Ids are entry ids, never chunk ids. An entry's date identifies it, so an id from before an index update still finds the same entry. Nothing may depend on the ranking staying the same between requests.
- The server speaks streamable HTTP and is reached only over a private link (ADR 0001). Access restrictions are cloud configuration, never application code.
- The search index and the embedding model are baked into the image (ADR 0002). A one-line pin file names the index release tag. The index is private data: download it with a BuildKit build secret (`--mount=type=secret`), never a plain `ARG` or `ENV`, so the token stays out of every layer. Check its sha256 and the manifest (model name, chunk settings, library versions) at build time and again at startup. A mismatch fails the build.
- Use CPU-only PyTorch. The container runs as non-root on port 8080.
- The NASA key used by the asteroid tool comes from the host's secret store at runtime, never from code.

## Repo hygiene (this repo is public)

- No secrets, keys or `.env` files.
- No archive text, no copyrighted images, no company or interviewer names. Test fixtures use invented or NASA-credited sample data only, and never a slice of the real index.
- Logs never print archive text.

## Do not touch

- Accepted ADRs and the docs in `docs/`. A changed decision becomes a new ADR that supersedes the old one, and only when I ask.
- The pin file. It changes only through a pull request I open after a new index release.
- `.env`, and anything under `.github/` unless the step says so.
