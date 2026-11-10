# Agent Change Log

This is the durable, newest-first record of repository changes for agents maintaining this fork. Update it in the same commit as every repository change. Do not record secrets or private data.

## 2026-11-09 — Sync upstream DIYgod/RSSHub & fix Vercel pnpm-lockfile dependency error

- **Summary:** Synchronized 6 commits from `DIYgod/RSSHub:master` (up to `301009ce2`), merged cleanly into `master`, and resolved the Vercel deployment failure (`BUILD_UTILS_SPAWN_1` / `ERR_PNPM_LOCKFILE_MISSING_DEPENDENCY`) by regenerating `pnpm-lock.yaml` to align `htmlparser2: 12.0.0` with `pnpm-workspace.yaml`.
- **Affected files:** `pnpm-lock.yaml`, `internal_docs/changelog.md`, and upstream route files (`lib/routes/bilibili/*`, `lib/routes/lvshunmuseum/*`, `lib/routes/smzdm/*`, `lib/routes/the-decoder/*`, etc.).
- **Rationale:** Previous Vercel build failed on `pnpm install --ignore-scripts --no-frozen-lockfile` because `pnpm-lock.yaml` had snapshot references pointing to `htmlparser2: 10.1.0` without a corresponding package definition. Under pnpm v12, `pnpm.overrides` inside `package.json` is ignored, and upstream's `pnpm-workspace.yaml` pins `htmlparser2: ^12.0.0`. Node 24 supports synchronous ESM `require()` properly, eliminating the historical need for `htmlparser2: 10.1.0`.
- **Upstream revision range:** `621561d0c...301009ce2` (6 commits).
- **Conflicts resolved & retained fork invariants:**
  - Merge completed cleanly with no conflicts.
  - Retained `vercel.json` (`installCommand: "pnpm install --ignore-scripts --no-frozen-lockfile"` and Patchright include-files).
  - Retained `.npmrc` (`ignore-scripts=true`).
  - Retained `.github/workflows/sync-fork.yml` (`if: false`).
  - Retained Node `24.x` engine and prepare `exit 0` in `package.json`.
- **Verification:** Verified `pnpm install --ignore-scripts --no-frozen-lockfile` exits 0 cleanly in 25ms; verified `build:routes` generates full route registry; verified `require('sanitize-html')` loads without error.
- **Commit:** `sync: merge upstream changes from DIYgod/RSSHub & fix lockfile for Vercel`.

## 2026-11-05 — Sync upstream DIYgod/RSSHub & fix Claude blog route

- **Summary:** Synchronized 141 commits from `DIYgod/RSSHub:master` (up to `8f4fd4356`), resolving the broken `/claude/blog` route while preserving all fork-specific Vercel deployment invariants.
- **Affected files:** `lib/routes/claude/blog.ts`, `package.json`, `pnpm-lock.yaml`, and 160+ upstream route/lib files.
- **Rationale:** The `/claude/blog` route was throwing `Article content was not found` on Anthropic research articles (e.g. `/research/formal-verification`) due to Anthropic's October 2026 hashed CSS class changes. Upstream commit `9601f8ba7` resolved this by adding the `article > div > div[class$="_body"]` selector.
- **Upstream revision range:** `3afc108d0...8f4fd4356` (141 commits).
- **Conflicts resolved & retained fork invariants:**
  - Resolved merge conflict in `package.json` to incorporate upstream `magpie` configuration while retaining fork-specific `prepare: "exit 0"`, Node `24.x` engine, and `htmlparser2: "10.1.0"` overrides.
  - Retained `vercel.json` (`installCommand: "pnpm install --ignore-scripts --no-frozen-lockfile"`).
  - Retained `.npmrc` (`ignore-scripts=true`).
  - Retained `.github/workflows/sync-fork.yml` (`if: false`).
  - Retained `pnpm-lock.yaml` pins for `htmlparser2: 10.1.0`.
- **Verification:** Validated `package.json` JSON structure, confirmed `git diff upstream/master HEAD` reflects only intended fork customizations, verified merge commit `d8c4cca0e`, and checked working tree status.
- **Commit:** `docs: record upstream sync in changelog`.

## 2026-11-05 — Establish agent change-log convention

- **Summary:** Added a mandatory internal change-log process for future repository changes.
- **Affected files:** `AGENTS.md`, `internal_docs/changelog.md`.
- **Rationale:** Preserve concise maintenance context across agents, especially for fork-specific Vercel and upstream-sync decisions.
- **Verification:** Confirmed the files are present and `git diff --check` passes.
- **Commit:** `docs: add agent change log`.
