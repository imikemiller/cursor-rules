---
description: Writing manual QA test scripts for new features
alwaysApply: false
---

# QA Test Script Standards

Write test scripts that are clear, reproducible, and focused on observable outcomes. Follow this format strictly.

## Test Script Template

```markdown
### Test Case: [FEATURE]-[NUMBER] - [Descriptive Name]

**Test ID**: [FEATURE]-[NUMBER]  
**Test Name**: [Short descriptive name]

---

**Prerequisites**:

- [User state, permissions, role - one per line]
- [Required test data or fixtures]
- [Environment configuration if relevant]

**Setup Scripts** (if required):

\`\`\`bash

# Commands to prepare test environment

\`\`\`

**Starting URL**: [Full URL with test parameters]

---

**Test Steps**:

1. [Action to perform]
2. [Next action]
3. [Continue sequentially]

---

**Expected UI State**:

- ✅ **[Element Name]**: [Observable state - visible/hidden, enabled/disabled, contains text]
- ✅ **[Element Name]**: [Observable state]
- ❌ **[Element Name]**: [Should NOT be present/visible]

---

**Expected Result**:

[Concrete, observable outcome in 1-2 sentences. Describe what the user sees, not what the system does internally.]

---

**Screenshots**:

<screenshot:qa/[feature]/screenshots/[FEATURE]-[NUMBER]_[step]-[element].png|[Description of what to capture]>
<screenshot:qa/[feature]/screenshots/[FEATURE]-[NUMBER]_[step]-[element].png|[Description of what to capture]>
```

## Writing Guidelines

### Test Identity

- **Test ID**: Use feature prefix and sequential number (e.g., `ACL-001`, `AUTH-012`, `DASH-003`)
- **Test Name**: Brief, describes what's being verified — not the steps

### Prerequisites

State everything needed before step 1 begins:

```markdown
**Prerequisites**:

- Logged in as Organisation Admin (`org:admin` role)
- Test organisation "Acme Corp" exists with at least 3 members
- Navigate to organisation settings page

**Setup Scripts**:

\`\`\`bash

# Seed test organisation

pnpm db:seed --fixture=acme-org

# Start mock server (if Mode: Mock)

pnpm mock:server --scenario=org-admin
\`\`\`

**Starting URL**: http://localhost:5173/org/acme-corp/settings
```

**Do not** include permission logic or conditional behaviour in prerequisites — different permissions require separate test cases.

### Test Steps

- Numbered, sequential actions
- One action per step
- Use precise UI language: "Click", "Enter", "Select", "Scroll to", "Hover over"
- Include specific values where relevant

```markdown
**Test Steps**:

1. Click "Add Member" button in the header
2. Enter "jane@example.com" in the email field
3. Select "Editor" from the role dropdown
4. Click "Send Invitation"
```

### Expected UI State

Document the observable state after completing all steps:

- Use ✅ for elements that **should** be present/visible/enabled
- Use ❌ for elements that **should NOT** be present
- Describe only what is observable — no internal logic or permission reasoning
- One element per line

```markdown
**Expected UI State**:

- ✅ **Success Toast**: "Invitation sent to jane@example.com"
- ✅ **Pending Members Table**: Contains row for "jane@example.com" with status "Pending"
- ✅ **Resend Button**: Visible and enabled in the jane@example.com row
- ❌ **Add Member Modal**: Should be closed
```

**Do not** include:

- Permission explanations ("visible because user has X permission")
- Conditional logic ("enabled when X OR Y")
- Internal state references

### Expected Result

A concrete statement of the observable outcome. Describe what the user sees.

```markdown
# ❌ BAD: Describes internal logic

**Expected Result**: The invitation system respects rate limits and permission boundaries while updating the UI state accordingly.

# ❌ BAD: Restates the steps

**Expected Result**: User can add a member and see them in the pending list.

# ✅ GOOD: Observable outcome

**Expected Result**: The pending members table displays "jane@example.com" with status "Pending" and a "Resend" action. The success toast confirms the invitation was sent.
```

### Screenshots

Use the placeholder format for searchability:

```
<screenshot:qa/[feature]/screenshots/[TEST-ID]_[step-number]-[element].png|[What to capture]>
```

Examples:

```markdown
**Screenshots**:

<screenshot:qa/acl/screenshots/ACL-001_03-acl-card-full.png|Full ACL card section showing header and member list>
<screenshot:qa/acl/screenshots/ACL-001_03-action-buttons.png|Close-up of Add Members and Approve buttons in card header>
<screenshot:qa/acl/screenshots/ACL-001_03-member-row.png|Single member row showing Remove and Replace actions>
```

Screenshots should capture:

- The specific UI state being verified
- Sufficient context to identify location in the app
- Any error states or edge cases

## One Test Case Per Scenario

Each test case should verify **one scenario**. Different permissions, states, or user types require separate test cases:

```markdown
# ❌ BAD: Multiple scenarios in one test

### Test Case: ACL-001 - ACL Visibility

Tests that admins can see all controls and viewers can only see the list...

# ✅ GOOD: Separate test cases

### Test Case: ACL-001 - Admin Can View All ACL Controls

### Test Case: ACL-002 - Viewer Can Only See Member List

### Test Case: ACL-003 - Admin Cannot Modify During Pending Changes
```

## File Organisation

Test scripts are organised by feature and mode. **Each script file is either mock or full — never mixed.**

```
qa/
├── acl/
│   ├── admin-access.mock.md
│   ├── admin-access.full.md
│   ├── viewer-permissions.mock.md
│   └── screenshots/
│       ├── ACL-001_03-acl-card-full.png
│       ├── ACL-001_03-action-buttons.png
│       └── ACL-002_05-readonly-view.png
├── auth/
│   ├── login-flow.mock.md
│   ├── login-flow.full.md
│   ├── mfa-setup.full.md
│   └── screenshots/
│       └── AUTH-001_02-login-form.png
└── dashboard/
    ├── widget-rendering.mock.md
    └── screenshots/
```

### Naming Convention

- **Script files**: `qa/[feature]/[descriptive-name].[mock|full].md`
- **Screenshots**: `qa/[feature]/screenshots/[TEST-ID]_[step]-[element].png`

### Mock vs Full

- **`.mock.md`** — Uses mocked APIs/services. Specify which services are mocked in prerequisites.
- **`.full.md`** — Requires live backend services. Specify environment and test credentials in prerequisites.

A single script file contains only mock OR full tests. If a feature needs both, create separate files:

```
qa/acl/admin-access.mock.md   # Tests against mocked API
qa/acl/admin-access.full.md   # Tests against staging/live
```

### Screenshot Paths

Screenshot placeholders should reference the feature's screenshot directory:

```markdown
<screenshot:qa/acl/screenshots/ACL-001_03-acl-card.png|Full ACL card section>
```

## Summary Checklist

Before submitting a test script, verify:

- [ ] File is saved to `qa/[feature]/[name].[mock|full].md`
- [ ] File contains only mock OR full tests — not mixed
- [ ] Test ID follows `[FEATURE]-[NUMBER]` format
- [ ] Prerequisites contain all setup — no assumptions
- [ ] Setup scripts are copy-pasteable
- [ ] Steps are numbered and atomic
- [ ] Expected UI State describes observables only — no permission logic
- [ ] Expected Result is concrete and observable
- [ ] Screenshot placeholders use `<screenshot:qa/[feature]/screenshots/[TEST-ID]_[step]-[element].png|description>` format
- [ ] One scenario per test case
