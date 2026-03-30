# Vue 2 Testing Patterns Reference

## Component Testing

### Setup Test Environment

```javascript
// tests/setup.js
import Vue from 'vue'
import Vuex from 'vuex'
import VueRouter from 'vue-router'
import { createLocalVue } from '@vue/test-utils'

export function createVue() {
  const localVue = createLocalVue()
  localVue.use(Vuex)
  localVue.use(VueRouter)
  return localVue
}

export function createStore(options = {}) {
  return new Vuex.Store(options)
}

export function createRouter(options = {}) {
  return new VueRouter({
    routes: [
      { path: '/', name: 'Home', component: { template: '<div>Home</div>' } }
    ],
    ...options
  })
}
```

## Common Test Patterns

### Testing User Interaction

```javascript
import { mount, createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'

describe('Counter', () => {
  test('increments on button click', async () => {
    const wrapper = mount({
      template: `
        <div>
          <span class="count">{{ count }}</span>
          <button @click="increment">Increment</button>
        </div>
      `,
      data() {
        return { count: 0 }
      },
      methods: {
        increment() {
          this.count++
        }
      }
    })
    
    await wrapper.find('button').trigger('click')
    
    expect(wrapper.find('.count').text()).toBe('1')
  })
  
  test('handles form submission', async () => {
    const wrapper = mount({
      template: `
        <form @submit.prevent="submit">
          <input v-model="email" />
          <button type="submit">Submit</button>
        </form>
      `,
      data() {
        return { email: '' }
      },
      methods: {
        submit() {
          this.$emit('submit', this.email)
        }
      }
    })
    
    await wrapper.find('input').setValue('test@example.com')
    await wrapper.find('form').trigger('submit')
    
    expect(wrapper.emitted('submit')[0]).toEqual(['test@example.com'])
  })
})
```

### Testing API Calls

```javascript
import { mount } from '@vue/test-utils'
import axios from 'axios'

jest.mock('axios')

describe('UserCard', () => {
  test('fetches user on mount', async () => {
    const mockUser = { id: 1, name: 'John' }
    axios.get.mockResolvedValue({ data: mockUser })
    
    const wrapper = mount(UserCard, {
      propsData: { userId: 1 }
    })
    
    await wrapper.vm.$nextTick()
    
    expect(axios.get).toHaveBeenCalledWith('/api/users/1')
    expect(wrapper.vm.user).toEqual(mockUser)
  })
  
  test('handles API error', async () => {
    axios.get.mockRejectedValue(new Error('Network error'))
    
    const wrapper = mount(UserCard, {
      propsData: { userId: 1 }
    })
    
    await wrapper.vm.$nextTick()
    
    expect(wrapper.vm.error).toBe('Network error')
  })
})
```

### Testing Vuex Integration

```javascript
import { mount, createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'

describe('Cart Component', () => {
  let store
  let actions
  
  beforeEach(() => {
    actions = {
      addToCart: jest.fn(),
      removeFromCart: jest.fn()
    }
    
    store = new Vuex.Store({
      modules: {
        cart: {
          namespaced: true,
          state: {
            items: [{ id: 1, name: 'Product', price: 10 }]
          },
          getters: {
            totalPrice: state => state.items.reduce((sum, item) => sum + item.price, 0)
          },
          actions
        }
      }
    })
  })
  
  test('displays cart items', () => {
    const wrapper = mount(Cart, { store })
    
    expect(wrapper.find('.item').text()).toContain('Product')
  })
  
  test('dispatches addToCart action', async () => {
    const wrapper = mount(Cart, { store })
    
    await wrapper.find('.add-button').trigger('click')
    
    expect(actions.addToCart).toHaveBeenCalled()
  })
})
```

## Mock Patterns

### Mock Global Objects

```javascript
test('uses $route', () => {
  const wrapper = mount(Component, {
    mocks: {
      $route: {
        params: { id: '123' }
      }
    }
  })
  
  expect(wrapper.vm.$route.params.id).toBe('123')
})

test('uses $store', () => {
  const mockGetters = {
    'user/isLoggedIn': () => true
  }
  
  const store = new Vuex.Store({
    getters: mockGetters
  })
  
  const wrapper = mount(Component, { store })
})
```

### Mock Timers

```javascript
test('debounces search', async () => {
  jest.useFakeTimers()
  
  const wrapper = mount({
    template: `
      <div>
        <input @input="handleInput" />
        <span v-if="showResult">Result</span>
      </div>
    `,
    data() {
      return { showResult: false }
    },
    methods: {
      handleInput() {
        setTimeout(() => {
          this.showResult = true
        }, 300)
      }
    }
  })
  
  await wrapper.find('input').setValue('test')
  
  // Not shown yet (debounced)
  expect(wrapper.find('span').exists()).toBe(false)
  
  // Fast-forward time
  jest.runAllTimers()
  
  // Now shown
  expect(wrapper.find('span').exists()).toBe(true)
  
  jest.useRealTimers()
})
```

### Mock Methods

```javascript
test('calls external method', () => {
  const wrapper = mount({
    template: '<div>{{ value }}</div>',
    data() {
      return { value: '' }
    },
    created() {
      this.fetchData()
    },
    methods: {
      async fetchData() {
        const data = await externalAPI.get()
        this.value = data
      }
    }
  })
  
  externalAPI.get = jest.fn().mockResolvedValue('test')
  
  // Re-mount to trigger created hook
  wrapper.vm.fetchData()
  
  expect(externalAPI.get).toHaveBeenCalled()
})
```

## Assertions

### DOM Assertions

```javascript
expect(wrapper.find('.title').exists()).toBe(true)
expect(wrapper.find('.title').text()).toBe('Hello')
expect(wrapper.find('.title').html()).toContain('<span>Hello</span>')
expect(wrapper.find('input').attributes('type')).toBe('text')
expect(wrapper.find('input').element.value).toBe('test')
expect(wrapper.find('input').classes()).toContain('active')
expect(wrapper.find('form').attributes('action')).toBe('/submit')
```

### Event Assertions

```javascript
expect(wrapper.emitted()).toHaveProperty('click')
expect(wrapper.emitted().click).toHaveLength(2)
expect(wrapper.emitted('click')[0]).toEqual([{ id: 1 }])
expect(wrapper.emitted().update[1]).toEqual([{ value: 'new' }])
```

### Vuex Assertions

```javascript
expect(store.state.user.name).toBe('John')
expect(store.getters['user/isLoggedIn']).toBe(true)
expect(store.commit).toHaveBeenCalledWith('SET_USER', expect.any(Object))
expect(store.dispatch).toHaveBeenCalledWith('user/login', credentials)
```

## Async Testing

### async/await

```javascript
test('loads data asynchronously', async () => {
  const mockData = { id: 1, name: 'Test' }
  api.getData = jest.fn().mockResolvedValue(mockData)
  
  const wrapper = mount(DataComponent)
  
  // Wait for next tick
  await wrapper.vm.$nextTick()
  
  // Or wait for specific condition
  await wrapper.vm.$nextTick(() => {
    expect(wrapper.vm.data).toEqual(mockData)
  })
})
```

### Testing Promise Resolution

```javascript
test('handles promise resolution', async () => {
  const promise = Promise.resolve({ success: true })
  
  const wrapper = mount({
    template: '<div>{{ result }}</div>',
    data() {
      return { result: null }
    },
    async mounted() {
      this.result = await promise
    }
  })
  
  await wrapper.vm.$nextTick()
  expect(wrapper.vm.result).toEqual({ success: true })
})
```

## Snapshot Testing

```javascript
test('matches snapshot', () => {
  const wrapper = mount(MyComponent, {
    propsData: { title: 'Test' }
  })
  
  expect(wrapper.html()).toMatchSnapshot()
})

// With snapshot serializers
test('matches snapshot with serializer', () => {
  const wrapper = mount(ComplexComponent)
  
  expect(wrapper.html()).toMatchSnapshot({
    timestamps: expect.any(Number)
  })
})
```

## Testing Best Practices

| Practice | Example |
|----------|---------|
| Test behavior, not implementation | Test UI changes, not internal methods |
| Use meaningful selectors | `.user-card` not `.vue-component-1` |
| Clean up after tests | `afterEach(() => wrapper.destroy())` |
| Use async/await | For async operations |
| Mock external dependencies | API calls, timers |
| Keep tests focused | One concern per test |
| Use describe blocks | Group related tests |
