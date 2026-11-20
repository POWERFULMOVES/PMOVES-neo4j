# PMOVES knowledge for Neo4j

This directory carries what PMOVES.AI knows about its Neo4j deployment, next to the source the PMOVES image is built
from. It changes nothing in upstream Neo4j source and nothing in the image build (see `PMOVES_BUILD.md` and fork PR #4).

**Source of truth: the main repository** (POWERFULMOVES/PMOVES.AI), snapshot `origin/main` `fd3cce40369fb3ebb24e5893342ac2344abb51c2`, until an image
built from this fork ships these seeds. Copies here are verbatim and carry their source path, last commit and blob.

| Path | What it is |
|---|---|
| [`docs/TAC_NEO4J.md`](docs/TAC_NEO4J.md) | Fork-side TAC: runtime contract the image must satisfy, seeds, mindmap, personas, consciousness, other writers, proposal |
| [`docs/main/`](docs/main/) | Verbatim main-repo docs (table below) |
| [`cypher/`](cypher/) | Verbatim seeds and migrations with provenance ([`cypher/README.md`](cypher/README.md)) |
| [`scripts/neo4j_bootstrap.sh`](scripts/neo4j_bootstrap.sh) | Verbatim main-repo bootstrap; it holds the only `Persona`/`Alias` loader |

## Verbatim docs and scripts

| File (this fork) | Main-repo source | Last main commit touching it | Blob | Class |
|---|---|---|---|---|
| `scripts/neo4j_bootstrap.sh` | `pmoves/scripts/neo4j_bootstrap.sh` | `9440503693c955758fd70ad241a3534a13499623` | `cf301ddfc00b` | bootstrap (holds the inline Persona/Alias loader) |
| `docs/main/TAC_NEO4J.md` | `pmoves/docs/TAC/TAC_NEO4J.md` | `5e07b31a1e20d0eae87394f8cda0b3599de57951` | `b3d2acb92354` | doc (main TAC, verbatim) |
| `docs/main/NEO4J_MIGRATIONS.md` | `pmoves/docs/services/neo4j/NEO4J_MIGRATIONS.md` | `9440503693c955758fd70ad241a3534a13499623` | `16700a388b06` | doc |
| `docs/main/NEO4J_SERVICE_README.md` | `pmoves/docs/services/neo4j/README.md` | `1b8b0a076f9677325f86403d92f0ca25228c53f2` | `72730e2d9c4a` | doc |
| `docs/main/TAC_CONSCIOUSNESS.md` | `pmoves/docs/TAC/TAC_CONSCIOUSNESS.md` | `9440503693c955758fd70ad241a3534a13499623` | `ce3566f1bcda` | doc (conch) |
| `docs/main/darkxside-persona-seeding-2026-09-05.md` | `pmoves/docs/handoffs/darkxside-persona-seeding-2026-09-05.md` | `45ff4cbac22e69c859b08d7bdee42eaa2e352905` | `145a74cd3e10` | doc (persona key across stores) |

Relative links inside the copies point at main-repo paths and do not resolve in this fork. Read them in the main repo.

## Linked, not copied

| Main-repo path | Topic | Why only linked |
|---|---|---|
| `pmoves/services/hi-rag-gateway-v2/routes/geometry.py`, `clients/neo4j.py` | mindmap read path | code; changes with the service |
| `pmoves/services/gateway/gateway/api/workflow.py`, `mindmap.py` | mindmap writer and older reader | code; service is in no compose file |
| `pmoves/scripts/mindmap_query.py`, `mindmap_to_notebook.py` | mindmap clients | code |
| `pmoves/scripts/neo4j_container.py`, `backup-neo4j.sh`, `verify_chr_conch.sh` | ops | code |
| `pmoves/docker-compose.yml` / `docker-compose.core.yml` (service `neo4j`), `docker-compose.neo4j-tailnet.yml`, `mk/neo4j-tailnet.mk`, `config/tailscale/neo4j-tailnet-serve.json` | deployment | guard-protected compose; the contract is summarised in `docs/TAC_NEO4J.md` section 2 |
| `pmoves/monitoring/grafana/dashboards/neo4j-overview.json` | ops | dashboard |
| `pmoves/tools/seed_agent_zero_mcp.py` | Agent Zero `neo4j` MCP | code (needs APOC) |
| `pmoves/services/consciousness-service/**`, `pmoves/docs/CONCH_INTEGRATION_MAP.md`, `pmoves/docs/geometry-bus/services/consciousness-service.md` | conch | no Neo4j driver in the service |
| `pmoves/data/consciousness/load_neo4j_consciousness.sh` | conch | manual duplicate the main TAC asks to retire |
| `pmoves/services/flute-gateway/persona_selector.py`, `pmoves/services/consciousness-service/persona_gate.py`, `pmoves/docs/AGENTS/PERSONAS.md`, `pmoves/supabase/migrations/*persona*` | grounded personas | Supabase-backed; they do not read Neo4j |
| `pmoves/docs/PMOVES.AI PLANS/SESSION_IMPLEMENTATION_PLAN.md`, `CREATOR_PIPELINE_TO_CHIT.md`, `HI-RAG_UPGRADE.md`, `MAKE_TARGETS.md`, `LOCAL_TOOLING_REFERENCE.md` | plans | historical; the parts still true are in `docs/TAC_NEO4J.md` |
| `PMOVES-HiRAG/hirag/_storage/gdb_neo4j.py` | HiRAG library labels | separate submodule |
| `pmoves/tests/test_neo4j_*.py` | contract tests | run in the main repo |
| `.kilo/command/cipher-village-pr-5-neo4j-graph.md` | cipher graph brief | lane brief |

Superseded (not copied): `pmoves/docs/NEO4J_INTEGRATION_GUIDE.md`, `NEO4J_SUBMODULE_INTEGRATION_COMPLETE.md`,
`NEO4J_SUBMODULE_PROMOTION.md` (they describe a submodule Makefile and `db/seeds/001_person_aliases.csv`, neither of
which exists; the main Makefile notes the delegation was removed), and the `pmoves/docs/pmoves_all_in_one/` and
`pmoves/docs/archive/` `.cql` patches (ported into `road/002` and `road/010`).
