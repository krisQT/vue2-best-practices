# Vue 2 Components Best Practices

## When to Use

Use this skill when:

- Designing Vue 2 components
- Implementing component communication
- Using props and events
- Working with slots
- Creating reusable components

## Activation Triggers

Use this skill when the user prompt contains any of these patterns:

- "create a component Vue"
- "component communication"
- "parent child Vue"
- "props emit Vue"
- "v-model component"
- "custom component Vue"
- "slot content"
- "scoped slot Vue"
- "named slots Vue"
- "provide inject"
- "component registration"
- "local component"
- "global component Vue"
- "async component"
- "lazy load component"
- "functional component"
- "component props"
- "component events"
- "reusable component"
- "component design"
- "single responsibility"
- "component naming"

Keywords: `<component>`, `props:`, `$emit`, `$on`, `v-slot`, `provide:`, `inject:`

## Component Communication Patterns

### Parent to Child: Props

```html
<!-- Parent -->
<user-card :user="currentUser" :compact="true" />

<!-- Child -->
export default {
  props: {
    user: {
      type: Object,
      required: true
    },
    compact: {
      type: Boolean,
      default: false
    }
  }
}
```

### Child to Parent: Events

```html
<!-- Child -->
<template>
  <button @click="handleClick">Click me</button>
</template>

<script>
export default {
  methods: {
    handleClick() {
      this.$emit('clicked', { id: 1, name: 'Item' })
    }
  }
}
</script>

<!-- Parent -->
<template>
  <my-button @clicked="handleItemClicked" />
</template>

<script>
export default {
  methods: {
    handleItemClicked(payload) {
      console.log('Clicked:', payload)
    }
  }
}
</script>
```

### v-model on Components

```html
<!-- Parent -->
<template>
  <search-input v-model="searchQuery" />
</template>

<!-- Child -->
<template>
  <input :value="value" @input="handleInput" />
</template>

<script>
export default {
  props: {
    value: String
  },
  methods: {
    handleInput(event) {
      this.$emit('input', event.target.value)
    }
  }
}
</script>
```

### .sync Modifier (Vue 2.3+)

> **版本说明**：`.sync` 修饰符从 Vue 2.3.0 开始支持。

```html
<!-- Parent -->
<template>
  <modal :visible.sync="isModalVisible" />
</template>

<!-- Child -->
<script>
export default {
  props: {
    visible: Boolean
  },
  methods: {
    close() {
      this.$emit('update:visible', false)
    }
  }
}
</script>
```

## Slots

### Named Slots

```html
<!-- Parent -->
<card>
  <template v-slot:header>
    <h1>Card Title</h1>
  </template>
  
  <template v-slot:default>
    <p>Main content</p>
  </template>
  
  <template v-slot:footer>
    <button>Action</button>
  </template>
</card>

<!-- Child: card.vue -->
<template>
  <div class="card">
    <header class="card-header">
      <slot name="header" />
    </header>
    
    <div class="card-body">
      <slot />  <!-- Default slot -->
    </div>
    
    <footer class="card-footer">
      <slot name="footer" />
    </footer>
  </div>
</template>
```

### Scoped Slots

```html
<!-- Child: data-table.vue -->
<template>
  <table>
    <thead>
      <tr>
        <th v-for="column in columns" :key="column.key">
          {{ column.label }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(row, index) in data" :key="row.id">
        <td v-for="column in columns" :key="column.key">
          <!-- Pass row data and column to parent -->
          <slot :row="row" :column="column" :index="index" />
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
export default {
  props: {
    data: Array,
    columns: Array
  }
}
</script>

<!-- Parent -->
<template>
  <data-table :data="users" :columns="columns">
    <!-- Receive row data via slot props -->
    <template v-slot:default="{ row, column }">
      <span v-if="column.key === 'name'">
        <strong>{{ row.name }}</strong>
      </span>
      <span v-else-if="column.key === 'actions'">
        <button @click="editUser(row)">Edit</button>
      </span>
      <span v-else>
        {{ row[column.key] }}
      </span>
    </template>
  </data-table>
</template>

<script>
export default {
  data() {
    return {
      columns: [
        { key: 'name', label: 'Name' },
        { key: 'email', label: 'Email' },
        { key: 'actions', label: 'Actions' }
      ]
    }
  }
}
</script>
```

### Fallback Content

```html
<!-- Child -->
<template>
  <button class="btn">
    <slot>Default Text</slot>
  </button>
</template>

<!-- Parent -->
<button-default>Save</button-default>  <!-- Shows "Save" -->
<button-default />  <!-- Shows "Default Text" -->
```

## Provide/Inject

### Basic Usage

```javascript
// Parent component
export default {
  provide: {
    theme: 'dark',
    user: () => this.currentUser  // Use function for reactivity
  }
}

// Child component (deep nested)
export default {
  inject: ['theme', 'user'],
  mounted() {
    console.log(this.theme)  // 'dark'
    console.log(this.user())  // Current user object
  }
}
```

### With Defaults

```javascript
export default {
  inject: {
    theme: {
      from: 'theme',
      default: 'light'
    }
  }
}
```

### Reactive Provide

```javascript
// Parent
export default {
  data() {
    return {
      user: null
    }
  },
  provide() {
    return {
      // Return object for reactivity
      user: () => this.user
    }
  }
}

// Child
export default {
  inject: ['user'],
  computed: {
    userData() {
      return this.user()  // Call function to get reactive value
    }
  }
}
```

## Component Design Principles

### 1. Single Responsibility

```javascript
// ❌ BAD - Too many responsibilities
export default {
  methods: {
    fetchData() { /* API calls */ },
    formatData() { /* Formatting */ },
    saveData() { /* Save */ },
    renderChart() { /* Chart rendering */ }
  }
}

// ✅ GOOD - Separate concerns
// DataFetcher.vue - handles data fetching
// DataFormatter.vue - handles formatting
// DataSaver.vue - handles saving
// ChartRenderer.vue - handles chart rendering
```

### 2. Props Validation

```javascript
export default {
  props: {
    // Required
    title: {
      type: String,
      required: true
    },
    
    // With default
    count: {
      type: Number,
      default: 0
    },
    
    // Object/Array with factory
    user: {
      type: Object,
      default() {
        return { name: '', email: '' }
      }
    },
    
    // Enum
    size: {
      type: String,
      default: 'medium',
      validator(value) {
        return ['small', 'medium', 'large'].includes(value)
      }
    },
    
    // Custom type
    customProp: {
      validator(value) {
        return value instanceof MyClass
      }
    }
  }
}
```

### TypeScript Props Definition

```typescript
// types/component.ts
export interface User {
  id: number
  name: string
  email: string
}

export type ButtonSize = 'small' | 'medium' | 'large'

// Component with TypeScript
import { PropType } from 'vue'

export default {
  props: {
    // String type
    title: {
      type: String,
      required: true
    },
    
    // Number type
    count: {
      type: Number,
      default: 0
    },
    
    // Boolean shorthand
    disabled: Boolean,
    
    // Object with type interface
    user: {
      type: Object as PropType<User>,
      required: true
    },
    
    // Array with type
    items: {
      type: Array as PropType<string[]>,
      default: () => []
    },
    
    // Enum with validator
    size: {
      type: String as PropType<ButtonSize>,
      default: 'medium',
      validator: (value: ButtonSize) => ['small', 'medium', 'large'].includes(value)
    },
    
    // Function prop
    onClick: {
      type: Function as PropType<(id: number) => void>,
      default: () => () => {}
    },
    
    // Custom class instance
    customData: {
      type: Object as PropType<InstanceType<typeof MyClass>>,
      validator: (value: any) => value instanceof MyClass
    }
  }
}
```

### 3. Emit Events with Payloads

```javascript
// ❌ BAD - Ambiguous events
this.$emit('change')
this.$emit('update')

// ✅ GOOD - Clear events with data
this.$emit('update:count', newCount)
this.$emit('change', { field: 'count', oldValue: 1, newValue: 2 })
this.$emit('submit', { email: this.email, password: this.password })
```

### 4. Use $refs Wisely

```html
<template>
  <div>
    <input ref="myInput" type="text" />
    <user-card ref="userCard" :user="user" />
  </div>
</template>

<script>
export default {
  methods: {
    focusInput() {
      // Access DOM element
      this.$refs.myInput.focus()
    },
    
    accessChild() {
      // Access child component methods
      this.$refs.userCard.refresh()
    }
  }
}
</script>
```

### 5. Functional Components

```javascript
// Functional component - no instance, no state
export default {
  functional: true,
  props: {
    level: {
      type: Number,
      required: true
    },
    text: String
  },
  render(h, { props, data, children }) {
    return h(
      `h${props.level}`,
      data,
      props.text || children
    )
  }
}

// With template
<!-- Functional: true in options -->
<!-- No data, no this -->
```

## Component Registration

### Local Registration

```javascript
import MyButton from './MyButton.vue'
import MyModal from './MyModal.vue'

export default {
  components: {
    MyButton,
    'my-modal': MyModal  // kebab-case alias
  }
}
```

### Global Registration

```javascript
// main.js
import Vue from 'vue'
import MyButton from './components/MyButton.vue'

Vue.component('my-button', MyButton)
Vue.component('MyButton', MyButton)  // Also available as <my-button>
```

## Async Components

### Basic Async

```javascript
components: {
  LazyComponent: () => import('./LazyComponent.vue')
}
```

### With Options

```javascript
components: {
  AsyncComponent: () => ({
    component: import('./AsyncComponent.vue'),
    loading: LoadingComponent,
    error: ErrorComponent,
    delay: 200,
    timeout: 3000
  })
}
```

## Component Naming

### PascalCase vs kebab-case

```html
<!-- Both work in templates -->
<MyComponent />
<my-component />

<!-- Recommended: kebab-case for consistency -->
<my-button />
<user-card />
<data-table />
```

### Name Collision

```javascript
// When same name from different sources
import Table from './components/Table.vue'
import { Table } from '@headlessui/vue'

export default {
  components: {
    Table,
    'HeadlessTable': Table  // Alias to avoid collision
  }
}
```

## Component Testing Considerations

```javascript
export default {
  props: {
    items: {
      type: Array,
      required: true,
      validator: items => items.every(item => item.id)
    }
  },
  
  data() {
    return {
      selectedId: null
    }
  },
  
  methods: {
    select(item) {
      this.selectedId = item.id
      this.$emit('select', item)
    }
  },
  
  computed: {
    selectedItem() {
      return this.items.find(item => item.id === this.selectedId)
    }
  }
}
```

## Anti-Patterns

### 1. Don't Use $parent/$children

```javascript
// ❌ WRONG
this.$parent.doSomething()
this.$children[0].doSomething()

// ✅ CORRECT - Use props and events
// Or provide/inject for deep nesting
```

### 2. Don't Mutate Props

```javascript
// ❌ WRONG
props: ['title'],
mounted() {
  this.title = 'Modified'  // Vue warns!
}

// ✅ CORRECT
props: ['initialTitle'],
data() {
  return {
    title: this.initialTitle
  }
}
```

### 3. Don't Use Too Many Props

```javascript
// ❌ WRONG
<user-card :name="user.name" :email="user.email" :phone="user.phone" ... />

// ✅ CORRECT - Pass the object
<user-card :user="user" />
```

### 4. Don't Forget Key in v-for

```html
<!-- ❌ WRONG -->
<div v-for="item in items">
  {{ item.name }}
</div>

<!-- ✅ CORRECT -->
<div v-for="item in items" :key="item.id">
  {{ item.name }}
</div>
```

## References

See the `references/` directory for:

- Component communication patterns
- Slot advanced usage
- Component testing strategies
