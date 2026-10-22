# Commit after work

> Part of **Andreas-standardoppsett** (topic: `workflow`). Installed into projects as `docs/standards/COMMIT-AFTER-WORK.md`.
> Rule for agents: `.cursor/rules/commit-after-work.mdc`.

## Why

A commit is a save point. If an AI change breaks something three steps later, you can go back to the last good step instead of untangling everything by hand. This complements the habit **"commit before and after each AI session"** (from Matt Murphy): that habit protects the session as a whole; this rule makes the agent add save points *inside* a session, after each finished piece of work.

## Project rules come first

Many projects already have a commit/release rule (chordinger: `agentic-workflow.mdc` + `iteration-closeout.mdc`; storsalen-sangoversikt: `deploy-to-github.mdc`; nrk-org-chart: `release-workflow.mdc`; tv-calendar: publish steps in `AGENTS.md`). Those decide **message format, versioning, changelog and pushing**. This standard only adds: *be up to date before committing* and *always commit after a job*.

## What the agent does

| Situation | Agent action |
|---|---|
| Before committing / before a large change | `git fetch`; if behind upstream, integrate first (`git pull --rebase` for unpushed commits, or merge if the project prefers; stash/restore uncommitted work); re-run quick checks |
| Conflicts it can't resolve confidently | Abort the rebase/merge (back to previous state), explain, ask you |
| No remote / no network | Says so, commits locally |
| Finished a job / substantial piece of work | `git status` + `git diff`, stage the relevant files, commit with a clear message |
| About to start a large change, tree is dirty | Commit the existing work first as a checkpoint (ask if the changes aren't the agent's own) |
| Small related fixes afterwards (e.g. from self-verification) | Separate small commit |
| Commit fails (hook, lint, conflict) | Report the error; fix the cause or leave uncommitted. Never `--no-verify` |
| Secrets (`.env`, keys) in the diff | Stop, don't commit, tell the user |
| Push | Only if the project's rules say so (e.g. chordinger pushes per iteration and has an opt-in auto-push hook); otherwise ask |

Never: force-push, `reset --hard`, rewriting pushed history, discarding local or remote work to fix a divergence, skipping hooks — unless you explicitly ask for that exact action.

## Good commit messages

- ✅ `Add Move up/down buttons to setlist reorder (keyboard alternative to drag)`
- ✅ `Fix login error not announced to screen readers`
- ❌ `update`, `fixes`, `changes from AI`, `wip` (without saying what)

## How you roll back (for you, the human)

```bash
git log --oneline -15              # find the step you want
git switch -c try-old <sha>        # look at that step safely on a new branch
git revert <sha>                   # undo one commit by adding a new "undo" commit (safe, keeps history)
git restore .                      # throw away uncommitted changes (careful: can't be undone)
```

In Cursor you can also use the Source Control panel (timeline/history) to see each commit. Ask the agent "undo the last commit with git revert" rather than "reset", so history stays intact.
