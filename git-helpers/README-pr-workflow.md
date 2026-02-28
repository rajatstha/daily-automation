# PR workflow scripts: pr-sync and pr-squash

Two scripts to automate syncing your branch with the base branch and squashing commits before or after creating a pull request.

## What they do

| Script      | Purpose |
|------------|---------|
| **pr-sync**  | Fetches the base branch from origin, merges it into your current branch, and pushes. Use before creating a PR or to bring in latest changes. |
| **pr-squash** | Squashes all commits on your branch (since it diverged from the base) into a single commit, then force-pushes. Use to clean up before opening a PR or after addressing feedback. |

## Requirements

- Run from inside a Git repository.
- For **pr-sync**: current branch must be pushed and able to merge with the base.
- For **pr-squash**: you must be on a feature branch (not the base branch and not in detached HEAD).

## Setup

1. Ensure the scripts are executable (they are by default after creation).
2. Add the directory containing the scripts to your `PATH`, e.g. in `~/.zshrc`:

   ```bash
   export PATH="$HOME/bin:$PATH"
   ```

   Then run `source ~/.zshrc` or open a new terminal.

## pr-sync

Syncs your current branch with the remote base branch.

**Commands run:**  
`git fetch origin <base>` → `git merge FETCH_HEAD` → `git push`

### Usage

```bash
pr-sync [base]
pr-sync -b <base>
PR_BASE_BRANCH=main pr-sync
```

### Options

- **base** — Base branch to sync with. Can be given as first argument or with `-b` / `--base`. If omitted, uses `PR_BASE_BRANCH` (if set) or `master`.

### Examples

```bash
pr-sync                  # sync with master
pr-sync main             # sync with main
pr-sync develop          # sync with develop
pr-sync -b release/2.0   # sync with release/2.0
export PR_BASE_BRANCH=main && pr-sync   # default to main for this shell
```

### Notes

- If the merge has conflicts, Git will stop at the merge step. Resolve conflicts, then run `git add`, `git commit`, and `git push` yourself. You can run `pr-squash` afterward if you want one commit again.

---

## pr-squash

Squashes all commits on your current branch (since the merge-base with the base branch) into one commit and force-pushes.

**Commands run:**  
`git reset --soft $(git merge-base origin/<base> HEAD)` → `git commit` (with `-m` or editor) → `git push --force-with-lease`

### Usage

```bash
pr-squash [base] [-m "commit message"]
pr-squash -b <base> [-m "commit message"]
PR_BASE_BRANCH=main pr-squash -m "message"
```

### Options

- **base** — Base branch to compute the merge-base from. Same as **pr-sync**: first argument, `-b` / `--base`, or `PR_BASE_BRANCH` / `master`.
- **-m "message"** — Use this as the single commit message. If omitted, Git opens your editor for the message.

### Examples

```bash
pr-squash -m "Add login flow"                    # squash against master, use given message
pr-squash main -m "Add login flow"               # squash against main
pr-squash develop -m "Address review feedback"   # squash against develop
pr-squash                                       # squash against default base; editor opens for message
PR_BASE_BRANCH=main pr-squash -m "Update docs"  # default base = main
```

### Safety checks

- **Detached HEAD** — Exits with an error; will not squash.
- **On base branch** — If current branch is the same as the base (e.g. you’re on `master`), exits with an error to avoid rewriting the base branch.

### Notes

- Uses `git push --force-with-lease` so the push fails if the remote branch was updated by someone else.
- After addressing PR feedback, you can make normal commits, push, then run **pr-squash** again to collapse everything into one commit and update the PR.

---

## Typical workflows

### Before creating a PR

1. Sync with the base:  
   `pr-sync` or `pr-sync main` (or your base branch).
2. Squash to one commit:  
   `pr-squash -m "Short description of your change"` (or `pr-squash main -m "..."`).
3. Create the PR in your Git host’s UI.

### After PR feedback

1. Make changes, then:  
   `git add ...`, `git commit -m "Address feedback"`, `git push`.
2. Squash again so the PR shows one commit:  
   `pr-squash -m "Short description of your change"` (same base and message as before, or updated).

---

## Base branch summary

| Base branch   | Sync              | Squash                    |
|---------------|-------------------|---------------------------|
| `master`      | `pr-sync`         | `pr-squash -m "msg"`      |
| `main`        | `pr-sync main`    | `pr-squash main -m "msg"` |
| `develop`     | `pr-sync develop` | `pr-squash develop -m "msg"` |
| Any branch    | `pr-sync <name>`  | `pr-squash <name> -m "msg"` |

To default to a branch other than `master` for the current shell or session:

```bash
export PR_BASE_BRANCH=main
pr-sync
pr-squash -m "Your message"
```

You can also add `export PR_BASE_BRANCH=main` to `~/.zshrc` if you usually use `main` as the base.
