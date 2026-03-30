---
name: vue2-options-api-best-practices
description: Deep dive into Vue 2 Options API including data(), methods, computed, watch, and this context. Essential for proper component structure and lifecycle management.
license: MIT
metadata:
  author: github.com/krisQT
  version: "1.1.0"
---

# Vue 2 Options API Best Practices

## When to Use

Use this skill when:

- Working with Vue 2 Options API
- Understanding `this` context
- Structuring component options correctly
- Using lifecycle hooks properly

## Activation Triggers

Use this skill when the user prompt contains any of these patterns:

- "this context in Vue"
- "arrow function Vue 2"
- "Options API structure"
- "Vue 2 component options"
- "data function Vue"
- "computed vs method"
- "watch vs computed"
- "Vue 2 lifecycle hooks"
- "beforeCreate created"
- "mounted updated"
- "beforeDestroy destroyed"
- "Vue 2 prop validation"
- "prop types Vue"
- "Vue 2 watch deep"
- "immediate watcher"
- "method naming Vue"
- "Vue options order"
- "Option merge Vue"
- "this undefined Vue"

Keywords: `this.$options`, `export default {`, `data()`, `methods:`, `computed:`, `watch:`

## The `this` Context

Understanding `this` in Vue 2 is crucial:

### Arrow Functions Do NOT Work

```javascript
// ❌ WRONG
export default {
  data: () => ({
    count: 0
  }),
  
  methods: {
    increment: () => {
      this.count++  // this is NOT the component instance!
    }
  },
  
  mounted: () => {
    console.log(this)  // Window or undefined!
  }
}

// ✅ CORRECT
export default {
  data() {
    return {
      count: 0
    }
  },
  
  methods: {
    increment() {
      this.count++  // this is the component instance
    }
  },
  
  mounted() {
    console.log(this)  // Component instance
  }
}
```

### Why Arrow Functions Don't Work

Vue 2 options are invoked with `.call(this)` to set the correct context. Arrow functions don't have their own `this`, so they inherit from the parent scope.

### Async Functions and `this`

```javascript
export default {
  methods: {
    async fetchData() {
      // Arrow function preserves this context
      const response = await api.get()
      this.data = response.data  // this is correct
      
      // Alternative: use .bind(this) or assign self
      const self = this
      api.get().then(function(response) {
        self.data = response.data
      })
    }
  }
}
```

## Data Function

### Always Use Function in Components

```javascript
// ❌ WRONG - Shared across all instances
export default {
  data: {
    count: 0  // All instances share this!
  }
}

// ✅ CORRECT - Each instance gets own copy
export default {
  data() {
    return {
      count: 0
    }
  }
}
```

### Root Vue Instance Exception

```javascript
// In main.js - root Vue instance
new Vue({
  data: {
    // Can use object (not function) for root
    count: 0
  }
})
```

### Return All Reactive Properties

```javascript
// ❌ WRONG - Adding properties later won't be reactive
export default {
  data() {
    return {
      count: 0
    }
  },
  created() {
    this.user = { name: 'John' }  // Not reactive!
  }
}

// ✅ CORRECT - Initialize all properties
export default {
  data() {
    return {
      count: 0,
      user: null  // Initialize with null
    }
  },
  created() {
    this.fetchUser()
  },
  methods: {
    async fetchUser() {
      this.user = await api.getUser()
    }
  }
}
```

## Props

### Prop Validation

```javascript
export default {
  props: {
    // Basic type check
    title: String,
    
    // Multiple types
    id: [String, Number],
    
    // Required
    name: {
      type: String,
      required: true
    },
    
    // Default value
    initialCount: {
      type: Number,
      default: 0
    },
    
    // Object/array defaults must be factory functions
    user: {
      type: Object,
      default() {
        return { name: 'Anonymous' }
      }
    },
    
    items: {
      type: Array,
      default() {
        return []
      }
    },
    
    // Custom validator
    status: {
      type: String,
      validator(value) {
        return ['pending', 'active', 'completed'].includes(value)
      }
    },
    
    // Custom default (function)
    callback: {
      type: Function,
      default() {
        return () => {}
      }
    }
  }
}
```

### Prop Types

Available prop types:

- `String`
- `Number`
- `Boolean`
- `Array`
- `Object`
- `Date`
- `Function`
- `Symbol`

### Prop Casing

```html
<!-- In template: kebab-case -->
<user-card :user-name="name" :is-active="active" />

<!-- In component: camelCase (auto-converted) -->
export default {
  props: {
    userName: String,
    isActive: Boolean
  }
}
```

## Computed Properties

### Basic Computed

```javascript
export default {
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe'
    }
  },
  
  computed: {
    fullName() {
      return `${this.firstName} ${this.lastName}`
    },
    
    // Computed with get/set
    fullNameWithSet: {
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
}
```

### Computed vs Methods

| Feature | Computed | Methods |
|---------|----------|---------|
| Cached | Yes (based on dependencies) | No (runs every call) |
| Reactive | Yes | Yes (when called reactively) |
| Use in templates | Yes | Yes |
| Performance | Better for expensive operations | Runs every time |

### Computed Setter Pattern

```javascript
computed: {
  // Read-only (default)
  fullName() {
    return `${this.firstName} ${this.lastName}`
  },
  
  // With setter
  fullName: {
    get() {
      return `${this.firstName} ${this.lastName}`
    },
    set(value) {
      const [first, ...rest] = value.split(' ')
      this.firstName = first
      this.lastName = rest.join(' ')
    }
  }
}
```

## Watchers

### Simple Watcher

```javascript
export default {
  data() {
    return {
      query: ''
    }
  },
  
  watch: {
    query(newQuery) {
      this.search(newQuery)
    }
  }
}
```

### Deep Watcher

```javascript
watch: {
  user: {
    handler(newUser) {
      this.saveUser(newUser)
    },
    deep: true  // Watches nested properties
  }
}
```

### Immediate Watcher

```javascript
watch: {
  query: {
    handler() {
      this.search()
    },
    immediate: true  // Runs on component creation
  }
}
```

### Watch with Old Value

```javascript
watch: {
  items(newItems, oldItems) {
    // oldItems is only available for arrays/objects
    // For primitives, old value is undefined on first call
    console.log('Changed:', oldItems, '->', newItems)
  }
}
```

### Async Watcher

```javascript
watch: {
  query: {
    handler() {
      this.debouncedSearch()
    },
    immediate: true
  }
}
```

## Methods

### Method Organization

```javascript
export default {
  // 1. Public methods (used in template)
  methods: {
    handleClick() {
      this.increment()
    },
    
    // 2. Public methods for external use
    reset() {
      this.count = 0
    },
    
    // 3. Private methods (by convention)
    increment() {
      this.count++
      this.persistCount()
    },
    
    persistCount() {
      localStorage.setItem('count', this.count)
    }
  }
}
```

### Method Naming Conventions

```javascript
export default {
  methods: {
    // Event handlers: handle<EventName>
    handleClick() {},
    handleInput() {},
    handleSubmit() {},
    
    // Async actions: load/fetch/update + Noun
    fetchUsers() {},
    updateUser() {},
    deleteItem() {},
    
    // Computed-like: get<Noun>
    getUserById() {},
    getTotalPrice() {},
    
    // Toggle/set methods
    toggleSidebar() {},
    setActiveTab() {},
    showModal() {},
    hideModal() {}
  }
}
```

## Lifecycle Hooks in Options

### Complete Lifecycle Order

```javascript
export default {
  name: 'MyComponent',
  
  // Parent-child relationships
  parent: null,  // Usually not needed
  
  // Component registration
  components: {
    ChildComponent,
    'kebab-case-name': Component
  },
  
  // Directives
  directives: {
    focus: { ... }
  },
  
  // Filters
  filters: {
    currency: function(value) { ... }
  },
  
  // Mixins
  mixins: [baseMixin],
  
  // Props
  props: {},
  
  // Data
  data() { return {} },
  
  // Computed
  computed: {},
  
  // Watch
  watch: {},
  
  // Lifecycle hooks
  beforeCreate() {},
  created() {},
  beforeMount() {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {},
  
  // Keep-alive hooks
  activated() {},
  deactivated() {},
  
  // Error handling
  errorCaptured() {},
  
  // Render
  render() {},
  renderError() {}
}
```

## Options Order

Vue 2 style guide recommends this order:

```javascript
export default {
  name: '',
  
  parent: null,
  
  inheritAttrs: true,
  
  model: { prop: 'value', event: 'input' },  // Vue 2.2+
  
  props: {},
  
  emits: [],  // Vue 2.3+
  
  setup() {},  // Vue 2.6+ (rarely used in Vue 2)
  
  data() {},
  
  computed: {},
  
  watch: {},
  
  provide() {},
  inject: [],
  
  methods: {},
  
  beforeCreate() {},
  created() {},
  beforeMount() {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  activated() {},
  deactivated() {},
  beforeDestroy() {},
  destroyed() {},
  
  errorCaptured() {},
  
  render(h) {},
  
  components: {},
  directives: {},
  filters: {}
}
```

### Version Notes for Options

| Option | Vue Version | Description |
|--------|-------------|-------------|
| `model` | 2.2.0+ | Custom v-model prop and event |
| `emits` | 2.3.0+ | Declare emitted events |
| `inheritAttrs` | 2.4.0+ | Control attribute inheritance |
| `functional` | 2.2.0+ | Functional component option |
| `setup` | 2.6.0+ | Composition API entry (rare in Vue 2) |
```

## Common Patterns

### Computed vs Method vs Watch

```javascript
export default {
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe',
      fullNameInput: ''
    }
  },
  
  computed: {
    // Use computed for: derived state
    fullName() {
      return `${this.firstName} ${this.lastName}`
    },
    
    // Computed with set
    fullNameWithSet: {
      get() {
        return `${this.firstName} ${this.lastName}`
      },
      set(value) {
        const parts = value.split(' ')
        this.firstName = parts[0]
        this.lastName = parts.slice(1).join(' ')
      }
    }
  },
  
  methods: {
    // Use methods for: actions, events
    updateFullName() {
      this.fullNameWithSet = this.fullNameInput
    },
    
    // Methods called from computed (helper methods)
    formatName(name) {
      return name.trim().toLowerCase()
    }
  },
  
  watch: {
    // Use watch for: side effects, async operations
    firstName() {
      this.saveToDatabase()
    }
  }
}
```

### Props, Data, Computed, Methods

```javascript
export default {
  props: {
    initialCount: {
      type: Number,
      default: 0
    }
  },
  
  data() {
    return {
      // Initialize data from props
      count: this.initialCount
    }
  },
  
  computed: {
    // Derived state
    isPositive() {
      return this.count > 0
    },
    
    // Conditional class
    buttonClass() {
      return {
        'btn-primary': this.isPositive,
        'btn-danger': this.count < 0,
        'btn-disabled': this.count === 0
      }
    }
  },
  
  methods: {
    increment() {
      this.count++
    },
    
    decrement() {
      this.count--
    },
    
    reset() {
      this.count = this.initialCount
    }
  }
}
```

## Anti-Patterns

### 1. Don't Use Arrow Functions

```javascript
// ❌ WRONG
methods: {
  getData: () => {
    return this.data  // this is undefined!
  }
}

// ✅ CORRECT
methods: {
  getData() {
    return this.data  // this is component instance
  }
}
```

### 2. Don't Mutate Props

```javascript
// ❌ WRONG
props: ['initialCount'],
mounted() {
  this.initialCount = 5  // Modifying prop!
}

// ✅ CORRECT
props: ['initialCount'],
data() {
  return {
    count: this.initialCount  // Copy to data
  }
}
```

### 3. Don't Put Side Effects in Computed

```javascript
// ❌ WRONG
computed: {
  fullName() {
    this.saveToDatabase(this.firstName)  // Side effect!
    return `${this.firstName} ${this.lastName}`
  }
}

// ✅ CORRECT
methods: {
  updateName() {
    this.saveToDatabase(this.fullName)
  }
}
```

### 4. Don't Use Methods When Computed Is Better

```javascript
// ❌ WRONG - Recalculates every call
<template>
  <div>{{ getTotal() }}</div>
</template>

methods: {
  getTotal() {
    return this.items.reduce((sum, item) => sum + item.price, 0)
  }
}

// ✅ CORRECT - Cached and reactive
computed: {
  total() {
    return this.items.reduce((sum, item) => sum + item.price, 0)
  }
}
```

## References

See the `references/` directory for:

- Complete Options API reference
- this context deep dive
- Property type validation
