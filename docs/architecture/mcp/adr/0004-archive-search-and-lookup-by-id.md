# ADR 0004: Archive search returns a ranked list; a lookup tool fetches entries by id

- Status: Accepted
- Date: 2026-11-12
- Repo: mcp
- Supersedes: the exclusion list and the "one result beyond the requested number" rule of [ADR 0003](0003-tools-return-data-only.md)
- Related: [App ADR 0013](../../app/adr/0013-show-more-by-reserved-ids.md), [Index ADR 0003](../../index/adr/0003-chunk-only-long-entries.md)

## Context
App ADR 0013 changes how "Show more" works: the first search keeps a ranked list longer than what is shown, and later requests fetch the reserved entries by id. The tools have to support that.

## Decision
- **Archive search** takes the query text and the number of entries wanted (default and upper limit are configuration). It merges chunks of the same entry, applies the relevance cut-off, and returns up to that number of entries in rank order. Fewer come back when the cut-off removes the rest. The app decides how many are shown and how many are held in reserve. There is no exclusion list.
- **Archive lookup** (`get_archive_entries`) takes a list of entry ids, at most 3, and returns those entries in the requested order, in the same shape the search returns. An id that is not in the index is left out, not an error. A list longer than 3 or an id in the wrong format is rejected.
- Both tools return data only, publish an output schema, and are validated by the app against its own models, as in ADR 0003.
- Ids are entry ids. The entry's date identifies it, so an id from before an index update still finds the same entry afterwards.
- The relevance cut-off value is still to be specified when the search tool is designed.

## Alternatives considered
- Keep the exclusion list and the extra result: rejected, see App ADR 0013.
- Add an offset or page parameter to search: rejected, it would repeat the search and rely on the ranking staying the same between requests.

## Consequences
- The entry lookup tool that ADR 0003 deferred "until follow-ups return" now exists, for Show more only. It is not a follow-up feature.
- Search no longer needs a stable ranking between requests, and chunk merging happens once, in the first search.
- The service has four data tools: archive search, archive lookup, asteroids, ISS.
