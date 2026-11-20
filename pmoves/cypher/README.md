# PMOVES Neo4j seeds and migrations (verbatim copies)

Every file under this directory is a **byte-for-byte copy** of a file in POWERFULMOVES/PMOVES.AI at `origin/main`
`fd3cce40369fb3ebb24e5893342ac2344abb51c2` (2026-11-20). Nothing here is applied by this fork or baked into its image yet.

**The main repository stays the source of truth** until an image built from this fork ships these files and the main
repo's compose and Make roads point at the image copy. Until then, edit the main-repo file and re-copy it here. A
mismatch between a blob below and the main-repo blob means this copy is stale, not that the main repo is wrong.

Check a copy: `git -C <PMOVES.AI> rev-parse origin/main:<main path>` must equal the Blob column (prefix shown).

| File (this fork) | Main-repo source | Last main commit touching it | Blob | Class |
|---|---|---|---|---|
| `road/001_init.cypher` | `pmoves/neo4j/cypher/001_init.cypher` | `b5e1355981bf946258e50d48815d5062078218e9` | `4685d52e818e` | migration (road) |
| `road/002_chit_constraints.cypher` | `pmoves/neo4j/cypher/002_chit_constraints.cypher` | `9440503693c955758fd70ad241a3534a13499623` | `dcb2f54e10c4` | migration (road) |
| `road/010_chit_geometry_fixture.cypher` | `pmoves/neo4j/cypher/010_chit_geometry_fixture.cypher` | `9440503693c955758fd70ad241a3534a13499623` | `d6236b475158` | seed (road, mindmap fixture) |
| `road/011_chit_geometry_smoke.cypher` | `pmoves/neo4j/cypher/011_chit_geometry_smoke.cypher` | `9440503693c955758fd70ad241a3534a13499623` | `7a0f30428492` | smoke (road) |
| `seeds/person_aliases_seed.csv` | `pmoves/neo4j/datasets/person_aliases_seed.csv` | `0f93e851bac7037c971b77054092329249602dcc` | `a08fae6d3413` | seed (persona aliases) |
| `seeds/chit_mindmap_seed.cypher` | `pmoves/tools/chit_mindmap_seed.cypher` | `af9b733309571dd05cc8d423352acc924d6c28f4` | `c0229e708331` | seed (CHIT taxonomy mindmap) |
| `seeds/neo4j-consciousness-schema.cypher` | `pmoves/data/consciousness/neo4j-consciousness-schema.cypher` | `9440503693c955758fd70ad241a3534a13499623` | `77132f2fa8b6` | seed (consciousness taxonomy) |
| `service-migrations/graph-linker_01_init.cypher` | `pmoves/services/graph-linker/migrations/01_init.cypher` | `fed8cbfe59b4ea8621977f3b53bb4546f19a8e14` | `313933857d23` | migration (applied by nothing) |
| `service-migrations/graphiti_0003_polarity_partition.cypher` | `pmoves/services/graphiti/migrations/0003_polarity_partition.cypher` | `6e53b09721c31bfc6e08863bd0442a03271c076f` | `a512317ed539` | migration (graphiti) |

## Order and roads (main repo)

- `road/`: `make -C pmoves neo4j-bootstrap` (`pmoves/scripts/neo4j_bootstrap.sh`) applies `001`, `002`, `010` in glob
  order (`011` is skipped in the loop), loads `seeds/person_aliases_seed.csv` as `Persona`/`Alias`/`HAS_ALIAS` through an
  inline `UNWIND`, then runs `011` and exits 1 unless it returns `CHIT_SMOKE_OK`. One file at a time:
  `make -C pmoves neo4j-migrate VERSION=001`.
- `seeds/chit_mindmap_seed.cypher`: `make -C pmoves chit-mindmap-seed`.
- `seeds/neo4j-consciousness-schema.cypher`: `make -C pmoves load-consciousness-neo4j`.
- `service-migrations/`: graph-linker's `01_init` is applied by nothing (graph-linker is in no compose file); graphiti's
  `0003` belongs to the graphiti service.
- Every road file keeps each statement self-contained (cypher-shell does not carry variables across `;`); main
  `pmoves/tests/test_neo4j_cypher_static.py` enforces that.

The main-repo doc for these is [`../docs/main/NEO4J_MIGRATIONS.md`](../docs/main/NEO4J_MIGRATIONS.md).
