# Codex Agent Workflow Reference

Detailed workflows for common coding scenarios.

## Workflow 1: New Feature Development

### When to use
Building a new feature that requires multiple files/components.

### Steps

1. **Check existing work**
   ```bash
   codex-status --project myapp
   ```

2. **Start feature**
   ```bash
   codex-do --project myapp "Implement user authentication:
   - Login form component with email/password validation
   - Auth context for state management
   - API integration for /login and /logout
   - Store JWT in localStorage
   - Add auth guards to protected routes"
   ```

3. **Monitor**
   ```bash
   codex-status --project myapp --follow
   ```

4. **Review changes**
   ```bash
   cd /home/dan/zoidcode/myapp
   git status
   git diff
   ```

5. **Test locally**
   ```bash
   npm run dev  # or equivalent
   ```

6. **Commit**
   ```bash
   git add .
   git commit -m "feat: add user authentication
   
   - Login form with validation
   - Auth context and state management
   - JWT storage and API integration
   - Route protection guards"
   ```

## Workflow 2: Bug Fix

### When to use
Fixing a reported bug requiring investigation and multi-file changes.

### Steps

1. **Start with context**
   ```bash
   codex-do --project myapp "Fix issue: Login button not working on mobile.
   
   Context:
   - Reported on iOS Safari
   - Button appears but doesn't trigger click handler
   - Works fine on desktop Chrome
   
   Steps:
   1. Find the LoginButton component
   2. Check for touch event handling issues
   3. Check CSS for pointer-events or z-index problems
   4. Fix and test the solution"
   ```

2. **Review output**
   ```bash
   codex-status --project myapp --output
   ```

3. **Verify fix**
   Test on actual device or browser dev tools.

4. **Commit**
   ```bash
   git commit -m "fix: resolve mobile login button touch issue
   
   - Add proper touch event handling
   - Fix z-index stacking context
   Fixes #123"
   ```

## Workflow 3: Code Review with Codex

### When to use
Want Codex to review code before committing.

### Steps

1. **Stage changes first**
   ```bash
   cd ~/clawd-team/agents/bender/workspace/myapp
   git add .
   ```

2. **Ask for review**
   ```bash
   codex-do --project myapp "Review the staged changes:
   - Check for security issues
   - Look for performance problems
   - Verify error handling
   - Check TypeScript types
   
   Output a summary of findings and suggestions."
   ```

3. **Apply suggestions**
   Either manually or with another codex-do pass.

## Workflow 4: Refactoring

### When to use
Restructuring code without changing behavior.

### Steps

1. **Document current state**
   ```bash
   git status
   git log --oneline -5
   ```

2. **Start refactor**
   ```bash
   codex-do --project myapp "Refactor the auth module:
   - Extract API calls to separate service file
   - Rename unclear variable names
   - Add JSDoc comments to public functions
   - Ensure all tests still pass
   
   Do NOT change any behavior, only structure and naming."
   ```

3. **Verify nothing broke**
   ```bash
   npm test
   ```

## Workflow 5: Resume After Interruption

### When to use
Session timed out, you killed it, or Codex asked questions.

### Steps

1. **Check what happened**
   ```bash
   codex-status --project myapp --output
   ```

2. **Answer question or continue**
   ```bash
   # If Codex asked a question:
   codex-resume --project myapp "Answer: Yes, use the approach you suggested."
   
   # If session timed out:
   codex-resume --project myapp "Continue from where you left off. The task was to implement the user dashboard."
   ```

## Workflow 6: Parallel Projects

### When to use
Working on multiple projects simultaneously.

### Example

Terminal 1 - Backend:
```bash
codex-do --project api "Add user endpoints"
```

Terminal 2 - Frontend:
```bash
codex-do --project web "Build user profile page"
```

Each has independent session tracking.

## Workflow 7: New Project from Scratch

### When to use
Starting a completely new codebase.

### Steps

1. **Create project via Mission Control first**
   ```bash
   ~/clawd-team/clawd project create newproject "New Project" "Description"
   # Note the FOLDER value from the output
   ```

2. **Scaffold with Codex**
   ```bash
   codex-do --project newproject "Create a new Node.js Express API:
   - Initialize package.json
   - Setup Express server with basic middleware
   - Add folder structure: src/routes, src/models, src/middleware
   - Create health check endpoint
   - Add Jest for testing
   - Setup ESLint and Prettier
   - Create README with setup instructions"
   ```

3. **Commit locally**
   ```bash
   cd /home/dan/zoidcode/newproject
   git add .
   git commit -m "Initial setup"
   ```

   (GitHub remote setup will come later when available.)

## Common Patterns

### Pattern: Iterative Refinement

When the first pass isn't perfect:

```bash
# First pass
codex-do --project myapp "Build a todo list"

# Review and refine
codex-resume --project myapp "The todo list works but needs:
- Add due dates
- Add priority levels
- Add filtering by status"

# Polish
codex-resume --project myapp "Almost done! Just need to:
- Fix the checkbox alignment
- Add hover effects
- Ensure it works on mobile"
```

### Pattern: Test-Driven with Codex

```bash
# Write tests first
codex-do --project myapp "Write Jest tests for a calculateTotal function:
- Should sum array of prices
- Should apply 10% discount over $100
- Should handle empty array
- Should throw on negative prices"

# Then implement
codex-resume --project myapp "Now implement the calculateTotal function to pass all those tests."
```

### Pattern: Split Large Tasks

For very large features, split into multiple sessions:

```bash
# Step 1: Data layer
codex-do --project myapp "Create database schema and models for user profiles"

# Step 2: API layer
codex-do --project myapp "Create REST API endpoints for profile CRUD operations"

# Step 3: UI layer
codex-do --project myapp "Build React components for profile display and editing"
```

## Anti-Patterns to Avoid

### ❌ The Kitchen Sink

Don't dump everything into one prompt:
```bash
# Bad - too much at once
codex-do --project myapp "Build a complete social media app with posts, comments, likes, DMs, stories, and live streaming"
```

Instead, break it down:
```bash
# Good - focused
codex-do --project myapp "Create the Post model and /api/posts endpoints"
```

### ❌ Vague Instructions

Don't be unclear about what you want:
```bash
# Bad - unclear
codex-do --project myapp "Make it better"
```

Instead, specify:
```bash
# Good - clear acceptance criteria
codex-do --project myapp "Improve the loading performance:
- Add skeleton screens while data loads
- Implement React.lazy for route code splitting
- Add loading states to all async buttons"
```

### ❌ No Context

Don't assume Codex knows the codebase:
```bash
# Bad - no context
codex-do --project myapp "Fix the bug"
```

Instead, provide context:
```bash
# Good - context included
codex-do --project myapp "Fix the navigation bug:
- Currently clicking 'Dashboard' doesn't navigate
- The NavLink component is in src/components/NavLink.tsx
- It should use react-router's useNavigate hook"
```

## Tips for Success

1. **Start small**: First session should be achievable in 10-15 minutes
2. **Be specific**: Include file names, function names, expected behavior
3. **Provide examples**: Show input/output examples for complex logic
4. **Iterate**: Resume and refine rather than starting over
5. **Review always**: Never commit without reviewing Codex's changes
6. **Test locally**: Run the code before pushing
7. **Commit often**: Small commits are easier to review and revert
