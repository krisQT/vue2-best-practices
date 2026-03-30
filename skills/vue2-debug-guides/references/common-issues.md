# Vue 2 Common Issues Reference

## Reactivity Issues

### Issue: Property Not Reactive

**Symptoms**: Changes don't trigger updates

**Common Causes**:
- Adding new property to object
- Setting array by index
- Property not initialized

**Solutions**:

```javascript
// ❌ Wrong
this.user.email = 'new@email.com'
this.items[0] = newItem

// ✅ Correct
this.$set(this.user, 'email', 'new@email.com')
this.$set(this.items, 0, newItem)
// Or
this.items.splice(0, 1, newItem)
```

## Component Issues

### Issue: Component Not Rendering

**Checklist**:
- [ ] Component imported correctly?
- [ ] Registered locally or globally?
- [ ] Props passed correctly?
- [ ] v-if condition met?
- [ ] No errors in console?

```javascript
// ✅ Ensure proper import
import MyComponent from '@/components/MyComponent.vue'

// ✅ Register component
export default {
  components: { MyComponent }
}
```

### Issue: Props Not Passing

**Common Mistakes**:

```javascript
// ❌ Wrong - prop name mismatch
<user-card :username="name" />  // 'username' prop doesn't exist

// ✅ Correct - matching prop name
props: {
  username: String
}

// ✅ Correct - pass correct prop
<user-card :user="userData" />
```

## Vuex Issues

### Issue: State Not Updating

**Check**:
- [ ] Mutation called?
- [ ] Using correct module namespace?
- [ ] Strict mode enabled?

```javascript
// ❌ Wrong - direct mutation
state.user.name = 'John'  // Won't work in strict mode!

// ✅ Correct - commit mutation
commit('SET_USER_NAME', 'John')

mutations: {
  SET_USER_NAME(state, name) {
    state.user.name = name
  }
}
```

### Issue: Getter Returns Wrong Value

```javascript
// ❌ Wrong - mutation outside mutation
actions: {
  updateUser({ state }, name) {
    state.user.name = name  // Direct mutation!
  }
}

// ✅ Correct
actions: {
  updateUser({ commit }, name) {
    commit('SET_USER_NAME', name)
  }
}
```

## Router Issues

### Issue: Route Not Found (404)

**Common Causes**:
- Route not defined
- Wrong path spelling
- Case sensitivity

```javascript
// ✅ Define route
{
  path: '/users/:id',
  name: 'UserProfile',
  component: UserProfile
}

// ✅ Use correct navigation
this.$router.push({ name: 'UserProfile', params: { id: 123 } })
```

### Issue: Navigation Guard Not Working

```javascript
// ✅ Correct guard syntax
router.beforeEach((to, from, next) => {
  // Must call next()
  if (authorized) {
    next()
  } else {
    next('/login')
  }
})
```

## Async Issues

### Issue: Data Not Ready

**Problem**: Trying to access async data before it's loaded

```javascript
// ❌ Wrong
created() {
  this.fetchData()
},
mounted() {
  console.log(this.data)  // Might be undefined!
}

// ✅ Correct
async created() {
  await this.fetchData()
  this.initialize()
}

mounted() {
  // Data is ready now
}
```

## Performance Issues

### Issue: Too Many Re-renders

**Symptoms**: UI lag, high CPU usage

**Solutions**:

```javascript
// 1. Use computed instead of methods in template
// ❌ Bad
<template>{{ getFullName() }}</template>

// ✅ Good
<template>{{ fullName }}</template>

// 2. Use Object.freeze for static data
data() {
  return {
    staticData: Object.freeze([
      { id: 1, label: 'Option A' },
      { id: 2, label: 'Option B' }
    ])
  }
}

// 3. Use v-memo for lists
<div v-for="item in items" :key="item.id" v-memo="[item.id]">
```

### Issue: Memory Leaks

**Common Sources**:

```javascript
// ❌ Wrong - not cleaned up
mounted() {
  this.timer = setInterval(this.tick, 1000)
  window.addEventListener('resize', this.handleResize)
}

destroyed() {
  // Forgot to clean up!
}

// ✅ Correct
beforeDestroy() {
  clearInterval(this.timer)
  window.removeEventListener('resize', this.handleResize)
}
```

## Template Issues

### Issue: Template Errors

**Common Mistakes**:

```html
<!-- ❌ Wrong - invalid template syntax -->
<div v-if="items.length">
  <div v-for="item in items">
    {{ item.name }}
  </div>
</div>

<!-- ✅ Correct - always use :key -->
<div v-if="items.length">
  <div v-for="item in items" :key="item.id">
    {{ item.name }}
  </div>
</div>
```

### Issue: Event Handler Issues

```html
<!-- ❌ Wrong - inline arrow function -->
<button @click="() => doSomething(this.value)">

<!-- ✅ Correct - use $event or method -->
<button @click="doSomething(value)">
<button @click="(e) => doSomething(e.target.value)">
```

## Debugging Tips

### Enable Verbose Warnings

```javascript
Vue.config.warnHandler = (msg, vm, trace) => {
  console.error('Warning:', msg)
  console.error('Trace:', trace)
}
```

### Check Vue Version

```javascript
console.log('Vue version:', Vue.version)
console.log('Vue config:', Vue.config)
```

### Inspect Component

```javascript
// In console
vm0.$root           // Root instance
vm0.$children[0]    // First child
vm0.$refs.myRef     // Named ref
```

### Track Changes

```javascript
// Watch for changes
this.$watch('someData', (newVal, oldVal) => {
  console.log('Changed:', oldVal, '->', newVal)
}, { deep: true })
```

## Quick Debugging Checklist

```
□ Is Vue loaded correctly?
□ Is component registered?
□ Are props passed correctly?
□ Is data reactive?
□ Are events emitting correctly?
□ Is Vuex state updating?
□ Are lifecycle hooks correct?
□ Is there memory leaks?
□ Are async operations awaited?
□ Are there console errors?
```
