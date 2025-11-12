---
layout: post
title: "Getting Started with MCP for Test Automation"
date: 2025-01-12 10:00:00 +0100
categories: [ai-assisted-development, testing]
tags: [mcp, playwright, ai, testing, automation]
---

Model Context Protocol (MCP) is changing how we approach test automation. In this post, I'll share my experience integrating MCP with Playwright and how it can accelerate your test development workflow.

## What is MCP?

Model Context Protocol is an open protocol that standardizes how applications provide context to Large Language Models. For test automation, this means AI assistants can:

- Access your project structure and existing tests
- Understand your testing patterns and conventions
- Generate tests that follow your project's style
- Maintain context across development sessions

## Why MCP for Testing?

Traditional test generation tools are rigid and produce generic code. With MCP-enabled AI assistants:

**Speed**: Generate comprehensive test suites in minutes instead of hours
**Consistency**: AI follows your project-specific rules and patterns
**Maintenance**: Update tests across your suite with natural language instructions
**Learning**: New team members can learn your patterns from AI-generated examples

## Getting Started

### 1. Set Up Your MCP-Enabled Tool

I use **Cursor IDE** and **Claude Code CLI**. Both support MCP natively:

```bash
# Claude Code CLI
npm install -g @anthropic/claude-code
```

### 2. Create MCP Rules for Your Project

Document your testing patterns in a `docs/MCP_RULES.md` file:

```markdown
# Test Generation Rules

## R1: File Naming
- Test files MUST end with `.spec.ts`
- Use descriptive names: `feature-name.spec.ts`

## R2: Locator Strategy Priority
1. User-facing attributes (getByRole, getByLabel)
2. Test IDs (getByTestId)
3. CSS selectors (last resort)

## R3: Test Structure
- Use describe blocks for grouping
- Use beforeEach for setup
- Use "should" statements for test names
```

### 3. Generate Your First Test

With MCP context, you can prompt:

```
Generate comprehensive login tests following the patterns in tests/auth/
and rules in docs/MCP_RULES.md. Include:
- Valid/invalid credentials
- Form validation
- Error handling
- Session management
```

The AI will generate tests that match your existing code style and follow your documented patterns.

## Real Example

Here's a test generated using MCP for a login flow:

```typescript
import { test, expect } from '@playwright/test';

test.describe('User Login', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('should successfully login with valid credentials', async ({ page }) => {
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('ValidPass123!');
    await page.getByRole('button', { name: 'Login' }).click();

    await expect(page).toHaveURL(/dashboard/);
    await expect(page.getByText('Welcome back')).toBeVisible();
  });

  test('should display error for invalid credentials', async ({ page }) => {
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('WrongPassword');
    await page.getByRole('button', { name: 'Login' }).click();

    await expect(page.getByTestId('error-message')).toBeVisible();
    await expect(page.getByTestId('error-message'))
      .toContainText('Invalid credentials');
  });
});
```

Notice how it:
- Uses semantic locators (getByLabel, getByRole)
- Follows describe/test structure
- Includes proper assertions
- Has descriptive test names

## My Workflow

1. **Define requirements** in natural language
2. **Reference existing patterns** in the prompt
3. **Review generated tests** for edge cases
4. **Run and iterate** if needed

## Challenges and Solutions

**Challenge**: AI generates tests that don't match your style
**Solution**: Create detailed MCP rules and reference them in prompts

**Challenge**: Tests are too generic
**Solution**: Provide specific requirements and edge cases upfront

**Challenge**: AI misses edge cases
**Solution**: Review generated code and iterate with follow-up prompts

## Resources

Check out my open-source framework demonstrating MCP integration:
[Playwright AI Testing Framework](https://github.com/ch0nkster/playwright-ai-testing)

It includes:
- Complete MCP documentation
- Sample AI-generated tests
- Best practices and patterns
- Ready-to-use templates

## Conclusion

MCP is a game-changer for test automation. It doesn't replace your expertise—it amplifies it. You still need to understand testing principles, but you can execute faster and maintain consistency more easily.

Start small: document your patterns, try generating a simple test suite, and iterate. The investment in MCP setup pays dividends as your test suite grows.

---

*Have questions about MCP or Playwright? Connect with me on [LinkedIn](https://linkedin.com/in/marcinwernecki) or check out my [GitHub](https://github.com/ch0nkster).*
