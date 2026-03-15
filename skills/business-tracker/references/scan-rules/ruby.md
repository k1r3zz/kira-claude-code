# Scan Rules — Ruby

File priority rules for Ruby projects (tag: `ruby`).

| Priority | File patterns |
|----------|--------------|
| P0 | `app/services/**`, `app/models/**` (with business methods), `app/jobs/**`, `app/workers/**`, `lib/domain/**` |
| P1 | `app/controllers/**`, `app/serializers/**`, `config/routes.rb`, `app/policies/**` |
| P2 | `lib/**` / `app/helpers/**` / `config/initializers/**` with business vocabulary |
| Skip | `vendor/`, `spec/`, `test/`, `db/migrate/` (auto-generated), `tmp/`, `log/`, `public/`, `node_modules/` |
