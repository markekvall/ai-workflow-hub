---
description: Review current branch changes before creating a PR
argument-hint: [quick]
---

# Review Command

**Goal:** Review code quality before creating a PR. Identify issues, confirm the feature achieves its purpose, and provide actionable feedback.

**Approach:** Adapt the review to the project context. Detect test frameworks, infer purpose from changes, focus on what matters for this specific feature.

## Modes

**Full mode (default):** Confirms purpose, evaluates against goal, provides context for downstream commands
**Quick mode:** Pass `quick` as argument to skip purpose discussion and review code as-is

Usage: `/review` or `/review quick`

When quick mode is used, it sets quick mode for entire workflow (`/commit` and `/pr` inherit it).

## Review Focus Areas

### 1. **Code Quality**
- Clean code principles and best practices
- Proper error handling and edge cases
- Code readability and maintainability
- DRY principle (Don't Repeat Yourself)
- Code duplication
- Unused variables or functions
- Proper use of language idioms

### 2. **Security**
- Potential security vulnerabilities
- Input validation and sanitization
- Authentication/authorization logic
- SQL injection risks
- XSS vulnerabilities
- Sensitive data handling

### 3. **Performance**
- Potential performance bottlenecks
- Database query efficiency
- N+1 query problems
- Memory leaks or resource issues
- Unnecessary computations
- Caching opportunities

### 4. **Testing**
- Adequate test coverage
- Test quality and edge cases
- Missing test scenarios
- Test organization and structure
- Use of test helpers to reduce duplication

### 5. **Documentation**
- Code comments for complex logic
- TODO comments for skeleton implementations
- README updates for new features
- API documentation accuracy
- CLAUDE.md or similar project docs updates

### 6. **Architecture & Design**
- Adherence to project patterns
- Separation of concerns
- Dependency injection
- Proper layering (routes → controllers → repos)
- Model design and validation

## Review Steps

1. **Fetch remote and gather changes:**
   - Run `git fetch origin` to get latest remote state
   - Determine base branch (origin/main or origin/master)
   - Check for uncommitted changes with `git status`
   - **Ask user: Should uncommitted changes be included in the review?**
     - If YES: Use `git diff origin/main...HEAD` for committed changes, then check `git status` output to note uncommitted files
     - If NO: Use `git diff origin/main...HEAD` (committed changes only on this branch)
   - Review commit history with `git log origin/main..HEAD --oneline` (or origin/master)
   - **IMPORTANT**: Always use three-dot syntax (`...`) to compare only changes made on this branch since it diverged from the base branch. This prevents including unrelated changes from the base branch.

2. **Infer and confirm purpose:**
   - **If $1 is "quick":** Skip this step entirely, proceed to review
   - **Otherwise (full mode):**
     - Analyze the changed files and code patterns to understand WHAT is being built
     - If JIRA MCP available: Fetch ticket description for context
     - Present what you detected: "I see you're adding [WHAT - e.g., JWT authentication with User model, login endpoint]."
     - **Ask for the WHY:** "Why are you adding this? What problem does it solve?"
     - Let user provide the context (e.g., "launching user accounts", "regulatory requirement", "security incident")
     - Use JIRA ticket description as starting point if available
     - Use this confirmed purpose as context for the entire review
     - **IMPORTANT:** Store this confirmed purpose in session context for `/commit` and `/pr` commands

3. **Analyze each changed file:**
   - Read the file contents using the Read tool
   - Identify issues by category
   - Note specific line numbers and code snippets
   - **Evaluate if the implementation effectively achieves the stated goal**

4. **Run tests:**
   - **Auto-detect and run appropriate test command:**
     - Check for `Makefile` with test target → `make test`
     - Check for `package.json` → `npm test`
     - Check for Python project (pytest.ini, setup.py, pyproject.toml) → `pytest`
     - Check for Go project (go.mod) → `go test ./...`
     - Check for Rust project (Cargo.toml) → `cargo test`
     - Check for Gradle project (build.gradle) → `./gradlew test`
     - If none found, ask user for test command
   - Check test coverage for new code
   - Run specific test suites if needed

5. **Generate comprehensive review report and ask for next steps:**
   - Use the confirmed purpose to provide relevant feedback
   - Check if edge cases for the specific feature are covered
   - Suggest better approaches for achieving the stated goal
   - Evaluate if the implementation matches the intent

   **After presenting the review:**
   - **Ask user:** "What would you like to do next?"
   - **Suggest options:**
     - Fix the issues found (you can help)
     - Proceed with commits using `/commit`
     - Adjust the implementation
     - Ask questions about specific findings
   - **Do NOT automatically run `/commit`** - wait for user decision

## Review Output Format

Provide a comprehensive review with:

### Overall Assessment
- **Purpose alignment**: Does the implementation achieve the stated goal?
- Grade and status recommendation (Approve/Request Changes/Comment)
- Key strengths (what's working well)
- Critical issues (must fix)
- Important improvements (should fix)
- Nice-to-have suggestions
- Test coverage summary

### Detailed Findings

**IMPORTANT**: Number each finding (1, 2, 3, etc.) so the user can reference them easily.

For each issue found, provide:
- **Number**: Sequential number for easy reference
- **Severity marker**: 🔴 Critical, 🟡 Important, 🔵 Nice-to-have, ✅ Praise
- **Category**: Code Quality, Security, Performance, Testing, Documentation, or Architecture
- **Location**: File path and line numbers
- **Issue description**: What's the problem?
- **Code example**: Show the problematic code
- **Suggested fix**: Provide concrete code snippet
- **Explanation**: Why this matters and benefits of the fix

## Review Severity Guidelines

**🔴 CRITICAL** - Must fix before production:
- Security vulnerabilities
- Data loss risks
- Breaking changes
- Critical bugs

**🟡 IMPORTANT** - Should fix before merge:
- Code quality issues
- Missing tests
- Unused code
- Code duplication
- Missing edge case handling

**🔵 NICE-TO-HAVE** - Consider for follow-up:
- Performance optimizations
- Future enhancements
- Documentation improvements
- Refactoring opportunities

**✅ PRAISE** - Recognize good work:
- Excellent patterns
- Clean code
- Good test coverage
- Helpful documentation

## Example Finding Format

```markdown
### [#] [Severity] [Category]: [Brief Title] ([file/path:line-numbers])

**Issue:**
Clear description of what the problem is.

**Current code:**
```language
// Show the problematic code snippet
```

**Suggested fix:**
```language
// Show the improved version
```

**Benefits:**
- Why this matters
- What improves
- Impact of the change
```

## Context Preservation (Conversation Memory)

After running this command, you'll remember these details for later commands in this conversation:

**Full mode - remember:**
- Feature goal/purpose
- What problem is being solved
- Why this approach was chosen
- Any important trade-offs discussed
- JIRA ticket number

**Quick mode - remember:**
- Quick mode flag (if user ran `/review quick`)
- Minimal description of changes

**How this works:**
When the user later runs `/commit` or `/pr`, you'll use your conversation memory of this review. You won't need to ask "what's the purpose?" again because you already know from this review.

If user used `quick` flag, you'll automatically use quick mode in `/commit` and `/pr` without asking.

## Notes

- **Always confirm the purpose with the user before reviewing** - Don't assume you understand the goal
- Evaluate the implementation in context of the confirmed purpose
- **Store confirmed purpose for downstream commands** (`/commit`, `/pr`)
- Always be constructive and specific in feedback
- Provide code examples for suggested changes
- Balance criticism with praise for good work
- Consider the context (e.g., skeleton implementation vs production code)
- Check if similar patterns exist elsewhere in the codebase
- Verify tests pass before and after suggested changes
- Be mindful of project conventions and existing patterns
- Suggest alternative approaches if there's a better way to achieve the stated goal

