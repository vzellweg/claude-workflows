---
name: asana-update
description: Generate and post bi-weekly project updates to Asana task
disable-model-invocation: true
allowed-tools: [Bash, Read, Grep, Write, AskUserQuestion]
---

# Asana Update Skill

Generate and post bi-weekly (every 2 weeks) project updates as comments to an existing Asana task. Updates are tailored for non-technical project managers and prioritize git history as the source of truth.

## Prerequisites Check

Before starting, verify the following environment variables:

1. **ASANA_PERSONAL_ACCESS_TOKEN** (system environment variable)

   - Check: `echo $ASANA_PERSONAL_ACCESS_TOKEN`
   - If missing: Display setup instructions and halt

2. **ASANA_TASK_GID** (project .env file)
   - Check: Read `.env` file for `ASANA_TASK_GID`
   - If missing: Display setup instructions and halt

**Setup Instructions (if prerequisites fail)**:

```
Missing environment variables. Please configure:

1. ASANA_PERSONAL_ACCESS_TOKEN (system env):
   - Get token from: https://app.asana.com/0/my-apps
   - Add to ~/.zshrc or ~/.bash_profile:
     export ASANA_PERSONAL_ACCESS_TOKEN="your_token_here"
   - Reload shell: source ~/.zshrc

2. ASANA_TASK_GID (project .env):
   - Extract from task URL: https://app.asana.com/0/PROJECT/TASK → use TASK
   - Add to .env file:
     ASANA_TASK_GID=your_task_gid_here
```

## Data Collection

Collect data from the following sources in order of priority:

### 1. Git Commit History (Primary Source of Truth)

```bash
git log --since="14 days ago" --pretty=format:"%h|%s|%an|%ar" --no-merges
```

- This is the PRIMARY source of truth for completed work
- Filter out merge commits
- Parse commit messages to extract:
  - Commit type (feat, fix, enhance, refactor, etc.)
  - Subject/description
  - Author and relative time

### 2. GitHub Activity (Additional Context)

```bash
# Get merged PRs from the past 14 days
gh pr list --state merged --json title,number,mergedAt --limit 50 | jq '[.[] | select(.mergedAt | fromdateiso8601 > (now - 1209600))]'

# Get closed issues from the past 14 days
gh issue list --state closed --json title,number,closedAt --limit 50 | jq '[.[] | select(.closedAt | fromdateiso8601 > (now - 1209600))]'
```

- Use to supplement git commit information
- Provides PR/issue titles for additional context

### 3. STATUS.md (Optional Reference)

- Read `docs/STATUS.md` if it exists
- Parse remaining tasks section
- **IMPORTANT**: May be outdated - trust git history over STATUS.md if there are conflicts
- If file doesn't exist, this is NOT an error - continue without it

### 4. PROJECT.md (Context Only)

- Read `docs/PROJECT.md` for project context and understanding
- Use to inform translation decisions and understand what's important
- **DO NOT** include a "Project Context" section in the output
- If file doesn't exist, this is NOT an error - continue without it

## Translation Rules

Convert technical git commit messages to PM-friendly language:

### Commit Type Mapping

- `feat:` → "New Feature:"
- `fix:` → "Bug Fix:"
- `enhance:` or `improve:` → "Improvement:"
- `refactor:` → "Improvement:"
- `docs:` → "Documentation:"
- `test:` → "Testing:"
- `chore:` → (omit from update unless significant)

### Pattern-Based Translation

Common patterns to look for and translate:

- `/accessibility/i` → "accessibility improvements"
- `/cpu/i` → "CPU handling"
- `/rotation/i` → "rotation control"
- `/ui|interface/i` → "user interface"
- `/button|control/i` → "controls"
- `/performance|optimize/i` → "performance improvements"
- `/bug|error|issue/i` → "bug fixes"

### Simplification Rules

- "refactor" → "improve"
- "implement" → "add"
- "utilize" → "use"
- Remove technical jargon (e.g., "O(n)", "TypeScript", "props", "state")
- Focus on user-facing impact, not implementation details

### Grouping Strategy

Group similar commits for conciseness:

**Example**:

```
Before:
- fix: CPU placement step
- fix: CPU rotation controls
- fix: CPU action handling

After:
Bug Fixes:
- Improved CPU handling across placement, rotation, and action steps
```

## Update Style Guidelines

**Keep it minimal and high-level:**

- Focus on requirements/features, bug fixes, and usability enhancements
- Avoid implementation details or technical deep-dives
- Group related work into single bullet points
- Aim for brevity - each section should have 2-5 bullet points maximum
- Only include what stakeholders need to know, not everything that changed

## Update Format

Generate the update in the following format:

```markdown
## Summary

Work completed: [Start Date] - [End Date]

## Completed Work

### New Features

- [Feature 1 description in PM-friendly language]
- [Feature 2 description]

### Bug Fixes

- [Fix 1 description in PM-friendly language]
- [Fix 2 description]

### Improvements

- [Improvement 1 description in PM-friendly language]
- [Improvement 2 description]

## Remaining Tasks

[If STATUS.md exists, list remaining tasks here]
[If STATUS.md doesn't exist: "See project documentation for full scope"]
```

## Process Flow

Follow these steps in order:

1. **Validate Prerequisites**

   - Check ASANA_PERSONAL_ACCESS_TOKEN
   - Check ASANA_TASK_GID
   - Halt if either is missing

2. **Collect Data**

   - Run git log command
   - Run GitHub PR/issue commands (if gh CLI available)
   - Read STATUS.md (if exists)
   - Read PROJECT.md (for context only - not to be included in output)

3. **Translate to PM-Friendly Language**

   - Parse commit messages using PROJECT.md context to understand priorities
   - Apply translation rules
   - Group similar commits aggressively for brevity
   - Remove technical jargon
   - Focus on user-facing impact and requirements progress

4. **Generate Formatted Update**

   - Apply update format template
   - Insert translated content
   - Calculate date range (14 days ago to today)

5. **Preview and Allow User Edits**

   - Display the generated update to the user
   - Ask: "Would you like to make any edits before posting to Asana?"
   - Wait for user confirmation or edits

6. **Post Comment to Asana**

   - Use Asana REST API to post comment via curl
   - Target task GID from ASANA_TASK_GID
   - Use the formatted update as comment text
   - Authenticate with ASANA_PERSONAL_ACCESS_TOKEN

7. **Report Success**
   - Confirm comment posted successfully
   - Provide link to Asana task (if available)

## Error Handling

Handle the following error scenarios:

### Missing Environment Variables

- Display setup instructions
- Halt execution
- Do NOT attempt Asana operations

### No Git History

- If no commits in past 14 days, offer to extend range
- Suggest: "No commits found in the past 14 days. Would you like to extend the range to 30 days?"

### Missing PROJECT.md

- This is NOT an error
- Continue without remaining tasks section
- Use fallback text: "See project documentation for full scope"

### Asana API Failure

- Save update to file: `asana-update-[timestamp].md`
- Display file location
- Provide manual instructions:

  ```
  Asana posting failed. Update saved to: [file path]

  To post manually:
  1. Open task: https://app.asana.com/0/[project]/[ASANA_TASK_GID]
  2. Copy content from saved file
  3. Paste as comment
  ```

### GitHub CLI Unavailable

- This is NOT an error
- Continue without GitHub PR/issue data
- Rely on git commit history only

## Translation Examples

**Technical → PM-Friendly**:

| Technical Commit                                   | PM-Friendly Translation                              |
| -------------------------------------------------- | ---------------------------------------------------- |
| `fix: Optimize displayIndex assignment to O(n)`    | "Bug Fix: Improved step ordering performance"        |
| `feat: Add CPURotationControl component`           | "New Feature: CPU orientation control"               |
| `refactor: Convert labStepParser.js to TypeScript` | "Improvement: Enhanced code quality and type safety" |
| `enhance: Update ActionButton accessibility`       | "Improvement: Better screen reader support"          |
| `fix: Broken CPU placement step`                   | "Bug Fix: Resolved CPU placement issues"             |
| `feat: Implement drag-and-drop for components`     | "New Feature: Drag-and-drop component placement"     |

## Conflict Resolution

If git history and STATUS.md conflict:

- **ALWAYS trust git history**
- Git commits represent actual completed work
- STATUS.md may be outdated or aspirational

**Example**:

- Git shows: 5 bug fixes related to CPU handling
- STATUS.md shows: "CPU handling - TODO"
- **Resolution**: Mark CPU handling as completed in the update, note discrepancy

## Tips for Success

1. **Be Minimal**: Keep the entire update brief - aim for 5-10 bullet points total across all sections
2. **Be High-Level**: Focus on what was delivered, not how it was implemented
3. **Be Concise**: Aggressively group similar commits into single bullets
4. **Be Clear**: Avoid technical jargon, focus on user-facing impact and requirements
5. **Be Accurate**: Trust git history as the source of truth
6. **Be Flexible**: Handle missing files gracefully, don't fail unnecessarily

## Testing the Skill

After implementation, verify:

1. ✅ Git log parsing works (14 days, filters merge commits)
2. ✅ GitHub PR/issue integration works (if gh CLI available)
3. ✅ Handles missing STATUS.md gracefully
4. ✅ Translation rules produce PM-friendly language
5. ✅ User can preview and edit before posting
6. ✅ Asana comment posts successfully via REST API
7. ✅ Error handling for missing ASANA_TASK_GID
8. ✅ Error handling for missing ASANA_PERSONAL_ACCESS_TOKEN
