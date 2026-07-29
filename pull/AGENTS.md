## Context

HTTP metrics exposer: embeds CivetWeb, serves collectables (default `/metrics`), optional basic auth and gzip, plus self-metrics for scrape bytes/latency.

## Tech

- Depends on `core`, `util`, CivetWeb; optional zlib when `ENABLE_COMPRESSION` / Bazel `HAVE_ZLIB` (`pull/CMakeLists.txt`, `pull/BUILD.bazel`)
- Gated by CMake `ENABLE_PULL` (default ON)
- Unit tests: `pull/tests/unit/`; integration samples + Telegraf scrape: `pull/tests/integration/`

## Architecture

`Exposer` owns a `CivetServer` and per-URI `Endpoint`s. `RegisterCollectable(weak_ptr)` attaches registries; `MetricsHandler` collects, serializes text, optionally gzip-compresses, and updates exposer self-metrics (`pull/src/handler.cc`, `pull/src/exposer.cc`).

## Patterns

- DO: keep the `shared_ptr<Registry>` alive after `RegisterCollectable` (`pull/include/prometheus/exposer.h`)
- DO: use `pull/tests/integration/sample_server.cc` as the pull usage reference
- DO: run scrape integration with Telegraf installed: `bazel test //pull/tests/integration:scrape-test`
- DON'T: assume compression is on in submodule CMake CI — that job sets `-DENABLE_COMPRESSION=OFF`
- DON'T: use CivetWeb < 1.14 (`pull/src/handler.cc` compile-time check)

## Key Files

- `pull/include/prometheus/exposer.h` — public exposer API
- `pull/src/exposer.cc` — server lifecycle / endpoint map
- `pull/src/handler.cc` — scrape handler, gzip, self-metrics
- `pull/src/basic_auth.cc` — HTTP basic auth (base64 via `util`)
- `pull/tests/integration/sample_server.cc` — end-to-end example
