## Context

libcurl client for Prometheus Pushgateway: Push / PushAdd / Delete (sync and async), optional basic auth, timeouts, and custom headers.

## Tech

- Depends on `core`, `util`, `CURL::libcurl` (`push/CMakeLists.txt`, `push/BUILD.bazel`)
- Gated by CMake `ENABLE_PUSH` (default ON; submodule CI example disables it)
- Tests: `push/tests/internal/` (label encoder); sample client in `push/tests/integration/`

## Architecture

`Gateway` builds `host:port/metrics/job/<job>` plus instance labels, holds `weak_ptr` collectables, serializes with `TextSerializer`, and sends via `detail::CurlWrapper` (`push/src/gateway.cc`, `push/src/detail/curl_wrapper.cc`).

## Patterns

- DO: construct with host/port/job (and optional labels/auth/timeout) or the `presetupCurl` overload for custom `CURL*` setup (`push/include/prometheus/gateway.h`)
- DO: keep registered registries alive across `Push` / `AsyncPush` (`push/include/prometheus/gateway.h`)
- DO: use `push/tests/integration/sample_client.cc` as the push usage reference
- DON'T: expect push to build in the README/submodule CMake snippet without curl (`-DENABLE_PUSH=OFF` there)
- DON'T: forget job label encoding goes through `push/src/detail/label_encoder.cc`

## Key Files

- `push/include/prometheus/gateway.h` — public Gateway API
- `push/src/gateway.cc` — URI assembly, push/delete orchestration
- `push/src/detail/curl_wrapper.cc` — libcurl wrapper
- `push/src/detail/label_encoder.cc` — path/label encoding
- `push/tests/integration/sample_client.cc` — example client
