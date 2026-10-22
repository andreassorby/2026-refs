# Agent instructions

<!-- Base template from Andreas-standardoppsett. Fill in the project-specific parts.
     Standards blocks are appended below by install.sh. -->

## Project

- **What it is:** <one sentence>
- **Stack:** <e.g. Next.js 16, React 19, Supabase, Tailwind>
- **Spec / source of truth:** <e.g. PROJECT_SPEC.md>

## Commands

- Install: `npm install`
- Dev: `npm run dev`
- Test: `npm run test`
- Lint: `npm run lint`
- Build: `npm run build`

## Workflow

1. Small, shippable changes. Add/update tests in the same change.
2. Run test + lint before committing.
3. Commit with a clear message; push to `main` (or the current branch).
4. Never commit secrets (`.env`, keys).

## Standards

Shared standards live in `docs/standards/` and `.cursor/rules/`, installed from Andreas-standardoppsett. See the sections below.

<!-- standardoppsett:accessibility:start -->
<!-- Added by Andreas-standardoppsett (topic: accessibility). Edit the kit, not this block; delete the whole block to reinstall. -->

## Accessibility (WCAG 2.2 AA)

All UI in this project must meet **WCAG 2.2 level AA**. Rule: `.cursor/rules/accessibility.mdc`. Full checklist, legal context (Norway/EU) and testing recipe: `docs/standards/ACCESSIBILITY.md`. Alt text: `docs/standards/ALT-TEXT.md`.

- Semantic HTML first; ARIA only when HTML can't do it.
- Keyboard-operable everything, visible focus, no hover-only or drag-only interactions, targets ≥ 24×24 px.
- Labels on every form control, correct `alt` on images, contrast 4.5:1 text / 3:1 large text and UI parts.
- Errors and status messages announced; modals trap focus, close on Esc and restore focus.
- `lang` matches UI language, headings in order, `prefers-reduced-motion` respected.
- **Before saying a UI task is done:** run the Definition of done in `docs/standards/ACCESSIBILITY.md` §6 (keyboard pass, jsx-a11y lint, axe check) and report which checks were run and which need a human (screen reader).

<!-- standardoppsett:accessibility:end -->

<!-- standardoppsett:production-readiness:start -->
<!-- Added by Andreas-standardoppsett (topic: production-readiness). Edit the kit, not this block; delete the whole block to reinstall. -->

## Production readiness

Rule (agent-requested): `.cursor/rules/production-readiness.mdc`. Checklist: `docs/standards/PRODUCTION-READINESS.md`.

- Before a launch / release / deploy to real users, or when asked for a readiness review: walk through all 8 areas (security, privacy, backup, monitoring, error handling, performance, accessibility, legal).
- Mark each item ✅ pass / ❌ fail / ❓ unknown with evidence; unknown is never pass. End with blockers and the top fixes.
- Review only: don't change production settings, data or billing. Legal points are not legal advice.

<!-- standardoppsett:production-readiness:end -->

<!-- standardoppsett:workflow:start -->
<!-- Added by Andreas-standardoppsett (topic: workflow). Edit the kit, not this block; delete the whole block to reinstall. -->

## Self-verification after larger work

Rule: `.cursor/rules/self-verification.mdc`. Details: `docs/standards/SELF-VERIFICATION.md`.

- After a larger piece of work (multi-file change, new feature, refactor, schema change), ask the user once and briefly whether they want a self-verification round, before any push/version-bump/deploy step. Don't ask for small changes.
- **No user present (cloud/background agent):** don't ask; run the round and include the report.
- The round: tests, lint, typecheck, build; start the app and click through the main flows in a browser (web apps); keyboard/accessibility spot-check (`docs/standards/ACCESSIBILITY.md`); mobile width.
- Report ✅ passed / ❌ failed / ⚠️ not checked. **Never claim something was verified unless you actually ran it.**

## Commit after work (rollback points)

Rule: `.cursor/rules/commit-after-work.mdc`. Details: `docs/standards/COMMIT-AFTER-WORK.md`.

- If this project has its own commit/release/publish rule, that rule decides message format, versioning and pushing; this section only adds the two baselines below.
- **Be up to date first:** `git fetch`; if the branch is behind its upstream, integrate first (`git pull --rebase` for unpushed local commits, or merge if the project prefers; stash and restore uncommitted work), then re-run quick checks. Stop and ask on conflicts you can't resolve confidently. No remote/network: say so and commit locally.
- Commit after each completed job or substantial piece of work, and before starting a large new change if the tree is dirty. Small logical commits are fine.
- Clear, descriptive messages (what and why). Never commit secrets or `.env` files.
- Never force-push, rewrite pushed history, `reset --hard`, discard local/remote work to fix a divergence, or skip hooks (`--no-verify`). If a commit fails, report the error.
- Pushing follows this project's own rules; the rule only requires the commit. Cloud agents already commit to their branch.

<!-- standardoppsett:workflow:end -->
