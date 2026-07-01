<div align="center">

<img src="assets/omniscience.png" alt="omniscience.md" width="330" height="330" />

# omniscience.md

**A Claude Code skill for fully autonomous git management.**<br>
*Every conceivable change is tracked and visible — nothing is lost, nothing is invisible.*

[![Release](https://img.shields.io/github/v/tag/justin06lee/omniscience.md?label=release&sort=semver)](https://github.com/justin06lee/omniscience.md/tags)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

</div>

---

[omniscience.md](https://github.com/justin06lee/omniscience.md) turns Claude into a meticulous git operator. It commits after every feature, branches for anything significant, tags everything with annotated tags, resolves merge conflicts on its own, runs parallel agents in isolated worktrees — and never, ever pushes unless you say so.

## Install

With [bmo](https://github.com/justin06lee/bmo):

```bash
bmo add justin06lee/omniscience.md
```

Or from a local clone:

```bash
bmo add .
```

Installs as the `omniscience` skill (`/omniscience` in Claude Code). It also triggers automatically whenever Claude finishes a feature, fix, or significant change in (or near) a git repo.

## What it does

| Behavior | Policy |
|---|---|
| **Commit** | Automatically, after every completed feature or logical unit — atomic, [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) format |
| **Push** | **Never**, unless you explicitly say so — then `git push --follow-tags` |
| **Branch** | Automatically for any macro/significant change (`feat/<slug>`, `fix/<slug>`, …), merged back `--no-ff`, short-lived trunk-based style |
| **Tag** | Every completed feature gets an annotated tag; releases get SemVer `vX.Y.Z` tags derived from accumulated commit types |
| **Conflicts** | Resolved autonomously by understanding both sides' intent — never blanket `--ours`/`--theirs`, tests run before concluding, `rerere` enabled |
| **Parallel agents** | One git worktree per subagent, each on its own branch; sequential integration, automatic worktree cleanup after merge |
| **Asking you** | Only two cases: pushing, or something genuinely dangerous (contradictory conflict intent, committed secret, diverged remote) |

## Core rules

1. **Commit after every completed feature.** No batching unrelated work, no leaving finished work uncommitted.
2. **Never push without explicit instruction.** "Commit" ≠ push. Finishing ≠ push.
3. **Branch automatically** for significant work — no confirmation, report afterwards.
4. **Tag every completed feature** with an annotated tag.
5. **Never rewrite pushed history.** Fix forward with `git revert`; local-only history may be cleaned freely.
6. **Operate autonomously** on all local plumbing — init, branch, commit, merge, resolve, tag, worktrees.
7. **Never destroy uncommitted or unmerged work.** Safe variants everywhere: `branch -d` not `-D`, no `reset --hard`, no forced worktree removal. Git's refusals are signals, not obstacles.

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
├── references/
│   └── git-facts.md          # factual cheatsheet backing every rule
└── assets/
    └── omniscience.png       # the eye
```

`references/git-facts.md` grounds the rules in sourced git facts: Conventional Commits → SemVer mapping, annotated vs. lightweight tag mechanics, `--follow-tags` vs `--tags`, merge/rerere behavior, worktree interlocks, reflog recovery, and branching-model background.

## License

[MIT](LICENSE)
