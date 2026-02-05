---
description: Commit changes and publish branch with PR
---

# Ship It

Commit the current changes with an accurate message, push the branch to the remote, and ensure a PR exists. If issue numbers are provided as arguments, attach them to the PR so they are closed.

## Inputs

- Optional arguments: one or more issue numbers (e.g., `ship-it 123 456`)

## Process

1. **Collect context**

   - `git status --porcelain` to understand the working tree.
   - `git diff` for the change overview.
   - `git log -5 --oneline` to match commit message style.

2. **Build commit message and PR description**

   - **CRITICAL:** Thoroughly analyze the COMPLETE `git diff` output to understand ALL changes across all files.
   - Review the full conversation context to understand the complete scope of work.
   - Identify ALL significant changes, not just the most recent or obvious ones.
   - The commit message should be concise but the PR description should comprehensively describe all changes.
   - Focus on the intent and outcome of the changes, not just the files touched.
   - Don't miss important features or modifications just because they weren't mentioned in the final messages.

3. **Stage and commit**

   - **CRITICAL:** Review `git status --porcelain` for untracked files (lines starting with `??`).
   - Stage ALL relevant files including:
     - Modified files (lines starting with `M`)
     - New untracked files that are part of the work (including new command files, config files, etc.)
   - Use `git add <file>` for each relevant file, or `git add .` if all untracked files should be included.
   - If there are no changes, stop and report that there is nothing to commit.
   - Create the commit with the drafted message.

4. **Determine base branch**

   - Prefer the reflog entry for branch creation (e.g., `checkout: moving from <base> to <current>`).
   - Fallback to `origin/HEAD` if no reflog entry exists.
   - If the base cannot be determined, stop and report the issue.

5. **Push or publish**

   - If the branch has no upstream, run `git push -u origin <branch>`.
   - If it already tracks a remote, run `git push`.

6. **Create or update PR**
   - Always ensure a PR exists on the base branch using `gh pr create` or `gh pr edit`.
   - Write a comprehensive PR description that includes:
     - A summary section listing ALL major changes
     - Specific technical details about what was modified
     - Testing notes or verification steps
   - Review the full diff output again to ensure no significant changes are missed in the PR description.
   - If issue numbers were provided, append `Closes #NNN` lines to the PR body.
   - Return the PR URL.

## Output

Report:

- Commit message used
- Branch push status
- PR URL
