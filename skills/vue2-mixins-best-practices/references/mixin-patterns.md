# Vue 2 Mixins Patterns Reference

## Basic Mixin Structure

```javascript
// mixins/eventLogger.js
export default {
  created() {
    console.log('Mixin created')
  },
  
  data() {
    return {
      mixinData: 'from mixin'
    }
  },
  
  methods: {
    logEvent(event) {
      console.log('Event:', event)
    }
  },
  
  computed: {
    mixinComputed() {
      return this.mixinData.toUpperCase()
    }
  }
}
```

## Common Mixin Patterns

### 1. API Data Fetcher

```javascript
// mixins/apiFetcher.js
export default {
  data() {
    return {
      items: [],
      loading: false,
      error: null
    }
  },
  
  methods: {
    async fetchItems(endpoint) {
      this.loading = true
      this.error = null
      
      try {
        this.items = await api.get(endpoint)
      } catch (err) {
        this.error = err.message
      } finally {
        this.loading = false
      }
    },
    
    async createItem(endpoint, data) {
      this.loading = true
      try {
        const created = await api.post(endpoint, data)
        this.items.push(created)
        return created
      } catch (err) {
        this.error = err.message
        throw err
      } finally {
        this.loading = false
      }
    }
  }
}

// Usage
export default {
  mixins: [apiFetcher],
  created() {
    this.fetchItems('/api/users')
  }
}
```

### 2. Form Handler

```javascript
// mixins/formHandler.js
export default {
  data() {
    return {
      form: {},
      errors: {},
      submitting: false
    }
  },
  
  methods: {
    resetForm() {
      this.form = {}
      this.errors = {}
    },
    
    validateForm(rules) {
      this.errors = {}
      let isValid = true
      
      for (const field in rules) {
        const rule = rules[field]
        const value = this.form[field]
        
        if (rule.required && !value) {
          this.$set(this.errors, field, rule.requiredMessage || 'Required')
          isValid = false
        }
        
        if (rule.pattern && !rule.pattern.test(value)) {
          this.$set(this.errors, field, rule.patternMessage || 'Invalid')
          isValid = false
        }
      }
      
      return isValid
    },
    
    async submitForm(endpoint, rules) {
      if (!this.validateForm(rules)) {
        return false
      }
      
      this.submitting = true
      try {
        await api.post(endpoint, this.form)
        this.resetForm()
        return true
      } catch (err) {
        this.errors.submit = err.message
        return false
      } finally {
        this.submitting = false
      }
    }
  }
}
```

### 3. Window Events

```javascript
// mixins/windowEvents.js
export default {
  data() {
    return {
      windowWidth: window.innerWidth,
      windowHeight: window.innerHeight
    }
  },
  
  mounted() {
    window.addEventListener('resize', this.handleResize)
    window.addEventListener('scroll', this.handleScroll)
  },
  
  beforeDestroy() {
    window.removeEventListener('resize', this.handleResize)
    window.removeEventListener('scroll', this.handleScroll)
  },
  
  methods: {
    handleResize() {
      this.windowWidth = window.innerWidth
      this.windowHeight = window.innerHeight
    },
    
    handleScroll() {
      this.scrollY = window.scrollY
    }
  }
}
```

### 4. Pagination

```javascript
// mixins/pagination.js
export default {
  data() {
    return {
      currentPage: 1,
      pageSize: 10,
      totalItems: 0
    }
  },
  
  computed: {
    totalPages() {
      return Math.ceil(this.totalItems / this.pageSize)
    },
    
    hasNextPage() {
      return this.currentPage < this.totalPages
    },
    
    hasPrevPage() {
      return this.currentPage > 1
    },
    
    paginatedItems() {
      const start = (this.currentPage - 1) * this.pageSize
      const end = start + this.pageSize
      return this.allItems.slice(start, end)
    }
  },
  
  methods: {
    goToPage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page
      }
    },
    
    nextPage() {
      if (this.hasNextPage) {
        this.currentPage++
      }
    },
    
    prevPage() {
      if (this.hasPrevPage) {
        this.currentPage--
      }
    },
    
    setPageSize(size) {
      this.pageSize = size
      this.currentPage = 1
    }
  }
}
```

### 5. Debounce

```javascript
// mixins/debounce.js
export default {
  data() {
    return {
      debounceTimers: {}
    }
  },
  
  methods: {
    debounce(method, delay = 300) {
      const name = method.name || method.toString()
      
      if (this.debounceTimers[name]) {
        clearTimeout(this.debounceTimers[name])
      }
      
      this.debounceTimers[name] = setTimeout(() => {
        method()
      }, delay)
    },
    
    debouncedMethod(method, delay) {
      let timeout
      return function(...args) {
        clearTimeout(timeout)
        timeout = setTimeout(() => {
          method.apply(this, args)
        }, delay)
      }
    }
  },
  
  beforeDestroy() {
    Object.values(this.debounceTimers).forEach(timer => {
      clearTimeout(timer)
    })
  }
}
```

## Mixin Conflict Resolution

### Data Conflicts

```javascript
// mixin A
export default {
  data() {
    return { shared: 'from A' }
  }
}

// mixin B
export default {
  data() {
    return { shared: 'from B' }
  }
}

// component
export default {
  mixins: [mixinA, mixinB],
  data() {
    return { shared: 'from component' }
  }
}

// Result: shared = 'from component' (component wins)
```

### Lifecycle Hooks

```javascript
// All lifecycle hooks are called
export default {
  mixins: [mixinA, mixinB],
  created() {
    console.log('Component created')
  }
}

// Order: mixinA.created -> mixinB.created -> Component.created
```

### Methods

```javascript
// Later mixin/component overrides earlier
export default {
  mixins: [mixinA, mixinB],
  methods: {
    greet() {
      return 'Hello from component'
    }
  }
}

// Result: Component method wins
```

## Using Multiple Mixins

```javascript
import apiFetcher from './apiFetcher'
import pagination from './pagination'
import formValidation from './formValidation'

export default {
  mixins: [
    apiFetcher,
    pagination,
    formValidation
  ],
  
  data() {
    return {
      form: {}
    }
  },
  
  created() {
    this.fetchItems('/api/data')
  }
}
```

## Vue 2 Mixins vs Vue 3 Composables

| Feature | Vue 2 Mixins | Vue 3 Composables |
|---------|--------------|------------------|
| State sharing | Via mixin data | Via return values |
| Naming conflicts | Automatic merge | Manual destructuring |
| TypeScript | Limited | Full support |
| Tree shaking | No | Yes |
| Testing | Harder | Easier |

```javascript
// Vue 3 Composable equivalent
import { ref } from 'vue'

export function useCounter(initialCount = 0) {
  const count = ref(initialCount)
  
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  return {
    count,
    increment,
    decrement
  }
}
```

## Best Practices

1. **Keep mixins focused** - One responsibility per mixin
2. **Name clearly** - Descriptive names prevent conflicts
3. **Document well** - Explain what the mixin provides
4. **Clean up** - Always clean up in beforeDestroy
5. **Prefer composables** - In Vue 3 or if migrating
