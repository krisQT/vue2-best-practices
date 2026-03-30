# Vue 2 Reactivity Deep Dive

## How Reactivity Works in Vue 2

Vue 2 uses `Object.defineProperty` to make properties reactive. This has important implications.

## Key Limitations

### 1. Cannot Detect Property Addition/Deletion

```javascript
const vm = new Vue({
  data: {
    user: {
      name: 'John'
    }
  }
})

// Adding a new property - NOT reactive
vm.user.age = 25  // Won't trigger updates!

// Solution: Use Vue.set
vm.$set(vm.user, 'age', 25)

// Or: Vue.set
Vue.set(vm.user, 'age', 25)
```

### 2. Cannot Detect Array Index Assignment

```javascript
const vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})

// Setting by index - NOT reactive
vm.items[1] = 'x'  // Won't trigger updates!

// Solutions:
vm.$set(vm.items, 1, 'x')  // Method 1
vm.items.splice(1, 1, 'x') // Method 2
```

### 3. Cannot Detect Array Length Changes

```javascript
const vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})

// Truncating array - NOT reactive
vm.items.length = 1  // Won't trigger updates!

// Solution:
vm.items.splice(1)
```

## Reactive Array Methods

These array mutation methods are reactive and will trigger updates:

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

```javascript
const vm = new Vue({
  data: {
    items: []
  }
})

// These all work reactively:
vm.items.push({ id: 1, name: 'Item 1' })
vm.items.splice(0, 1)  // Remove first item
vm.items.reverse()
```

## Replacement is Always Reactive

While mutations have limitations, replacing the entire object/array is always reactive:

```javascript
const vm = new Vue({
  data: {
    user: { name: 'John' },
    items: [1, 2, 3]
  }
})

// These are reactive:
vm.user = { name: 'Jane', age: 30 }
vm.items = [1, 2, 3, 4, 5]
```

## Watched Arrays

When watching arrays:

```javascript
export default {
  watch: {
    // This will trigger on array replacement but not on mutations
    items(newVal, oldVal) {
      // Called when items is reassigned
    },
    
    // For deep watching (includes mutations)
    items: {
      handler(newVal, oldVal) {
        // Called on any change
      },
      deep: true
    }
  }
}
```

## Computed Properties and Reactivity

Computed properties automatically track dependencies:

```javascript
const vm = new Vue({
  data: {
    firstName: 'John',
    lastName: 'Doe'
  },
  computed: {
    fullName() {
      // This computed property depends on firstName and lastName
      // It will re-evaluate whenever either changes
      return `${this.firstName} ${this.lastName}`
    }
  }
})
```

## Practical Examples

### Adding Multiple Properties

```javascript
export default {
  methods: {
    addUser(userData) {
      // Instead of:
      // Object.assign(this.newUser, userData)  // Not reactive
      
      // Do this:
      this.newUser = Object.assign({}, this.newUser, userData)
    }
  }
}
```

### Replacing Array Contents

```javascript
export default {
  methods: {
    filterItems(criteria) {
      const filtered = this.allItems.filter(item => 
        item.name.includes(criteria)
      )
      // This triggers reactivity
      this.items = filtered
    }
  }
}
```

### Working with Nested Objects

```javascript
export default {
  data() {
    return {
      user: {
        profile: {
          settings: {
            theme: 'dark'
          }
        }
      }
    }
  },
  
  methods: {
    updateTheme(newTheme) {
      // Deep nesting requires Vue.set or replacement
      this.$set(this.user.profile.settings, 'theme', newTheme)
      // Or: this.user.profile.settings = { ...this.user.profile.settings, theme: newTheme }
    }
  }
}
```

## Performance Considerations

1. **Avoid deep reactivity when not needed**
   - Deep watchers are expensive
   - Consider using `shallowReactive` pattern (Vue 2 doesn't have this, but be mindful)

2. **Batch updates**
   ```javascript
   // Vue automatically batches updates in event handlers
   // But in async code, you might want:
   this.$nextTick(() => {
     // DOM updates after current microtask queue is flushed
   })
   ```

3. **Change detection tricks**
   ```javascript
   // Instead of watching individual properties
   watch: {
     a() { this.handleChange() },
     b() { this.handleChange() },
     c() { this.handleChange() }
   }
   
   // Consider a combined approach
   watch: {
     '$data': {
       handler() { this.handleChange() },
       deep: true
     }
   }
   ```

## Common Pitfalls

### Pitfall 1: Forgetting $set

```javascript
// ❌ WRONG
this.user.status = 'active'

// ✅ CORRECT
this.$set(this.user, 'status', 'active')
```

### Pitfall 2: Object.assign on Reactive Object

```javascript
// ❌ WRONG - Mutates original
Object.assign(this.user, { name: 'New Name' })

// ✅ CORRECT - Creates new object
this.user = Object.assign({}, this.user, { name: 'New Name' })
```

### Pitfall 3: Array Index Assignment

```javascript
// ❌ WRONG
this.items[0] = newItem

// ✅ CORRECT
this.$set(this.items, 0, newItem)
// Or
this.items.splice(0, 1, newItem)
```

## Testing Reactivity

```javascript
// In tests, you can verify reactivity:
const vm = new Vue({
  data: {
    count: 0
  }
})

// Initial value
expect(vm.count).toBe(0)

// Update
vm.count = 5
// Wait for next tick
vm.$nextTick(() => {
  expect(vm.count).toBe(5)
})
```

## Best Practices Summary

1. ✅ Always initialize data with all expected properties
2. ✅ Use `Vue.set()` or `$set()` for dynamic property addition
3. ✅ Use `Vue.delete()` or `$delete()` for property removal
4. ✅ Use array mutation methods (push, splice, etc.) for array changes
5. ✅ Use object/array replacement when adding multiple items
6. ✅ Use computed properties instead of methods for derived state
7. ✅ Be aware of deep watcher performance implications
