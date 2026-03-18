# Create and Start a Git Worktree

Create a new git worktree for isolated feature development. Handles both git repos and non-git directories.

## Steps

1. **Check if current directory is a git repo**
   - Run `git rev-parse --is-inside-work-tree 2>/dev/null`
   - If not a git repo, ask the user: "This directory is not a git repo. Initialize one with `git init`?"
   - If yes, run `git init` then create an initial commit: `git add . && git commit -m "initial commit"` (or just `git commit --allow-empty -m "initial commit"` if nothing to add)

2. **Get worktree name**
   - If the user provided a name in `$ARGUMENTS`, use that
   - Otherwise ask: "What should the worktree be named?"

3. **Create the worktree**
   - Run: `claude --worktree <name>`
   - This creates `.claude/worktrees/<name>` with branch `worktree-<name>`

4. **Confirm to the user**
   - Tell the user the worktree was created at `.claude/worktrees/<name>` on branch `worktree-<name>`
   - Remind them Claude will prompt to keep or remove it when the session ends
