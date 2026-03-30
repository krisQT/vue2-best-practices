# Vue 2 Testing Best Practices

## When to Use

Use this skill when:

- Writing unit tests for Vue 2 components
- Testing Vuex stores
- Testing Vue Router integration
- Setting up E2E tests

## Activation Triggers

Use this skill when the user prompt contains any of these patterns:

- "test Vue component"
- "Vue test utils"
- "shallowMount mount"
- "Jest Vue"
- "component testing"
- "unit test Vue"
- "integration test Vue"
- "e2e test Vue"
- "Vue testing"
- "test props"
- "test emit"
- "test event"
- "mock Vuex"
- "mock router"
- "test watcher"
- "test computed"
- "test lifecycle"
- "test slot"
- "test async"
- "Vue test"
- "snapshot test"
- "test coverage"
- "stub component"
- "mock component"
- "jest.config"
- "test-utils"
- "vue-jest"

Keywords: `describe(`, `it(`, `expect(`, `shallowMount(`, `mount(`, `wrapper`

## Testing Tools

- **Jest** - Test runner
- **Vue Test Utils** - Vue component testing library
- **Nightwatch** - E2E testing

## Setup

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  moduleFileExtensions: ['js', 'json', 'vue'],
  transform: {
    '^.+\\.vue$': 'vue-jest',
    '^.+\\.js$': 'babel-jest'
  },
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  snapshotSerializers: ['jest-serializer-vue'],
  testMatch: [
    '**/tests/unit/**/*.spec.js',
    '**/__tests__/**/*.js'
  ]
}
```

### Vue Test Utils Setup

```javascript
// tests/setup.js
import { createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'

const localVue = createLocalVue()
localVue.use(Vuex)

export { localVue }
```

## Component Testing

### Basic Component Test

```javascript
// tests/unit/MyComponent.spec.js
import { shallowMount } from '@vue/test-utils'
import MyComponent from '@/components/MyComponent.vue'

describe('MyComponent', () => {
  test('renders correctly', () => {
    const wrapper = shallowMount(MyComponent, {
      propsData: {
        title: 'Hello'
      }
    })
    
    expect(wrapper.text()).toContain('Hello')
  })
  
  test('emits event on click', () => {
    const wrapper = shallowMount(MyComponent)
    
    wrapper.find('button').trigger('click')
    
    expect(wrapper.emitted()).toHaveProperty('click')
  })
})
```

### Mounting Options

```javascript
const wrapper = mount(Component, {
  propsData: {},        // Component props
  data() {},            // Component data
  slots: {},            // Slot content
  scopedSlots: {},      // Scoped slot content
  stubs: {},            // Stub child components
  mocks: {},            // Mock injected values
  localVue,             // Local Vue instance
  attachToDocument: true // For E2E style tests
})
```

## Finding Elements

```javascript
// By CSS selector
wrapper.find('.class')
wrapper.find('#id')
wrapper.find('button')

// By component
wrapper.findComponent(ChildComponent)

// With options
wrapper.find('.item', { ref: 'second' })
wrapper.findAll('li').at(0)  // First item

// Returns
wrapper.find()    // First match (Wrapper)
wrapper.findAll() // All matches (Wrappers)
```

## Testing Props

```javascript
test('receives props correctly', () => {
  const wrapper = shallowMount(MyComponent, {
    propsData: {
      title: 'Test Title',
      count: 5
    }
  })
  
  expect(wrapper.props().title).toBe('Test Title')
  expect(wrapper.props().count).toBe(5)
})
```

## Testing Events

```javascript
test('emits click event', () => {
  const wrapper = shallowMount(MyButton)
  
  wrapper.find('button').trigger('click')
  
  expect(wrapper.emitted('click')).toBeTruthy()
})

test('emits event with payload', () => {
  const wrapper = shallowMount(MyButton)
  
  wrapper.vm.$emit('update', 42)
  
  expect(wrapper.emitted('update')[0]).toEqual([42])
})

test('emits multiple times', () => {
  const wrapper = shallowMount(Counter)
  
  wrapper.find('button').trigger('click')
  wrapper.find('button').trigger('click')
  wrapper.find('button').trigger('click')
  
  expect(wrapper.emitted('increment')).toHaveLength(3)
})
```

## Testing Form Input

```javascript
test('updates data on input', async () => {
  const wrapper = shallowMount({
    template: `
      <input v-model="message" />
      <span>{{ message }}</span>
    `,
    data() {
      return { message: '' }
    }
  })
  
  const input = wrapper.find('input')
  
  await input.setValue('Hello')
  
  expect(wrapper.find('span').text()).toBe('Hello')
  expect(wrapper.vm.message).toBe('Hello')
})
```

## Testing Async Operations

```javascript
test('fetches data async', async () => {
  const mockData = { id: 1, name: 'Test' }
  jest.spyOn(api, 'fetchUser').mockResolvedValue(mockData)
  
  const wrapper = shallowMount(UserCard, {
    propsData: { userId: 1 }
  })
  
  await wrapper.vm.$nextTick()
  
  expect(api.fetchUser).toHaveBeenCalledWith(1)
  expect(wrapper.vm.user).toEqual(mockData)
})

test('handles async error', async () => {
  jest.spyOn(api, 'fetchUser').mockRejectedValue(new Error('API Error'))
  
  const wrapper = shallowMount(UserCard, {
    propsData: { userId: 1 }
  })
  
  await wrapper.vm.$nextTick()
  
  expect(wrapper.vm.error).toBe('API Error')
})
```

## Testing Computed Properties

```javascript
test('computed property works correctly', () => {
  const wrapper = shallowMount({
    template: '<div>{{ fullName }}</div>',
    data() {
      return {
        firstName: 'John',
        lastName: 'Doe'
      }
    },
    computed: {
      fullName() {
        return `${this.firstName} ${this.lastName}`
      }
    }
  })
  
  expect(wrapper.text()).toBe('John Doe')
})
```

## Testing Watchers

```javascript
test('watches data changes', async () => {
  const wrapper = shallowMount({
    template: '<div>{{ upperCaseMessage }}</div>',
    data() {
      return {
        message: 'Hello',
        upperCaseMessage: 'HELLO'
      }
    },
    watch: {
      message(newVal) {
        this.upperCaseMessage = newVal.toUpperCase()
      }
    }
  })
  
  wrapper.vm.message = 'World'
  await wrapper.vm.$nextTick()
  
  expect(wrapper.vm.upperCaseMessage).toBe('WORLD')
})
```

## Testing Lifecycle Hooks

```javascript
test('calls created hook', () => {
  const createdSpy = jest.fn()
  
  shallowMount({
    template: '<div>Test</div>',
    created: createdSpy
  })
  
  expect(createdSpy).toHaveBeenCalled()
})
```

## Testing Slots

```javascript
test('renders slot content', () => {
  const wrapper = shallowMount(MyComponent, {
    slots: {
      default: '<div class="slot-content">Slot content</div>'
    }
  })
  
  expect(wrapper.find('.slot-content').exists()).toBe(true)
})

test('renders named slots', () => {
  const wrapper = shallowMount(MyComponent, {
    slots: {
      header: '<h1>Header</h1>',
      footer: '<p>Footer</p>'
    }
  })
  
  expect(wrapper.find('h1').text()).toBe('Header')
  expect(wrapper.find('p').text()).toBe('Footer')
})

test('renders scoped slots', () => {
  const wrapper = shallowMount(MyComponent, {
    scopedSlots: {
      default: '<span>{{ props.item }}</span>'
    }
  })
  
  expect(wrapper.find('span').text()).toBe('item')
})
```

## Stubs

```javascript
test('stubs child components', () => {
  const wrapper = shallowMount(ParentComponent)
  
  // Child components are stubbed
  expect(wrapper.findComponent(ChildComponent).exists()).toBe(true)
  // But they don't render their actual content
})

test('stubs with custom component', () => {
  const wrapper = shallowMount(ParentComponent, {
    stubs: {
      'child-component': {
        template: '<div class="stub">Stubbed</div>'
      }
    }
  })
  
  expect(wrapper.find('.stub').text()).toBe('Stubbed')
})
```

## Mocking

### Mock Components

```javascript
test('mocks child component', () => {
  const MockChild = {
    name: 'MockChild',
    template: '<div>Mocked</div>'
  }
  
  const wrapper = shallowMount(ParentComponent, {
    stubs: {
      'child-component': MockChild
    }
  })
})
```

### Mock Methods

```javascript
test('mocks component method', () => {
  const wrapper = shallowMount(MyComponent)
  
  wrapper.vm.fetchData = jest.fn().mockResolvedValue({ id: 1 })
  
  await wrapper.vm.loadData()
  
  expect(wrapper.vm.fetchData).toHaveBeenCalled()
})
```

### Mock Global Properties

```javascript
test('mocks $route', () => {
  const wrapper = shallowMount(MyComponent, {
    mocks: {
      $route: {
        params: { id: '123' }
      }
    }
  })
  
  expect(wrapper.vm.$route.params.id).toBe('123')
})
```

### Mock Vuex Store

```javascript
import { createLocalVue } from '@vue/test-utils'
import Vuex from 'vuex'

test('works with Vuex', () => {
  const localVue = createLocalVue()
  localVue.use(Vuex)
  
  const store = new Vuex.Store({
    state: {
      count: 5
    },
    getters: {
      doubleCount: state => state.count * 2
    }
  })
  
  const wrapper = shallowMount(MyComponent, {
    localVue,
    store
  })
  
  expect(wrapper.vm.$store.state.count).toBe(5)
  expect(wrapper.vm.doubleCount).toBe(10)
})
```

## Testing Vuex

### Test Store Actions

```javascript
import { actions } from '@/store/modules/user'

describe('user actions', () => {
  test('fetchUser commits SET_USER', async () => {
    const commit = jest.fn()
    const mockUser = { id: 1, name: 'John' }
    api.getUser = jest.fn().mockResolvedValue(mockUser)
    
    await actions.fetchUser({ commit }, 1)
    
    expect(commit).toHaveBeenCalledWith('SET_USER', mockUser)
  })
})
```

### Test Store Getters

```javascript
import { getters } from '@/store/getters'

describe('getters', () => {
  test('activeUsers filters correctly', () => {
    const state = {
      users: [
        { id: 1, active: true },
        { id: 2, active: false },
        { id: 3, active: true }
      ]
    }
    
    const result = getters.activeUsers(state)
    
    expect(result).toHaveLength(2)
  })
})
```

## Testing Vue Router

```javascript
import { createLocalVue } from '@vue/test-utils'
import VueRouter from 'vue-router'

test('uses router correctly', () => {
  const localVue = createLocalVue()
  localVue.use(VueRouter)
  
  const router = new VueRouter({
    routes: [
      { path: '/', name: 'Home', component: { template: '<div>Home</div>' } }
    ]
  })
  
  const wrapper = shallowMount(App, {
    localVue,
    router
  })
  
  expect(wrapper.vm.$route.path).toBe('/')
})
```

## Snapshot Testing

```javascript
test('matches snapshot', () => {
  const wrapper = shallowMount(MyComponent, {
    propsData: {
      title: 'Test'
    }
  })
  
  expect(wrapper.html()).toMatchSnapshot()
})

// Update snapshot
// jest --updateSnapshot
```

## Coverage

```javascript
// jest.config.js
module.exports = {
  collectCoverage: true,
  collectCoverageFrom: [
    'src/**/*.{js,vue}',
    '!src/main.js',
    '!**/node_modules/**'
  ]
}
```

## Best Practices

### 1. Test Behavior, Not Implementation

```javascript
// ❌ WRONG - Testing implementation
test('calls increment method', () => {
  const wrapper = shallowMount(Counter)
  wrapper.vm.increment()
  expect(wrapper.vm.count).toBe(1)
})

// ✅ CORRECT - Testing behavior
test('increments on click', async () => {
  const wrapper = shallowMount(Counter)
  
  await wrapper.find('button').trigger('click')
  
  expect(wrapper.find('.count').text()).toBe('1')
})
```

### 2. Use describe Blocks

```javascript
describe('MyComponent', () => {
  describe('when logged in', () => {
    test('shows user info', () => {
      // ...
    })
  })
  
  describe('when logged out', () => {
    test('shows login button', () => {
      // ...
    })
  })
})
```

### 3. Use Meaningful Assertions

```javascript
// ❌ WRONG
expect(wrapper.classes()).toContain('active')

// ✅ CORRECT
expect(wrapper.find('.tab.active').exists()).toBe(true)
```

### 4. Clean Up

```javascript
afterEach(() => {
  wrapper.destroy()
})

// Or for global cleanup
afterAll(() => {
  jest.clearAllMocks()
})
```

## Common Issues

### Async Updates

```javascript
// ❌ WRONG
test('async update', () => {
  wrapper.vm.update()
  expect(wrapper.text()).toBe('Updated')
})

// ✅ CORRECT
test('async update', async () => {
  wrapper.vm.update()
  await wrapper.vm.$nextTick()
  expect(wrapper.text()).toBe('Updated')
})
```

### Event Triggering

```javascript
// ❌ WRONG
wrapper.find('input').element.value = 'test'
wrapper.find('input').trigger('input')

// ✅ CORRECT
await wrapper.find('input').setValue('test')
```

## Anti-Patterns

### 1. Testing Too Much

```javascript
// ❌ WRONG
test('renders everything correctly', () => {
  const wrapper = mount(ComplexComponent)
  // 100 assertions!
})

// ✅ CORRECT - Split tests
describe('renders header', () => { ... })
describe('renders body', () => { ... })
describe('renders footer', () => { ... })
```

### 2. Testing Private Methods

```javascript
// ❌ WRONG
test('processData works', () => {
  expect(wrapper.vm.processData()).toBe('processed')
})

// ✅ CORRECT - Test public behavior
test('shows processed data', () => {
  wrapper.vm.showData()
  expect(wrapper.find('.result').text()).toBe('processed')
})
```

### 3. Brittle Selectors

```javascript
// ❌ WRONG
expect(wrapper.find('.vue-component-123 .inner .text').text()).toBe('Hello')

// ✅ CORRECT - Use meaningful selectors
expect(wrapper.find('.user-name').text()).toBe('John')
```

## References

See the `references/` directory for:

- Vue Test Utils API reference
- Mocking strategies
- E2E testing patterns
