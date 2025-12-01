---
layout: post
title: "Playwright AI Agents: Testing on Autopilot with Claude Code"
date: 2025-12-01 10:00:00 +0100
categories: [ai-assisted-development, testing]
tags: [playwright, ai, testing, automation, claude, vscode]
---

Playwright just changed the game for test automation. With the introduction of AI agents in version 1.56, you can now have specialized AI assistants write, debug, and fix your tests automatically. In this guide, I'll show you how to set up Playwright's AI agents with Claude Code and VS Code.

## What Are Playwright Agents?

Think of Playwright agents as a team of specialized QA engineers built into your development workflow. Instead of one general-purpose AI trying to do everything, you get three focused agents, each with a specific job:

**The Planner** - Your test strategist. It explores your application like a manual QA engineer would, clicking around, understanding user flows, and creating detailed test plans in plain Markdown.

**The Generator** - Your test developer. It takes those Markdown plans and writes actual Playwright test code, verifying that every selector works as it goes.

**The Healer** - Your maintenance engineer. When tests break (and they will), it automatically debugs them, updates selectors, adds missing waits, and gets things working again.

## Why This Matters

Traditional AI test generation gives you generic code that might work once. Playwright agents give you:

- **Context-aware tests** that understand your app's structure
- **Self-healing tests** that adapt when your UI changes
- **Collaborative planning** with Markdown specs that non-technical stakeholders can review
- **Customizable workflows** since agents are just Markdown files you can edit

## Prerequisites

You'll need:
- Node.js installed
- Playwright installed in your project
- **VS Code v1.105 or later** (released October 2025) for the full agentic experience
- Claude Code CLI or Claude Desktop

## Getting Started with Claude Code

### Step 1: Initialize the Agents

In your Playwright project, run:

```bash
npx playwright init-agents --loop=claude
```

This creates a `.github/` folder with three agent definition files. These are Markdown files that define how each agent behaves—think of them as specialized instruction manuals for Claude.

**Pro tip**: Run this command again whenever Playwright updates to get access to new tools and capabilities.

### Step 2: Understand Your Project Structure

After initialization, you'll have:

```
your-project/
├── .github/              # Agent definitions (customizable!)
│   ├── planner.md
│   ├── generator.md
│   └── healer.md
├── specs/                # Human-readable test plans
├── tests/
│   └── seed.spec.ts      # Your testing baseline
└── playwright.config.ts
```

### Step 3: Create Your Seed File

Here's where **seed.spec.ts** comes in. This file is your testing template and environment setup rolled into one.

**What is seed.spec.ts?**

The seed file serves two critical purposes:

1. **It runs your setup code** - global setup, project dependencies, fixtures, and hooks that need to execute before tests
2. **It's an example** - the Planner uses it as a reference for code style, patterns, and conventions

Think of it as showing the AI: "This is how we write tests in this project."

Here's a simple seed.spec.ts:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Application seed', () => {
  test.beforeEach(async ({ page }) => {
    // Any setup that needs to run before tests
    await page.goto('/');
  });

  test('basic navigation works', async ({ page }) => {
    // Example test showing your preferred patterns
    await expect(page.getByRole('heading')).toBeVisible();
    await page.getByRole('link', { name: 'About' }).click();
    await expect(page).toHaveURL(/.*about/);
  });
});
```

You can leave it blank, but including example tests helps the agents understand your project better.

## The Agent Workflow

### 1. Planning Phase

Talk to the Planner agent in Claude Code:

```
Create a test plan for user registration including:
- Form validation
- Successful registration flow
- Duplicate email handling
- Password strength requirements
```

The Planner will:
- Navigate your application
- Identify the UI elements involved
- Create a detailed Markdown test plan in `specs/`

Example output (`specs/user-registration.md`):

```markdown
# User Registration Tests

## Scenario 1: Successful Registration
1. Navigate to /register
2. Fill email field with valid email
3. Fill password field with strong password
4. Fill confirm password field with matching password
5. Click "Create Account" button
6. Expect redirect to /dashboard
7. Expect welcome message to be visible

## Scenario 2: Password Validation
1. Navigate to /register
2. Fill email field with valid email
3. Fill password field with "weak"
4. Expect error message: "Password must be at least 8 characters"
```

### 2. Generation Phase

The Planner automatically hands off to the Generator, which:
- Reads the Markdown plan
- Writes actual Playwright code
- **Verifies selectors live** against your running app
- Saves tests to `tests/`

Generated test (`tests/user-registration.spec.ts`):

```typescript
import { test, expect } from '@playwright/test';

test.describe('User Registration', () => {
  test('should successfully register with valid credentials', async ({ page }) => {
    await page.goto('/register');

    await page.getByLabel('Email').fill('newuser@example.com');
    await page.getByLabel('Password').fill('StrongPass123!');
    await page.getByLabel('Confirm Password').fill('StrongPass123!');

    await page.getByRole('button', { name: 'Create Account' }).click();

    await expect(page).toHaveURL(/dashboard/);
    await expect(page.getByText('Welcome')).toBeVisible();
  });

  test('should reject weak password', async ({ page }) => {
    await page.goto('/register');

    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('weak');

    await expect(page.getByText('Password must be at least 8 characters'))
      .toBeVisible();
  });
});
```

### 3. Healing Phase

Six months later, your UI changes. The "Create Account" button is now "Sign Up". Your tests fail.

To fix them, start a new chat in Claude Code and ask the Healer agent:

```
Fix the failing user registration test
```

The Healer agent:
1. Runs the test in debug mode
2. Inspects page snapshots and console logs
3. Identifies that the button text changed
4. Updates the test automatically
5. Reruns until it passes

**Updated code**:

```typescript
// Changed automatically by Healer
await page.getByRole('button', { name: 'Sign Up' }).click();
```

If the Healer can't fix it (maybe the feature is actually broken), it marks the test as skipped and notifies you.

## Using with VS Code

If you prefer VS Code over the Claude Code CLI:

```bash
npx playwright init-agents --loop=vscode
```

This integrates the agents directly into your VS Code workflow. Make sure you're running **VS Code v1.105+** for full agent support.

With VS Code, you can:
- Trigger agents from the command palette
- See test plans and generated code side-by-side
- Review agent changes in your normal diff viewer

## Customizing the Agents

The real power is customization. The agent definitions are just Markdown files:

```markdown
# .github/planner.md

You are the Planner agent for Playwright tests.

## Your role
Explore the application and create test plans.

## Custom rules for this project
- Always test mobile viewports first
- Include accessibility checks in every plan
- Never use CSS selectors, only semantic locators
- Check for loading states explicitly
```

Edit these files to match your team's standards, coding conventions, and testing philosophy.

## My Workflow

1. **Run the Planner** with high-level requirements
2. **Review the Markdown spec** with product/QA team
3. **Let the Generator write the code**
4. **Run tests** and let the Healer handle initial issues
5. **Commit everything** - specs, tests, and seed files

## Challenges and Tips

**Challenge**: Agents generate tests that are too generic

**Solution**: Improve your seed.spec.ts with detailed examples showing your preferred patterns

**Challenge**: The Healer keeps fixing the same test differently

**Solution**: The functionality might actually be broken. Review the changes manually and fix the root cause

**Challenge**: Test plans miss edge cases

**Solution**: Be specific in your prompts. List the edge cases you care about explicitly

## When to Use AI Agents vs. Manual Testing

AI agents are amazing for:
- Regression test suites
- Happy path flows
- UI component testing
- Test maintenance at scale

Write tests manually for:
- Complex business logic
- Security-critical flows
- Tests requiring deep domain knowledge
- One-off debugging scenarios

## Real-World Results

In my projects, Playwright agents have:
- Reduced test writing time by 60%
- Cut test maintenance effort by 40%
- Improved test coverage because it's now "cheap" to add tests
- Made test specs reviewable by non-technical team members

## Getting Help

If you run into issues:
- Check the [Playwright Agents docs](https://playwright.dev/docs/test-agents)
- Review the generated `.github/` agent files
- Regenerate agents after Playwright updates
- Ensure you're on VS Code v1.105+ if using VS Code

## Conclusion

Playwright's AI agents aren't replacing QA engineers—they're making them more effective. You still need to understand testing principles, define what to test, and review the results. But the tedious parts (writing boilerplate, maintaining selectors, debugging flaky tests) are now automated.

Start simple:
1. Initialize agents in an existing project
2. Create a detailed seed.spec.ts
3. Have the Planner generate one test suite
4. See how it goes

The three-agent approach (plan, generate, heal) is more reliable than generic AI test generation because each agent is specialized and can be customized to your needs.

---

*Questions about Playwright agents or AI-assisted testing? Connect with me on [LinkedIn](https://linkedin.com/in/marcinwernecki) or check out my [GitHub](https://github.com/ch0nkster) for more testing frameworks and examples.*
