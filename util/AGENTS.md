## Context

Header-only base64 helpers shared by pull basic-auth and push; CMake exports an INTERFACE `util` target, Bazel visibility is package-internal.

## Tech

- INTERFACE / header-only (`util/CMakeLists.txt`, `util/BUILD.bazel`)
- Single public header: `util/include/prometheus/detail/base64.h`
- Tests: `util/tests/unit/base64_test.cc`

## Architecture

No runtime library. Pull `basic_auth.cc` and push internals include `prometheus/detail/base64.h` for encode/decode (including base64url).

## Patterns

- DO: include `prometheus/detail/base64.h` only from library internals that need auth/encoding (`pull/src/basic_auth.cc`)
- DO: extend coverage via `util/tests/unit/base64_test.cc` when changing encode/decode behavior
- DON'T: treat `detail::` symbols as stable app-facing API
- DON'T: broaden Bazel visibility beyond `//:__subpackages__` without intent (`util/BUILD.bazel`)

## Key Files

- `util/include/prometheus/detail/base64.h` — encode/decode implementation
- `util/tests/unit/base64_test.cc` — unit coverage
- `util/CMakeLists.txt` — INTERFACE target export
- `util/BUILD.bazel` — Bazel library visibility
- `pull/src/basic_auth.cc` — primary in-tree consumer
