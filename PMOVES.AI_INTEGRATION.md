# PMOVES.AI Integration — Neo4j

## Role in PMOVES.AI

Neo4j serves as the **graph database backbone** for PMOVES.AI, providing:

- **Knowledge management** — Entity-relationship storage for Hi-RAG v2 hybrid retrieval
- **CHIT consciousness taxonomy** — Graph representation of Compressed Hierarchical Information Transfer structures
- **Agent memory** — Persistent graph-based memory for Cipher Memory service and agent coordination
- **Relationship traversal** — Multi-hop queries across knowledge entities, agents, and content

## Service Integration

| Service | Port | Protocol | Purpose |
|---------|------|----------|---------|
| Neo4j HTTP | 7474 | HTTP | Cypher transaction API |
| Neo4j Bolt | 7687 | Bolt | Driver connections |

## Docker Compose Profile

Neo4j runs under the `neo4j-local` profile in `pmoves/docker-compose.yml`:

```bash
make -C pmoves neo4j-local-up
```

## Consumers

- **Hi-RAG v2** (port 8086/8087) — Graph queries for hybrid retrieval
- **Cipher Memory** (port 8096) — Knowledge-graph memory storage
- **Extract Worker** (port 8083) — Entity indexing

## Health Check

```bash
curl http://localhost:7474/db/neo4j/health
```

## NATS Subjects

Neo4j state changes propagate via:
- `model.registry.updated.v1` — Catalog mutations that may affect graph state

## Security Notes

- Default credentials must be overridden via `NEO4J_AUTH` env var
- Cypher queries must use parameterized queries (no f-string label construction)
- See `pmoves/docs/security/` for hardening requirements
