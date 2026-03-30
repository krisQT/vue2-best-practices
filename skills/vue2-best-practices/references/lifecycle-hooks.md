# Vue 2 Lifecycle Hooks

## Lifecycle Diagram

```
Creation
  │
  ├─ beforeCreate
  │   (Instance initialized, no reactivity yet)
  │
  └─ created
      (Reactive data available, but DOM not mounted)
          │
          ├─ beforeMount
          │   (Template compiled, about to mount)
          │
          └─ mounted
              (DOM is ready and in document)
                  │
                  ├─ beforeUpdate
                  │   (Reactive data changed, re-render needed)
                  │
                  └─ updated
                      (DOM updated)
                      
                      (Loop: beforeUpdate → updated)
                      
                  ├─ beforeDestroy
                  │   (Instance still functional, about to tear down)
                  │
                  └─ destroyed
                      (Instance destroyed)
```

## Detailed Hook Descriptions

### beforeCreate

```javascript
export default {
  beforeCreate() {
    // Called synchronously immediately after instance initialization
    // No access to: data, props, computed, methods, watchers
    // No access to: $el, $refs, $root, $parent
    
    console.log('beforeCreate:', this.$options.name)
    // this.message is undefined here
  }
}
```

**Use cases:**
- Initialize non-reactive instance properties
- Set up simple instance configuration

### created

```javascript
export default {
  data() {
    return {
      users: null,
      loading: false
    }
  },
  
  created() {
    // Access to: data, props, computed, methods, watchers
    // No access to: $el (not in DOM), $refs (template not rendered)
    
    console.log('created:', this.users)  // null (initialized)
    this.fetchUsers()  // Common to start data fetching here
    
    // Can access this.message
    // Can access this.$options.name
  }
}
```

**Use cases:**
- Initialize component data from props
- Start data fetching
- Set up event listeners (if not using attached/detached)
- Initialize non-reactive state

### beforeMount

```javascript
export default {
  beforeMount() {
    // Template has been compiled
    // Virtual DOM created but not yet applied to real DOM
    // Last chance to modify data before render
    
    console.log('beforeMount:', this.$el)  // undefined
  }
}
```

**Use cases:**
- Final data preparation
- Last-minute setup before render
- Modify component state (rare use case)

### mounted

```javascript
export default {
  mounted() {
    // Instance is mounted, DOM is ready
    // Can access: this.$el, this.$refs
    // Guaranteed to be in document (unlike Vue 3)
    
    console.log('mounted:', this.$el)
    console.log('refs:', this.$refs.myInput)
    
    // Initialize third-party libraries
    this.initChart()
  }
}
```

**Use cases:**
- Access DOM elements with $refs
- Initialize third-party libraries (charts, maps, etc.)
- Add event listeners to DOM elements
- Start intervals or timers
- DOM manipulations

**Note:** In Vue 2, `mounted` does not guarantee the component is in the document. Use `$nextTick` and `nextTick` callback:

```javascript
mounted() {
  this.$nextTick(function() {
    // DOM is now guaranteed to be in document
    this.initThirdPartyLibrary()
  })
}
```

### beforeUpdate

```javascript
export default {
  data() {
    return {
      count: 0
    }
  },
  
  beforeUpdate() {
    // Called when reactive data changes
    // DOM has not been updated yet
    // Good for accessing DOM state before update
    
    console.log('beforeUpdate: count =', this.count)
    // Can access old DOM state here
  }
}
```

**Use cases:**
- Access DOM state before update
- Save scroll position
- Grab information from DOM before re-render

### updated

```javascript
export default {
  updated() {
    // DOM has been updated
    // Be careful of infinite loops if updating reactive data here
    
    console.log('updated: DOM is now updated')
  }
}
```

**Use cases:**
- Execute code after DOM update
- Reset values after update
- Interact with third-party libraries after update

**Important:** Always use `$nextTick` for accessing updated DOM:

```javascript
updated() {
  this.$nextTick(function() {
    // DOM is now fully updated
    this.resizeChart()
  })
}
```

### beforeDestroy

```javascript
export default {
  data() {
    return {
      timer: null
    }
  },
  
  created() {
    this.timer = setInterval(() => {
      this.fetchData()
    }, 5000)
  },
  
  beforeDestroy() {
    // Instance is still fully functional
    // Good for cleanup
    
    clearInterval(this.timer)
    clearTimeout(this.timer)
    
    // Remove event listeners
    window.removeEventListener('resize', this.handleResize)
    
    // Cancel pending requests
    if (this.request) {
      this.request.cancel()
    }
  }
}
```

**Use cases:**
- Cancel timers and intervals
- Remove event listeners
- Cancel ongoing HTTP requests
- Clean up third-party library instances

### destroyed

```javascript
export default {
  destroyed() {
    // Component is destroyed
    // Child components also destroyed
    // Directives unbound
    // Event listeners removed (if properly cleaned up in beforeDestroy)
    
    console.log('destroyed: Component is fully cleaned up')
  }
}
```

**Use cases:**
- Final cleanup (though beforeDestroy is preferred)
- Debugging to verify cleanup
- Log destruction for monitoring

## Parent-Child Lifecycle Order

When a parent component creates a child:

```
Parent beforeCreate
Parent created
Parent beforeMount
  Child beforeCreate
  Child created
  Child beforeMount
  Child mounted
Parent mounted
```

When a parent is destroyed:

```
Parent beforeDestroy
  Child beforeDestroy
  Child destroyed
Parent destroyed
```

## Keep-Alive and Activated/Deactivated

When using `<keep-alive>`:

```javascript
export default {
  activated() {
    // Called when component is activated (brought back to life)
    // Similar to mounted but for kept-alive components
    
    this.resumeUpdates()
    this.startTimer()
  },
  
  deactivated() {
    // Called when component is deactivated (cached)
    // Similar to beforeDestroy but component is preserved
    
    this.pauseUpdates()
    this.stopTimer()
  }
}
```

## Error Captured Hook (Vue 2.5+)

```javascript
export default {
  errorCaptured(err, vm, info) {
    // Called when a descendant component throws an error
    // Return false to prevent error propagation
    
    console.error('Error captured:', err)
    console.error('Component:', vm)
    console.error('Error info:', info)
    
    // Return false to stop propagation
    return false
  }
}
```

## Server-Side Rendering (SSR)

In SSR, lifecycle hooks are limited:

- `beforeCreate` and `created` are called on server
- `mounted`, `beforeUpdate`, `updated`, `beforeDestroy`, `destroyed` are NOT called on server
- `activated` and `deactivated` are NOT called in SSR

```javascript
export default {
  created() {
    // Runs on both server and client
    this.initializeData()
    
    // For client-only code:
    if (typeof window !== 'undefined') {
      this.initClientOnlyFeature()
    }
  },
  
  mounted() {
    // Runs only on client
    this.initThirdPartyLibrary()
  }
}
```

## Async Components and Lifecycle

For async components:

```javascript
// Component definition
const AsyncComponent = () => ({
  component: import('./AsyncComponent.vue'),
  loading: LoadingComponent,
  error: ErrorComponent,
  delay: 200,
  timeout: 3000
})

// In parent
export default {
  components: {
    AsyncComponent
  }
}
```

Async components have the same lifecycle, but loading states can affect when hooks are called.

## Best Practices

1. **Use created for data fetching**
   - DOM is not needed
   - Faster than waiting for mounted
   - Consistent between server and client render

2. **Use mounted for DOM operations**
   - Need to wait for DOM to be ready
   - Good for third-party library initialization
   - Remember to use $nextTick

3. **Always clean up in beforeDestroy**
   - Prevent memory leaks
   - Remove timers, listeners, subscriptions
   - Cancel pending requests

4. **Be careful in updated**
   - Don't update reactive state (causes infinite loops)
   - Use $nextTick for batch DOM operations

5. **Use activated/deactivated for keep-alive**
   - Better than mounted/destroyed for cached components
   - More efficient than reinitializing everything

## Common Mistakes

### Mistake 1: Not Cleaning Up

```javascript
// ❌ WRONG
export default {
  created() {
    this.timer = setInterval(this.tick, 1000)
    // Timer never cleared!
  }
}

// ✅ CORRECT
export default {
  created() {
    this.timer = setInterval(this.tick, 1000)
  },
  
  beforeDestroy() {
    clearInterval(this.timer)
  }
}
```

### Mistake 2: Arrow Functions in Lifecycle

```javascript
// ❌ WRONG
export default {
  created: () => {
    this.fetchData()  // this is undefined!
  }
}

// ✅ CORRECT
export default {
  created() {
    this.fetchData()  // this is the component instance
  }
}
```

### Mistake 3: Trying to Access DOM in created

```javascript
// ❌ WRONG
export default {
  created() {
    const element = document.querySelector('.my-class')
    // Element doesn't exist yet!
  }
}

// ✅ CORRECT
export default {
  mounted() {
    const element = document.querySelector('.my-class')
    // Element exists now
  }
}
```
