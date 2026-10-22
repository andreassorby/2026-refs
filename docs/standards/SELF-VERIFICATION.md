# Self-verification

> Part of **Andreas-standardoppsett** (topic: `workflow`). Installed into projects as `docs/standards/SELF-VERIFICATION.md`.
> Rule for agents: `.cursor/rules/self-verification.mdc`.

## Why

AI agents often say "done" after writing code they never ran. A short verification round catches broken builds, failing tests, runtime errors and inaccessible UI before you find them yourself. It costs a few minutes, so we only offer it after **larger** work.

## Larger vs small work

| Larger → offer a round | Small → no question |
|---|---|
| Changes in several files | Typo, text change |
| New feature, page, or component | One CSS/Tailwind tweak |
| Refactor / renames | One-line bug fix with an obvious test |
| Database schema / migration / RLS policy | Docs or comments only |
| Auth, payments, deleting data | |
| Dependency upgrades, build/deploy config | |

## The question (once, short)

> "Want me to do a self-verification round (tests, build, click through the app) before I finish?"

Ask it before the project's push / version-bump / deploy step, so a broken change isn't pushed. A local checkpoint commit (see [`COMMIT-AFTER-WORK.md`](COMMIT-AFTER-WORK.md)) may already exist; fixes from the round become their own commit. Cloud/background agents with no user to answer skip the question and just run the round.

## The round

1. **Static checks:** tests, lint, typecheck, build – using the scripts in `package.json` / README.
2. **Database:** migrations only against local/dev, never production without explicit permission.
3. **Run the app** (web): start the dev server, open it in a browser, walk through the 2–5 most important flows touched by the change plus one error path; watch the console for errors.
4. **Accessibility spot-check** on changed screens: keyboard-only pass, visible focus, labels, quick axe run. Full checklist: [`ACCESSIBILITY.md`](ACCESSIBILITY.md) (topic `accessibility`).
5. **Mobile width:** ~375 px for changed screens.
6. Fix and re-run; after two failed attempts at the same problem, stop and report.

## Report format

```
Self-verification
✅ Passed: npm run test – 128 tests passed
✅ Passed: npm run build – OK
✅ Passed: Setlist flow in browser – created, reordered (buttons + keyboard), deleted
❌ Failed: npm run lint – 3 jsx-a11y errors in SongCard.tsx; fixed 2, 1 needs a design decision
⚠️ Not checked: VoiceOver on iPhone – needs a human with a device
```

## Honesty rule

Only what was actually run in this session and observed counts as verified. Everything else goes under "Not checked", with the reason.
