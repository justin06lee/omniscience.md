# Git facts & conventions cheatsheet

Condensed, factual background for the rules in SKILL.md. Sourced from the
Conventional Commits v1.0.0 spec (conventionalcommits.org), Semantic
Versioning (semver.org), the Pro Git book, and Atlassian/trunk-based
development guides.

## Conventional Commits v1.0.0

Structure:

```
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

- `feat:` → correlates with a SemVer **MINOR** bump.
- `fix:` → correlates with a SemVer **PATCH** bump.
- `BREAKING CHANGE:` footer, or `!` appended after type/scope
  (e.g. `feat(api)!: remove v1 endpoints`) → SemVer **MAJOR** bump,
  regardless of type.
- Other common types (from the Angular convention, allowed by the spec):
  `build`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf`, `test`,
  `revert`.
- Scope is a noun in parentheses describing the affected area:
  `feat(parser): ...`.
- The spec makes machine-parseable history possible: automated changelogs,
  automated SemVer bumps (semantic-release, release-please), and
  commit linting (commitlint + husky).

## Commit message style (Tim Pope / Pro Git conventions)

- Subject line: imperative mood ("Fix bug", not "Fixed bug" — it completes
  the sentence "If applied, this commit will ___").
- Subject ≈ 50 chars target, 72 hard-ish limit; no trailing period.
- Blank line between subject and body (git uses this to split them).
- Body wrapped at 72 columns; explains **why** and context, not the diff.

## Atomic commits

One logical change per commit. Benefits: `git bisect` works (every commit
builds), `git revert` cleanly undoes one change, review is tractable.
Tools: `git add -p` (stage hunks selectively), `git commit --fixup` +
`git rebase --autosquash` for local cleanup before sharing.

## Branching models

- **Trunk-based development** (current industry best practice for CD):
  short-lived branches (hours–days) off `master`, merged back quickly;
  master is always releasable. Favored because small, frequent integrations
  minimize merge conflicts and keep CI meaningful.
- **GitFlow** (Driessen, 2010): long-lived `develop` + `master`, release and
  hotfix branches. Now generally considered legacy except when maintaining
  multiple released versions in parallel.
- **GitHub Flow**: branch off master → PR → merge. The middle ground this
  skill effectively implements locally.
- `git merge --no-ff` forces a merge commit even when fast-forward is
  possible, preserving the branch topology in history.
- Branch naming: lowercase, hyphens, `type/slug` prefixes group branches
  and make `git branch --list 'feat/*'` useful.

## Tags

- **Annotated tags** (`git tag -a`) are full objects: tagger name, email,
  date, message, optional GPG signature (`-s`). Use for anything meaningful.
- **Lightweight tags** (`git tag <name>`) are bare refs to a commit — no
  metadata. Only for private, temporary bookmarks.
- Tags are NOT pushed by default. `git push --follow-tags` pushes annotated
  tags reachable from the pushed commits (safer than `git push --tags`,
  which pushes everything including lightweight tags).
- Never re-point or delete a published tag; consumers may have fetched it
  and git will not update a changed tag on fetch by default.
- SemVer release tags: `vMAJOR.MINOR.PATCH`, optionally with pre-release
  suffixes (`v2.0.0-rc.1`).

## History safety

- Rewriting (rebase, amend, reset) is fine for commits that exist ONLY
  locally. Once pushed/shared, history is immutable — undo with
  `git revert` (new inverse commit) instead of `git reset`.
- `git push --force` overwrites the remote ref unconditionally, discarding
  any commits others pushed since. `--force-with-lease` refuses if the
  remote moved since your last fetch — safer, but it still rewrites shared
  history and belongs only on branches provably nobody else uses.
- `git reflog` can recover almost anything local for ~90 days; still,
  prefer committed checkpoints over relying on it.
- `git stash` entries are invisible to `git log` and easy to lose —
  checkpoint commits are more visible and durable.

## Worktrees (parallel checkouts)

- `git worktree add <path> -b <branch>` creates an extra working directory
  linked to the same `.git` — each worktree has its own checked-out branch,
  index, and untracked files; commits/branches/tags are shared instantly.
- Git refuses to check out the same branch in two worktrees.
- Remove with `git worktree remove <path>`; recover from a hand-deleted
  worktree with `git worktree prune`. `git worktree list` shows all.
- This is the correct mechanism for concurrent agents/tasks in one repo —
  cheaper than clones (shared object database) and safer than sharing a
  single checkout.

## Merges & conflicts

- A conflict only marks *textual* overlap; a clean merge can still be
  semantically broken. Building/testing after every merge is the only check.
- `git log --merge -p <path>` shows the commits from both sides that touched
  a conflicted file — the fastest way to understand both intents.
- `git diff --check` finds leftover conflict markers and whitespace errors.
- `git merge --abort` restores the exact pre-merge state; it is always safe
  while a merge is unconcluded.
- `git rerere` ("reuse recorded resolution", enable with
  `git config rerere.enabled true`) records how you resolved a conflict and
  auto-replays it when the identical conflict reappears — valuable when
  repeatedly integrating parallel branches.
- Blanket `-X ours` / `-X theirs` strategy options silently discard one
  side's changes wherever conflicts occur — avoid for real integrations.
- `git branch -d` refuses to delete a branch not merged into HEAD; `-D`
  overrides that safety and can orphan work (recoverable only via reflog).
- A branch checked out in any worktree cannot be deleted until that
  worktree is removed; `git worktree remove` refuses when the worktree is
  dirty (uncommitted changes) unless forced.

## Hygiene

- `.gitignore` before first commit; never commit secrets/`.env`,
  dependencies (node_modules, venvs), build outputs, OS/editor junk.
  If a secret lands in history, rotating the secret is mandatory —
  deleting the commit is not enough once pushed.
- `git status --short --branch` and `git log --oneline --graph --decorate`
  are the fastest state overviews.
- Initial branch: `git init -b master` (git ≥ 2.28 supports `-b`; never
  rely on the machine's `init.defaultBranch` — pass it explicitly).
