# AGENTS.md

## Scope
This file applies to the entire repository.

## Project Summary
This repository is a Cloudflare Workers application for RSS subscription and aggregation.
It uses GitHub OAuth for access control, stores feed definitions in Cloudflare KV, stores aggregated RSS content in Cloudflare R2, and exposes both authenticated management APIs and a public RSS content API.

## Stack
- Runtime: Cloudflare Workers
- Language: TypeScript with `strict` mode enabled
- HTTP framework: Hono
- RSS parsing: `rss-parser`
- Storage:
  - `RSS_FEEDS` for feed metadata and app settings
  - `RSS_BUCKET` for generated RSS payloads
- Local/dev tooling: Wrangler

## Repository Layout
- `src/index.ts`: main Worker entry, routes, auth middleware, cron handler, feed refresh logic
- `src/types.ts`: shared TypeScript types and Cloudflare bindings
- `public/`: static HTML assets
- `wrangler.jsonc`: Worker config, bindings, assets, cron trigger
- `README.md`: deployment and product overview

## Commands
- Install dependencies: `npm install`
- Start local development: `npm run dev`
- Deploy Worker: `npm run deploy`
- Type-check before finishing a non-trivial change: `npx tsc --noEmit`

## Environment And Bindings
Keep bindings and code in sync. This project currently depends on:
- `RSS_FEEDS`
- `RSS_BUCKET`
- `GITHUB_CLIENT_ID`
- `GITHUB_CLIENT_SECRET`
- `ALLOWED_GITHUB_USERS`
- `APP_URL`
- `IMG_PROXY_URL`

If you add, rename, or remove a binding:
- update `src/types.ts`
- update `wrangler.jsonc`
- update `README.md` if setup steps changed

## Working Rules
- Prefer small, targeted edits. This project is compact and most behavior is centralized in `src/index.ts`.
- Preserve Cloudflare Workers compatibility. Avoid introducing Node-only APIs unless Wrangler compatibility explicitly supports them.
- Keep route behavior, binding names, and storage keys stable unless the task explicitly requires a breaking change.
- When changing API payloads or route semantics, update both frontend callers and backend handlers in the same change.
- Keep TypeScript strict-safe. Do not silence type errors without a concrete reason.
- Reuse existing types from `src/types.ts` before adding new ad hoc shapes.

## Frontend Notes
- There are currently two UI sources for the main pages:
  - inline HTML returned from routes in `src/index.ts`
  - static files in `public/index.html` and `public/login.html`
- Before changing login or dashboard UI, verify which version is actually serving the route and keep behavior aligned.
- If you consolidate the UI to a single source, do it explicitly and remove the duplicate path in the same change.

## Storage And Data Rules
- `RSS_FEEDS` stores the feed list under the `feeds` key. Preserve that key unless performing an intentional migration.
- `RSS_BUCKET` stores aggregated content in `rss.json`. Keep this contract stable for the public API unless coordinated changes are made.
- Feed refresh logic should tolerate partial failures. Do not let one broken feed abort the entire refresh job.

## Auth And Security
- Protected routes must continue to require GitHub authentication.
- Authorization is allowlist-based via `ALLOWED_GITHUB_USERS`.
- Be careful with cookies, redirects, and CORS changes. Small regressions here can break login or cross-origin usage.
- Never commit real secrets, tokens, or Cloudflare identifiers that are environment-specific beyond what is already present in the repo.

## Validation Expectations
For code changes, run or describe the most relevant checks:
- `npx tsc --noEmit`
- manual route smoke test with `npm run dev`
- if auth, cron, KV, or R2 behavior changes, mention what could not be fully verified locally

## Known Project-Specific Risks
- Some repository text appears to have encoding issues. Preserve existing file encoding unless the task explicitly includes cleanup.
- `src/index.ts` currently mixes routing, HTML rendering, auth, and feed refresh logic in one file. Refactors should avoid accidental behavior changes and should be done incrementally.
- `public/` assets and inline HTML may diverge. Treat that duplication as a maintenance risk during UI work.

## Preferred Change Style
- For bug fixes, patch the smallest viable surface first.
- For larger work, separate functional changes from cleanup where practical.
- Add concise comments only when the code would otherwise be hard to understand.
