# AI Workflow Hub

A collection of commands for collaborative AI-assisted development workflows. Works with Cursor, Claude Code, and other AI coding assistants. These commands create a seamless experience for code review, commits, and pull requests—with context that flows between steps.

## Philosophy

These commands treat AI as a thinking partner, not just a code generator. They:

- **Preserve context** across the entire workflow (review → commit → PR)
- **Tell stories** through commits that explain how features were built
- **Confirm understanding** before acting, avoiding assumptions
- **Adapt** to your project's conventions and templates

## Commands

### `/review`
Review your changes before creating a PR. Identifies code quality issues, security concerns, performance bottlenecks, and missing tests.

```
/review        # Full review with purpose discussion
/review quick  # Skip purpose, review code as-is
```

**What it does:**
- Analyzes all changes on your branch
- Infers what you're building and confirms the purpose with you
- Provides categorized feedback (🔴 Critical, 🟡 Important, 🔵 Nice-to-have)
- Runs your test suite automatically
- Stores context for downstream commands

### `/commit`
Create meaningful, story-driven commits that explain the evolution of your feature.

```
/commit        # Story-driven commits with context
/commit quick  # Simple commits, minimal ceremony
```

**What it does:**
- Groups related changes into logical commits
- Extracts JIRA tickets from branch names
- Creates commits in chronological order (foundation → functionality → tests)
- Prevents commits to protected branches (main/master)
- Uses confirmed context from `/review` if available

### `/pr`
Create a pull request with a complete description that explains what changed and why.

```
/pr            # Creates PR immediately
/pr --preview  # Shows description before creating
```

**What it does:**
- Generates title and description from commits
- Fills your PR template while preserving HTML comments
- Links JIRA tickets in the description
- Uses context from earlier commands (no redundant questions)

### `/thinking-partner`
Explore complex problems through structured questioning before jumping to implementation.

**What it does:**
- Asks clarifying questions instead of rushing to solutions
- Tracks insights and connections throughout the conversation
- Surfaces hidden assumptions
- Helps you think through the problem space

## Workflow Example

```
# 1. Review your changes
/review
> "I see you're adding JWT authentication. Why are you adding this?"
> "We're launching user accounts next sprint, need auth first"

# 2. Create story-driven commits  
/commit
> Creates: feat(PRD-123): add user model
>          feat(PRD-123): implement JWT service  
>          feat(PRD-123): add login endpoint
>          test(PRD-123): add auth tests

# 3. Create the PR
/pr
> Uses context from steps 1 & 2—no redundant questions
```

## Installation

**Cursor:**
1. Copy the `commands/` directory to your project's `.cursor/` folder
2. Use commands with `/command-name`

**Claude Code:**
1. Copy the `commands/` directory to your project's `.claude/` folder
2. Use commands with `/command-name`

## Customization

### Commit Footer
Commands include a `🤖 Generated with AI` footer. Modify this in `commit.md` and `pr.md` if desired.

### PR Templates
The `/pr` command automatically detects and fills templates at:
- `.github/pull_request_template.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `docs/pull_request_template.md`

## Conventions Used

- **Conventional Commits**: `type(scope): description`
- **Semantic Types**: feat, fix, refactor, test, docs, chore, perf
- **JIRA Integration**: Ticket extracted from branch name or commit messages
- **Story-driven commits**: Logical progression showing how a feature was built

## License

MIT

