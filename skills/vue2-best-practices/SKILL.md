---
name: vue2-best-practices
description: Core Vue 2 patterns, reactivity system, Options API, lifecycle hooks, and common gotchas. Use for Vue 2 projects, Options API implementation, Vue 2 component design, and legacy Vue.js applications.
license: MIT
metadata:
  author: github.com/krisQT
  version: "1.1.0"
---

# Vue 2 Best Practices

## When to Use

Use this skill when:

- Working with Vue 2 projects (not Vue 3)
- Using Options API
- Need to understand Vue 2 reactivity system
- Building or maintaining Vue 2 applications

Do not use this skill when working with Vue 3 (use vue-best-practices instead).

## Activation Triggers

Use this skill when the user prompt contains any of these patterns:

- "create a Vue 2 component"
- "working with Vue 2"
- "Vue 2 Options API"
- "Vue 2 reactivity"
- "Vue 2 lifecycle"
- "build a Vue 2 app"
- "Vue 2 data function"
- "Vue 2 computed property"
- "Vue 2 watcher"
- "Vue 2 event handling"
- "Vue 2 two-way binding"
- "Vue 2 form handling"
- "Vue 2 async data"
- "Vue 2 performance"
- "Vue 2 best practices"
- "migrating from Vue 2"
- "legacy Vue project"
- "Vue 2.x"
- "Vue 2"

Keywords that indicate Vue 2 context: `Vue.version === '2.x'`, `vue2`, `options-api`

## Core Concepts

### Options API Structure

Vue 2 uses Options API with these key options:

```javascript
export default {
  name: 'MyComponent',
  
  // Data
  data() {
    return {
      message: 'Hello',
      count: 0
    }
  },
  
  // Properties
  props: {
    title: String,
    initialCount: {
      type: Number,
      default: 0
    }
  },
  
  // Computed
  computed: {
    doubleCount() {
      return this.count * 2
    }
  },
  
  // Watchers
  watch: {
    count(newVal, oldVal) {
      console.log(`Count changed from ${oldVal} to ${newVal}`)
    }
  },
  
  // Methods
  methods: {
    increment() {
      this.count++
    }
  },
  
  // Lifecycle Hooks
  created() {
    console.log('Component created')
  },
  
  mounted() {
    console.log('Component mounted')
  }
}
```

### Vue 2 Reactivity Caveats

Vue 2's reactivity system has specific limitations:

#### DO: Use Vue.set() for Adding Properties

```javascript
// ❌ BAD - Won't be reactive
this.user.email = 'test@example.com'

// ✅ GOOD - Will be reactive
this.$set(this.user, 'email', 'test@example.com')
// Or: Vue.set(this.user, 'email', 'test@example.com')
```

#### DO: Use Vue.delete() for Removing Properties

```javascript
// ❌ BAD - Won't trigger reactivity
delete this.user.email

// ✅ GOOD - Will trigger reactivity
this.$delete(this.user, 'email')
// Or: Vue.delete(this.user, 'email')
```

#### DO: Use Array Methods for Modifying Arrays

```javascript
// ❌ BAD - Won't trigger reactivity
this.items[0] = { id: 1, name: 'Updated' }

// ✅ GOOD - These will trigger reactivity
this.$set(this.items, 0, { id: 1, name: 'Updated' })
this.items.splice(0, 1, { id: 1, name: 'Updated' })
```

### Data Function

Always use a function for `data` in components (except in root Vue instance):

```javascript
export default {
  data() {
    return {
      count: 0,
      user: null
    }
  }
}
```

The data function ensures each instance has isolated data:

```javascript
// In a component
data() {
  return {
    message: 'instance specific'
  }
}

// NOT
data: {
  message: 'shared across all instances'  // ❌ WRONG in components
}
```

### Lifecycle Hooks

Understanding lifecycle hooks is crucial:

1. **beforeCreate**: Called synchronously immediately after instance initialization
   - No access to data, computed, methods, or template
   - Good for: initializing non-reactive properties

2. **created**: Called after instance is created
   - Can access data, computed, methods
   - No access to DOM ($el not available)
   - Good for: API calls, setting up listeners

3. **beforeMount**: Called right before mounting begins
   - Template compilation completed
   - Good for: last-minute preparations

4. **mounted**: Instance is mounted
   - Can access DOM elements via this.$el, this.$refs
   - Good for: DOM manipulations, third-party library initialization
   - Note: Not guaranteed to be in document - use $nextTick

5. **beforeUpdate**: Called when reactive data changes and re-render needed
   - Good for: accessing DOM state before update

6. **updated**: Called after DOM update
   - Use $nextTick for batch DOM operations
   - Good for: DOM-dependent operations

7. **beforeDestroy**: Called before instance is destroyed
   - Good for: cleanup (remove listeners, timers, etc.)

8. **destroyed**: Called after instance is destroyed
   - Good for: post-destruction cleanup

### Computed Properties

Computed properties are cached based on their dependencies:

```javascript
computed: {
  fullName() {
    return `${this.firstName} ${this.lastName}`
  },
  
  // With getter and setter
  fullNameWithSetter: {
    get() {
      return `${this.firstName} ${this.lastName}`
    },
    set(value) {
      const [first, last] = value.split(' ')
      this.firstName = first
      this.lastName = last
    }
  }
}
```

Use computed for:
- Derived state from other state
- Expensive calculations
- Filtering/sorting lists

### Watchers

Use watchers for:
- Async operations
- Performing side effects
- Watching nested properties

```javascript
watch: {
  // Simple
  count(newVal, oldVal) {
    console.log(`Count changed: ${oldVal} -> ${newVal}`)
  },
  
  // Deep watcher
  user: {
    handler(newVal, oldVal) {
      console.log('User changed')
    },
    deep: true
  },
  
  // Immediate
  message: {
    handler(newVal) {
      this.processMessage(newVal)
    },
    immediate: true
  }
}
```

## Common Patterns

### Handling Async Data

```javascript
export default {
  data() {
    return {
      users: [],
      loading: false,
      error: null
    }
  },
  
  created() {
    this.fetchUsers()
  },
  
  methods: {
    async fetchUsers() {
      this.loading = true
      this.error = null
      
      try {
        const response = await fetch('/api/users')
        this.users = await response.json()
      } catch (err) {
        this.error = err.message
      } finally {
        this.loading = false
      }
    }
  }
}
```

### Event Handling

```javascript
methods: {
  handleClick(event) {
    console.log('Clicked!', event)
  },
  
  handleInput(event) {
    this.searchQuery = event.target.value
  },
  
  // With event modifier
  handleSubmit() {
    // Form handling
  }
}
```

```html
<button @click="handleClick">Click</button>
<input @input="handleInput" />
<form @submit.prevent="handleSubmit">...</form>
```

### Two-Way Binding

```html
<!-- Text input -->
<input v-model="message" />

<!-- Checkbox -->
<input type="checkbox" v-model="checked" />

<!-- Multiple checkboxes -->
<input type="checkbox" value="A" v-model="selected" />
<input type="checkbox" value="B" v-model="selected" />

<!-- Select -->
<select v-model="selected">
  <option value="a">A</option>
  <option value="b">B</option>
</select>
```

### v-model with Custom Components

```javascript
// Parent
<custom-input v-model="searchQuery" />

// Child
export default {
  props: {
    value: String
  },
  methods: {
    updateValue(event) {
      this.$emit('input', event.target.value)
    }
  }
}
```

## Performance Tips

1. **Use `v-show` for Frequent Toggles**
   ```html
   <div v-show="isVisible">...</div>  <!-- Keeps in DOM, toggles display -->
   <div v-if="isVisible">...</div>   <!-- Removes from DOM -->
   ```

2. **Use `key` for List Re-rendering**
   ```html
   <div v-for="item in items" :key="item.id">...</div>
   ```

3. **Lazy Load Components**
   ```javascript
   components: {
     HeavyComponent: () => import('./HeavyComponent.vue')
   }
   ```

4. **Use Functional Components for Simple Components**
   ```javascript
   export default {
     functional: true,
     render(h, { props }) {
       return h('div', props.message)
     }
   }
   ```

5. **Debounce Watchers for Search**
   ```javascript
   watch: {
     searchQuery: {
       handler() {
         this.search()
       },
       debounce: 300  // Note: Requires lodash-decorators or similar
     }
   }
   ```

## Anti-Patterns to Avoid

1. ❌ **Don't use arrow functions for component options**
   ```javascript
   // ❌ WRONG
   export default {
     data: () => ({ count: 0 }),  // this won't be correct
     methods: {
       increment: () => { this.count++ }  // this won't be correct
     }
   }
   
   // ✅ CORRECT
   export default {
     data() {
       return { count: 0 }
     },
     methods: {
       increment() {
         this.count++
       }
     }
   }
   ```

2. ❌ **Don't modify props directly**
   ```javascript
   props: ['initialCount'],
   mounted() {
     // ❌ WRONG
     this.initialCount = 5
     
     // ✅ CORRECT - Use local data
     this.localCount = this.initialCount
   }
   ```

3. ❌ **Don't use v-for without :key**
   ```html
   <!-- ❌ WRONG -->
   <div v-for="item in items">{{ item.name }}</div>
   
   <!-- ✅ CORRECT -->
   <div v-for="item in items" :key="item.id">{{ item.name }}</div>
   ```

4. ❌ **Don't use $parent/$children**
   ```javascript
   // ❌ WRONG
   this.$parent.doSomething()
   
   // ✅ CORRECT - Use props and events
   this.$emit('do-something')
   ```

## Vue 2 vs Vue 3 Key Differences

| Feature | Vue 2 | Vue 3 |
|---------|-------|-------|
| API Style | Options API (default) | Composition API (default) |
| Reactivity | Object.defineProperty | Proxies |
| Data | Must use function in components | Can use plain object |
| Multiple v-models | Not supported | Multiple v-model support |
| Fragments | Not supported | Supported |
| Teleport | Not available | Built-in |
| Suspense | Not available | Built-in |
| Composition | Mixins | Composables (setup function) |

## References

See the `references/` directory for more detailed information on:

- Vue 2 Reactivity in Depth
- Vue 2 Lifecycle Diagram
- Common Gotchas and Solutions
