---
name: vue2-router-best-practices
description: Vue Router 3 patterns for Vue 2 including navigation guards (beforeRouteEnter, beforeEach), route params, queries, nested routes, lazy loading, and programmatic navigation.
license: MIT
metadata:
  author: github.com/krisQT
  version: "1.1.0"
---

# Vue Router 3 Best Practices

## When to Use

Use this skill when:

- Working with Vue Router 3 (for Vue 2)
- Implementing navigation guards
- Managing route params and queries
- Building multi-page Vue 2 applications

## Activation Triggers

Use this skill when the user prompt contains any of these patterns:

- "Vue Router"
- "navigation guard"
- "beforeRouteEnter"
- "beforeEach router"
- "route params"
- "route query"
- "dynamic route"
- "nested routes"
- "lazy load route"
- "route transition"
- "router history mode"
- "hash mode router"
- "router push"
- "router replace"
- "programmatic navigation"
- "route meta"
- "route redirect"
- "route alias"
- "scroll behavior"
- "router-link"
- "$route.params"
- "$router.push"
- "route children"
- "named routes"
- "Vue Router 3"

Keywords: `this.$router`, `this.$route`, `router-view`, `router-link`, `beforeEnter`

## Router Setup

### Basic Setup

```javascript
// router/index.js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '@/views/Home.vue'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/views/About.vue')  // Lazy load
  }
]

const router = new VueRouter({
  mode: 'history',  // or 'hash'
  routes
})

export default router
```

### Main.js Integration

```javascript
// main.js
import Vue from 'vue'
import router from './router'

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

## Route Parameters

### Defining Parameters

```javascript
const routes = [
  {
    path: '/users/:id',
    name: 'UserProfile',
    component: UserProfile
  },
  {
    path: '/users/:userId/posts/:postId',
    name: 'UserPost',
    component: UserPost
  },
  {
    path: '/search/:query',
    name: 'Search',
    component: Search,
    props: true  // Route params as props
  }
]
```

### Accessing Parameters

```javascript
// In component - Option 1: $route
export default {
  mounted() {
    const userId = this.$route.params.id
    console.log(userId)
  },
  
  watch: {
    '$route'(to, from) {
      this.fetchUser(to.params.id)
    }
  }
}

// In component - Option 2: Props (when props: true)
export default {
  props: ['id'],  // Automatically receives :id param
  mounted() {
    this.fetchUser(this.id)
  }
}
```

### Query Parameters

```javascript
// Navigating with query
<router-link :to="{ path: '/users', query: { page: 1, sort: 'name' } }">
  Users
</router-link>

// Or programmatically
this.$router.push({
  path: '/users',
  query: { page: 1, sort: 'name' }
})

// Accessing query
export default {
  computed: {
    page() {
      return parseInt(this.$route.query.page) || 1
    },
    sort() {
      return this.$route.query.sort || 'name'
    }
  }
}
```

## Navigation Guards

### Guard Types

| Guard | When it runs |
|-------|--------------|
| `beforeRouteEnter` | Before route is confirmed |
| `beforeRouteUpdate` | When route changes but same component |
| `beforeRouteLeave` | Before leaving the route |

### Component Guards

```javascript
export default {
  data() {
    return {
      user: null,
      hasUnsavedChanges: false
    }
  },
  
  beforeRouteEnter(to, from, next) {
    // Called before the route is confirmed
    // Cannot access `this` (component not created yet)
    
    // Pass callback to access component after creation
    next(vm => {
      // vm is component instance
      console.log(vm.user)
    })
  },
  
  beforeRouteUpdate(to, from, next) {
    // Called when route changes but same component
    // (e.g., /users/1 -> /users/2)
    
    this.fetchUser(to.params.id)
    next()
  },
  
  beforeRouteLeave(to, from, next) {
    // Called when navigating away from this route
    
    if (this.hasUnsavedChanges) {
      const answer = window.confirm('You have unsaved changes. Leave?')
      if (answer) {
        next()
      } else {
        next(false)  // Cancel navigation
      }
    } else {
      next()
    }
  }
}
```

### Global Guards

```javascript
// router/index.js

// Before any route
router.beforeEach((to, from, next) => {
  // Check authentication
  if (to.meta.requiresAuth && !isAuthenticated()) {
    next({ name: 'Login', query: { redirect: to.fullPath } })
  } else {
    next()  // Don't forget to call next!
  }
})

// After route is confirmed
router.afterEach((to, from) => {
  // Analytics, scroll to top, etc.
  window.scrollTo(0, 0)
})

// Before route resolution
router.beforeResolve((to, from, next) => {
  // Similar to beforeEach but called after all async components resolved
  next()
})
```

### Per-Route Guards

```javascript
const routes = [
  {
    path: '/admin',
    component: Admin,
    beforeEnter: (to, from, next) => {
      // Route-specific guard
      if (!isAdmin()) {
        next('/')
      } else {
        next()
      }
    }
  }
]
```

## Route Meta Fields

```javascript
const routes = [
  {
    path: '/admin',
    component: Admin,
    meta: {
      requiresAuth: true,
      role: 'admin',
      title: 'Admin Dashboard'
    }
  }
]

// Access in guards
router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth) {
    // Handle auth
  }
  
  // Update document title
  if (to.meta.title) {
    document.title = to.meta.title
  }
  
  next()
})
```

### TypeScript Route Meta Types

```typescript
// types/router.ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    role?: 'admin' | 'user' | 'guest'
    title?: string
    keepAlive?: boolean
  }
}

// Usage in component
export default {
  mounted() {
    if (this.$route.meta.requiresAuth) {
      // TypeScript knows this exists
    }
  }
}
```

## Programmatic Navigation

```javascript
// Navigate to path
this.$router.push('/users')

// Navigate with name
this.$router.push({ name: 'UserProfile', params: { id: 1 } })

// Navigate with query
this.$router.push({ name: 'Search', query: { q: 'vue' } })

// Go back
this.$router.back()

// Go forward
this.$router.forward()

// Go with delta
this.$router.go(-1)  // Back
this.$router.go(1)   // Forward

// Replace current route (no history entry)
this.$router.replace('/new-route')
```

## Route Component Lifecycle

```javascript
// When navigating between routes with different components:

// Current component
beforeRouteLeave(to, from, next) {
  // Last chance to do something
  next()
}

// Destroyed
destroyed() {
  // Component is destroyed
}

// New component
beforeRouteEnter(to, from, next) {
  // Setup for new component
  next()
}

// Mounted
mounted() {
  // Component is ready
}
```

## Nested Routes

```javascript
const routes = [
  {
    path: '/users/:id',
    component: User,
    children: [
      {
        // No path = renders in parent's <router-view>
        path: '',
        name: 'UserProfile',
        component: UserProfile
      },
      {
        path: 'posts',
        name: 'UserPosts',
        component: UserPosts
      },
      {
        path: 'settings',
        name: 'UserSettings',
        component: UserSettings
      }
    ]
  }
]
```

```html
<!-- User.vue (parent) -->
<template>
  <div class="user">
    <div class="user-header">
      User Info
    </div>
    <div class="user-content">
      <router-view />
    </div>
  </div>
</template>
```

## Named Views

```javascript
const routes = [
  {
    path: '/',
    components: {
      default: DefaultLayout,
      sidebar: Sidebar,
      header: Header
    }
  }
]
```

```html
<router-view />  <!-- default -->
<router-view name="header" />
<router-view name="sidebar" />
```

## Lazy Loading Routes

```javascript
// Basic lazy load
const User = () => import('./views/User.vue')

// With named chunk
const User = () => import(/* webpackChunkName: "user" */ './views/User.vue')

// Lazy load with preloading
const User = () => import('./views/User.vue')
// Later, when user hovers over link:
router.match('/user').preload()
```

## Scroll Behavior

```javascript
const router = new VueRouter({
  routes,
  scrollBehavior(to, from, savedPosition) {
    // Scroll to saved position (back button)
    if (savedPosition) {
      return savedPosition
    }
    
    // Scroll to element by hash
    if (to.hash) {
      return {
        selector: to.hash
      }
    }
    
    // Scroll to top
    return { x: 0, y: 0 }
  }
})
```

## History Modes

### Hash Mode (Default)

```javascript
// URL: example.com/#/user/1
// No server configuration needed
// Not SEO friendly

const router = new VueRouter({
  mode: 'hash',
  routes
})
```

### History Mode

```javascript
// URL: example.com/user/1
// Requires server configuration
// SEO friendly

const router = new VueRouter({
  mode: 'history',
  routes
})

// Server configuration needed (nginx example)
// location / {
//   try_files $uri $uri/ /index.html;
// }
```

## Route Object Properties

```javascript
this.$route.path        // Current path
this.$route.params      // Route parameters
this.$route.query       // Query parameters
this.$route.hash        // Hash fragment
this.$route.name        // Route name
this.$route.fullPath    // Full path with query
this.$route.meta        // Meta fields
this.$route.matched     // Matched routes (for nested)
```

## Common Patterns

### 1. Authentication Guard

```javascript
router.beforeEach((to, from, next) => {
  const isAuthenticated = store.getters['auth/isAuthenticated']
  
  if (to.meta.requiresAuth && !isAuthenticated) {
    next({
      name: 'Login',
      query: { redirect: to.fullPath }
    })
  } else {
    next()
  }
})
```

### 2. Data Fetching on Route Change

```javascript
export default {
  watch: {
    '$route': 'fetchData'
  },
  
  methods: {
    async fetchData() {
      this.loading = true
      this.user = await api.getUser(this.$route.params.id)
      this.loading = false
    }
  },
  
  created() {
    this.fetchData()
  }
}
```

### 3. Prevent Navigation

```javascript
export default {
  data() {
    return {
      form: { name: '' },
      hasUnsavedChanges: false
    }
  },
  
  beforeRouteLeave(to, from, next) {
    if (this.hasUnsavedChanges) {
      const answer = confirm('Leave without saving?')
      if (answer) {
        next()
      } else {
        next(false)
      }
    } else {
      next()
    }
  }
}
```

## Vue Router 2 vs Vue Router 4

| Feature | Vue Router 2 | Vue Router 4 (Vue 3) |
|---------|--------------|---------------------|
| Vue Version | Vue 2 | Vue 3 |
| Navigation | this.$router | useRouter() |
| Route | this.$route | useRoute() |
| Routes | `routes` option | `routes` option |
| Scroll | scrollBehavior | scrollBehavior |
| Meta | this.$route.meta | this.$route.meta |
| Children | children | children |

## Route Transitions

### Basic Transitions

```html
<template>
  <transition name="fade" mode="out-in">
    <router-view />
  </transition>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
```

### Slide Transitions

```html
<template>
  <transition name="slide" mode="out-in">
    <router-view :key="$route.path" />
  </transition>
</template>

<style>
.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.slide-enter {
  transform: translateX(30px);
  opacity: 0;
}

.slide-leave-to {
  transform: translateX(-30px);
  opacity: 0;
}
</style>
```

### Dynamic Transitions

```html
<template>
  <transition :name="transitionName" mode="out-in">
    <router-view />
  </transition>
</template>

<script>
export default {
  data() {
    return {
      transitionName: 'fade'
    }
  },
  
  watch: {
    '$route'(to, from) {
      // Slide from child routes
      if (to.meta.depth > from.meta.depth) {
        this.transitionName = 'slide-left'
      } else {
        this.transitionName = 'slide-right'
      }
    }
  }
}
</script>
```

### Page-Level Transitions

```html
<!-- Use transition on route components -->
<template>
  <router-view v-slot="{ Component }">
    <transition name="page" mode="out-in">
      <component :is="Component" />
    </transition>
  </router-view>
</template>

<style>
.page-enter-active,
.page-leave-active {
  transition: opacity 0.2s ease;
}

.page-enter,
.page-leave-to {
  opacity: 0;
}
</style>
```

## Anti-Patterns

### 1. Direct Mutation

```javascript
// ❌ WRONG
this.$route.params.id = 'newId'  // Read-only!

// ✅ CORRECT
this.$router.push({ params: { id: 'newId' } })
```

### 2. Not Calling Next

```javascript
// ❌ WRONG
router.beforeEach((to, from) => {
  // Forgot to call next!
})

// ✅ CORRECT
router.beforeEach((to, from, next) => {
  // ...
  next()
})
```

### 3. Memory Leaks with Watch

```javascript
// ❌ WRONG - Not cleaned up
created() {
  this.$watch('$route.params.id', this.fetchData)
}

// ✅ CORRECT - Use beforeRouteUpdate
beforeRouteUpdate(to, from, next) {
  this.fetchData(to.params.id)
  next()
}
```

### 4. Hardcoded Navigation

```javascript
// ❌ WRONG
window.location.href = '/users'

// ✅ CORRECT
this.$router.push('/users')
```

## References

See the `references/` directory for:

- Navigation guard examples
- Route transitions
- Router testing strategies
