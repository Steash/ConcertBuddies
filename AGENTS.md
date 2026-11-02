<!-- BEGIN:nextjs-agent-rules -->

## This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

## Automated Verification Rules

1. **Static Analysis Check:** After modifying any TypeScript file, you MUST trigger `npx tsc --noEmit` via terminal execution to confirm no type regressions were introduced.
2. **Execution Check:** If modifying business logic, Server Actions, or utilities, run the relevant Vitest/Jest suite before asking for human review.
3. **Self-Correction:** If a build, lint, or type check fails, inspect the error output, patch the code, and rerun the check automatically. Do NOT pass broken builds back to the user.

# Repository Agent Guidelines & Execution Protocol

## Architecture & Code Standards
- **Framework:** Next.js App Router (`src/app`).
- **Components:** Favor React Server Components (RSC) by default. Use `'use client'` strictly at the lowest leaves of the UI tree.
- **Data Fetching:** Never fetch data on the client side when it can be streamed via Server Components or handled via Server Actions.
- **Type Safety:** Strict TypeScript rules apply. No `any` or loose type assertions (`as ...`) unless explicitly justified.

## Mandatory Closed-Loop Verification Rules
Before handing code modifications or completing a task, you MUST execute and pass automated checks:
1. Run `npx tsc --noEmit` via the terminal/tool execution to verify no static type regressions exist.
2. If changing business logic, API routes, or utility helpers, execute `npm run test`.
3. If an error occurs, inspect the error log, patch the code, and re-run the verification script automatically. **Do not pass broken code to the user.**

<!-- END:nextjs-agent-rules -->

## Agent skills

### Issue tracker

Issues and specs live as GitHub issues in `Steash/ConcertBuddies`, via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Five canonical triage roles mapped to `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: root `GLOSSARY.md` + `docs/adr/`. See `docs/agents/domain.md`.
