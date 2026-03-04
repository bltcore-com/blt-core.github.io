# BLT
Primary project context loaded at the start of every session. Add coding standards, build commands, architectural decisions, and domain knowledge here.

## About This Project
BLT is an acronym of the company name Beyond Limited Thinking, LLC (bltwai.com).  BLT Core is an AI-native, modular point-of-sale (POS) infrastructure 
built for restaurants and retail. BLTPos A vibe coded  AI-native restaurant point-of-sale system. Our focus is on composability, open standards, and AI-first workflows—supporting rapid, flexible integration with banks, independent sales prganization (ISOs), and strategic partners. BLT provides a payments-agnostic alternative to legacy POS software, enabling full control of the merchant relationship and economics.

## About the Company
- **Industry:** Tech / Software — Enterprise Software / Fintech / Payments
- **Company Name:** Beyond Limited Thinking, LLC (a.k.a BLT)
- **Product:** BLT Core — AI-native restaurant point-of-sale infrastructure
- **Value proposition:** "Payments freedom" — gives banks, ISOs, and strategic partners full ownership of the merchant relationship and payments economics
- **Differentiation:** Modern, scalable, payments-agnostic alternative to legacy POS incumbents. No forced processor lock-in, no proprietary hardware requirements, no hidden fees. Competitive parity with legacy systems but built AI-native.
- **Vision:** Modern, minimal, typographic—no proprietary lock-in

## Tech Stack
- **Runtime:** Bun >= 1.0.0
- **Language:** TypeScript (ES2022 target, ESNext modules)
- **CLI Framework:** `@caporal/core` — command/subcommand registration, args, options, logger
- **Database:** `pg` (PostgreSQL), Supabase (`@supabase/supabase-js`)
- **Storage:** Supabase Storage (bucket operations)
- **Image Processing:** `sharp` (optional)
- **PDF:** `pdf-lib`
- **YAML/SQL:** `js-yaml` for YAML→SQL conversion pipeline
- **Debug:** `debug` (namespaced logging)
- **Build:** `bunx --bun tsc` → rename to `dist/blt` → chmod +x

## Key Conventions
- Commands live in `src/commands/<name>.ts`; subcommands in `src/commands/<name>/<sub>.ts`
- Each command file default-exports a function `(program: Program) => void` that registers with Caporal
- Top-level command has `.action()` that prints help; subcommands `.hide()` and call implementations
- Shared libs in `src/lib/` (constants, database-runner, yaml-converter, sql-builder, etc.)
- Config loaded via `src/config/config.ts` from `blt.config.json`, `.bltrc`, `~/.blt/config.json`, or env vars
- Supabase client via `src/utils/supabase.ts` (uses `SUPABASE_DATA_API` + `SUPABASE_SERVICE_ROLE_KEY`)
- Error handling: try/catch in handlers, `logger.error()`, `process.exit(1)`
- Output formats: many commands support `--format table|json`

## Documentation & Source
- `README.md` — CLI usage reference
- `blt.md` — Extended guide (env setup, workflows, troubleshooting, smashdata pattern)
- `version.json` — Auto-generated build metadata (postbuild hook)
- Schema/data layout: `schema/<name>/sql/` and `schema/instances/<instance>/{sql,yaml}/`
- BLT repos: tools, pos, data, gateway, deploy, customers, logz

## Build & Run
```bash
bun install              # install dependencies
bun run build            # tsc + rename dist/blt.js → dist/blt + chmod +x
bun run dev              # tsc --watch
bun run start            # bun run src/blt.ts (dev entry point)
bun run test             # placeholder (tests TBD)
```
Postbuild automatically runs `src/commands/version/update.ts` to refresh `version.json`.

## Architecture
Entry point `src/blt.ts` imports each command module and calls its registration function, then `program.run()`. Commands:
- **image** — convert (WebP), sharpen, enhance, color
- **pdf** — binder (combine PDFs), folio (combine with page numbers/TOC)
- **version** — update, string, sql
- **init** — generate shell script to clone/pull BLT repos
- **wai** — scaffold AI config files (CLAUDE.md, AGENTS.md, skills, rules, MCP config)
- **build** — schema (DDL→SQL), data (instance YAML→SQL)
- **deploy** — schema, data, sql, functions (Supabase Edge), secrets
- **bucket** — names, list, upload, upload-folder, download, url, clear
- **workflow** — GitHub Actions: list, show, delete, reload, deploy
- **show** — schema info, row counts, env vars, db version, repos
- **cleanup** — remove generated SQL, strip rarely used YAML tags

## Coding Standards
- TypeScript strict mode, ESM (`"type": "module"`)
- Async handlers with Caporal's `{ args, options, logger }` signature
- Use `debug` for verbose/trace logging, Caporal `logger` for user-facing output
- Prefer `Bun.file` / Bun APIs where applicable; `pg` for direct SQL execution
- No tests yet — `bun run test` is a placeholder

## Design Language
### Philosophy
CLI-only tool — no GUI. Output is terminal text using Caporal's built-in logger levels (`info`, `warn`, `error`). Structured output uses `--format table|json`. Database runner uses box-drawing characters for visual structure.

### Colors
N/A — relies on terminal defaults and Caporal logger coloring.

### Typography
N/A — monospace terminal output.

### Borders & Spacing
Database runner output uses Unicode box-drawing characters for tables and status markers.

### Interactive Elements
N/A — non-interactive CLI; all input via arguments and options.

### Component Patterns
- Subcommand pattern: registration file + implementation directory
- Shared discovery modules (`schema-discovery.ts`, `instance-discovery.ts`) for filesystem scanning
- `yaml-converter.ts` handles env variable substitution and multi-target SQL generation
- `sql-builder.ts` generates SQL file headers and orchestrates file writes

### Do's
- Follow the subcommand file layout (`commands/<cmd>.ts` + `commands/<cmd>/<sub>.ts`)
- Use `getConfig()` / `getPaths()` for all path resolution
- Handle errors with try/catch, log with `logger.error()`, exit with `process.exit(1)`
- Use `getSupabaseClient()` for Supabase operations, `SUPABASE_CONNECTION_STRING` for direct pg

### Don'ts
- Don't hardcode paths — use `lib/constants.ts` and config resolution
- Don't use `console.log` — use Caporal `logger` or `debug`
- Don't add dependencies without checking if Bun has a built-in equivalent
- Don't skip the postbuild version update step
