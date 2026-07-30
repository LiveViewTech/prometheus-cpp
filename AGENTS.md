## Purpose

C++ client library implementing the Prometheus data model so services can collect metrics and expose them via HTTP pull (`Exposer`) or Pushgateway push (`Gateway`).

## Project Snapshot

- Tech: C++11, CMake and Bazel dual builds, GoogleTest, Google Benchmark
- Type: single multi-library repo (`core`, `pull`, `push`, `util`)
- Version: 1.3.0 (`CMakeLists.txt`, `MODULE.bazel`)

## Commands

```bash
git submodule update --init
mkdir _build && cd _build
cmake .. -DBUILD_SHARED_LIBS=ON -DENABLE_PUSH=OFF -DENABLE_COMPRESSION=OFF
cmake --build . --parallel 4
ctest -V
cmake --install .
```

```bash
bazel build //...
bazel test //...
bazel run -c opt //core/benchmarks
bazel test //pull/tests/integration:scrape-test
```

CMake submodule/example flow above turns push and compression off because zlib/curl are not in `3rdparty`. Enable them only when system/vcpkg deps are present (`ENABLE_PUSH`, `ENABLE_COMPRESSION`). Scrape integration needs Telegraf installed.

## Conventions

- Style: Google C++ Style Guide; clang-format before PRs (`.clang-format`)
- Commits: follow https://chris.beams.io/posts/git-commit/
- Public APIs live under `*/include/prometheus/`; `detail/` and `*/src/` are internal
- Prefer pre-created `Family::Add` label sets over hot-path dynamic `Add` (see `README.md` usage notes)
- Keep `Registry` (or other `Collectable`) alive for the lifetime of `Exposer`/`Gateway` registration (`std::weak_ptr`)
- Duplicate family names: `InsertBehavior::Merge` vs `Throw` (`core/include/prometheus/registry.h`)

## Directory Map

- `core/` — metrics, registry, text serializer
- `pull/` — CivetWeb HTTP exposer (optional zlib when `ENABLE_COMPRESSION`)
- `push/` — libcurl Pushgateway client
- `util/` — header-only base64 helpers (`detail/`, not app-facing)
- `cmake/` — package config, pkg-config templates, import smoke tests
- `bazel/` — export-header and third-party BUILD helpers
- `3rdparty/` — git submodules (`civetweb`, `googletest`); empty until `git submodule update --init`
- `doc/` — Doxygen (`doc/Doxyfile`)
- `.github/workflows/` — CMake, Bazel, coverage, lint, release, doxygen CI

## Gotchas

- **Registry lifetime:** `Exposer`/`Gateway` store `weak_ptr<Collectable>`; destroying the registry drops metrics from scrapes/pushes (`pull/include/prometheus/exposer.h`, `push/include/prometheus/gateway.h`).
- **CivetWeb ≥ 1.14:** `pull/src/handler.cc` `#error`s on older versions.
- **Link order (manual):** pull needs `-lprometheus-cpp-pull -lprometheus-cpp-core -lz`; push needs `-lprometheus-cpp-push -lprometheus-cpp-core -lcurl -lz` (`README.md`).
