# Scan Rules — Rust

File priority rules for Rust projects (tag: `rust`).

| Priority | File patterns |
|----------|--------------|
| P0 | `src/service/**`, `src/domain/**`, `src/handler/**`, `src/usecase/**` |
| P1 | `src/model/**`, `src/repository/**`, `src/routes/**`, `src/middleware/**` |
| P2 | `src/utils/**` / `src/config/**` / `src/constants/**` with business vocabulary |
| Skip | `target/`, `tests/`, `benches/`, `examples/`, `*.lock` |
