# Scan Rules — .NET

File priority rules for .NET projects (tag: `dotnet`).

| Priority | File patterns |
|----------|--------------|
| P0 | `**/Services/**`, `**/Domain/**`, `**/Handlers/**`, `**/UseCases/**` |
| P1 | `**/Models/**`, `**/Entities/**`, `**/Controllers/**`, `**/Repositories/**`, `**/Middleware/**` |
| P2 | `**/Helpers/**` / `**/Utils/**` / `**/Constants/**` / `**/Configuration/**` with business vocabulary |
| Skip | `bin/`, `obj/`, `Migrations/`, `**/Tests/**`, `*.Designer.cs`, `*.g.cs` |
