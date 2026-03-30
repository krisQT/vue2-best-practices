# Vue 2 Mixins Best Practices

## When to Use

Use this skill when:

- Working with Vue 2 mixins
- Sharing code between components
- Understanding mixin conflicts
- Implementing reusable logic patterns

## Activation Triggers

Use this skill when the user prompt contains any of these patterns:

- "Vue mixin"
- "mixins Vue"
- "share code Vue"
- "mixin conflict"
- "global mixin"
- "local mixin"
- "mixin merge"
- "option merge"
- "reusable logic Vue"
- "cross-cutting concerns"
- "mixin vs composable"
- "Vue 2 code sharing"
- "component reuse"
- "mixin options"
- "form mixin"
- "API mixin"
- "validation mixin"
- "scroll mixin"

Keywords: `mixins:`, `Vue.mixin(`, `mixin`, `merge strategy`

## What are Mixins

Mixins allow you to create reusable pieces of component logic. When a component uses a mixin, all options from the mixin are "mixed in" with the component's own options.

## Basic Mixin

```javascript
// mixins/formatter.js
export default {
  methods: {
    formatDate(date) {
      return new Date(date).toLocaleDateString()
    },
    
    formatCurrency(amount) {
      return new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD'
      }).format(amount)
    }
  }
}

// Using in component
import formatter from '@/mixins/formatter'

export default {
  mixins: [formatter],
  
  methods: {
    displayPrice() {
      return this.formatCurrency(100)  // $100.00
    }
  }
}
```

## Global Mixin

```javascript
// main.js
import Vue from 'vue'
import myMixin from '@/mixins/myMixin'

// Global mixin - applied to ALL components
Vue.mixin({
  methods: {
    log(message) {
      console.log(`[${this.$options.name}]:`, message)
    }
  }
})

// Use sparingly - global mixins affect all components
```

## Mixin Options Merging

When mixins and component have same options:

| Option | Merge Strategy |
|--------|----------------|
| `data` | Deep merge (component data takes precedence) |
| `methods` | Component method overrides mixin method |
| `computed` | Component computed overrides mixin computed |
| `watch` | Both are called |
| `lifecycle` | Both are called (mixin first) |

### Data Merging

```javascript
// mixin
export default {
  data() {
    return {
      shared: 'mixin value',
      mixinOnly: 'mixin'
    }
  }
}

// component
export default {
  mixins: [mixin],
  data() {
    return {
      shared: 'component value',  // Component takes precedence
      componentOnly: 'component'
    }
  }
}

// Result:
// shared: 'component value'
// mixinOnly: 'mixin'
// componentOnly: 'component'
```

### Lifecycle Hooks

```javascript
// mixin
export default {
  created() {
    console.log('Mixin created')
  }
}

// component
export default {
  mixins: [mixin],
  created() {
    console.log('Component created')
  }
}

// Output:
// "Mixin created"
// "Component created"
```

## Common Mixin Patterns

### 1. Form Validation

```javascript
// mixins/formValidation.js
export default {
  data() {
    return {
      errors: {},
      touched: {}
    }
  },
  
  methods: {
    validateField(field, rules) {
      const value = this.form[field]
      
      if (rules.required && !value) {
        this.$set(this.errors, field, 'This field is required')
        return false
      }
      
      if (rules.minLength && value.length < rules.minLength) {
        this.$set(this.errors, field, `Minimum ${rules.minLength} characters`)
        return false
      }
      
      if (rules.email && !this.isValidEmail(value)) {
        this.$set(this.errors, field, 'Invalid email')
        return false
      }
      
      this.$delete(this.errors, field)
      return true
    },
    
    isValidEmail(email) {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
    },
    
    touchField(field) {
      this.$set(this.touched, field, true)
    },
    
    resetValidation() {
      this.errors = {}
      this.touched = {}
    }
  }
}

// Usage
export default {
  mixins: [formValidation],
  
  data() {
    return {
      form: {
        email: '',
        name: ''
      }
    }
  },
  
  methods: {
    submitForm() {
      const emailValid = this.validateField('email', {
        required: true,
        email: true
      })
      const nameValid = this.validateField('name', {
        required: true,
        minLength: 2
      })
      
      if (emailValid && nameValid) {
        this.sendForm()
      }
    }
  }
}
```

### 2. Scroll Detection

```javascript
// mixins/scrollDetector.js
export default {
  data() {
    return {
      isScrolled: false,
      scrollY: 0
    }
  },
  
  mounted() {
    window.addEventListener('scroll', this.handleScroll)
  },
  
  beforeDestroy() {
    window.removeEventListener('scroll', this.handleScroll)
  },
  
  methods: {
    handleScroll() {
      this.scrollY = window.scrollY
      this.isScrolled = this.scrollY > 50
    }
  }
}
```

### 3. Media Query Detection

```javascript
// mixins/responsive.js
export default {
  data() {
    return {
      windowWidth: window.innerWidth,
      windowHeight: window.innerHeight
    }
  },
  
  computed: {
    isMobile() {
      return this.windowWidth < 768
    },
    
    isTablet() {
      return this.windowWidth >= 768 && this.windowWidth < 1024
    },
    
    isDesktop() {
      return this.windowWidth >= 1024
    }
  },
  
  mounted() {
    window.addEventListener('resize', this.handleResize)
  },
  
  beforeDestroy() {
    window.removeEventListener('resize', this.handleResize)
  },
  
  methods: {
    handleResize() {
      this.windowWidth = window.innerWidth
      this.windowHeight = window.innerHeight
    }
  }
}
```

### 4. Local Storage Persistence

```javascript
// mixins/persist.js
export default {
  created() {
    const savedState = localStorage.getItem(this.persistKey)
    if (savedState) {
      const parsed = JSON.parse(savedState)
      Object.keys(parsed).forEach(key => {
        this.$set(this.$data, key, parsed[key])
      })
    }
    
    this.$watch(this.persistKey, {
      handler() {
        const state = {}
        this.persistFields.forEach(field => {
          state[field] = this[field]
        })
        localStorage.setItem(this.persistKey, JSON.stringify(state))
      },
      deep: true
    })
  }
}

// Usage
export default {
  mixins: [persist],
  
  data() {
    return {
      searchQuery: '',
      filters: {}
    }
  },
  
  computed: {
    persistKey() {
      return 'my-component-state'
    },
    
    persistFields() {
      return ['searchQuery', 'filters']
    }
  }
}
```

## Multiple Mixins

```javascript
import formatter from '@/mixins/formatter'
import validator from '@/mixins/validator'
import logger from '@/mixins/logger'

export default {
  mixins: [formatter, validator, logger],
  
  created() {
    this.log('Component created')  // From logger mixin
    this.formatDate(new Date())    // From formatter mixin
  }
}
```

## Mixin vs Mixin Conflict

When multiple mixins have the same option:

```javascript
// mixinA
export default {
  data() {
    return { message: 'from mixin A' }
  }
}

// mixinB
export default {
  data() {
    return { message: 'from mixin B' }
  }
}

// Component
export default {
  mixins: [mixinA, mixinB],
  data() {
    return { message: 'from component' }
  }
}

// Final message: 'from component'
// Component always wins for data, methods, computed
```

## Custom Merge Strategies

```javascript
// Custom merge for 'customOption'
Vue.config.optionMergeStrategies.custom = function(toVal, fromVal) {
  // Return merged value
  if (!toVal) return fromVal
  if (!fromVal) return toVal
  
  return toVal.concat(fromVal)
}
```

## Composition API vs Mixins (Vue 2 vs Vue 3)

In Vue 3 with Composition API, `mixins` are replaced by composables:

```javascript
// Vue 2 Mixin
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

// Vue 3 Composable
import { ref } from 'vue'

export function useCounter() {
  const count = ref(0)
  
  function increment() {
    count.value++
  }
  
  return { count, increment }
}
```

## Best Practices

### 1. Use Descriptive Names

```javascript
// ❌ BAD
export default {
  mixins: [foo, bar]  // What are these?
}

// ✅ GOOD
import formValidation from '@/mixins/formValidation'
import dataFetcher from '@/mixins/dataFetcher'

export default {
  mixins: [formValidation, dataFetcher]
}
```

### 2. Keep Mixins Focused

```javascript
// ❌ BAD - Mixin does too much
export default {
  data() { return { /* lots of data */ } },
  methods: { /* lots of methods */ },
  mounted() { /* lots of logic */ }
}

// ✅ GOOD - Focused mixins
export default {
  mixins: [
    scrollDetector,      // Handles scroll detection
    formValidation,      // Handles form validation
    localStoragePersist  // Handles persistence
  ]
}
```

### 3. Document Mixins

```javascript
/**
 * @mixin formValidation
 * 
 * Provides form validation functionality
 * 
 * @param {Object} form - The form data object
 * @param {Object} rules - Validation rules per field
 * 
 * @example
 * import formValidation from '@/mixins/formValidation'
 * 
 * rules: {
 *   email: { required: true, email: true },
 *   password: { required: true, minLength: 6 }
 * }
 */
export default {
  // ... implementation
}
```

### 4. Use for Cross-Cutting Concerns

```javascript
// ✅ GOOD - Good use of mixins
export default {
  mixins: [
    logger,           // Logging across all components
    analytics,        // Analytics tracking
    errorHandler      // Error handling
  ]
}

// ❌ BAD - Use inheritance or composition instead
export default {
  mixins: [specificFeatureMixin]  // One component feature
}
```

## Anti-Patterns

### 1. Overusing Mixins

```javascript
// ❌ WRONG - Too many mixins
export default {
  mixins: [
    mixin1, mixin2, mixin3, mixin4, mixin5, mixin6
  ]
}

// ✅ CORRECT - Keep it manageable
// Group related functionality into fewer, well-designed mixins
```

### 2. Unclear Source of Options

```javascript
// ❌ WRONG - Where does this come from?
this.formatDate()
this.validate()
this.track()

// ✅ CORRECT - Use namespaced modules or Vuex
this.$formatters.formatDate()
this.$validators.validate()
this.$analytics.track()
```

### 3. Mixins for One-Time Use

```javascript
// ❌ WRONG - Only used in one component
import singleUseMixin from '@/mixins/singleUseMixin'

export default {
  mixins: [singleUseMixin]
}

// ✅ CORRECT - Just put it in the component
```

### 4. Deeply Nested Mixins

```javascript
// ❌ WRONG - Hard to debug
export default {
  mixins: [
    {
      mixins: [
        {
          mixins: [deepMixin]
        }
      ]
    }
  ]
}
```

## Testing Mixins

```javascript
// mixins/__tests__/formValidation.spec.js
import { createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'
import formValidation from '../formValidation'

describe('formValidation mixin', () => {
  let componentOptions
  
  beforeEach(() => {
    componentOptions = {
      mixins: [formValidation],
      data() {
        return {
          form: { email: '' },
          errors: {},
          touched: {}
        }
      }
    }
  })
  
  test('validateField sets error for required field', () => {
    const vm = new Vue(createLocalVue())
      .extend(componentOptions)
      .$mount()
    
    vm.validateField('email', { required: true })
    
    expect(vm.errors.email).toBe('This field is required')
  })
})
```

## When to Use Mixins vs Other Patterns

| Pattern | Use When |
|---------|----------|
| Mixins | Sharing reusable component logic in Vue 2 |
| Vuex | Complex state management across app |
| Composition (Vue 3) | Vue 3 code sharing |
| Plugins | Adding global functionality |
| Utility functions | Pure functions, no component state |

## References

See the `references/` directory for:

- Mixin conflict resolution
- Testing mixins
- Mixin patterns library
