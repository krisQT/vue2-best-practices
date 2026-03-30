# Vue2-best-practices

Agent skills for Vue 2 development.

🚧 Early Experiment / Community Project

This repository is an early experiment in creating specialized skills for AI agents to enhance Vue 2 development. Skills are derived from real-world issues and patterns commonly found in Vue 2 legacy projects.

## Installation

Using npm:

```bash
npx skills add your-username/vue2-best-practices
```

Using Claude Code Marketplace:

```bash
# Add marketplace
/plugin marketplace add your-username/vue2-best-practices

# Install all skills at once
/plugin install vue2-skills-bundle@vue2-best-practices

# Install individual skills
/plugin install vue2-best-practices@vue2-best-practices

# Install multiple skills
/plugin install vue2-best-practices@vue2-best-practices vue2-router-best-practices@vue2-best-practices
```

## Usage

For most reliable results, prefix your prompt with `use vue2 skill`:

```
Use vue2 skill, <your prompt here>
```

This explicitly triggers the skill and ensures the AI follows the documented patterns. Without the prefix, skill triggering may be inconsistent depending on how closely your prompt matches the skill's description keywords.

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
create a todo app with Vue 2
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
create a user authentication store with Vuex
```

Changes after using skill:

- Proper Vuex module structure
- Correct mutation and action patterns
- State hydration from localStorage
- Proper getters usage

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
