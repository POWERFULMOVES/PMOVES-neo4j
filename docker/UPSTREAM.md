# Vendored from neo4j/docker-neo4j

- Repository: https://github.com/neo4j/docker-neo4j (Apache-2.0, see `LICENSE.docker-neo4j`)
- Tag: `neo4j-5.26.30`, commit `d00c64fde2cc0939c103408e0338ab1b3c29f4c0`

| File here | Upstream path | Change |
|---|---|---|
| `local-package/docker-entrypoint.sh` | `docker-image-src/5/coredb/docker-entrypoint.sh` | the one line `    #%%DEPRECATION_WARNING_PLACEHOLDER%%` removed, which is what upstream `build-scripts/build-docker-image.sh` does for a non-deprecated 5.x image |
| `local-package/neo4j-admin-report.sh` | `docker-image-src/5/coredb/neo4j-admin-report.sh` | none |
| `local-package/neo4j-plugins.json` | `docker-image-src/5/coredb/neo4j-plugins.json` | none |
| `local-package/utilities.sh` | `docker-image-src/common/utilities.sh` | none |
| `local-package/semver.jq` | `docker-image-src/common/semver.jq` | none |

## Proof these are the files the vendor image runs

sha256 of each file here against `/startup/<file>` inside `neo4j:5.26.30-community@sha256:037cf5756f0135cbfd66b739b6df7c7c4bb100f9ce11602f6f9538e17e02c74d`
(measured 2026-10-01): `docker-entrypoint.sh` 9527409f6c44..., `utilities.sh` 5cd8fcb5f1e5..., `semver.jq` be7b5d1d5573...,
`neo4j-plugins.json` 7885136f3526... are byte-identical. `neo4j-admin-report.sh` is moved to `bin/neo4j-admin-report` by the image build.

## Re-vendoring

Check out the matching `neo4j-<version>` tag of docker-neo4j, copy the five files, re-apply the one-line removal,
and re-measure the hashes against the vendor image of the same version.
