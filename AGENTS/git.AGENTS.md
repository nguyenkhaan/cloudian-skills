# Git Discipline: Atomic Commits, Clean History — AGENTS Protocol

## Git Discipline: Atomic Commits, Clean History

Git is not a save button. A commit is a unit of reviewable, revertable,
understandable change. If the operator needs to undo step 5 of your plan,
they should be able to `git revert <hash>` and lose only step 5 — not
steps 3 through 7 smushed into one giant commit with the message "updates".

### One PLAN.md Step = One Commit

Each completed step from `INTERNAL/PLAN.md` gets its own commit. Not
two steps. Not half a step. One step, one commit, one clear message.

```bash
# GOOD: Each commit maps to exactly one plan step
git log --oneline
a1b2c3d  [Step 5] Add integration tests for auth module
f4e5d6c  [Step 4] Implement token refresh endpoint
9a8b7c6  [Step 3] Add input validation to login handler
3d2e1f0  [Step 2] Create AuthError exception hierarchy
c0b1a2d  [Step 1] Add User model and database migration

# BAD: One commit with everything
git log --oneline
x9y8z7w  fix stuff and add features

# ALSO BAD: Tiny meaningless commits
git log --oneline
aaa1111  wip
bbb2222  fix
ccc3333  more fixes
ddd4444  actually fix it this time
eee5555  ok now it works
```

### Commit Message Format

```
[Step N] <imperative verb> <what changed>

<optional body: why this change was made, referencing PLAN.md>
```

The subject line uses the imperative mood ("Add", "Fix", "Remove", not
"Added", "Fixed", "Removed"). It references the plan step number so the
operator can trace any commit back to the plan.

```bash
# GOOD
git commit -m "[Step 3] Add input validation to parse_config()

Validates that 'timeout' is a positive integer and 'host' is non-empty.
Raises ValueError with descriptive message on invalid input.
Ref: INTERNAL/PLAN.md step 3"

# BAD
git commit -m "updated config.py"
```

### What Gets Committed

- **Source code changes** for the current step
- **Tests** written for the current step
- **Updated PLAN.md** with the step marked as complete

What does **not** get committed:
- `.env` files (ever)
- Build artifacts, `node_modules`, `__pycache__`, `.pyc`
- IDE configuration (`.vscode/`, `.idea/`) unless project-shared
- Temporary debug prints or `console.log` statements

### The Revert Test

Before committing, mentally ask: "If the operator runs `git revert` on
this commit, will the project still work?" If the answer is no — if your
commit has tangled dependencies on other uncommitted work — your commit
is too big or in the wrong order. Split it.

### Commit Often, Push Once

Commits are local and cheap. Pushes are remote and visible. Do not push
after every commit — batch them.

The heuristic: **~5 commits per 1 push**. Complete a logical chunk of the
plan (several related steps), verify the tests pass, then push once. This
gives the operator a clean, reviewable unit of progress on the remote
without polluting the history with half-finished intermediate states.

Exception: if a single commit is large (a major step that touches many
files), push it immediately. A large commit sitting only locally is a
large commit that can be lost.

```bash
# GOOD: Work locally, push a batch
git commit -m "[Step 1] Add User model and migration"
git commit -m "[Step 2] Create AuthError exception hierarchy"
git commit -m "[Step 3] Add input validation to login handler"
git commit -m "[Step 4] Implement token refresh endpoint"
git commit -m "[Step 5] Add integration tests for auth module"
git push origin feat/add-auth-module  # one push, five commits

# BAD: Push after every single commit
git commit -m "[Step 1] ..." && git push
git commit -m "[Step 2] ..." && git push
git commit -m "[Step 3] ..." && git push  # three pushes, three CI runs, noise

# ALSO BAD: 20 commits sitting locally, never pushed
# if your machine dies, all work is lost
```

Before pushing, always run the full test suite. A push that breaks the
remote branch is worse than no push at all.

### Branching

For multi-step plans, work on a feature branch. Not on `main`. Not on
`master`. The branch name should be descriptive:

```bash
# GOOD
git checkout -b feat/add-auth-module
git checkout -b fix/config-parsing-timeout-validation
git checkout -b refactor/extract-database-layer

# BAD
git checkout -b fix
git checkout -b updates
git checkout -b my-branch
```

When the plan is fully executed and all tests pass, the branch is ready
for merge. The agent should inform the operator that the branch is
complete and ready for review.

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
