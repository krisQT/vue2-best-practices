# Component Communication Patterns Reference

## Parent-Child Communication

### Props Down, Events Up

```
Parent Component
    │
    ├── Props (data flows down)
    │    ↓
Child Component
    │
    └── Events (actions flow up)
         ↑
Parent Component
```

### Basic Props and Events

```javascript
// Child: UserCard.vue
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
  },
  
  methods: {
    selectUser() {
      this.$emit('select', this.user)
    }
  }
}
```

```html
<!-- Parent -->
<template>
  <user-card 
    :user="selectedUser"
    :compact="true"
    @select="handleSelect"
  />
</template>

<script>
export default {
  data() {
    return {
      selectedUser: null
    }
  },
  
  methods: {
    handleSelect(user) {
      this.selectedUser = user
    }
  }
}
</script>
```

### v-model Pattern

```javascript
// Child: CustomInput.vue
export default {
  props: {
    value: String
  },
  
  methods: {
    updateValue(event) {
      this.$emit('input', event.target.value)
    }
  }
}
```

```html
<!-- Parent -->
<template>
  <custom-input v-model="searchQuery" />
  <!-- Equivalent to: -->
  <custom-input :value="searchQuery" @input="searchQuery = $event" />
</template>
```

### .sync Modifier

```javascript
// Child: Modal.vue
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
```

```html
<!-- Parent -->
<template>
  <modal :visible.sync="isModalOpen" />
</template>
```

## Deep Nesting Communication

### Provide/Inject

```javascript
// Grandparent
export default {
  provide() {
    return {
      theme: this.theme,
      user: () => this.currentUser  // Function for reactivity
    }
  }
}

// Grandchild (multiple levels deep)
export default {
  inject: ['theme', 'user'],
  
  computed: {
    currentUserData() {
      return this.user()  // Call the function
    }
  }
}
```

### Event Bus (Legacy)

```javascript
// event-bus.js
import Vue from 'vue'
export const EventBus = new Vue()

// Emit in component A
import { EventBus } from '@/event-bus'
EventBus.$emit('user-selected', user)

// Listen in component B
import { EventBus } from '@/event-bus'
EventBus.$on('user-selected', user => {
  // Handle event
})

// Clean up
EventBus.$off('user-selected')
```

### Vuex (Global State)

Best for:
- Shared state across multiple components
- Complex state logic
- Persistence requirements

```javascript
// store/modules/user.js
export default {
  namespaced: true,
  
  state: {
    currentUser: null
  },
  
  mutations: {
    SET_USER(state, user) {
      state.currentUser = user
    }
  },
  
  actions: {
    async fetchUser({ commit }, userId) {
      const user = await api.getUser(userId)
      commit('SET_USER', user)
    }
  },
  
  getters: {
    isLoggedIn: state => !!state.currentUser
  }
}
```

## Sibling Communication

### Via Parent

```html
<!-- Parent -->
<template>
  <div>
    <component-a @update="handleUpdate" />
    <component-b :value="sharedValue" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      sharedValue: ''
    }
  },
  
  methods: {
    handleUpdate(value) {
      this.sharedValue = value
    }
  }
}
</script>
```

### Via Event Bus

```javascript
// Component A
EventBus.$emit('value-changed', newValue)

// Component B
EventBus.$on('value-changed', newValue => {
  this.value = newValue
})
```

### Via Vuex

```javascript
// Component A - dispatch action
this.$store.dispatch('module/updateValue', newValue)

// Component B - read from state
computed: {
  value() {
    return this.$store.state.module.value
  }
}
```

## Slot Communication

### Scoped Slots

```html
<!-- DataTable.vue -->
<template>
  <table>
    <tbody>
      <tr v-for="row in data" :key="row.id">
        <td v-for="col in columns" :key="col.key">
          <slot :row="row" :column="col" :value="row[col.key]" />
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
<data-table :data="users" :columns="columns">
  <template v-slot:default="{ row, column }">
    <span v-if="column.key === 'name'">
      <strong>{{ row.name }}</strong>
    </span>
    <span v-else>{{ row[column.key] }}</span>
  </template>
</data-table>
```

### Render Props

```javascript
// ListRenderer.vue
export default {
  props: {
    items: Array,
    renderItem: Function
  },
  
  render(h) {
    return h('div', 
      this.items.map(item => this.renderItem(h, item))
    )
  }
}

// Parent
<list-renderer :items="users" :render-item="renderUser" />
```

## Best Practices Summary

| Pattern | Use Case | Complexity |
|---------|----------|------------|
| Props/Events | Parent-child | Low |
| v-model | Two-way binding | Low |
| .sync | Parent control | Low |
| Provide/Inject | Deep nesting | Medium |
| Event Bus | Siblings | Medium |
| Vuex | Global state | High |

## Anti-Patterns

### $parent / $children

```javascript
// ❌ WRONG
this.$parent.doSomething()
this.$children[0].doSomething()

// ✅ CORRECT - Use props and events
this.$emit('do-something')
```

### Inline Handlers with Too Much Logic

```html
<!-- ❌ WRONG -->
<button @click="count++; saveToDatabase(); showNotification()">
  Click
</button>

<!-- ✅ CORRECT -->
<button @click="handleClick">
  Click
</button>
```

### Prop Drilling

```javascript
// ❌ WRONG - Too many intermediate components
<GrandParent>
  <Parent :data="data">
    <Child :data="data">
      <GrandChild :data="data">  <!-- Got it, but too many props -->
```

```javascript
// ✅ CORRECT - Use provide/inject or Vuex
// GrandParent provides data
// GrandChild injects data
```
