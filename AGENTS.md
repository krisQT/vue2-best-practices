# Vue2-best-practices Agent Guidelines

This document outlines the guidelines and patterns for AI agents working with Vue 2 projects using these skills.

## Project Overview

Vue2-best-practices is a collection of specialized skills designed to help AI agents understand and work effectively with Vue 2 codebase. Vue 2 has different patterns and APIs compared to Vue 3, making these skills essential for maintaining and developing Vue 2 legacy applications.

**Repository**: https://github.com/krisQT/vue2-best-practices

## Quick Start

### Installation

```bash
# 方式一：使用 npm（推荐）
npx skills add krisQT/vue2-best-practices

# 方式二：使用 Claude Code Marketplace
/plugin marketplace add krisQT/vue2-best-practices
/plugin install vue2-skills-bundle@vue2-best-practices

# 方式三：本地开发模式
git clone https://github.com/krisQT/vue2-best-practices.git
ln -s /path/to/vue2-best-practices/skills /your-project/.skills/vue2-best-practices
```

### Usage

Prefix your prompt with `use vue2 skill` for best results:

```
Use vue2 skill, <your prompt here>
```

## Core Principles

### 1. Vue 2 Specific Patterns

When working with Vue 2:

- Use Options API by default (`data`, `methods`, `computed`, `watch`)
- Prefer `Vue.observable()` for simple state management
- Use `Vuex` for complex state management
- Understand Vue 2 lifecycle hooks (beforeCreate, created, beforeMount, mounted, beforeUpdate, updated, beforeDestroy, destroyed)
- Use `v-bind` and `v-on` modifiers appropriately

### 2. Reactivity System

Vue 2 uses a different reactivity system than Vue 3:

- Avoid adding new properties to reactive objects - use `Vue.set()` or `this.$set()`
- Arrays: avoid directly setting array indexes - use array methods or `Vue.set()`
- Object property addition/deletion requires `Vue.set()` or `Vue.delete()`
- Watch for reactivity caveats with nested objects

### 3. Component Patterns

Key component patterns for Vue 2:

- Use `props` with type validation
- Use `$emit()` for parent-child communication
- Use `provide/inject` for deep component tree communication
- Use `v-model` with `.sync` modifier for two-way binding
- Avoid `$parent` and `$children` - use props and events
- Use `ref` for accessing DOM elements

### 4. Vue Router 3

Vue Router 3 (for Vue 2) has specific patterns:

- Navigation guards: `beforeRouteEnter`, `beforeRouteUpdate`, `beforeRouteLeave`
- Route params and query parameters
- Scroll behavior control
- Lazy loading with dynamic imports

### 5. Vuex Patterns

Vuex in Vue 2 projects:

- Module-based architecture
- Strict mode for development
- Namespaced modules
- Action vs Mutation distinction
- Plugin system (persistence, logging)
- HMR support

### 6. Common Gotchas

Be aware of these Vue 2 specific issues:

- `this` context in callbacks and async functions
- Component naming (PascalCase vs kebab-case)
- Functional components vs stateful components
- Scoped CSS vs CSS modules
- Async components and dynamic imports
- Template expressions limitations

## Skill Categories

### Core Skills

1. **vue2-best-practices**: General Vue 2 patterns and best practices
2. **vue2-options-api-best-practices**: Deep dive into Options API
3. **vue2-components-best-practices**: Component design and communication

### Ecosystem Skills

4. **vue2-router-best-practices**: Vue Router 3 patterns
5. **vue2-vuex-best-practices**: Vuex state management
6. **vue2-mixins-best-practices**: Mixin patterns
7. **vue2-directives-best-practices**: Custom directives

### Quality Assurance

8. **vue2-testing-best-practices**: Testing strategies
9. **vue2-debug-guides**: Debugging and troubleshooting

## When to Use Each Skill

### Starting a New Vue 2 Project

Use: `vue2-best-practices`, `vue2-components-best-practices`

- Establish proper project structure
- Set up component architecture
- Configure Vuex and Vue Router

### Maintaining Legacy Code

Use: `vue2-debug-guides`, `vue2-options-api-best-practices`

- Understand existing patterns
- Debug issues effectively
- Make safe modifications

### Adding New Features

Use: `vue2-best-practices`, relevant ecosystem skill

- Follow existing patterns
- Maintain consistency
- Apply best practices

### Migration Planning

Use: All skills for understanding Vue 2 patterns

- Document current patterns
- Understand Vue 2 specifics
- Prepare for Vue 3 migration

## Best Practices for Using Skills

1. **Read SKILL.md files**: Each skill directory contains detailed guidelines
2. **Check references/**: Reference materials provide deeper context
3. **Follow patterns**: Maintain consistency with existing codebase
4. **Consider Vue 2 specifics**: Always think about Vue 2 vs Vue 3 differences
5. **Test changes**: Vue 2 has different behavior than Vue 3

## Anti-Patterns to Avoid

### In Vue 2 Code

- ❌ Adding properties directly to reactive objects
- ❌ Using array[index] = value
- ❌ Using arrow functions for component methods
- ❌ Accessing parent component via $parent/$children
- ❌ Using Vue 3 Composition API patterns in Vue 2

### In AI Generation

- ❌ Generating Vue 3 patterns without Vue 2 equivalents
- ❌ Ignoring Vue 2 reactivity caveats
- ❌ Using Options API incorrectly (wrong lifecycle hooks, etc.)
- ❌ Generating code without considering Vue 2 compatibility

## Validation

Each skill has been validated through automated testing to ensure it provides value in:
- Solving Vue 2 specific problems
- Improving code quality
- Reducing common mistakes
- Maintaining consistency

## Updates and Maintenance

These skills are community-maintained and updated based on:
- Real-world usage feedback
- Vue 2 documentation updates
- New patterns discovered in legacy codebases
- Community contributions

Last updated: 2024
