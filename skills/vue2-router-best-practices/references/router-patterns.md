# Vue Router Patterns Reference

## Route Configuration

### Basic Routes

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
    component: Home,
    meta: {
      title: 'Home'
    }
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/views/About.vue'),
    meta: {
      title: 'About Us'
    }
  }
]

const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    }
    if (to.hash) {
      return { selector: to.hash }
    }
    return { x: 0, y: 0 }
  }
})

export default router
```

### Dynamic Routes

```javascript
// User routes
{
  path: '/users/:id',
  name: 'UserProfile',
  component: () => import('@/views/UserProfile.vue'),
  props: true  // Pass route params as props
}

// Optional params
{
  path: '/users/:id?',
  name: 'User',
  component: User
}

// Multiple params
{
  path: '/users/:userId/posts/:postId',
  name: 'UserPost',
  component: UserPost,
  props: true
}

// Regex constraints
{
  path: '/order/:id(\\d+)',
  name: 'OrderDetail',
  component: OrderDetail
}
```

## Navigation Guards

### Route Meta

```javascript
// Define meta in route
{
  path: '/admin',
  component: Admin,
  meta: {
    requiresAuth: true,
    role: 'admin',
    title: 'Admin Panel'
  }
}

// Use in global guard
router.beforeEach((to, from, next) => {
  // Check auth
  if (to.meta.requiresAuth && !isAuthenticated()) {
    next({
      name: 'Login',
      query: { redirect: to.fullPath }
    })
    return
  }
  
  // Check role
  if (to.meta.role && !hasRole(to.meta.role)) {
    next({ name: 'Forbidden' })
    return
  }
  
  // Update title
  document.title = to.meta.title || 'Default Title'
  
  next()
})
```

### Component Guards

```javascript
export default {
  data() {
    return {
      user: null
    }
  },
  
  beforeRouteEnter(to, from, next) {
    // Cannot access `this` yet
    // Use callback to get component instance
    next(vm => {
      vm.fetchUser(to.params.id)
    })
  },
  
  beforeRouteUpdate(to, from, next) {
    // Called when route changes but same component
    // Good for /users/:id -> /users/:id
    this.fetchUser(to.params.id)
    next()
  },
  
  beforeRouteLeave(to, from, next) {
    // Prevent leaving with unsaved changes
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

## Nested Routes

```javascript
{
  path: '/dashboard',
  component: DashboardLayout,
  children: [
    {
      path: '',
      name: 'DashboardHome',
      component: DashboardHome,
      meta: { title: 'Dashboard' }
    },
    {
      path: 'settings',
      name: 'DashboardSettings',
      component: DashboardSettings,
      meta: { title: 'Settings' }
    },
    {
      path: 'analytics',
      name: 'DashboardAnalytics',
      component: DashboardAnalytics,
      meta: { title: 'Analytics' }
    }
  ]
}
```

```html
<!-- DashboardLayout.vue -->
<template>
  <div class="dashboard">
    <aside class="sidebar">
      <router-link to="/dashboard">Home</router-link>
      <router-link to="/dashboard/settings">Settings</router-link>
    </aside>
    <main class="content">
      <router-view />
    </main>
  </div>
</template>
```

## Named Views

```javascript
{
  path: '/',
  components: {
    default: MainLayout,
    header: Header,
    sidebar: Sidebar,
    footer: Footer
  }
}
```

```html
<router-view />  <!-- default -->
<router-view name="header" />
<router-view name="sidebar" />
<router-view name="footer" />
```

## Route Transitions

```html
<template>
  <transition name="fade" mode="out-in">
    <router-view />
  </transition>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
```

```html
<!-- With named views -->
<transition name="slide" mode="out-in">
  <router-view key="unique" />
</transition>
```

## Lazy Loading

### Basic Lazy Load

```javascript
const User = () => import('./views/User.vue')
const Admin = () => import(/* webpackChunkName: "admin" */ './views/Admin.vue')
```

### Named Chunks

```javascript
const UserProfile = () => import(/* webpackChunkName: "user" */ './UserProfile.vue')
const UserSettings = () => import(/* webpackChunkName: "user" */ './UserSettings.vue')

// Both will be in same chunk: user.js
```

### Prefetching

```javascript
// In router
const User = () => import('./User.vue')

// Later, trigger prefetch
router.getMatchedComponents().forEach(component => {
  if (component.prefetch) {
    component.prefetch()
  }
})
```

## Programmatic Navigation

```javascript
// Navigate to path
this.$router.push('/users')

// Navigate with params
this.$router.push({ name: 'User', params: { id: 123 } })

// Navigate with query
this.$router.push({ path: '/users', query: { page: 2 } })

// Replace current route
this.$router.replace('/new-route')

// Go back/forward
this.$router.go(-1)
this.$router.go(1)

// Named routes
this.$router.push({ name: 'UserProfile', params: { id: user.id } })
```

## Route Object Properties

```javascript
// Inside component
this.$route.path        // '/users/123'
this.$route.params      // { id: '123' }
this.$route.query       // { page: '2' }
this.$route.hash        // '#section'
this.$route.name        // 'UserProfile'
this.$route.fullPath    // '/users/123?page=2#section'
this.$route.matched     // Array of matched routes
this.$route.meta        // Meta from route
```

## Navigation with Callbacks

```javascript
// Promise-based navigation
this.$router.push('/users')
  .then(() => {
    // Navigation successful
  })
  .catch(err => {
    // Navigation prevented
  })

// Async navigation (Vue Router 2.2+)
this.$router.push('/users', () => {}, (err) => {
  console.log(err)
})
```

## Scroll Behavior

```javascript
const router = new VueRouter({
  scrollBehavior(to, from, savedPosition) {
    // 1. Saved position (back button)
    if (savedPosition) {
      return savedPosition
    }
    
    // 2. Hash target
    if (to.hash) {
      return {
        selector: to.hash,
        offset: { x: 0, y: 10 }
      }
    }
    
    // 3. Scroll to top
    return { x: 0, y: 0 }
  }
})
```

## History Modes

### Hash Mode (Default)

```javascript
// URL: http://example.com/#/user/123
// Works without server config
// Not SEO friendly

const router = new VueRouter({
  mode: 'hash',
  routes
})
```

### History Mode (Requires Server)

```javascript
// URL: http://example.com/user/123
// SEO friendly
// Requires server-side config

const router = new VueRouter({
  mode: 'history',
  routes
})

// Server must redirect all requests to index.html
```

### Nginx Configuration

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

### Apache Configuration

```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

## Data Fetching

### Fetch on Navigation

```javascript
export default {
  data() {
    return {
      user: null,
      loading: false
    }
  },
  
  watch: {
    '$route': 'fetchUser'
  },
  
  created() {
    this.fetchUser()
  },
  
  methods: {
    async fetchUser() {
      this.loading = true
      const id = this.$route.params.id
      this.user = await api.getUser(id)
      this.loading = false
    }
  }
}
```

### Fetch with Navigation Guards

```javascript
{
  path: '/users/:id',
  component: UserProfile,
  beforeEnter(to, from, next) {
    fetchUser(to.params.id)
      .then(user => {
        to.params.user = user  // Pass data through route
        next()
      })
      .catch(err => {
        next({ name: 'NotFound' })
      })
  }
}
```

## Route Matching

### Catch All

```javascript
// 404 route
{
  path: '*',
  name: 'NotFound',
  component: NotFound
}

// With params
{
  path: '/user-*',
  component: UserHome
}
```

### Multiple Segments

```javascript
// Match /guide, /guide/vue, /guide/vue/reactivity
{
  path: '/guide/*',
  component: Guide
}
```

## Route Aliases

```javascript
{
  path: '/users',
  component: UserList,
  alias: '/people'  // Both /users and /people work
}

{
  path: '/home',
  component: Home,
  alias: ['/', '/homepage']  // Multiple aliases
}
```

## Redirects

```javascript
// Basic redirect
{ path: '/old', redirect: '/new' }

// Redirect with params
{ path: '/old/:id', redirect: to => `/new/${to.params.id}` }

// Redirect with name
{ path: '/home', redirect: { name: 'Dashboard' } }

// Global redirect (catch all)
{ path: '*', redirect: '/404' }
```

## Common Patterns

### Tab Navigation

```html
<!-- With params -->
<router-link :to="{ name: 'UserTab', params: { tab: 'posts' } }">Posts</router-link>
<router-link :to="{ name: 'UserTab', params: { tab: 'followers' } }">Followers</router-link>

<!-- Route -->
{
  path: '/user/:id/:tab',
  name: 'UserTab',
  component: UserTab
}
```

### Search with Query Params

```html
<template>
  <div>
    <input v-model="search" @input="updateSearch" />
    <router-link 
      :to="{ query: { q: search } }" 
      class="btn"
    >
      Search
    </router-link>
  </div>
</template>

<script>
export default {
  data() {
    return {
      search: this.$route.query.q || ''
    }
  },
  
  methods: {
    updateSearch() {
      this.$router.push({
        query: { q: this.search }
      })
    }
  },
  
  watch: {
    '$route.query.q'(newQuery) {
      this.search = newQuery || ''
      this.fetchResults()
    }
  }
}
</script>
```

## Testing Routes

```javascript
import { createLocalVue } from '@vue/test-utils'
import VueRouter from 'vue-router'

test('navigates correctly', async () => {
  const localVue = createLocalVue()
  localVue.use(VueRouter)
  
  const router = new VueRouter({
    routes: [
      { path: '/', name: 'Home', component: { template: '<div>Home</div>' } },
      { path: '/about', name: 'About', component: { template: '<div>About</div>' } }
    ]
  })
  
  const wrapper = mount(App, {
    localVue,
    router
  })
  
  await router.push('/about')
  
  expect(wrapper.text()).toContain('About')
})
```
