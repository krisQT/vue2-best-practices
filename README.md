# Vue2-best-practices

Agent skills for Vue 2 development.

🚧 Early Experiment / Community Project

This repository is an early experiment in creating specialized skills for AI agents to enhance Vue 2 development. Skills are derived from real-world issues and patterns commonly found in Vue 2 legacy projects.

## ⚠️ Prerequisites

- **Node.js**: v16.0.0 or higher recommended (for CLI installation)
- **Git**: Required for cloning

> **Note**: If you're using an older Node.js version (like v14), please use the **Local Development Mode** method below instead of the `npx` command.

## Installation

### Method 1: Local Development Mode (Recommended - No Node.js version issues)

This method copies the skills directly to your project without using the `skills` CLI tool.

**Step 1**: Clone or download this repository

```bash
# Option A: Clone to a separate directory
git clone https://github.com/krisQT/vue2-best-practices.git ~/vue2-skills

# Option B: Download and extract ZIP from GitHub
# Visit: https://github.com/krisQT/vue2-best-practices/releases
```

**Step 2**: Copy skills to your project

```bash
# In your Vue 2 project root directory
mkdir -p .skills

# Copy the skills folder
cp -r ~/vue2-skills/skills .skills/vue2-best-practices

# Or create a symlink (changes sync automatically)
ln -s ~/vue2-skills/skills .skills/vue2-best-practices
```

**Step 3**: Verify installation

```bash
ls -la .skills/vue2-best-practices/
# You should see all 9 skill folders
```

### Method 2: Using npm (Requires Node.js 16+)

If you have Node.js v16 or higher, you can use the skills CLI:

```bash
npx skills add krisQT/vue2-best-practices
```

**If you encounter `SyntaxError: Unexpected token '{'` error**, it means your Node.js version is too old. Please use **Method 1** instead.

### Method 3: Claude Code Marketplace

For Claude Code users:

```bash
# Add marketplace
/plugin marketplace add krisQT/vue2-best-practices

# Install all skills at once
/plugin install vue2-skills-bundle@vue2-best-practices

# Install individual skills
/plugin install vue2-best-practices@vue2-best-practices

# Install multiple skills
/plugin install vue2-best-practices@vue2-best-practices vue2-router-best-practices@vue2-best-practices
```

## Usage

### For Trae IDE / Claude Code

**Option A**: Explicit trigger (most reliable)

```
Use vue2 skill, <your prompt here>

Example:
Use vue2 skill, 帮我创建一个用户列表组件
```

**Option B**: Automatic detection

Trae will automatically detect Vue 2 context based on keywords in your prompt:

- "Vue 2" → vue2-best-practices
- "Vue Router" → vue2-router-best-practices
- "Vuex" → vue2-vuex-best-practices
- And more...

### Example Prompts

```
# Create a Vue 2 component
Use vue2 skill, create a user profile component with props and events

# Debug reactivity issues
Use vue2 skill, why isn't my array update triggering re-render?

# Setup Vuex
Use vue2 skill, create an authentication module with Vuex

# Route guards
Use vue2 skill, implement a login guard for routes
```

## Available Skills

| Skill | When to use | Description |
|-------|-------------|-------------|
| [vue2-best-practices](./skills/vue2-best-practices) | Vue 2 + Options API | Core Vue 2 patterns, reactivity, lifecycle hooks, common gotchas |
| [vue2-options-api-best-practices](./skills/vue2-options-api-best-practices) | Options API (data, methods, computed) | `this` context, lifecycle, proper Options API usage |
| [vue2-components-best-practices](./skills/vue2-components-best-practices) | Component design | Component communication, props, events, slots |
| [vue2-router-best-practices](./skills/vue2-router-best-practices) | Vue Router 3 | Navigation guards, route params, route-component lifecycle |
| [vue2-vuex-best-practices](./skills/vue2-vuex-best-practices) | Vuex for state management | Store setup, mutations, actions, getters, modules |
| [vue2-mixins-best-practices](./skills/vue2-mixins-best-practices) | Mixins usage | Mixin patterns, conflicts resolution, composition |
| [vue2-directives-best-practices](./skills/vue2-directives-best-practices) | Custom directives | Directive creation, hooks, argument modifiers |
| [vue2-testing-best-practices](./skills/vue2-testing-best-practices) | Writing component or E2E tests | Jest, Vue Test Utils, Nightwatch |
| [vue2-debug-guides](./skills/vue2-debug-guides) | Debugging Vue 2 issues | Runtime errors, warnings, Vue DevTools usage |

## Examples

### vue2-best-practices

Demo - Todo App

Prompt:
```
Use vue2 skill, create a todo app with Vue 2
```

Changes after using skill:

- More readable code
- Proper reactive data initialization
- Correct lifecycle hook usage
- Optimized watchers

### vue2-vuex-best-practices

Demo - User Authentication Store

Prompt:
```
Use vue2 skill, create a user authentication store with Vuex
```

Changes after using skill:

- Proper Vuex module structure
- Correct mutation and action patterns
- State hydration from localStorage
- Proper getters usage

## Documentation

- [USAGE_GUIDE.md](./USAGE_GUIDE.md) - Detailed usage guide
- [AGENTS.md](./AGENTS.md) - AI agent guidelines
- [IMPLEMENTATION.md](./IMPLEMENTATION.md) - Implementation details
- [CHANGELOG.md](./CHANGELOG.md) - Version history

## Methodology

### Skill Types

Skills are classified into two categories:

- **Capability**: AI cannot solve the problem without the skill. These address version-specific issues, undocumented behaviors, Vue 2 specific patterns, or edge cases outside AI's training data.

- **Efficiency**: AI can solve the problem but not well. These provide optimal patterns, best practices, and consistent approaches that improve solution quality.

### Validation Process

Each skill rule is validated through automated evals:

- **Baseline**: Run without skill installed
- **With-skill**: Run with skill installed

A rule is kept only if it enables the model to solve problems it couldn't solve without it.

| Baseline | With Skill | Action |
|----------|------------|--------|
| Fail | Pass | Keep |
| Pass | Pass | Considered removed |

## Troubleshooting

### Error: `SyntaxError: Unexpected token '{'`

**Cause**: Your Node.js version is too old for the skills CLI.

**Solution**: Use **Method 1: Local Development Mode** instead.

```bash
# 1. Clone the repository
git clone https://github.com/krisQT/vue2-best-practices.git ~/vue2-skills

# 2. Copy to your project
cd your-vue2-project
mkdir -p .skills
cp -r ~/vue2-skills/skills .skills/vue2-best-practices
```

### Error: `skills: command not found`

**Solution**: The skills CLI isn't installed globally. This is fine - just use local development mode.

## Contributing

Development happens on the `dev` branch. The `main` branch is reserved for published skills only.

1. Create a feature branch from `dev`
2. Submit a PR to `dev`
3. After approval, changes are merged to `dev`
4. Maintainers sync `dev` → `main` via GitHub Action when ready to publish

## Related Projects

- [vuejs-ai/skills](https://github.com/vuejs-ai/skills) - Vue 3 official skills
- [vuejs/vue2](https://github.com/vuejs/vue) - Vue 2 source code
- [vuejs/docs-next](https://github.com/vuejs/docs-next) - Vue 2 documentation

## License

MIT
