# TAC Tree: Neo4j, fork side (PMOVES-neo4j, line PMOVES.AI-Edition-5.26)

> **Source of truth.** The main repository (POWERFULMOVES/PMOVES.AI) owns the Neo4j contract until an image built
> from this fork ships the seeds. The main-repo TAC is copied verbatim at [`main/TAC_NEO4J.md`](main/TAC_NEO4J.md)
> (source `pmoves/docs/TAC/TAC_NEO4J.md`, provenance in [`../README.md`](../README.md)). This file does not restate
> it. It records what the fork needs to know to build the image, plus the graph knowledge the main TAC does not cover
> yet: personas, the consciousness ("conch") taxonomy, and the HiRAG library's labels.
>
> Snapshot: main-repo `origin/main` at `fd3cce40369fb3ebb24e5893342ac2344abb51c2` (2026-11-20).

## 1. Identity

| Item | Value | Where it is defined |
|---|---|---|
| Service / container | `neo4j` / `pmoves-neo4j` | main `pmoves/docker-compose.yml` (core.yml is generated from it) |
| Runtime image today | `neo4j:5.26.30-community@sha256:037cf5756f0135cbfd66b739b6df7c7c4bb100f9ce11602f6f9538e17e02c74d` | main `pmoves/docker-compose.core.yml:1144` |
| Image this fork will produce | `pmoves-neo4j` 5.26.30 Community + APOC core, from this line | fork PR #4 (`build/image-5.26.30`), `PMOVES_BUILD.md` |
| Main-repo gitlink | `9632778b26dae0b0d1a63cd34432bc4a092cce85` (2026.09 source on `PMOVES.AI-Edition-Hardened`) | main `.gitmodules:295-298` (`branch = PMOVES.AI-Edition-Hardened`) |
| Database | Community Edition has exactly one standard database, `neo4j`. Every consumer shares it, separated by label only. | main TAC section 5 |

The gitlink and the runtime are on different lines. The pin is 2026.09 source; the runtime is 5.26.30. Building an
image "from the pinned submodule" would therefore produce a different store version from the one the live graph runs.
The image must come from this line (`PMOVES.AI-Edition-5.26`), and the gitlink should move to it in the same main-repo
change that pins the image digest.

## 2. Runtime contract the image must satisfy

These come from main compose (`pmoves/docker-compose.core.yml:1133-1225`) and `pmoves/scripts/neo4j_bootstrap.sh`.
An image that breaks any of them breaks the deployment, even if it starts.

| # | Contract | Why it matters |
|---|---|---|
| R1 | Entrypoint is the vendor `/startup/docker-entrypoint.sh` under `tini -g`; compose does not override `entrypoint` or `command` | compose relies on the vendor env-to-conf loop |
| R2 | `NEO4J_PLUGINS=["apoc"]` installs APOC **without network**: the jar must exist at `/var/lib/neo4j/labs/apoc-*-core.jar` (`neo4j-plugins.json` `location`) | otherwise the entrypoint downloads from `neo4j.github.io` at start |
| R3 | `NEO4J_dbms_security_procedures_unrestricted=apoc.coll.*,apoc.text.*,apoc.path.*,apoc.algo.*` must override the plugin step's `apoc.*` | the explicit list is the hardening; without it all of `apoc.*` is unrestricted |
| R4 | `NEO4J_apoc_*` settings go to `conf/apoc.conf` (entrypoint `add_env_setting_to_conf`, `docker-entrypoint.sh:276-283`) | APOC 5 reads file-import settings only from `apoc.conf` |
| R5 | `internal.dbms.cypher_ip_blocklist=0.0.0.0/0,::/0` with `server.config.strict_validation.enabled=false` must be accepted and enforced | LOAD CSV egress block; `internal.*` is unsupported upstream, so only the A/B proves the source build keeps it |
| R6 | Heap 2g/2g and page cache 512m from env; container limit 4G | sized by `neo4j-admin server memory-recommendation --memory=4g` |
| R7 | `cap_drop: ALL` plus `CHOWN DAC_OVERRIDE FOWNER SETGID SETUID NET_BIND_SERVICE` | the entrypoint chowns and drops to uid 7474 via `su-exec` |
| R8 | Healthcheck runs `wget -qO- http://localhost:7474` inside the container | `wget` must survive the image's package purge |
| R9 | `cypher-shell` on `PATH` | `neo4j-bootstrap`, `neo4j-migrate`, `chit-mindmap-seed`, `load-consciousness-neo4j` all `docker exec ... cypher-shell` |
| R10 | `neo4j-admin database dump/load` | `scripts/backup-neo4j.sh`, `neo4j-backup`, `neo4j-restore` |
| R11 | Volumes `/data`, `/logs`; read-only mount `./neo4j/cypher:/cypher` | the mount exists, but nothing reads `/cypher` today: `neo4j_bootstrap.sh` pipes host files into `cypher-shell` over `docker exec` |

## 3. Seeds and migrations (the `cypher/` bundle)

Copied verbatim into [`../cypher/`](../cypher/); per-file provenance is in [`../cypher/README.md`](../cypher/README.md).

| Layer | Files | Applied by (main repo) |
|---|---|---|
| Road | `road/001_init.cypher` (`Entity.value` unique), `road/002_chit_constraints.cypher` (`Anchor.id`, `Constellation.id`, `Point.id`, `MediaRef.uid` unique), `road/010_chit_geometry_fixture.cypher`, `road/011_chit_geometry_smoke.cypher` | `make -C pmoves neo4j-bootstrap` applies every file once in glob order, skips 011, then loads the alias CSV, then runs 011 and requires `CHIT_SMOKE_OK` |
| Persona aliases | `seeds/person_aliases_seed.csv` | the same bootstrap, through an inline Python-to-Cypher `UNWIND` (no LOAD CSV), creating `(:Persona {slug})-[:HAS_ALIAS]->(:Alias {value})`. The loader lives in `scripts/neo4j_bootstrap.sh`, copied here for reference |
| CHIT taxonomy mindmap | `seeds/chit_mindmap_seed.cypher` (`CHITPillar`, `NATSSubject`, `CGPElement`, `Agent {id}`, `CHITModule`) | `make -C pmoves chit-mindmap-seed` |
| Consciousness taxonomy | `seeds/neo4j-consciousness-schema.cypher` (`ConsciousnessRoot/Category/Subcategory/Theory`, `Proponent`, `Implication`; `HAS_CATEGORY`, `HAS_SUBCATEGORY`, `CONTAINS_THEORY`, `PROPOSED_BY`, `HAS_IMPLICATION`, ...) | `make -C pmoves load-consciousness-neo4j` (CONCH Phase 4b) |
| Service migrations | `service-migrations/graph-linker_01_init.cypher` (`Agent.name` unique), `service-migrations/graphiti_0003_polarity_partition.cypher` (`Agent.agent_id`) | graph-linker's is applied by nothing; graph-linker is in no compose file |

## 4. What feeds the mindmap

- **Read path.** `GET /mindmap/{constellation_id}` on hi-rag-gateway-v2 (`services/hi-rag-gateway-v2/routes/geometry.py`)
  walks `(:Constellation {id})-[:HAS]->(:Point)-[:LOCATES]->(:MediaRef)`. Clients: `scripts/mindmap_query.py` and
  `scripts/mindmap_to_notebook.py` (pushes a mindmap into Open Notebook). A second, older reader is
  `services/gateway/gateway/api/mindmap.py`.
- **Write path.** The only code writer is `services/gateway/gateway/api/workflow.py:426-467`
  (`Video-[:HAS_CONSTELLATION]->Constellation-[:HAS]->Point-[:LOCATES]->MediaRef`), and services/gateway is in no
  compose file. Live mindmap data is therefore fixture data (`road/010`).
- **Persona anchors.** The alias graph (`Persona`, `Alias`) is the anchor set the creator pipeline planned to attach
  constellations to (`docs/PMOVES.AI PLANS/CREATOR_PIPELINE_TO_CHIT.md:13`). No code creates that edge yet.

## 5. Personas in the graph (not covered by the main TAC)

- Nodes: `(:Persona {slug, name})` and `(:Alias {value, alias_type, source, source_reference, confidence, notes})`
  joined by `HAS_ALIAS`, from the bootstrap. Seed slugs: `powerfulmoves`, `darkxsideshows`, with 4 aliases.
- **No uniqueness constraint** on `Persona.slug` or `Alias.value`, yet both are MERGE keys (same gap class as
  `Constellation.id` before `002`).
- The grounded-persona runtime does **not** read Neo4j. `services/flute-gateway/persona_selector.py`,
  `services/consciousness-service/persona_gate.py` and the A0 / Mavis personas resolve from Supabase
  (`pmoves_core.personas`, migrations `20250101000000_grounded_personas_kb.sql` onward) and YAML. The graph's persona
  nodes are an alias dictionary only.
- Key drift (main `docs/handoffs/darkxside-persona-seeding-2026-09-05.md`, copied to [`main/`](main/)): the registry
  key is `darkxside-persona`, Supabase uses `personas.name`, Neo4j uses `Persona.slug = darkxsideshows`. The handoff's
  resolution is an alias, not a rename.
- Planned but never built: `SESSION_IMPLEMENTATION_PLAN.md:63-80` names `neo4j/cypher/002_load_person_aliases.cypher`
  and `(:Persona)-[:FEATURES_IN]->(:Media)` / `COLLABORATES_WITH` edges. The `002` file never existed; the loader was
  folded into `neo4j_bootstrap.sh`, and the `002` slot now holds `002_chit_constraints.cypher`.

## 6. The consciousness ("conch") pipeline and Neo4j

- `consciousness-service` (port 8106) has **no Neo4j driver** (main `TAC_CONSCIOUSNESS.md`, copied to [`main/`](main/)).
  It emits CGP packets to NATS. Persisting a signature beside a graph node is planned, not implemented.
- The taxonomy graph is loaded out of band: `make -C pmoves load-consciousness-neo4j` applies
  `seeds/neo4j-consciousness-schema.cypher`. `data/consciousness/load_neo4j_consciousness.sh` is a manual duplicate that
  leaves unlabeled nodes and a second `Entity.id` key; the main TAC asks to retire it (LINK only here).
- `scripts/verify_chr_conch.sh` checks the CHR/CONCH path; its Neo4j step (`:44-49`) counts `Entity` nodes in the
  container named by `scripts/neo4j_container.py`.

## 7. Other writers sharing the database

| Writer | Labels | Note |
|---|---|---|
| cipher-api (`Pmoves-cipher/src/pmoves/graph.ts`) | `:Memory`, `SAME_CATEGORY` | creates its own constraints |
| hi-rag-gateway-v2 (`clients/neo4j.py`) | reads `Entity` | relies on `road/001` |
| PMOVES-HiRAG library (`hirag/_storage/gdb_neo4j.py`) | upper-case allowlisted labels (`ENTITY`, `PERSON`, `ORGANIZATION`, ...) plus a namespace label | measured on the locally checked-out submodule (`d8484b2`), not the main-repo pin (`03d6134`). `ENTITY` and `Entity` are different labels in Neo4j, so this library's nodes are invisible to `clients/neo4j.py` |
| Agent Zero MCP `neo4j` | any | needs APOC (R2) |
| graph-linker, graphiti | `Agent` with keys `name` and `agent_id` | `Agent` has three competing unique keys (`id`, `name`, `agent_id`); one owner has to choose |

## 8. Proposal: what the image could carry (not implemented here)

Proposed in this PR's description only. This PR does not change the `Dockerfile`.

1. Bake `pmoves/cypher/` into the image at `/pmoves/cypher` (read-only, owned by root). The compose mount
   `./neo4j/cypher:/cypher:ro` stays authoritative until the main repo drops it.
2. Ship a `neo4j-pmoves-bootstrap` helper (the bootstrap logic, reading from `/pmoves/cypher`) run by `docker exec`,
   **not** from the entrypoint. Seeding at every start would re-run the fixture against production data.
3. Add `cypher-shell --version` and `neo4j-admin --version` to the CI smoke (R9, R10), and compare the
   `/startup/docker-entrypoint.sh` sha256 with a pinned value instead of only printing it.

## 9. Open items

- A/B of the source build against the vendor image: not yet run (PMOVES_BUILD.md, A/B acceptance).
- Main-repo GHCR: `.github/workflows/integrations-ghcr.yml` lists `PMOVES-Neo4j` in its path filters, but
  `integrations-ghcr.matrix.json` has no Neo4j entry, so a gitlink bump triggers the workflow with nothing to build.
  This is intentional for now: the image build lives in this fork (`pmoves-image.yml`, 150 min) because the main matrix
  job is limited to 45 min.
- Constraints for `Persona.slug` and `Alias.value` (section 5) and one owner for the `Agent` key (section 7).
