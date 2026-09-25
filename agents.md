# Agent instructions

## Scope and hierarchy

- These instructions apply to the whole `zed-pkg/zed-lib` repository unless a deeper lowercase `agents.md` adds narrower rules.
- Before editing, resolve the current working directory and load every readable ancestor `agents.md` from the filesystem root to the working directory. Do not search siblings. Resolve symlinks, deduplicate resolved files, and report unreadable or cyclic instruction files.
- `.claude/CLAUDE.md`, `.gemini/GEMINI.md`, and `.openai/AGENTS.md` are pointers only. Never duplicate instructions in tool-specific files.

## Repository role

This repository implements the contract defined by `zed-pkg/zed-interfaces`. Interfaces hold shape — types, serialization, validation; this holds behavior — resolution, planning, policy. The dependency runs one way and must stay that way: `zed-lib` depends on `zed-interfaces`, never the reverse.

## Working rules

- Do not copy a module out of `zed-interfaces`. Depend on it until it is *moved* — a duplicated implementation drifts silently, and drift between the CLI and a front end is the failure this repository exists to prevent.
- Moving behavior out of `zed-interfaces` is a breaking change for that crate. Land each move as its own change with its consumers updated in contract-first order, not as a batch.
- Cross-language behavior is specified by `conformance/cases/*.json` before it is implemented. Add the case, watch it fail, then write the code; a case written afterwards tests the code's opinion rather than the contract.
- Every implementation slice runs the same corpus. A case that only one language can satisfy is a bug in the case.
- The slices are native implementations, not bindings, and they take no semver dependency. `pub_semver` and npm's `semver` implement a different dialect from Cargo's — a bare `1.0.0` is exact there and a caret range here — so a wrapper would smuggle in disagreements the corpus was built to prevent. When behavior is ambiguous, the Rust `semver` crate is the reference: probe it, then pin the answer as a case.
- A new corpus case must be verified against Rust *first*. Rust is the slice consumers already run through `zed-cli`, so a case Rust fails is a case that mis-states the contract.
- Errors carry a stable `kind()` string shared with the corpus. Renaming one is a breaking change for every implementation and every consumer that matches on it.
- Resolution returns versions in their **published spelling**. Normalizing is for comparison only — the store address and the VCS tag must stay faithful to what the publisher tagged.
- Keep `Cargo.lock` committed; do not allow CI to update it implicitly.
- Pin GitHub Actions by immutable commit SHA and keep workflow permissions read-only unless a documented write is required.

## Validation

```sh
cargo test --all-targets --locked
cargo clippy --all-targets --locked -- -D warnings
cargo fmt --all -- --check
```

The crate resolves `zed-interfaces` through a sibling checkout at
`../zed-interfaces`, so clone both into the same parent directory.

## Repository-local Git worktrees

- Create or use a Git worktree only when the human operator explicitly authorizes it for the current task. Concurrency or a dirty checkout is not permission by itself.
- Put every authorized worktree at `<repository-root>/tmp/worktrees/<name>`; from the repository root, use `./tmp/worktrees/<name>`. Never place worktrees beside repositories or organization directories.
- Keep `tmp`, `temp`, `tmp/worktrees`, and `temp/worktrees` ignored in the repository-root `.gitignore`. Do not commit files from those directories.
- Relocate or remove a worktree only when the operator explicitly requests it. Before removal, preserve and publish intended changes, verify its commit is represented on the target branch, and confirm there are no tracked, untracked, ignored-sensitive, or in-use files that must survive. Remove it with `git worktree remove <path>` without `--force`; never delete a worktree directory with `rm`.

<!-- BEGIN ores-agents-pointer: managed by ORESoftware/my-ai; edit there, not here -->

## Canonical agent instructions

Before doing anything else in this repository, also read:

    .ores/agents/AGENTS.md

That path is a symlink to `~/codes/oresoftware/my-ai/AGENTS.md`, whose canonical copy is
<https://github.com/ORESoftware/my-ai/blob/main/AGENTS.md>.

It exists at a fixed path *inside* the repository because some agents cannot walk up past
the repository root, so machine-wide instructions one or more directories above are
invisible to them. This pointer plus that path make the same file reachable from a working
directory anywhere in the tree.

The symlink is deliberately **not committed**: it names an absolute path that is only valid
on a machine with `~/codes/oresoftware/my-ai` checked out, so committing it would produce a
broken link for everyone else and for CI. `.ores/` is git-ignored for that reason. If
`.ores/agents/AGENTS.md` is missing on your machine, create it with:

    mkdir -p .ores/agents
    ln -sfn "$HOME/codes/oresoftware/my-ai/AGENTS.md" .ores/agents/AGENTS.md

or run `~/codes/oresoftware/my-ai/scripts/link-repo-agents.sh` once to do it for every git
repository under `~/codes`, and `--check` to verify them.

A missing `.ores/agents/AGENTS.md` is a setup gap on the reader's machine, never a reason to
skip the canonical instructions: fetch them from the URL above instead.

<!-- END ores-agents-pointer -->
