---
description: Create story-driven commits with context awareness
argument-hint: [quick]
---

# Commit Command

**Goal:** Create meaningful commits that tell the story of how a feature was built. Each commit should be a logical step that makes sense independently.

**Approach:** Adapt to the changes you see. Understand the purpose (from conversation or by asking), group related changes logically, create a sequence that tells the story. Don't force a specific number of commits—create as many or as few as makes sense for the changes.

## Modes

**Full mode (default):** Story-driven commits with full context (WHY and WHAT)
**Quick mode (inherited or explicit):** Simple commit descriptions (WHAT only), minimal ceremony

Usage: `/commit` or `/commit quick`

When quick mode is used, it sets quick mode for `/pr` command.

## Context Awareness

**What is "session context"?**
Session context = your conversation memory. If the user ran `/review` or discussed the feature earlier in this conversation, you already know the purpose and JIRA ticket. Use that knowledge.

**Priority order for understanding the change:**
1. **Conversation memory** - Check if purpose was already discussed (e.g., from `/review` or earlier in this chat)
2. **Commit history** - Review recent commits on branch for context
3. **Infer from diff** - Analyze changes to understand the WHY
4. **Confirm with user** - Present inference and ask user to confirm or adjust

**Note:** Branch names are good for JIRA tickets but rarely explain WHY. Always infer purpose from actual code changes and confirm with user.

**If you already know the context from earlier in the conversation, use it. Don't ask again.**

## Story-Driven Commits (Full Mode)

Create **multiple logical commits** that tell the story of the feature being built:

**Good** (incremental story):
```
feat(PRD-123): add user authentication model
feat(PRD-123): implement JWT token generation
feat(PRD-123): add login endpoint
feat(PRD-123): add authentication middleware
test(PRD-123): add authentication test coverage
```

**Bad** (one giant commit):
```
feat(PRD-123): add authentication feature
```

**Principles:**
- Each commit should represent a logical step in building the feature
- Order commits chronologically (foundation → functionality → tests)
- Group related changes together
- Keep commits focused and atomic
- Make each commit independently understandable

## JIRA Ticket Extraction

**Extract JIRA ticket from:**
1. **Branch name** - `feat/PRD-123-add-auth` → `PRD-123`
2. **Existing commits** - Check if other commits already have ticket
3. **Ask user** - Only if ticket not found

**Ticket format:** Project key + number (e.g., `PRD-123`, `PDAC-164`, `BAT-456`)

## Commit Steps

### 0. **Branch Safety Check**

**CRITICAL: Never commit directly to main/master**

Before doing anything else, check if we're on a protected branch:

```bash
# Check current branch
git branch --show-current
```

**If on main or master:**
1. Automatically create a feature branch with a sensible name
2. Use pattern: `feat/<description>` or `fix/<description>` or `docs/<description>`
3. Derive description from JIRA ticket if available, otherwise use generic description
4. Create and switch to the new branch:
   ```bash
   git checkout -b <branch-name>
   ```

**Example:**
```
⚠️ You're on 'main'. Creating feature branch: feat/add-authentication

git checkout -b feat/add-authentication
```

**Branch naming:**
- If JIRA ticket found in diff/context: `feat/PRD-123-short-description`
- If no ticket: `feat/descriptive-name` (infer from changes)
- Use semantic prefix: `feat/`, `fix/`, `docs/`, `refactor/`, etc.

**Only proceed with commits after switching to a feature branch.**

### 1. **Gather Complete Context**

**CRITICAL: Check BOTH modified AND untracked files to avoid missing major changes**

**Instead of complex bash scripts, use individual commands in sequence:**

```bash
# Check current branch
git branch --show-current
# Show all changes (modified, new, deleted)
git status --short
# Review recent commits for context
git log origin/main..HEAD --oneline
# Count modified files
git diff --name-only origin/main | wc -l
# Count new untracked files
git ls-files --others --exclude-standard | wc -l
# Count deleted files
git diff --name-only --diff-filter=D origin/main | wc -l
# Show new top-level directories
git ls-files --others --exclude-standard | cut -d/ -f1 | sort -u
# Check for implementation documentation
git ls-files --others --exclude-standard | grep -iE 'IMPLEMENTATION|SUMMARY|DESIGN' | head -5
# Check for files to exclude (credentials, local config, etc.)
git ls-files --others --exclude-standard | grep -iE '\.env|cookie|credential|secret|\.claude|\.vscode|\.idea|\.log'
# Get diff statistics for modified files
git diff --stat origin/main
# Show first 20 untracked files
git ls-files --others --exclude-standard | head -20
# Calculate total lines in new files (excluding sensitive files)
git ls-files --others --exclude-standard | grep -vE '^\.claude/|cookie|credential|secret' | xargs wc -l 2>/dev/null | tail -1
```

**Analysis approach:**
- Run each command individually to gather information
- Analyze the output yourself (don't rely on shell variables)
- If you see >10 new files → warn about large addition
- If you see >1000 new lines → warn about major feature
- If you see implementation docs → read the first one for context
- If you see sensitive files → explicitly note they should NOT be committed

**Pre-flight checklist:**
- ✅ Checked both `git diff` (modified) AND `git ls-files --others` (new)
- ✅ Identified large directories or additions (>1000 lines = major feature)
- ✅ Read implementation docs if present
- ✅ Identified files to exclude (credentials, local config)
- ✅ Understood FULL scope (not just current session)

**If total changes > 50 files, explicitly confirm with user before proceeding.**

### 2. **Understand Purpose**

**Check for quick mode first:**
- If session has `quick_mode=true` or $1 is "quick": Skip to simple commits (WHAT only)
- If $1 is "quick": Set `quick_mode=true` in session for `/pr`

**Full mode - check session context:**
- Was this feature reviewed with `/review`?
- Did the user explain the purpose earlier in this session?
- Can we extract context from recent commits or branch name?

**If context exists:** Use it directly, state your understanding

**If context unclear:** Infer from changes and confirm with user

### 3. **Analyze Changes for Story**

**Full mode:** Group changes into logical increments:
- **Foundation** - Models, schemas, migrations
- **Core logic** - Services, business logic
- **API/Interface** - Controllers, routes, endpoints
- **Integration** - Middleware, utilities, helpers
- **Tests** - Unit tests, integration tests
- **Documentation** - README, comments, CLAUDE.md

**Quick mode:** Group by file type or simple logical units, less granular

### 4. **Extract JIRA Ticket**

**Use individual commands and analyze the output yourself:**

```bash
# Try to extract from branch name (pattern: PRD-123, PDAC-456, etc.)
git branch --show-current | grep -oE '[A-Z]+-[0-9]+'
# If no result, try from existing commits on the branch
git log origin/main..HEAD --format=%s | grep -oE '[A-Z]+-[0-9]+' | head -1
```

**Analysis approach:**
- Run the branch name extraction first
- If you get a ticket (e.g., "PRD-123"), use it for all commits
- If no ticket from branch, check recent commits
- If still no ticket, proceed without ticket (skip the parenthesis in commit format)

### 5. **Create Commit Sequence**

**Full mode - detailed commits (with ticket):**
```bash
git commit -m "feat(PRD-123): add authentication model

- Create User model with email and password fields
- Add password hashing utility
- Implement JWT token generation

🤖 Generated with AI"
```

**Full mode - detailed commits (without ticket):**
```bash
git commit -m "feat: add authentication model

- Create User model with email and password fields
- Add password hashing utility
- Implement JWT token generation

🤖 Generated with AI"
```

**Quick mode - simple commits (with ticket):**
```bash
git commit -m "feat(PRD-123): add User model and auth utilities

🤖 Generated with AI"
```

**Quick mode - simple commits (without ticket):**
```bash
git commit -m "feat: add User model and auth utilities

🤖 Generated with AI"
```

**Semantic commit types:**
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Code restructuring
- `test` - Adding/updating tests
- `docs` - Documentation only
- `chore` - Build/tooling changes
- `perf` - Performance improvements

**Format:**
- **With JIRA ticket:** `type(TICKET): short description`
- **Without JIRA ticket:** `type: short description` (skip the parenthesis entirely)

### 6. **Confirm Sequence**

Before executing commits:
- Show proposed commit sequence
- Preview commit messages
- Ask user to confirm or adjust

Example:
```
📝 Proposed commit sequence:

1. feat(PRD-123): add authentication model
2. feat(PRD-123): implement JWT token service
3. feat(PRD-123): add login endpoint
4. test(PRD-123): add authentication tests

Proceed with these commits? (or suggest adjustments)
```

### 7. **Execute Commits**

Stage and commit each increment sequentially:

```bash
# Commit 1
git add src/models/User.ts src/utils/password.ts
git commit -m "feat(PRD-123): add authentication model..."

# Commit 2
git add src/services/auth.ts
git commit -m "feat(PRD-123): implement JWT token service..."

# And so on...
```

### 8. **Provide Summary**

```
✅ Created 4 commits:
  1. feat(PRD-123): add authentication model
  2. feat(PRD-123): implement JWT token service
  3. feat(PRD-123): add login endpoint
  4. test(PRD-123): add authentication tests

📊 Changes:
  - 8 files changed
  - 245 insertions (+)
  - 12 deletions (-)

Next: Run /pr to create pull request
```

## Context Preservation (Conversation Memory)

After running this command, you'll remember these details for the next command:

**Full mode - remember:**
- Feature purpose/goal
- JIRA ticket number
- What was built and why
- Any important decisions or trade-offs
- Commit structure you created

**Quick mode - remember:**
- Quick mode flag (if user ran `/commit quick`)
- JIRA ticket number
- Simple list of changes

**How this works:**
When the user later runs `/pr`, you'll use your conversation memory from this commit process. You won't need to ask "what's the JIRA ticket?" or "what's the purpose?" again because you already know.

If user used `quick` flag here, `/pr` will automatically use quick mode too.

## Example Workflow

**Scenario 1: After `/review`**
```
User: /review
Claude: [reviews changes]
User confirms: "Yes, adding authentication to the API"

User: /commit
Claude: "I'll create commits for the authentication feature (from our review).

Proposed sequence:
1. feat(PRD-123): add user authentication model
2. feat(PRD-123): implement JWT token generation
3. feat(PRD-123): add login endpoint
4. test(PRD-123): add auth test coverage

Proceed?"
```
**No redundant questions - context already established.**

**Scenario 2: Standalone usage**
```
User: /commit
Claude: [checks git history, analyzes diff]
"I see you're adding JWT-based authentication (User model, JWT service, login endpoint).

Why are you adding this? What's the context?"
User: "We're launching user accounts next sprint, need auth first"
Claude: "Got it! Creating story-driven commits with that context..."
```

**Scenario 3: Working with Claude from start**
```
User: "Help me add authentication"
[Claude builds the feature throughout session]

User: /commit
Claude: "I'll commit the authentication feature we just built.

Proposed sequence:
1. feat(PRD-123): add user model and password hashing
2. feat(PRD-123): implement JWT service
3. feat(PRD-123): add login endpoint
..."
```
**Seamless - Claude already knows what was built because it built it.**

## Safety Features

- Show full commit sequence before executing
- Allow user to adjust commit grouping
- Verify no uncommitted changes remain
- Check if commits already exist
- Preserve existing commits (only add new ones)

## Error Handling

- **No changes to commit:** "No uncommitted changes found"
- **Already committed:** "Changes already committed. Ready for /pr?"
- **Merge conflicts:** Guide user through resolution
- **No JIRA ticket found:** Create commits with format `type: description` (no parenthesis)

## Notes

- Always preserve session context for `/pr` command
- Group related changes logically, not by file type
- Each commit should be independently understandable
- Commit messages should explain WHAT and WHY, not HOW
- Tests can be in separate commits or bundled with features (user preference)
- Use conventional commits format consistently
- Include ticket in every commit message for traceability
