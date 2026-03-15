# Scan Rules — Node.js / TypeScript

File priority rules for Node.js projects (tags: `node-frontend`, `node-backend`).

| Priority | File patterns |
|----------|--------------|
| P0 | `services/**`, `handlers/**`, `controllers/**`, `usecases/**`, `domain/**`, `**/store.ts`, `**/store.js`, `**/actions.ts`, `**/reducer.ts`, `**/slice.ts` |
| P1 | `models/**`, `schemas/**`, `routes/**`, `middleware/**`, `entities/**`, `repositories/**`, `**/hooks/*.ts` |
| P2 | `utils/**` / `helpers/**` with business vocabulary, `config/**`, `constants.*`, `**/api/*.ts` |
| Skip | `node_modules/`, `dist/`, `build/`, `.next/`, `.nuxt/`, `*.test.ts`, `*.spec.ts`, `__tests__/`, `*.d.ts`, `*.config.js` (build config), `public/`, `static/` |
