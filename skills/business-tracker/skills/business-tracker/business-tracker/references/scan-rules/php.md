# Scan Rules — PHP

File priority rules for PHP projects (tag: `php`).

| Priority | File patterns |
|----------|--------------|
| P0 | `app/Services/**`, `app/Domain/**`, `app/Actions/**`, `app/Handlers/**` |
| P1 | `app/Models/**`, `app/Http/Controllers/**`, `app/Repositories/**`, `app/Http/Middleware/**`, `routes/**` |
| P2 | `app/Helpers/**` / `app/Utils/**` / `config/**` / `app/Enums/**` with business vocabulary |
| Skip | `vendor/`, `storage/`, `bootstrap/cache/`, `tests/`, `database/migrations/`, `public/`, `resources/views/**` (pure templates) |
