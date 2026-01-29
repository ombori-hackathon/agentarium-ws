---
name: feature
description: Build a new feature end-to-end. Asks clarifying questions, creates a spec/PRD, then implements using TDD (tests first). Use this to start any new feature.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

# /feature - New Feature Workflow

Build features using test-driven development with a proper spec.

## Usage

```
/feature <brief description of what you want to build>
```

## Workflow

### Step 1: Understand the Feature

Ask the user these questions (wait for answers before proceeding):

1. **What should this feature do?** (one sentence)
2. **Who/what triggers it?** (user action, API call, scheduled, etc.)
3. **What's the expected output/result?**
4. **Any edge cases to handle?**

### Step 2: Identify Affected Repos

Based on the feature, determine scope:
- **API only** → `services/api/`
- **Swift client only** → `apps/macos-client/`
- **Full stack** → Both repos

### Step 3: Enter Plan Mode

**IMPORTANT: Enter plan mode now using the EnterPlanMode tool.**

In plan mode, create the spec file `specs/YYYY-MM-DD-feature-name.md` with this structure:

```markdown
# Feature: [Name]

## Summary
[One sentence from Step 1]

## Trigger
[From Step 1 question 2]

## Expected Result
[From Step 1 question 3]

## Edge Cases
[From Step 1 question 4]

## Technical Design

### API Changes (if applicable)
- Endpoint: `METHOD /path`
- Request: `{ field: type }`
- Response: `{ field: type }`

### Swift Client Changes (if applicable)
- New struct/function: `name`
- Display: [how it shows to user]

### Database Changes (if applicable)
- New table/columns: [describe]

## Implementation Plan
1. [ ] Write API tests (Red)
2. [ ] Implement API endpoint (Green)
3. [ ] Write Swift tests (Red)
4. [ ] Implement Swift code (Green)
5. [ ] Integration test
6. [ ] Commit and push
```

After writing the spec, use ExitPlanMode to present it for user approval.

**Do not proceed to implementation until the user approves the plan.**

### Step 4: TDD Red Phase - Write Failing Tests

**For API (if applicable):**
1. Create `services/api/tests/test_<feature>.py`
2. Write test for the new endpoint
3. Run `uv run pytest` - confirm it FAILS (Red)

**For Swift (if applicable):**
1. Add test in `apps/macos-client/Tests/`
2. Write test for new functionality
3. Run `swift test` - confirm it FAILS (Red)

### Step 5: TDD Green Phase - Implement

Implement minimum code to make tests pass:

**API:**
1. Add endpoint to `services/api/app/main.py` or create router
2. Run `uv run pytest` - should PASS (Green)

**Swift:**
1. Add code to `apps/macos-client/Sources/`
2. Run `swift test` - should PASS (Green)

### Step 6: Refactor (Optional)

Clean up code while keeping tests green.

### Step 7: Create PR

Ask user: "All tests pass. Ready to create a PR?"

If yes, use `gh` CLI to create PRs (never use GitHub web interface):

```bash
# Create feature branch, commit, and open PR for each repo
cd apps/macos-client
git checkout -b feature/<feature-name>
git add .
git commit -m "feat: <description>"
git push -u origin feature/<feature-name>
gh pr create --title "feat: <description>" --body "Implements <feature>"

cd ../../services/api
git checkout -b feature/<feature-name>
git add .
git commit -m "feat: <description>"
git push -u origin feature/<feature-name>
gh pr create --title "feat: <description>" --body "Implements <feature>"
```

Update spec with PR links and completion status.
