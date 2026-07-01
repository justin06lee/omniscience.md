---
name: omniscience
description: Use for ALL git management during coding sessions — committing, branching, tagging, and repo hygiene. Triggers whenever a feature, fix, or significant change is completed in a git repo (or a repo should exist but doesn't), and on phrases like "commit this", "manage git", "track my changes", "version this", "push", "tag a release". Enforces commit-after-every-feature, auto-branching for significant work, annotated tags per feature, and never pushing without explicit user instruction.
---

# omniscience.md

You are managing git proactively on the user's behalf. The prime directive: **every conceivable change is tracked and visible in git history — nothing is lost, nothing is invisible.** You act autonomously for local operations and NEVER touch the remote without explicit instruction.

## Core rules (non-negotiable)

1. **Commit after every completed feature or logical unit of work.** Do not batch unrelated work into one commit. Do not leave completed work uncommitted at the end of a turn.
2. **Never push unless the user explicitly says to push** (e.g. "push", "push it", "publish this"). "Commit" does not mean push. Finishing a feature does not mean push. When the user does say push, push the branch AND its tags (`git push --follow-tags`).
3. **Branch automatically for any macro/significant feature** — no confirmation, just do it and tell the user afterwards.
4. **Tag every completed feature** with an annotated tag.
5. **Never rewrite history that has been pushed.** Local, unpushed history may be amended/rebased freely to keep it clean; anything on the remote is immutable — fix forward with new commits (`git revert`, never `git reset` on shared branches).
6. **Operate autonomously.** All local git plumbing — init, branch, commit, merge, conflict resolution, tag, worktree create/remove — happens without asking permission. Ask the user ONLY when: (a) they need to explicitly request a push, or (b) something genuinely dangerous/ambiguous comes up that risks losing their work (e.g. a conflict you cannot safely resolve, a secret already committed, diverged remote history). Everything else: do it, then report it.
7. **Never destroy uncommitted or unmerged work.** Before any operation that could discard changes (`checkout`/`switch` over dirty files, `reset --hard`, `clean`, worktree/branch removal), verify via `git status` that nothing uncommitted would be lost — commit a checkpoint first if in doubt. Use `git branch -d` (not `-D`) so git itself refuses to delete unmerged branches; if `-d` refuses, that's a signal to investigate, not to force.

## Session start: repo state check

Before any work in a session, run `git status` (and `git log --oneline -5`). If the directory is not a git repo, initialize one immediately:

```bash
git init -b main
```

Then create a sensible `.gitignore` for the project's stack (node_modules, build outputs, .env files, OS junk like .DS_Store, editor dirs) and make an initial commit (`chore: initial commit`). Secrets, credentials, and `.env` files must never be committed — add them to `.gitignore` before the first commit. Large binaries and generated artifacts stay out of history too.

If there are uncommitted changes from before you arrived, commit them first as `chore: checkpoint pre-existing changes` (or sort them into logical commits if they're readable) so the baseline is clean and visible.

## Branching

**Decide branch vs. mainline by scope, without asking:**

- **Branch** (create automatically): a new feature, a refactor touching multiple files, anything experimental or risky, any change you'd describe with more than one commit, or anything a reviewer would call "a piece of work" rather than "a tweak."
- **Stay on the current branch**: typo fixes, single-file tweaks, config bumps, follow-up fixes to the feature you're already on.

Naming convention — lowercase, hyphen-separated, type-prefixed:

```
feat/<short-slug>        fix/<short-slug>
refactor/<short-slug>    chore/<short-slug>
docs/<short-slug>        experiment/<short-slug>
```

Create with `git switch -c feat/<slug>` from an up-to-date mainline. Keep branches **short-lived** (trunk-based philosophy): merge back into the mainline as soon as the feature is complete and working — long-lived divergence breeds merge conflicts. Merge with `git merge --no-ff feat/<slug>` so the branch's existence stays visible in history, then delete the branch (`git branch -d`) — its story is preserved by the merge commit and the feature tag. Always announce to the user which branch you created/merged.

## Commits

Follow **Conventional Commits** (conventionalcommits.org):

```
<type>(<optional scope>): <description>

[optional body — the WHY, wrapped at 72 chars]

[optional footer, e.g. BREAKING CHANGE: ...]
```

Types: `feat`, `fix`, `refactor`, `perf`, `docs`, `style`, `test`, `build`, `ci`, `chore`, `revert`. `feat` maps to a SemVer MINOR, `fix` to PATCH, and a `BREAKING CHANGE:` footer (or `!` after the type) to MAJOR.

Rules for every commit:

- **Atomic**: one logical change per commit. It should build and pass tests on its own. Stage selectively (`git add <paths>` or `git add -p`) — never blind `git add -A` when unrelated changes are present.
- **Subject**: imperative mood ("add", not "added"/"adds"), ≤ 50 chars ideally (hard cap ~72), no trailing period.
- **Body**: explain *why* the change was made and any trade-offs, not a restatement of the diff. Omit only for trivial changes.
- Reference issues in the footer when known (`Refs: #123`).
- Before committing, review the actual staged diff (`git diff --staged`) — describe what's really there, not what you intended.
- Run the project's tests/build before committing when they exist; a red commit defeats atomicity. If you must checkpoint broken work, say so in the message (`wip:` prefix) and fix it in the next commit before merging.

For work-in-progress across long tasks, prefer real checkpoint commits over `git stash` — stashes are invisible history; commits are the point of this skill.

## Tagging

Tag **every completed feature** at its merge/completion point with an **annotated** tag (annotated tags store tagger, date, and message — lightweight tags store nothing and are for throwaway bookmarks only):

```bash
git tag -a feat/<slug> -m "Feature: <one-line summary of what shipped>"
```

For releases (when the user asks, or a milestone is clearly reached), use SemVer tags `vMAJOR.MINOR.PATCH` (e.g. `v1.2.0`), bumping by the Conventional Commit types accumulated since the last release tag:

```bash
git tag -a v1.2.0 -m "Release v1.2.0

- <feat/fix highlights since last tag>"
```

Never move, reuse, or delete a tag that has been pushed — cut a new one instead. Tags are only pushed as part of an explicit user-requested push (`--follow-tags` pushes annotated tags reachable from the pushed refs).

## Parallel work: worktrees

A checkout can hold only one branch at a time — so when multiple agents (or parallel tasks) need to work simultaneously, use **git worktrees**, never separate clones and never two agents in one directory.

- One worktree + one branch per parallel workstream, created as sibling directories outside the repo root so they don't pollute each other's `git status`:

  ```bash
  git worktree add ../<repo>-wt/feat-auth -b feat/auth
  git worktree add ../<repo>-wt/feat-search -b feat/search
  ```

- When spawning subagents via the Agent tool, prefer its built-in `isolation: "worktree"` option — it creates and cleans up the worktree automatically.
- Each agent works ONLY inside its own worktree, committing normally on its own branch. Git enforces that no two worktrees check out the same branch — never work around that.
- Worktrees share one `.git`: branches, tags, and commits are visible across all of them instantly; only the index and untracked files are per-worktree.
- Integrate sequentially: as each workstream completes, merge its branch into the mainline (`--no-ff`) from the main worktree, tag the feature, then run the cleanup routine below.
- `git worktree list` is part of the "where are we" visibility report whenever any extra worktrees exist.

### Post-merge cleanup (automatic, every time)

Immediately after a feature branch merges into the mainline, without being asked:

```bash
git worktree remove <path>    # only if the branch had a worktree; never rm -rf
git branch -d feat/<slug>     # -d, never -D: git verifies the branch is fully merged
git worktree prune            # clears metadata for any hand-deleted worktrees
```

Order matters: remove the worktree before deleting its branch (a branch checked out in a worktree can't be deleted). `git worktree remove` refuses if the worktree has uncommitted changes — that refusal means unfinished work exists; commit or merge it first, never `--force` through it. A merged branch's history lives on in the merge commit and the feature tag, so nothing is lost by this cleanup.

## Merge conflict resolution

Resolve conflicts autonomously, with this protocol:

1. Merge with the mainline up to date and the working tree clean, so the merge is the only thing in flight.
2. On conflict, run `git status` and `git diff` to list every conflicted path, then resolve each by **understanding both sides' intent** — read the surrounding code and both branches' commits (`git log --merge <path>` shows exactly the commits that touched the conflicted file on each side). Never resolve by blanket `--ours`/`--theirs`, and never delete one side just to make the markers go away; if both sides made real changes, the resolution usually combines them.
3. Check for **semantic** conflicts, not just textual ones: a merge can apply cleanly and still be broken (e.g. one branch renamed a function the other branch calls). After resolving, run the project's build/tests before concluding the merge commit.
4. Ensure no conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) remain: `git diff --check`.
5. Conclude with `git commit` (keep the default merge message, appending a short note of how each conflict was resolved), then report to the user which files conflicted and how you resolved them.
6. **Escalate to the user only if** the two sides embody genuinely contradictory intent (e.g. one branch deletes a module the other builds upon) and any resolution would silently discard someone's work — describe both sides and ask which wins. If a merge goes sideways, `git merge --abort` restores the pre-merge state exactly; aborting and retrying is always safe and always preferable to committing a resolution you're unsure of.
7. Enable `git config rerere.enabled true` in the repo so git records and replays your conflict resolutions if the same conflicts recur (common when integrating several parallel branches).

## Visibility & reporting

After every git operation, tell the user what happened in one or two lines: branch created/merged, commit hash + subject, tag created. When the user asks "where are we," show:

```bash
git log --oneline --graph --decorate -15
git status --short --branch
git tag --sort=-creatordate | head
```

End-of-task checklist (run mentally every time you finish work):

1. All changes committed? (`git status` clean)
2. Feature branch merged back with `--no-ff` and deleted?
3. Merged worktrees removed and pruned? (`git worktree list` shows only the main one, unless work is still in flight)
4. Feature tagged (annotated)?
5. Merge conflicts (if any) resolved with tests passing and no stray conflict markers?
6. User told what was committed/branched/merged/tagged — and what conflicted?
7. Nothing pushed unless the user explicitly asked?
8. Nothing force-deleted, force-removed, or hard-reset at any point?

## Reference

`references/git-facts.md` in this skill contains a condensed cheatsheet of the underlying git facts and conventions (Conventional Commits types, SemVer mapping, tag mechanics, branching-model background). Consult it when unsure about a rule's rationale.
