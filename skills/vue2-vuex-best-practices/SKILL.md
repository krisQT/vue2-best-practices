# Vuex Best Practices

## When to Use

Use this skill when:

- Managing complex application state
- Using Vuex in Vue 2 projects
- Implementing state management patterns
- Building scalable Vue 2 applications

## Activation Triggers

Use this skill when the user prompt contains any of these patterns:

- "Vuex"
- "state management"
- "Vuex store"
- "Vuex mutation"
- "Vuex action"
- "Vuex getter"
- "Vuex module"
- "namespaced Vuex"
- "mapState"
- "mapGetters"
- "mapActions"
- "mapMutations"
- "commit mutation"
- "dispatch action"
- "Vuex strict mode"
- "Vuex plugin"
- "Vuex persistence"
- "Vuex hot reload"
- "store state"
- "centralized state"
- "shared state Vue"
- "Vuex CRUD"
- "Vuex async"

Keywords: `new Vuex.Store`, `this.$store`, `commit(`, `dispatch(`, `mapState`, `mapGetters`

## Vuex Core Concepts

```
┌─────────────┐
│   Vuex Store │
└──────┬──────┘
       │
       ├── State (Single source of truth)
       │    │
       │    └── Components read state
       │
       ├── Getters (Computed state)
       │    │
       │    └── Components access computed values
       │
       ├── Mutations (Sync state changes)
       │    │
       │    └── Components commit mutations
       │
       ├── Actions (Async operations)
       │    │
       │    └── Components dispatch actions
       │
       └── Modules (Organize store)
              │
              └── Split store into namespaces
```

## Basic Store Setup

```javascript
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    count: 0,
    users: []
  },
  
  getters: {
    doubleCount: state => state.count * 2,
    getUserById: state => id => state.users.find(user => user.id === id)
  },
  
  mutations: {
    SET_COUNT(state, count) {
      state.count = count
    },
    ADD_USER(state, user) {
      state.users.push(user)
    }
  },
  
  actions: {
    async fetchUsers({ commit }) {
      const users = await api.getUsers()
      commit('ADD_USER', users)
    }
  },
  
  modules: {}
})
```

## State

### Defining State

```javascript
// store/index.js
export default new Vuex.Store({
  state: {
    // Primitive values
    loading: false,
    error: null,
    
    // Objects
    user: null,
    
    // Arrays
    items: [],
    
    // Nested state
    cart: {
      items: [],
      total: 0
    }
  }
})
```

### Accessing State

```javascript
// In component - Option 1: Using $store
export default {
  computed: {
    count() {
      return this.$store.state.count
    },
    
    // Multiple state values
    ...mapState(['loading', 'error', 'user'])
    
    // With different names
    ...mapState({
      myLoading: 'loading',
      currentUser: 'user'
    })
    
    // Using functions
    ...mapState({
      count: state => state.count,
      userName: state => state.user?.name || 'Guest'
    })
  }
}
```

## Getters

### Basic Getters

```javascript
export default new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: 'Learn Vue', done: true },
      { id: 2, text: 'Learn Vuex', done: false }
    ]
  },
  
  getters: {
    // Simple getter
    doneTodos: state => state.todos.filter(todo => todo.done),
    
    // Getter with parameters
    getTodoById: state => id => state.todos.find(todo => todo.id === id),
    
    // Getter using other getters
    completedCount: (state, getters) => getters.doneTodos.length
  }
})
```

### Accessing Getters

```javascript
export default {
  computed: {
    // Single getter
    doneTodos() {
      return this.$store.getters.doneTodos
    },
    
    // Multiple getters
    ...mapGetters(['doneTodos', 'completedCount'])
    
    // With aliases
    ...mapGetters({
      completed: 'doneTodos',
      count: 'completedCount'
    })
  }
}
```

## Mutations

### Defining Mutations

```javascript
export default new Vuex.Store({
  state: {
    count: 0
  },
  
  mutations: {
    // Simple mutation
    INCREMENT(state) {
      state.count++
    },
    
    // Mutation with payload
    SET_COUNT(state, count) {
      state.count = count
    },
    
    // Mutation with object payload
    UPDATE_USER(state, payload) {
      state.user.name = payload.name
      state.user.email = payload.email
    }
  }
})
```

### Committing Mutations

```javascript
export default {
  methods: {
    increment() {
      this.$store.commit('INCREMENT')
    },
    
    setCount(count) {
      this.$store.commit('SET_COUNT', count)
    },
    
    updateUser(payload) {
      // Object shorthand
      this.$store.commit({
        type: 'UPDATE_USER',
        name: 'John',
        email: 'john@example.com'
      })
    }
  }
}
```

### Important Rules

1. **Mutations must be synchronous**
2. **Always use constants for mutation types**
3. **Use Vue.set for adding properties**

```javascript
// ❌ WRONG - Mutation must be synchronous
INCREMENT(state) {
  setTimeout(() => {
    state.count++  // Don't do this in mutation
  }, 1000)
}

// ✅ CORRECT - Use action for async
async INCREMENT(state) {
  const result = await api.getCount()
  state.count = result
}
```

## Actions

### Defining Actions

```javascript
export default new Vuex.Store({
  mutations: {
    SET_USER(state, user) {
      state.user = user
    }
  },
  
  actions: {
    // Context object has: commit, dispatch, state, getters, rootState, rootGetters
    async fetchUser({ commit, state }, userId) {
      try {
        const user = await api.getUser(userId)
        commit('SET_USER', user)
        return user
      } catch (error) {
        throw error
      }
    },
    
    async fetchMultipleUsers({ dispatch }, userIds) {
      // Dispatch other actions
      const users = await Promise.all(
        userIds.map(id => dispatch('fetchUser', id))
      )
      return users
    }
  }
})
```

### Dispatching Actions

```javascript
export default {
  methods: {
    async loadUser() {
      try {
        const user = await this.$store.dispatch('fetchUser', 1)
        console.log(user)
      } catch (error) {
        console.error('Failed to load user:', error)
      }
    }
  }
}
```

### Using mapActions

```javascript
import { mapActions } from 'vuex'

export default {
  methods: {
    // Map single action
    ...mapActions(['fetchUser', 'fetchUsers']),
    
    // Map with different names
    ...mapActions({
      loadUser: 'fetchUser',
      loadAllUsers: 'fetchUsers'
    })
  }
}
```

## Modules

### Basic Module Structure

```javascript
// store/modules/user.js
const state = {
  profile: null,
  loading: false
}

const getters = {
  isLoggedIn: state => !!state.profile
}

const mutations = {
  SET_PROFILE(state, profile) {
    state.profile = profile
  },
  SET_LOADING(state, loading) {
    state.loading = loading
  }
}

const actions = {
  async login({ commit }, credentials) {
    commit('SET_LOADING', true)
    try {
      const profile = await api.login(credentials)
      commit('SET_PROFILE', profile)
    } finally {
      commit('SET_LOADING', false)
    }
  }
}

export default {
  namespaced: true,  // Enable namespacing
  state,
  getters,
  mutations,
  actions
}
```

### Registering Modules

```javascript
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'
import user from './modules/user'
import cart from './modules/cart'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {},
  getters: {},
  mutations: {},
  actions: {},
  modules: {
    user,
    cart
  }
})
```

### Access Namespaced Module

```javascript
export default {
  computed: {
    // With namespaced module
    ...mapState('user', {
      profile: state => state.profile,
      loading: state => state.loading
    }),
    
    ...mapGetters('user', ['isLoggedIn']),
    
    // Access without mapping
    profile() {
      return this.$store.state.user.profile
    }
  },
  
  methods: {
    ...mapActions('user', ['login']),
    
    // Call directly
    async handleLogin() {
      await this.$store.dispatch('user/login', credentials)
    }
  }
}
```

## Vuex Store Setup in Vue

```javascript
// main.js
import Vue from 'vue'
import store from './store'

new Vue({
  store,
  render: h => h(App)
}).$mount('#app')
```

## Getters Patterns

### Filtering

```javascript
getters: {
  activeUsers: state => state.users.filter(user => user.active),
  adminUsers: (state, getters) => getters.activeUsers.filter(user => user.role === 'admin')
}
```

### Grouping

```javascript
getters: {
  groupByRole: state => {
    return state.users.reduce((groups, user) => {
      const role = user.role
      if (!groups[role]) {
        groups[role] = []
      }
      groups[role].push(user)
      return groups
    }, {})
  }
}
```

## Vuex with Local Storage

```javascript
// plugins/persist.js
export default function persistPlugin(store) {
  // Load from localStorage on init
  if (localStorage.getItem('vuex')) {
    store.replaceState(JSON.parse(localStorage.getItem('vuex')))
  }
  
  // Subscribe to mutations
  store.subscribe((mutation, state) => {
    localStorage.setItem('vuex', JSON.stringify(state))
  })
}

// store/index.js
import createPersistedState from 'vuex-persistedstate'
import * as Cookies from 'js-cookie'

export default new Vuex.Store({
  // ... state, mutations, actions, getters, modules
  plugins: [
    createPersistedState({
      storage: window.localStorage,
      paths: ['user', 'cart.items']  // Only persist specific paths
    })
  ]
})
```

## Vuex Strict Mode

```javascript
export default new Vuex.Store({
  // ...
  strict: process.env.NODE_ENV !== 'production'
})
```

Strict mode throws errors when state is mutated outside mutations.

```javascript
// ❌ WRONG - Will throw error in strict mode
computed: {
  count() {
    return this.$store.state.count++
  }
}

// ✅ CORRECT - Commit mutation
methods: {
  increment() {
    this.$store.commit('INCREMENT')
  }
}
```

## Common Patterns

### 1. CRUD Operations

```javascript
// actions
async createItem({ commit }, item) {
  const created = await api.createItem(item)
  commit('ADD_ITEM', created)
  return created
},

async updateItem({ commit }, { id, data }) {
  const updated = await api.updateItem(id, data)
  commit('UPDATE_ITEM', updated)
  return updated
},

async deleteItem({ commit }, id) {
  await api.deleteItem(id)
  commit('DELETE_ITEM', id)
  return id
}
```

### 2. Form Handling

```javascript
// store/modules/form.js
const state = {
  fields: {},
  errors: {},
  submitting: false
}

const mutations = {
  SET_FIELD(state, { field, value }) {
    Vue.set(state.fields, field, value)
  },
  SET_ERRORS(state, errors) {
    state.errors = errors
  },
  CLEAR_FORM(state) {
    state.fields = {}
    state.errors = {}
  }
}

const actions = {
  async submitForm({ commit, state }) {
    commit('SET_ERRORS', {})
    try {
      await api.submitForm(state.fields)
      commit('CLEAR_FORM')
      return true
    } catch (error) {
      commit('SET_ERRORS', error.response.data.errors)
      return false
    }
  }
}
```

### 3. Loading States

```javascript
// store/modules/data.js
const state = {
  loading: false,
  error: null,
  data: null
}

const mutations = {
  SET_LOADING(state, loading) {
    state.loading = loading
  },
  SET_ERROR(state, error) {
    state.error = error
    state.loading = false
  },
  SET_DATA(state, data) {
    state.data = data
    state.loading = false
    state.error = null
  }
}

const actions = {
  async fetchData({ commit }) {
    commit('SET_LOADING', true)
    try {
      const data = await api.getData()
      commit('SET_DATA', data)
    } catch (error) {
      commit('SET_ERROR', error.message)
    }
  }
}
```

## Vuex vs Vue 2 Reactivity

Remember Vue 2 reactivity limitations:

```javascript
// ❌ WRONG - Won't be reactive
mutations: {
  ADD_ITEM(state) {
    state.newItem = {}  // Just adding property
  }
}

// ✅ CORRECT - Use Vue.set
mutations: {
  ADD_ITEM(state) {
    Vue.set(state, 'newItem', {})
    // Or: state.newItem = {} and ensure it was initialized in state
  }
}
```

## Testing Vuex

```javascript
import { createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'
import user from '@/store/modules/user'

describe('user module', () => {
  let store
  
  beforeEach(() => {
    const localVue = createLocalVue()
    localVue.use(Vuex)
    
    store = new Vuex.Store({
      modules: {
        user: {
          ...user,
          state: {
            profile: null,
            loading: false
          }
        }
      }
    })
  })
  
  test('login action commits SET_PROFILE', async () => {
    const mockUser = { id: 1, name: 'John' }
    api.login = jest.fn().mockResolvedValue(mockUser)
    
    await store.dispatch('user/login', {})
    
    expect(store.state.user.profile).toEqual(mockUser)
  })
})
```

## Anti-Patterns

### 1. Mutations in Actions (Without Committing)

```javascript
// ❌ WRONG
actions: {
  async fetchUser({ state }, id) {
    state.user = await api.getUser(id)  // Direct mutation!
  }
}

// ✅ CORRECT
actions: {
  async fetchUser({ commit }, id) {
    const user = await api.getUser(id)
    commit('SET_USER', user)
  }
}
```

### 2. Not Using Namespaced Modules

```javascript
// ❌ WRONG - Potential name collisions
mutations: {
  SET_USER() {},
  SET_PRODUCT() {}  // Conflicting name?
}

// ✅ CORRECT
export default {
  namespaced: true,
  mutations: {
    SET() {}  // Clear namespace
  }
}
```

### 3. Getting All State Manually

```javascript
// ❌ WRONG
export default {
  computed: {
    user() {
      return this.$store.state.user
    },
    products() {
      return this.$store.state.products
    },
    cart() {
      return this.$store.state.cart
    }
  }
}

// ✅ CORRECT
import { mapState } from 'vuex'

export default {
  computed: {
    ...mapState(['user', 'products', 'cart'])
  }
}
```

### 4. Complex Logic in Components

```javascript
// ❌ WRONG
export default {
  computed: {
    totalPrice() {
      return this.$store.state.cart.items.reduce((total, item) => {
        return total + (item.price * item.quantity)
      }, 0)
    }
  }
}

// ✅ CORRECT - Put in getter
// store/getters.js
const getters = {
  totalPrice: state => state.cart.items.reduce((total, item) => {
    return total + (item.price * item.quantity)
  }, 0)
}

// Component
computed: {
  ...mapGetters(['totalPrice'])
}
```

## References

See the `references/` directory for:

- Vuex module patterns
- State hydration strategies
- Performance optimization
