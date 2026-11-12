# MCP repo: C4 component view (level 3)

Part of the [main architecture document](../space-agent-architecture.md). Decisions: [ADR 0001](adr/0001-http-mcp-server-on-a-private-link.md), [ADR 0002](adr/0002-index-and-model-baked-into-image.md), [ADR 0003](adr/0003-tools-return-data-only.md), [ADR 0004](adr/0004-archive-search-and-lookup-by-id.md).

```mermaid
flowchart TB
    app["App service<br/>(agent, one MCP connection per question)"]
    subgraph mcp["MCP service (private Cloud Run, ingress internal)"]
        srv["Streamable HTTP MCP server"]
        t1["Tool: archive search<br/>query + number wanted"]
        t4["Tool: archive lookup<br/>up to 3 entry ids"]
        t2["Tool: asteroids<br/>7-day window"]
        t3["Tool: ISS<br/>10 timepoints"]
        idx[("FAISS index<br/>in memory")]
        emb["Embedding model<br/>baked in, CPU only"]
        chk["Startup manifest check"]
    end
    neows(["NASA NeoWs"])
    iss(["wheretheiss.at"])

    app -- "private link" --> srv
    srv --> t1
    srv --> t4
    srv --> t2
    srv --> t3
    t1 --> emb
    t1 --> idx
    t4 --> idx
    t2 --> neows
    t3 --> iss
    chk -. "model, chunks, versions must match" .-> idx
```

Build time (not shown above): the Dockerfile reads the pin file, downloads the index release with a build secret, verifies its checksum, runs the manifest check and downloads the model weights.
