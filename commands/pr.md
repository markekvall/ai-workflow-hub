---
description: Create pull request with JIRA integration and template preservation
argument-hint: [--preview]
---

# Pull Request Command

**Goal:** Create a pull request with a clear description that explains what changed and why. Make it easy for reviewers to understand the context.

**Approach:** Use information from earlier conversation (if available), extract JIRA ticket from branch/commits, fill the PR template with meaningful content. Adapt to the project's template structure and preserve any guidance comments.

## Modes

**Full mode (default):** Comprehensive PR description with purpose, context, and detailed changes
**Quick mode (inherited):** Basic PR with change list, minimal explanation

Usage: `/pr` or `/pr --preview` (to show description before creating)

**Preview Mode:**
- By default, creates PR immediately without preview
- Use `--preview` flag to review description before creation
- Quick mode is inherited from conversation context if `/review --quick` was used earlier

## Context Awareness

**What is "session context"?**
Session context = your conversation memory. If the user ran `/review` or `/commit` earlier in this conversation, you already know the purpose, JIRA ticket, and what was built. Use that knowledge.

**Priority order for understanding the change:**
1. **Conversation memory** - Check if purpose was discussed (e.g., from `/review` or `/commit` earlier in this chat)
2. **Commit messages** - Extract purpose from commit history on branch
3. **Infer from changes** - Analyze code changes to understand the WHY
4. **Confirm with user** - Present inference and ask user to confirm or adjust

**Note:** Branch names are good for JIRA tickets but rarely explain WHY. Always infer purpose from commits/changes and confirm with user.

**If you already know the context from earlier in the conversation, use it. Don't ask again.**

## JIRA Integration

### Extract Ticket Number

**Sources (in priority order):**
1. **Branch name** - `feat/PRD-123-add-auth` → `PRD-123`
2. **Commit messages** - Check existing commits for ticket references
3. **User input** - Ask only if not found above

**Format:** `[A-Z]+-[0-9]+` (e.g., `PRD-123`, `PDAC-164`, `BAT-456`)

### Add JIRA Link

Add to "Related Documents" section:
```markdown
## Related documents

- PRD-123
```

## PR Template Handling

### Read Template

**Check for template at:**
1. `.github/pull_request_template.md`
2. `.github/PULL_REQUEST_TEMPLATE.md`
3. `docs/pull_request_template.md`

### Preserve Hidden Comments

Templates may contain HTML comments for guidance:
```markdown
<!-- Instructions for filling out this PR -->
## Summary
<!-- Brief description of what this PR does -->
```

**IMPORTANT:** Preserve all HTML comments exactly as-is. These are intentional.

### Fill Template Sections

Common sections:
- **Summary/Description** - What was built and why
- **Changes** - List of key changes
- **Testing** - How it was tested
- **Related Documents** - JIRA ticket link
- **Screenshots** - If UI changes
- **Checklist** - Pre-merge requirements

## PR Steps

### 1. **Gather Context**

```bash
# Check if already on feature branch
git branch --show-current

# Get commit history for this branch
git log origin/main..HEAD --oneline

# Check if changes are committed
git status

# Get repository info for PR URL
git remote get-url origin
```

### 2. **Extract JIRA Ticket**

**Use individual commands and analyze output yourself:**

```bash
# Try to extract from branch name (pattern: PRD-123, PDAC-456, etc.)
git branch --show-current | grep -oE '[A-Z]+-[0-9]+'

# If no result, try from commits
git log origin/main..HEAD --format=%s | grep -oE '[A-Z]+-[0-9]+' | head -1
```

**Analysis approach:**
- Run branch extraction first
- If you get a ticket (e.g., "PRD-123"), use it for PR
- If no ticket from branch, check commit messages
- If still no ticket, ask user: "No JIRA ticket found. What's the ticket number? (or 'none' if not applicable)"

### 3. **Understand Purpose**

**Check for quick mode first:**
- If session has `quick_mode=true` or $1 is "quick": Skip deep context, use simple descriptions

**Full mode - check session context:**
- Was purpose discussed in `/review`?
- Was purpose confirmed in `/commit`?
- Did user explain goal earlier in session?
- Can we extract from commit messages?

**If context exists:** Use it directly

**If not:** Infer from commits and confirm with user

### 4. **Generate PR Title**

**Format:**
- **With JIRA ticket:** `type(TICKET): description`
- **Without JIRA ticket:** `type: description` (skip the parenthesis entirely)

**Examples with ticket:**
- `feat(PRD-123): add JWT authentication to API`
- `fix(PDAC-164): resolve timeout on video processing`
- `refactor(BAT-456): migrate dashboard from Loki to metrics`

**Examples without ticket:**
- `feat: add JWT authentication to API`
- `fix: resolve timeout on video processing`
- `refactor: migrate dashboard from Loki to metrics`

**Determine type from commits:**
- If all commits are `feat` → PR is `feat`
- If all commits are `fix` → PR is `fix`
- If mixed → Use most significant type

**Title should summarize ALL commits**, not just one.

### 5. **Read PR Template**

```bash
# Find template
if [ -f .github/pull_request_template.md ]; then
  template=".github/pull_request_template.md"
elif [ -f .github/PULL_REQUEST_TEMPLATE.md ]; then
  template=".github/PULL_REQUEST_TEMPLATE.md"
elif [ -f docs/pull_request_template.md ]; then
  template="docs/pull_request_template.md"
fi

# Read template content
cat "$template"
```

### 6. **Fill Template**

**Full mode - comprehensive description:**

**Summary Section:**
- **What:** Describe what was built (from commits)
- **Why:** Explain purpose/goal (from session context or ask)
- **How:** High-level approach (if relevant)

**Changes Section:**
- List key changes from commits
- Group related changes
- Highlight important decisions or trade-offs

**Testing Section:**
- How was it tested?
- What edge cases were covered?
- Any manual testing steps?

**Related Documents:**
- Add JIRA ticket reference
- Add any other relevant docs from session context

**Checklist:**
- Pre-populate with ✅ for items known to be done
- Leave unchecked items for user review

**Quick mode - minimal description:**

**Summary Section:**
- Brief what changed (from commit messages)

**Changes Section:**
- Bullet list from commits

**Testing Section:**
- Basic checklist (not pre-filled)

**Related Documents:**
- JIRA ticket link only

### 7. **Create PR (with optional preview)**

**Default behavior:** Create PR immediately

**If `--preview` flag provided:** Show filled template first:
```markdown
📝 PR Description Preview:

---
[Full template with all sections filled]
---

Looks good? (or request changes)
```

### 8. **Push and Create PR**

```bash
# Ensure branch is pushed
git push -u origin $(git branch --show-current)

# Create PR with gh cli
gh pr create \
  --title "feat(PRD-123): add JWT authentication to API" \
  --body "$(cat <<'EOF'
## Summary
<!-- Brief description of what this PR does -->
Adds JWT-based authentication to the API to secure user endpoints.

## What changed
- Added User model with password hashing
- Implemented JWT token generation and validation
- Created login endpoint for authentication
- Added authentication middleware
- Added comprehensive test coverage

## Why
Enable secure user authentication for the API before launching user-facing features.

## Testing
- ✅ Unit tests for all new services
- ✅ Integration tests for login flow
- ✅ Manual testing with Postman
- ✅ Verified token expiration works correctly

## Related documents
- [PRD-123](https://your-org.atlassian.net/browse/PRD-123)

🤖 Generated with AI
EOF
)" \
  --base main
```

### 9. **Provide Summary**

```
✅ Pull Request Created!

🔗 https://github.com/user/repo/pull/42

Title: feat(PRD-123): add JWT authentication to API
Base: main ← feature/PRD-123-add-auth

Next steps:
- Request reviews from team members
- Add labels if needed
- Monitor CI/CD checks
```

## Context Preservation (Conversation Memory)

This command uses information from earlier in the conversation:

**Full mode:**
- Purpose/goal discussed in `/review`
- Commit structure created in `/commit`
- Feature details from earlier conversation
- JIRA ticket extracted previously

**Quick mode:**
- If user ran `/review --quick` earlier, you'll remember to use quick mode here
- Basic change list from commits

**Important:** "Context" means your memory of this conversation, not files. If the user discussed the feature purpose 10 messages ago, you still remember it. Use that knowledge instead of asking again.

## Example Workflows

### Scenario 1: Full workflow with all commands

```
User: /review
Claude: [reviews changes, user confirms purpose]

User: /commit
Claude: [creates story-driven commits using confirmed purpose]

User: /pr
Claude: "I'll create the PR for the authentication feature (from our review).

Title: feat(PRD-123): add JWT authentication to API

[Shows filled template with all context from previous commands]

Looks good?"
```
**No redundant questions across entire workflow.**

### Scenario 2: PR only (no previous commands)

```
User: /pr
Claude: [analyzes commits and changes]
"I see you're adding JWT-based authentication (auth model, JWT service, login endpoint).

Why are you adding this? What's the business context?"
User: "We're launching user accounts next sprint, need auth before that"
Claude: [creates PR with WHAT and WHY captured]
```

### Scenario 3: After working with Claude

```
User: "Help me add authentication"
[Claude builds feature throughout session]

User: /commit
[Claude creates commits]

User: /pr
Claude: "Creating PR for the authentication we just built.

Title: feat(PRD-123): add JWT authentication to API
..."
```
**Seamless - Claude knows everything because it built the feature.**

## Template Example

**Common template structure:**
```markdown
## Summary
<!-- Brief description of what this PR does -->

## What changed
<!-- List of key changes -->

## Why
<!-- Purpose and motivation -->

## Testing
<!-- How it was tested -->
- [ ] Unit tests added
- [ ] Integration tests added
- [ ] Manual testing completed

## Related documents
<!-- Links to JIRA, design docs, etc. -->

## Screenshots
<!-- If applicable -->
```

**Filled example:**
```markdown
## Summary
<!-- Brief description of what this PR does -->
Adds JWT-based authentication to the API to secure user endpoints.

## What changed
<!-- List of key changes -->
- Added User model with password hashing
- Implemented JWT token generation and validation
- Created login endpoint
- Added authentication middleware
- Comprehensive test coverage

## Why
<!-- Purpose and motivation -->
Enable secure user authentication before launching user-facing features. Required for PRD-123 milestone.

## Testing
<!-- How it was tested -->
- ✅ Unit tests added
- ✅ Integration tests added
- ✅ Manual testing completed

## Related documents
<!-- Links to JIRA, design docs, etc. -->
- [PRD-123](https://your-org.atlassian.net/browse/PRD-123)
```

## Safety Features

- Preview full PR description before creating
- Verify branch is pushed before creating PR
- Check if PR already exists for branch
- Preserve all template comments
- Validate JIRA ticket format
- Confirm base branch (usually main/master)

## Error Handling

- **No commits:** "No commits found. Run /commit first?"
- **PR already exists:** Show existing PR URL
- **Template not found:** Use default format
- **No JIRA ticket:** Create PR with format `type: description` (no parenthesis)
- **Push fails:** Check permissions and guide user

## Notes

- Always preserve HTML comments in templates
- JIRA ticket goes in BOTH title and Related Documents
- Use semantic commit type for PR title
- Summary should explain WHAT and WHY, not HOW
- Include context from earlier session work when available
- Don't ask redundant questions if context exists
- PR title should summarize all commits, not just latest
- Test checklist items can be pre-checked if known to be done
