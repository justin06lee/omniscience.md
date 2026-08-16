<div align="center">

<img src="assets/omniscience.svg" alt="omniscience.md" width="330" />

# omniscience.md

**A Claude Code skill for fully autonomous git management.**<br>
*Every conceivable change is tracked and visible — nothing is lost, nothing is invisible.*

</div>

---

[omniscience.md](https://github.com/justin06lee/omniscience.md) turns Claude into a meticulous git operator. It commits after every feature, branches for anything significant, tags everything with annotated tags, resolves merge conflicts on its own, runs parallel agents in isolated worktrees — and pushes finished work on its own, carefully: never a force-push, never a broken state, and "don't push" always wins.

## Install

With [bmo](https://github.com/justin06lee/bmo):

```bash
bmo add justin06lee/omniscience.md
```

Or from a local clone:

```bash
bmo add ./
```

Installs as the `omniscience` skill (`/omniscience` in Claude Code). It also triggers automatically whenever Claude is about to build or modify anything in (or near) a git repo — read before the work starts, governing the whole git lifecycle around it.

## What it does

| Behavior | Policy |
|---|---|
| **Commit** | Automatically, after every completed feature or logical unit — atomic, [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) format |
| **Push** | Allowed by default at completion points — `git push --follow-tags` of a clean, verified state; never forced, never broken, and an explicit "don't push" is always honored |
| **Branch** | Automatically for any macro/significant change (`feat/<slug>`, `fix/<slug>`, …), merged back `--no-ff`, short-lived trunk-based style |
| **Tag** | Every completed feature gets an annotated tag; releases get SemVer `vX.Y.Z` tags derived from accumulated commit types |
| **Conflicts** | Resolved autonomously by understanding both sides' intent — never blanket `--ours`/`--theirs`, tests run before concluding, `rerere` enabled |
| **Parallel agents** | One git worktree per subagent, each on its own branch; sequential integration, automatic worktree cleanup after merge |
| **Docs** | Kept in sync with every code change — README, docs, comments, examples, shipped skills; stale prose is hunted down by grepping for the old names |
| **Asking you** | Only for genuine danger: contradictory conflict intent, a committed secret, a diverged remote it can't cleanly integrate, renaming a published `main` |

## Core rules

1. **Commit after every completed feature.** No batching unrelated work, no leaving finished work uncommitted.
2. **Push freely, never destructively.** Finished, verified work is pushed without asking; force-pushes, broken states, and secrets never leave the machine — and "don't push" always wins.
3. **Branch automatically** for significant work — no confirmation, report afterwards.
4. **Tag every completed feature** with an annotated tag.
5. **Never rewrite pushed history.** Fix forward with `git revert`; local-only history may be cleaned freely.
6. **Operate autonomously** on all local plumbing — init, branch, commit, merge, resolve, tag, worktrees.
7. **Never destroy uncommitted or unmerged work.** Safe variants everywhere: `branch -d` not `-D`, no `reset --hard`, no forced worktree removal. Git's refusals are signals, not obstacles.
8. **Docs are part of the change.** A feature isn't complete while anything in the repo still describes the old behavior — doc updates land in the same commit as the code they describe.

## Baked-in practices

- **Conventional Commits v1.0.0** — `feat` → SemVer MINOR, `fix` → PATCH, `BREAKING CHANGE`/`!` → MAJOR; imperative ≤50-char subjects, 72-char-wrapped "why" bodies, staged-diff review before every message.
- **Trunk-based branching** — short-lived typed branches (`feat/`, `fix/`, `refactor/`, `chore/`, `docs/`, `experiment/`), `--no-ff` merges to keep topology visible, immediate cleanup.
- **Annotated tags only** for anything meaningful (tagger, date, message — lightweight tags are throwaway bookmarks).
- **Worktree-per-agent parallelism** — sibling directories outside the repo root, shared object database, per-worktree index; `git worktree remove` + `prune` after every merge.
- **Session hygiene** — a fresh repo, `.gitignore`, and initial commit whenever none exists; secrets never committed; pre-existing dirty state checkpointed so the baseline is clean.
- **Visibility** — every operation reported; `git log --oneline --graph --decorate`, `git status --short --branch`, tag and worktree listings on demand; an end-of-task checklist verifies nothing was left untracked, untagged, or force-deleted.

## Layout

```
omniscience.md/
├── SKILL.md                  # the skill — policies and protocols Claude follows
├── .bmoignore                # keeps repo scaffolding (assets, README) out of installs
├── references/
│   └── git-facts.md          # factual cheatsheet backing every rule
└── assets/
    └── omniscience.svg       # the eye (vector — crisp at any size)
```

`references/git-facts.md` grounds the rules in sourced git facts: Conventional Commits → SemVer mapping, annotated vs. lightweight tag mechanics, `--follow-tags` vs `--tags`, merge/rerere behavior, worktree interlocks, reflog recovery, and branching-model background.

## License

[MIT](LICENSE)
