# PMOVES build of Neo4j Community 5.26.30

Branch `PMOVES.AI-Edition-5.26` = upstream `neo4j/neo4j` tag `5.26.30` (tag object `88e8e8a0`, commit
`d3ee27440d2c5fee6d4087cf1e05f30bf687e9c3`), so the image matches the store version PMOVES.AI runs today
(`neo4j:5.26.30-community`). The 2026.x line stays on `PMOVES.AI-Edition-Hardened` and is a separate migration.

## What the image is made of

The vendor image comes from three repositories. This build reproduces all three:

| Part | Source | License | How |
|---|---|---|---|
| Server tarball | this repository | GPL-3.0 (`LICENSE.txt`) | Maven, `packaging/standalone/standalone-community` -> `neo4j-community-5.26.30-unix.tar.gz` |
| APOC core | `neo4j/apoc` tag `5.26.30` (tag object `064f8c71`, commit `a0ffb507`) | Apache-2.0 | Gradle `:core:shadowJar`, placed at `labs/apoc-5.26.30-core.jar` as the vendor tarball has it |
| Image packaging | `neo4j/docker-neo4j` tag `neo4j-5.26.30` | Apache-2.0 | `docker/local-package/` (see `docker/UPSTREAM.md`) + the trixie image recipe |

The image build file itself is a protected path in PMOVES.AI's tooling and lands in a follow-up commit on this PR once
the grant is given. Its shape: stage 1 Maven (JDK 17, as upstream README.asciidoc requires for 5.26), stage 2 Gradle
APOC with the tag object and commit both verified, stage 3 = docker-neo4j's trixie recipe at `neo4j-5.26.30` with only
these deviations: base images pinned by digest; the tarball and APOC jar copied from stages 1 and 2 instead of a
download plus sha256 check; the APOC jar moved into `labs/`; OCI labels. Everything else, including the vendor's
permission model, is unchanged so the A/B compares builds, not packaging choices.

## The `-SNAPSHOT` version caveat (measured)

- Every pom at tag `5.26.30` carries `5.26.30-SNAPSHOT` (161 poms; `mvn help:evaluate -Dexpression=project.version` = `5.26.30-SNAPSHOT`).
- The jars' `Implementation-Version` comes from `project.version` (`pom.xml:425`, `addDefaultImplementationEntries`), and
  `org.neo4j.kernel.internal.Version` reads it. The vendor jars say `Implementation-Version: 5.26.30`
  (`neo4j-kernel-5.26.30.jar` from the vendor image), so the vendor release rewrites the version.
- Without the rewrite, `5.26.30-SNAPSHOT` would surface in `neo4j --version`, `dbms.components()`, the HTTP discovery
  document, and the Bolt `server` agent (`HelloStateTransition`). APOC's startup check compares only major.minor
  (`apoc/cypher/CypherInitializer.java:98-106`), so it would not warn either way.
- Fix used here: `mvn versions:set -DnewVersion=5.26.30 -DprocessAllModules=true` rewrites all 161 poms; afterwards
  `project.version` = `5.26.30` and 0 poms contain `-SNAPSHOT` (measured in a throwaway maven container). The build
  asserts the 0.
- Not yet measured: a full image build and the A/B against the vendor image (the PMOVES sandbox road reported
  could-not-measure on 2026-10-01, and the build host lacked memory headroom for a full Maven reactor).

## A/B acceptance before PMOVES.AI pins this image

`neo4j --version`; `CALL dbms.components()`; `RETURN apoc.version()`; `ls lib labs plugins` diff against the vendor image;
`/startup` hashes; start against a COPY of a 5.26.30 store (`neo4j-admin database load` into a throwaway volume) and
compare node/relationship counts. GHCR publish and the PMOVES.AI digest pin follow the A/B.
