# Vue2-best-practices for Claude Code / Trae IDE

## Project Info

- **Repository**: https://github.com/krisQT/vue2-best-practices
- **Version**: 1.1.0
- **Vue Version**: Vue 2.x

## Installation

### Option 1: NPM (Recommended)

```bash
npx skills add krisQT/vue2-best-practices
```

### Option 2: Claude Code Marketplace

```bash
/plugin marketplace add krisQT/vue2-best-practices
/plugin install vue2-skills-bundle@vue2-best-practices
```

### Option 3: Local Development

```bash
git clone https://github.com/krisQT/vue2-best-practices.git
# Then symlink skills to your project:
ln -s /path/to/vue2-best-practices/skills /your-project/.skills/vue2-best-practices
```

## Usage

When working with Vue 2 projects, activate relevant skills using:

```
Use vue2 skill, <your prompt here>
```

This project provides specialized knowledge for Vue 2 development, focusing on:

- **Options API** - Vue 2 component structure
- **Reactivity System** - Vue 2 limitations and workarounds
- **Vuex** - State management patterns
- **Vue Router 3** - Routing and navigation guards
- **Testing** - Jest and Vue Test Utils
- **Debugging** - Common issues and solutions

## Available Skills

| Skill | Keyword | Description |
|-------|---------|-------------|
| vue2-best-practices | Vue 2, Options API | Core Vue 2 patterns |
| vue2-options-api | this context | Deep dive into Options API |
| vue2-components | props, slots | Component design |
| vue2-router | Vue Router | Navigation guards |
| vue2-vuex | Vuex | State management |
| vue2-mixins | mixins | Code sharing |
| vue2-directives | directives | Custom directives |
| vue2-testing | Jest, testing | Testing practices |
| vue2-debug-guides | debug | Debugging guides |

## Version Compatibility

| Feature | Vue Version | Note |
|---------|------------|------|
| `.sync` modifier | 2.3.0+ | Two-way binding |
| `emits` option | 2.3.0+ | Event declarations |
| `model` option | 2.2.0+ | Custom v-model |
| `inheritAttrs` | 2.4.0+ | Attribute inheritance |

## Quick Examples

### Create a Vue 2 Component

```
Use vue2 skill, create a user profile component with props and events
```

### Debug Reactivity Issues

```
Use vue2 skill, why isn't my array update triggering re-render?
```

### Setup Vuex Store

```
Use vue2 skill, create an authentication module with Vuex
```

## Documentation

- [README.md](README.md) - Project overview
- [USAGE_GUIDE.md](USAGE_GUIDE.md) - Detailed usage guide
- [AGENTS.md](AGENTS.md) - AI agent guidelines
- [CHANGELOG.md](CHANGELOG.md) - Version history

## Related Projects

- [vuejs-ai/skills](https://github.com/vuejs-ai/skills) - Vue 3 official skills
- [vuejs/vue2](https://github.com/vuejs/vue) - Vue 2 source code
