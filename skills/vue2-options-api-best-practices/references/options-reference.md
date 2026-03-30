# Vue 2 Options API Reference

## Component Options

### name

```javascript
export default {
  name: 'MyComponent',
  // Used in Vue DevTools
  // Used for recursive components
  // Used for component matching in keep-alive
}
```

### components

```javascript
import ChildComponent from './Child.vue'

export default {
  components: {
    // Key: local name, Value: component
    ChildComponent,          // <child-component />
    'special-name': ChildComponent  // <special-name />
  }
}
```

### props

```javascript
export default {
  props: {
    // Basic
    title: String,
    count: Number,
    isActive: Boolean,
    items: Array,
    user: Object,
    
    // Required
    requiredProp: {
      type: String,
      required: true
    },
    
    // Default
    initialCount: {
      type: Number,
      default: 0
    },
    
    // Object/Array default (factory function)
    defaultItems: {
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
    
    // Multiple types
    mixed: [String, Number]
  }
}
```

### data

```javascript
export default {
  data() {
    return {
      message: 'Hello',
      count: 0,
      user: null,
      items: []
    }
  }
}

// ❌ Wrong - in components, must be function
export default {
  data: {
    message: 'Hello'  // Shared across instances!
  }
}

// ✅ Correct - function returns fresh object
export default {
  data() {
    return { message: 'Hello' }
  }
}
```

### computed

```javascript
export default {
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe'
    }
  },
  
  computed: {
    // Getter only
    fullName() {
      return `${this.firstName} ${this.lastName}`
    },
    
    // With getter and setter
    fullNameWithSet: {
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
}
```

### methods

```javascript
export default {
  methods: {
    handleClick() {
      console.log('Clicked!')
    },
    
    async fetchData() {
      try {
        const response = await api.get()
        this.data = response
      } catch (error) {
        this.error = error.message
      }
    }
  }
}
```

### watch

```javascript
export default {
  data() {
    return {
      searchQuery: '',
      user: { name: '' }
    }
  },
  
  watch: {
    // Simple watcher
    searchQuery(newVal, oldVal) {
      this.search(newVal)
    },
    
    // Deep watcher
    user: {
      handler(newVal, oldVal) {
        this.saveUser(newVal)
      },
      deep: true
    },
    
    // Immediate (runs on creation)
    searchQuery: {
      handler() {
        this.search()
      },
      immediate: true
    }
  }
}
```

### Lifecycle Hooks

```javascript
export default {
  beforeCreate() {
    // Instance initialized
    // No access to data, props, etc.
  },
  
  created() {
    // Instance created with reactivity
    // Good for: data fetching, setting up
  },
  
  beforeMount() {
    // Template compiled
    // About to mount
  },
  
  mounted() {
    // DOM ready
    // Good for: DOM manipulation, third-party init
  },
  
  beforeUpdate() {
    // Before re-render
    // DOM not updated yet
  },
  
  updated() {
    // After re-render
    // Use $nextTick for DOM operations
  },
  
  beforeDestroy() {
    // Before destruction
    // Good for: cleanup
  },
  
  destroyed() {
    // After destruction
    // Good for: final cleanup
  }
}
```

### emits

```javascript
export default {
  emits: ['click', 'update', 'delete'],
  
  // Or with validation
  emits: {
    click: null,
    update: (value) => {
      return typeof value === 'string'
    }
  }
}
```

### provide/inject

```javascript
// Parent component
export default {
  provide() {
    return {
      theme: 'dark',
      user: () => this.currentUser  // Function for reactivity
    }
  }
}

// Child component
export default {
  inject: ['theme', 'user'],
  
  computed: {
    currentUserData() {
      return this.user()  // Call the function
    }
  }
}
```

### model (Vue 2.2+)

```javascript
export default {
  model: {
    prop: 'value',
    event: 'change'
  },
  
  props: {
    value: String
  },
  
  methods: {
    updateValue(newValue) {
      this.$emit('change', newValue)
    }
  }
}
```

### inheritAttrs (Vue 2.4+)

```javascript
export default {
  inheritAttrs: false,  // Don't inherit attrs as props
  
  props: ['value', 'placeholder'],
  
  mounted() {
    // Access attrs
    console.log(this.$attrs)
  }
}
```

### functional

```javascript
export default {
  functional: true,
  // No instance, no this
  // Receive props and context
  render(h, { props, data, parent }) {
    return h('div', data, props.text)
  }
}
```

### render

```javascript
export default {
  render(h) {
    return h('div', {
      class: 'container',
      attrs: { id: 'app' },
      on: { click: this.handleClick }
    }, [
      h('h1', 'Hello'),
      h('p', this.message)
    ])
  }
}
```

### directives

```javascript
export default {
  directives: {
    focus: {
      inserted(el) {
        el.focus()
      }
    },
    'click-outside': {
      bind(el, binding) {
        // Implementation
      }
    }
  }
}
```

### filters

```javascript
export default {
  filters: {
    capitalize(value) {
      if (!value) return ''
      return value.charAt(0).toUpperCase() + value.slice(1)
    },
    
    currency(value, currency = 'USD') {
      return new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency
      }).format(value)
    }
  }
}
```

### mixins

```javascript
import formatter from '@/mixins/formatter'
import validator from '@/mixins/validator'

export default {
  mixins: [formatter, validator]
  // Options merge automatically
}
```

## Option Merging

| Option | Merge Strategy |
|--------|---------------|
| data | Deep merge, component wins |
| methods | Component wins |
| computed | Component wins |
| watch | Both called |
| lifecycle | Both called (mixin first) |
| props | Merged into array |
| inject | Merged |
| directives | Merged |
| components | Merged |

## Vue 2 vs Vue 3 Options

| Vue 2 | Vue 3 |
|-------|-------|
| Options API | Composition API |
| data() | setup() |
| this | reactive refs |
| mixins | composables |
| $refs | ref() |
| $parent | provide/inject |
| filters | methods or pipe |
| $listeners | attrs |
