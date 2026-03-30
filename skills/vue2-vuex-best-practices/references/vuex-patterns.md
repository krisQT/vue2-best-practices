# Vuex Module Patterns Reference

## Module Structure

```
store/
├── index.js           # Root store
├── modules/
│   ├── auth.js        # Authentication module
│   ├── users.js       # User management module
│   ├── products.js    # Product module
│   └── cart.js        # Shopping cart module
└── plugins/
    ├── logger.js      # Development logger
    └── persist.js     # State persistence
```

## Basic Module

```javascript
// modules/auth.js
export default {
  namespaced: true,
  
  state: {
    user: null,
    token: null,
    loading: false,
    error: null
  },
  
  getters: {
    isAuthenticated: state => !!state.token,
    currentUser: state => state.user,
    hasError: state => !!state.error
  },
  
  mutations: {
    SET_USER(state, user) {
      state.user = user
    },
    
    SET_TOKEN(state, token) {
      state.token = token
    },
    
    SET_LOADING(state, loading) {
      state.loading = loading
    },
    
    SET_ERROR(state, error) {
      state.error = error
    },
    
    CLEAR_AUTH(state) {
      state.user = null
      state.token = null
      state.error = null
    }
  },
  
  actions: {
    async login({ commit }, credentials) {
      commit('SET_LOADING', true)
      commit('SET_ERROR', null)
      
      try {
        const response = await authAPI.login(credentials)
        commit('SET_TOKEN', response.token)
        commit('SET_USER', response.user)
        return response
      } catch (error) {
        commit('SET_ERROR', error.message)
        throw error
      } finally {
        commit('SET_LOADING', false)
      }
    },
    
    async logout({ commit }) {
      await authAPI.logout()
      commit('CLEAR_AUTH')
    },
    
    async fetchCurrentUser({ commit, state }) {
      if (!state.token) return null
      
      commit('SET_LOADING', true)
      try {
        const user = await authAPI.getCurrentUser()
        commit('SET_USER', user)
        return user
      } catch (error) {
        commit('CLEAR_AUTH')
        throw error
      } finally {
        commit('SET_LOADING', false)
      }
    }
  }
}
```

## Store Setup

```javascript
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'
import auth from './modules/auth'
import users from './modules/users'
import products from './modules/products'
import cart from './modules/cart'

Vue.use(Vuex)

export default new Vuex.Store({
  strict: process.env.NODE_ENV !== 'production',
  
  modules: {
    auth,
    users,
    products,
    cart
  },
  
  state: {
    // Root state (shared across modules)
    appVersion: '1.0.0',
    notifications: []
  },
  
  mutations: {
    ADD_NOTIFICATION(state, notification) {
      state.notifications.push({
        id: Date.now(),
        ...notification
      })
    },
    
    REMOVE_NOTIFICATION(state, id) {
      const index = state.notifications.findIndex(n => n.id === id)
      if (index > -1) {
        state.notifications.splice(index, 1)
      }
    }
  },
  
  actions: {
    notify({ commit }, notification) {
      commit('ADD_NOTIFICATION', notification)
      
      // Auto remove after timeout
      setTimeout(() => {
        commit('REMOVE_NOTIFICATION', notification.id)
      }, notification.duration || 3000)
    }
  },
  
  getters: {
    appVersion: state => state.appVersion,
    notifications: state => state.notifications
  }
})
```

## Module with Submodules

```javascript
// modules/products.js
import categories from './products/categories'
import items from './products/items'

export default {
  namespaced: true,
  
  modules: {
    categories,
    items
  },
  
  state: {
    selectedCategoryId: null
  },
  
  getters: {
    selectedCategory(state, getters, rootState) {
      return state.selectedCategoryId
    }
  },
  
  mutations: {
    SET_CATEGORY(state, id) {
      state.selectedCategoryId = id
    }
  },
  
  actions: {
    selectCategory({ commit, dispatch }, id) {
      commit('SET_CATEGORY', id)
      dispatch('items/fetchItems', { categoryId: id }, { root: true })
    }
  }
}
```

## Cross-Module Communication

### Root State Access

```javascript
// In any module
export default {
  actions: {
    someAction({ state, rootState, commit }) {
      console.log(rootState.appVersion)  // Root state
      
      // Another module's state
      console.log(rootState.auth.user)
      
      // Dispatch another module's action
      commit('auth/SET_USER', user, { root: true })
    }
  },
  
  getters: {
    computed(state, getters, rootState, rootGetters) {
      return rootGetters['auth/isAuthenticated']
    }
  }
}
```

### Module Action Dispatch

```javascript
// Dispatch from one module to another
export default {
  actions: {
    async checkout({ dispatch, commit }) {
      // Dispatch action in another module
      await dispatch('cart/clearCart', null, { root: true })
      
      // Commit mutation in another module
      commit('orders/addOrder', order, { root: true })
    }
  }
}
```

## Plugins

### Logger Plugin

```javascript
// plugins/logger.js
export default function loggerPlugin(store) {
  // Subscribe to mutations
  store.subscribe((mutation, state) => {
    console.log('%c Mutation ', 'background: #41b883; color: white;', mutation.type)
    console.log('Payload:', mutation.payload)
    console.log('State after:', state)
  })
  
  // Subscribe to actions
  store.subscribeAction((action, state) => {
    console.log('%c Action ', 'background: #35495e; color: white;', action.type)
    console.log('Payload:', action.payload)
  })
}
```

### Persistence Plugin

```javascript
// plugins/persist.js
export default function persistPlugin(store) {
  // Load initial state
  if (localStorage.getItem('vuex_state')) {
    const savedState = JSON.parse(localStorage.getItem('vuex_state'))
    store.replaceState(savedState)
  }
  
  // Subscribe to changes
  store.subscribe((mutation, state) => {
    localStorage.setItem('vuex_state', JSON.stringify(state))
  })
}
```

### HMR Support

```javascript
// store/hot-update.js
if (module.hot) {
  module.hot.accept([
    './modules/auth',
    './modules/users'
  ], () => {
    const _auth = require('./modules/auth').default
    const _users = require('./modules/users').default
    
    store.hotUpdate({
      modules: {
        auth: _auth,
        users: _users
      }
    })
  })
}
```

## State Hydration

### From LocalStorage

```javascript
// modules/auth.js
export default {
  state: {
    user: null,
    token: null
  },
  
  mutations: {
    HYDRATE_AUTH(state, authData) {
      state.user = authData.user
      state.token = authData.token
    }
  },
  
  actions: {
    hydrateAuth({ commit }) {
      const savedAuth = localStorage.getItem('auth')
      if (savedAuth) {
        try {
          const authData = JSON.parse(savedAuth)
          commit('HYDRATE_AUTH', authData)
        } catch (e) {
          localStorage.removeItem('auth')
        }
      }
    },
    
    saveAuth({ state }) {
      const authData = {
        user: state.user,
        token: state.token
      }
      localStorage.setItem('auth', JSON.stringify(authData))
    }
  }
}
```

### From API (SSR)

```javascript
// In Vuex store
const store = new Vuex.Store({
  state: {
    initialState: window.__INITIAL_STATE__ || {}
  }
})

// On server
context.state = store.state
return context
```

## Performance Patterns

### Lazy Module Registration

```javascript
// store/index.js
export default new Vuex.Store({
  modules: {
    // Only load essential modules initially
  }
})

// Load heavy modules on demand
async function loadAdminModule(store) {
  const module = await import('./modules/admin')
  store.registerModule('admin', module.default)
}
```

### Getter Caching

```javascript
getters: {
  // ❌ BAD - Creates new array every time
  activeUsers: state => state.users.filter(u => u.active)
  
  // ✅ GOOD - Use memoization or computed
  // Vue automatically caches based on dependencies
  activeUsers(state) {
    return state.users.filter(u => u.active)
  }
}
```

### Batch Mutations

```javascript
// ❌ BAD - Multiple mutations
actions: {
  updateUser({ commit }, data) {
    commit('SET_NAME', data.name)
    commit('SET_EMAIL', data.email)
    commit('SET_AVATAR', data.avatar)
  }
}

// ✅ GOOD - Single mutation
mutations: {
  SET_USER(state, data) {
    Object.assign(state.user, data)
  }
}

actions: {
  updateUser({ commit }, data) {
    commit('SET_USER', data)
  }
}
```

## Testing Vuex

### Test Store

```javascript
import { createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'
import auth from '@/store/modules/auth'

describe('auth module', () => {
  let store
  let apiMock
  
  beforeEach(() => {
    const localVue = createLocalVue()
    localVue.use(Vuex)
    
    apiMock = {
      login: jest.fn(),
      logout: jest.fn(),
      getCurrentUser: jest.fn()
    }
    
    store = new Vuex.Store({
      modules: {
        auth: {
          ...auth,
          actions: {
            ...auth.actions,
            // Mock API calls
            async login({ commit }, credentials) {
              const response = await apiMock.login(credentials)
              commit('SET_USER', response.user)
              return response
            }
          }
        }
      }
    })
  })
  
  test('login action commits user', async () => {
    const mockUser = { id: 1, name: 'John' }
    apiMock.login.mockResolvedValue({
      token: 'token123',
      user: mockUser
    })
    
    await store.dispatch('auth/login', { email: 'test@test.com' })
    
    expect(store.state.auth.user).toEqual(mockUser)
  })
})
```

## Common Patterns Summary

| Pattern | Use Case |
|---------|----------|
| Namespaced modules | Large apps with multiple developers |
| Root state | Truly global data |
| Cross-module | Related but separate concerns |
| Plugins | Cross-cutting functionality |
| Hydration | Persistence, SSR |
| Lazy loading | Performance optimization |
