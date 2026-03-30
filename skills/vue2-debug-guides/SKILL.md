---
name: vue2-debug-guides
description: Vue 2 debugging guides including common Vue 2 issues, reactivity problems, Vue DevTools usage, error message interpretation, and performance debugging.
license: MIT
metadata:
  author: github.com/krisQT
  version: "1.1.0"
---

# Vue 2 Debug Guides

## When to Use

Use this skill when:

- Debugging Vue 2 applications
- Solving common Vue 2 issues
- Using Vue DevTools
- Understanding error messages

## Activation Triggers

Use this skill when the user prompt contains any of these patterns:

- "debug Vue"
- "Vue not updating"
- "Vue reactivity issue"
- "Vue error"
- "Vue warning"
- "Vue DevTools"
- "Vue this undefined"
- "memory leak Vue"
- "Vue component not rendering"
- "props not working"
- "v-for not updating"
- "computed not updating"
- "watcher not triggering"
- "Vue performance"
- "Vue memory leak"
- "Vue lifecycle issue"
- "Vue debugging"
- "Vue console log"
- "Vue error message"
- "avoid mutating prop"
- "Vue $set"
- "Vue $delete"
- "Vue.array not reactive"
- "Vue change detection"
- "Vue devtools"

Keywords: `Vue.config.performance`, `$nextTick`, `$watch`, `console.log(this`

## Vue DevTools

### Installation

- Chrome: [Vue DevTools Extension](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
- Firefox: [Vue DevTools Extension](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)

### Key Features

1. **Components Tab**
   - View component tree
   - Inspect component state
   - Edit props and data in real-time
   - Filter components

2. **Vuex Tab**
   - View state tree
   - Track mutations
   - Time travel debugging
   - Take snapshots

3. **Events Tab**
   - View emitted events
   - Filter by component
   - Track event payloads

4. **Timeline**
   - Performance monitoring
   - Component render times
   - Vuex mutation timeline

## Common Issues

### 1. Changes Not Reflecting in DOM

**Symptoms**: Data changed but UI not updating

**Causes**:
- Object property added after initialization
- Array index assignment
- Added property to array

**Solutions**:

```javascript
// Problem: Adding new property
this.user.email = 'test@example.com'  // Won't work!

// Solution 1: Use Vue.set
this.$set(this.user, 'email', 'test@example.com')

// Solution 2: Replace object
this.user = { ...this.user, email: 'test@example.com' }

// Problem: Array index assignment
this.items[0] = newItem  // Won't work!

// Solution
this.$set(this.items, 0, newItem)
// Or
this.items.splice(0, 1, newItem)
```

### 2. this Context Issues

**Symptoms**: `this` is undefined or wrong in methods

**Cause**: Using arrow functions in component options

**Solution**:

```javascript
// ❌ WRONG
export default {
  methods: {
    handleClick: () => {
      console.log(this)  // undefined or window
    }
  }
}

// ✅ CORRECT
export default {
  methods: {
    handleClick() {
      console.log(this)  // Component instance
    }
  }
}
```

### 3. Memory Leaks

**Symptoms**: Performance degrades over time, console warnings

**Common Causes**:
- Event listeners not removed
- Timers not cleared
- Subscriptions not unsubscribed

**Solutions**:

```javascript
export default {
  created() {
    // ❌ WRONG
    window.addEventListener('resize', this.handleResize)
    this.timer = setInterval(this.tick, 1000)
    
    // ✅ CORRECT
    window.addEventListener('resize', this.handleResize)
    this.timer = setInterval(this.tick, 1000)
  },
  
  beforeDestroy() {
    // Clean up!
    window.removeEventListener('resize', this.handleResize)
    clearInterval(this.timer)
    clearTimeout(this.timer)
    
    // Unsubscribe from observables
    this.subscription?.unsubscribe()
  }
}
```

### 4. Props Mutated

**Symptoms**: Vue warns about direct prop mutation

**Cause**: Modifying props in component

**Solution**:

```javascript
// ❌ WRONG
props: ['initialCount'],
mounted() {
  this.initialCount = 5  // Warning!
}

// ✅ CORRECT
props: ['initialCount'],
data() {
  return {
    count: this.initialCount  // Copy to data
  }
}
```

### 5. Computed Not Updating

**Symptoms**: Computed property returns stale value

**Causes**:
- Computed doesn't track dependency
- Using non-reactive data in computed

**Solution**:

```javascript
// ❌ WRONG - Doesn't track date change
computed: {
  formattedDate() {
    return new Date(this.date).toLocaleDateString()
  }
}

// ✅ CORRECT - Ensure reactive access
computed: {
  formattedDate() {
    const date = this.date  // Access reactively
    return new Date(date).toLocaleDateString()
  }
}
```

### 6. Watch Not Triggering

**Symptoms**: Watch callback not called on change

**Causes**:
- Watching nested property without deep option
- Property doesn't exist in initial data

**Solution**:

```javascript
// ❌ WRONG - Nested property not watched
watch: {
  'user.name'(newVal) {
    console.log(newVal)
  }
}

// ✅ CORRECT
watch: {
  user: {
    handler(newVal) {
      console.log(newVal.name)
    },
    deep: true  // Watch nested properties
  }
}

// ✅ ALTERNATIVE - Initialize and watch
data() {
  return {
    user: {
      name: '',
      email: ''  // Initialize!
    }
  }
}
```

### 7. Async Component Not Loading

**Symptoms**: Component doesn't appear

**Causes**:
- Import path wrong
- webpack chunk name issue
- Error in async component

**Solutions**:

```javascript
// ✅ Use correct import
const AsyncComponent = () => import('./AsyncComponent.vue')

// ✅ Add error handling
components: {
  AsyncComponent: () => ({
    component: import('./AsyncComponent.vue'),
    error: ErrorComponent,
    loading: LoadingComponent
  })
}
```

### 8. v-for Key Issues

**Symptoms**: Components not updating correctly, unexpected behavior

**Cause**: Using index as key instead of unique identifier

**Solution**:

```javascript
// ❌ WRONG
<div v-for="(item, index) in items" :key="index">

// ✅ CORRECT
<div v-for="item in items" :key="item.id">

// ✅ For dynamic keys
<div v-for="item in items" :key="item.id || item.name">
```

## Error Messages

### "Avoid mutating a prop directly"

```
Error: Avoid mutating a prop directly...
```

**Cause**: Directly modifying a prop value

**Solution**: Copy prop to data and modify that

```javascript
props: ['initialValue'],
data() {
  return {
    value: this.initialValue
  }
},
methods: {
  updateValue(newVal) {
    this.value = newVal
    this.$emit('update', newVal)
  }
}
```

### "Expected Array/Object"

```
Error: Expected Array/Object with v-for
```

**Cause**: v-for used on non-iterable value

**Solution**: Ensure data is array/object

```javascript
data() {
  return {
    items: []  // Always initialize arrays
  }
}
```

### "Unknown custom element"

```
Error: Unknown custom element: <my-component>
```

**Cause**: Component not registered

**Solution**:

```javascript
// Local registration
export default {
  components: {
    MyComponent
  }
}

// Or global registration
Vue.component('my-component', MyComponent)
```

### "did you register the component correctly?"

```
Error: Failed to resolve component: MyComponent
```

**Cause**: Import not found or component not imported

**Solutions**:

```javascript
// Check import
import MyComponent from '@/components/MyComponent.vue'

// Check registration
export default {
  components: {
    MyComponent  // Don't forget!
  }
}
```

## Console Logging

### Useful Vue Debug Statements

```javascript
export default {
  mounted() {
    // Log component instance
    console.log('Component:', this)
    
    // Log reactive data
    console.log('User:', this.user)
    
    // Log computed values
    console.log('Full name:', this.fullName)
    
    // Log props
    console.log('Props:', this.$props)
    
    // Log DOM ref
    console.log('Input:', this.$refs.myInput)
    
    // Watch changes
    this.$watch('user', (newVal, oldVal) => {
      console.log('User changed:', oldVal, '->', newVal)
    }, { deep: true })
  }
}
```

### Vue DevTools Console API

```javascript
// In DevTools console
vm.$root           // Root instance
vm.$children       // Child components
vm.$refs.myComponent  // Named refs

// Vue utilities
Vue.set(obj, key, value)
Vue.delete(obj, key)
Vue.nextTick(callback)
```

## Performance Debugging

### Identify Render Performance

```javascript
const components = []

// Enable performance mode
Vue.config.performance = true

// Or use DevTools Timeline
// In Vue DevTools > Timeline > Components
```

### Memory Profiling

1. Open Chrome DevTools
2. Go to Performance tab
3. Record interaction
4. Check for memory leaks in Memory tab
5. Look for detached DOM nodes

### Large List Performance

```javascript
// Problem: Large list causes lag
<div v-for="item in items">{{ item }}</div>

// Solutions:

// 1. Use virtual scrolling
<virtual-list :items="items" />

// 2. Use v-show for visibility toggling
<div v-for="item in items" v-show="item.visible">

// 3. Limit visible items
<div v-for="item in visibleItems">{{ item }}</div>
```

## Vuex Debugging

### Inspect State Changes

```javascript
// Add logging plugin
const loggerPlugin = store => {
  store.subscribe((mutation, state) => {
    console.log('Mutation:', mutation.type)
    console.log('Payload:', mutation.payload)
    console.log('New State:', state)
  })
}

export default new Vuex.Store({
  plugins: [loggerPlugin]
})
```

### Track Specific Mutations

```javascript
// In Vue DevTools > Vuex > Mutations
// Filter by mutation type
```

### Debug Getters

```javascript
// Add getter tracking
getters: {
  computedValue: (state) => {
    console.log('Getter called')
    return state.value * 2
  }
}
```

## Network Debugging

### API Request Issues

```javascript
methods: {
  async fetchData() {
    try {
      const response = await fetch('/api/data')
      console.log('Status:', response.status)
      const data = await response.json()
      console.log('Data:', data)
    } catch (error) {
      console.error('Fetch error:', error)
    }
  }
}
```

### Vue Resource Interceptor

```javascript
Vue.http.interceptors.push((request, next) => {
  console.log('Request:', request)
  
  next(response => {
    console.log('Response:', response)
    
    if (!response.ok) {
      console.error('Request failed:', response.status)
    }
  })
})
```

## Template Debugging

### Check Template Compilation

```javascript
// Check if template is compiled
console.log(vm.$options.render)  // Render function
console.log(vm.$options.template)  // Template string

// See compiled template
console.log(Vue.compile(template))
```

### Debug Computed Access

```javascript
computed: {
  result() {
    console.log('Computing...')
    return this.input.map(x => x * 2)
  }
}
```

## Browser Debugging Tips

### Chrome DevTools

1. **Elements Tab**
   - Inspect Vue components
   - View computed styles
   - Edit DOM

2. **Console**
   - Access Vue instance: `vm0`, `vm1`, etc.
   - Use `$vm0` for root

3. **Sources**
   - Set breakpoints in Vue code
   - Debug render functions

4. **Network**
   - Monitor API calls
   - Check request/response

## Common Patterns to Debug

### Pattern 1: Double API Call

```javascript
// Problem
mounted() {
  this.fetchData()  // Called twice?
}
watch: {
  $route() {
    this.fetchData()  // This also triggers!
  }
}

// Solution
mounted() {
  this.fetchData()
},
watch: {
  $route: {
    handler() {
      this.fetchData()
    },
    immediate: false  // Don't run on mount
  }
}
```

### Pattern 2: Stale Closure

```javascript
// Problem
mounted() {
  setTimeout(() => {
    console.log(this.data)  // this might be stale
  }, 1000)
}

// Solution
mounted() {
  const vm = this
  setTimeout(() => {
    console.log(vm.data)
  }, 1000)
}
```

### Pattern 3: Array Filter Pitfalls

```javascript
// Problem - Original array modified!
const filtered = this.items.filter(...)
console.log(this.items.length)  // Changed!

// Solution - Make a copy first
const filtered = [...this.items].filter(...)
// Or
const filtered = this.items.slice().filter(...)
```

## Best Practices

### 1. Use Production Build in Development

```javascript
// Check Vue version
console.log(Vue.version)  // Should be 2.x.x

// In development, you might see more warnings
Vue.config.devtools = true
Vue.config.performance = true
```

### 2. Enable Detailed Warnings

```javascript
Vue.config.warnHandler = function(msg, vm, trace) {
  console.warn('Vue Warning:', msg)
  console.warn('Component:', vm)
  console.warn('Trace:', trace)
}
```

### 3. Error Boundaries

```javascript
export default {
  errorCaptured(err, vm, info) {
    console.error('Error captured:', err)
    console.error('Component:', vm)
    console.error('Info:', info)
    
    // Return false to prevent propagation
    return false
  }
}
```

## Quick Debugging Checklist

- [ ] Is the component registered?
- [ ] Are props passed correctly?
- [ ] Is data reactive?
- [ ] Are event handlers correct?
- [ ] Is Vuex state updating properly?
- [ ] Are there memory leaks?
- [ ] Is DevTools showing changes?

## References

See the `references/` directory for:

- Common error solutions
- Performance optimization guide
- Debugging tools list
