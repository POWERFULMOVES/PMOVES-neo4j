# PMOVES.AI Integration Dossier

_Last updated: 2026-10-01 (branch PMOVES.AI-Edition-5.26)_

## Module
- Name: PMOVES-Neo4j
- Path: PMOVES-Neo4j (gitlink in POWERFULMOVES/PMOVES.AI)

## Purpose in PMOVES.AI
- The single Community graph database (`neo4j`) behind the mindmap (`GET /mindmap/{constellation_id}` on
  hi-rag-gateway-v2), Cipher graph memory (`:Memory`), Hi-RAG's entity dictionary (`:Entity`), and Agent Zero's
  Neo4j MCP server. Community Edition has exactly one standard database, so these share one graph and are
  separated by label only.

## PMOVES Overlay Surface
- pmoves-integrations/ overlay path: none
- Compose/profile wiring: service `neo4j` in `pmoves/docker-compose.yml` (core.yml is generated from it), container
  `pmoves-neo4j`, volume `pmoves_neo4j-data`, networks pmoves_app/bus/data with alias `neo4j` plus `pmoves_graph_front`
  for the tailnet forwarder (`pmoves/docker-compose.neo4j-tailnet.yml`)
- Env/secret inputs: `NEO4J_PASSWORD` (compose interpolation into `NEO4J_AUTH`); consumers read `NEO4J_URL`, `NEO4J_USER`, `NEO4J_PASSWORD`
- Auth/JWT requirements: Neo4j native auth; clients use `bolt://neo4j:7687`

## Contracts and Topics
- NATS subjects: none
- Supabase schema/tables touched: none
- MCP endpoints/skills: Agent Zero seeds `mcp://neo4j` (`pmoves/tools/seed_agent_zero_mcp.py`); neo4j/mcp requires APOC

## Boot Order and Health
- Bring-up dependency order: data tier (`make -C pmoves up-data-tier DATA_SERVICES=neo4j`), before cipher-api, hi-rag, agent-zero
- Health endpoints: HTTP 7474 (liveness only; Community has no unauthenticated database-availability endpoint)
- Smoke targets: `make -C pmoves neo4j-bootstrap` applies `pmoves/neo4j/cypher/*.cypher` (001, 003, 010, 011 smoke)

## Hardening Notes
- Image pinning / provenance: today `neo4j:5.26.30-community@sha256:037cf575...`; this branch builds the same version
  from source (PMOVES_BUILD.md). GHCR publish + digest pin follow an A/B.
- Secrets source: env var today; `NEO4J_AUTH_FILE` (Docker secrets) is the vendor-recommended form, an operator decision
- Network/security policy constraints: internal-only plus tailnet forwarder; LOAD CSV blocklist
  `internal.dbms.cypher_ip_blocklist=0.0.0.0/0,::/0`; APOC unrestricted list kept to what consumers call

## Source Documentation
- Upstream docs entrypoint: README.asciidoc; build: PMOVES_BUILD.md; packaging: docker/UPSTREAM.md
- PMOVES docs: `pmoves/docs/TAC/TAC_NEO4J.md` in POWERFULMOVES/PMOVES.AI

## Owner / Audit
- Owning lane: ops/knuckles-neo4j-from-fork (B850-CLAUDE, Knuckles)
- Last integration audit run: 2026-10-01
