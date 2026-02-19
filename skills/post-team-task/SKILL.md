---
name: post-team-task
description: Create GitHub issues for each team member and add them to a project board
disable-model-invocation: true
allowed-tools: [Bash, Read, Grep, AskUserQuestion]
---

# Post Team Task

Create a GitHub issue for each team member and add it to a shared GitHub Projects board with "In progress" status. The invoking command provides the issue title and body; this skill handles creation and project board management.

## Prerequisites Check

Before starting, read `~/.claude/skills/post-team-task/.env` and verify the following variables are present:

1. **GH_ORG** — GitHub organization name
2. **GH_TEAM_REPO** — Repository for team issues
3. **GH_PROJECT_NUMBER** — GitHub Projects board number
4. **GH_TEAM_MEMBERS** — Comma-separated list of GitHub usernames

Also verify GitHub CLI authentication:

```bash
gh auth status
```

**Setup Instructions (if prerequisites fail)**:

```
Missing configuration. Please set up the .env file:

1. Create ~/.claude/skills/post-team-task/.env with:

   GH_ORG=your-org
   GH_TEAM_REPO=your-repo
   GH_PROJECT_NUMBER=1
   GH_TEAM_MEMBERS=user1,user2,user3

2. Ensure GitHub CLI is authenticated:
   gh auth login
   gh auth refresh -s project
```

## Process Flow

Follow these steps in order:

1. **Validate Prerequisites**

   - Read `~/.claude/skills/post-team-task/.env` and parse all four variables
   - Run `gh auth status` to confirm authentication
   - Halt with setup instructions if anything is missing

2. **Receive Task Details**

   - The invoking command provides the issue title and body
   - Use them verbatim — do not modify or prompt for them

3. **Preview & Confirm**

   - Show the user:
     - Issue title and body
     - List of team members who will each get an issue
     - Target repo (`GH_ORG/GH_TEAM_REPO`) and project board number
   - Ask for confirmation via `AskUserQuestion` before proceeding

4. **Create Issues**

   For each team member in `GH_TEAM_MEMBERS`:

   ```bash
   gh issue create --repo "$GH_ORG/$GH_TEAM_REPO" \
     --assignee "$USERNAME" --title "$TITLE" --body "$BODY"
   ```

   Capture each returned issue URL.

5. **Add to Project Board & Set Status**

   First, fetch project metadata (once, then reuse):

   ```bash
   # Get project ID
   gh project view $GH_PROJECT_NUMBER --owner $GH_ORG --format json --jq '.id'

   # Get Status field ID and "In progress" option ID
   gh project field-list $GH_PROJECT_NUMBER --owner $GH_ORG --format json
   ```

   Then for each issue:

   ```bash
   # Add issue to project (returns item ID)
   gh project item-add $GH_PROJECT_NUMBER --owner $GH_ORG --url $ISSUE_URL --format json

   # Set status to "In progress"
   gh project item-edit --id $ITEM_ID --project-id $PROJECT_ID \
     --field-id $STATUS_FIELD_ID --single-select-option-id $IN_PROGRESS_OPTION_ID
   ```

6. **Report Results**

   Display a summary table showing each team member, their issue URL, and whether creation/board-add succeeded or failed.

## Error Handling

### Missing `.env` or Variables

- Display setup instructions (see above)
- Halt execution — do NOT attempt any GitHub operations

### `gh` Auth Failure

- Instruct user to run:
  ```
  gh auth login
  gh auth refresh -s project
  ```
- Halt execution

### Single Issue Creation Failure

- Log the error for that member
- Continue creating issues for remaining members
- Report all failures in the final summary

### "In progress" Option Not Found

- List available status options from the project field metadata
- Ask the user (via `AskUserQuestion`) which option to use instead
