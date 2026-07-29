## Context

Metric types (`Counter`, `Gauge`, `Histogram`, `Summary`, `Info`), labeled `Family` collections, thread-safe `Registry`, and Prometheus text exposition serialization.

## Tech

- C++11 library target `prometheus-cpp-core` (`core/CMakeLists.txt`, `core/BUILD.bazel`)
- Public headers under `core/include/prometheus/`
- Tests: GoogleTest in `core/tests/`; optional Google Benchmark in `core/benchmarks/`

## Architecture

Builders (`BuildCounter`, `BuildGauge`, …) register a `Family<T>` into `Registry`. Scrapers call `Registry::Collect()` → `TextSerializer` for exposition text. `InsertBehavior::Merge` vs `Throw` controls duplicate family-name handling (`core/include/prometheus/registry.h`).

## Patterns

- DO: register families once, then `Family::Add` fixed low-cardinality label sets (`core/include/prometheus/family.h`, `pull/tests/integration/sample_server.cc`)
- DO: treat `Registry` as long-lived `shared_ptr` owned by the process (`core/include/prometheus/registry.h`)
- DON'T: call `Family::Add` on every request with unbounded label values (cardinality + lock cost; see `README.md` usage loop)
- DON'T: mix metric types under the same family name (registry throws)

## Key Files

- `core/include/prometheus/registry.h` — collection hub / insert behavior
- `core/include/prometheus/family.h` — labeled metric dimensions
- `core/include/prometheus/counter.h` / `gauge.h` / `histogram.h` / `summary.h` / `info.h` — metric types
- `core/include/prometheus/text_serializer.h` — text exposition format
- `core/src/registry.cc` — registration and collect implementation
