# Scan Rules — Go

File priority rules for Go projects (tag: `go`).

| Priority | File patterns |
|----------|--------------|
| P0 | `internal/service/**`, `internal/domain/**`, `internal/usecase/**`, `pkg/service/**`, `handler/**` |
| P1 | `internal/model/**`, `internal/repository/**`, `internal/middleware/**`, `api/**`, `cmd/**` (entry logic) |
| P2 | `pkg/utils/**` / `internal/config/**` / `internal/constants/**` with business vocabulary |
| Skip | `vendor/`, `*_test.go`, `mock_*`, `*_mock.go`, `testdata/`, `docs/`, `scripts/`, `*.pb.go`, `*_gen.go` |
