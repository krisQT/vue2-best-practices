# Vue 2 Directives Best Practices

## When to Use

Use this skill when:

- Creating custom directives
- Understanding directive hooks
- Working with DOM manipulation
- Implementing reusable low-level functionality

## Activation Triggers

Use this skill when the user prompt contains any of these patterns:

- "Vue directive"
- "custom directive"
- "v-focus"
- "v-click-outside"
- "directive hooks"
- "bind inserted"
- "update unbind"
- "directive arguments"
- "directive modifiers"
- "directive binding"
- "DOM manipulation Vue"
- "functional directive"
- "global directive"
- "local directive"
- "directive example"
- "click outside Vue"
- "autofocus directive"
- "v-drag directive"
- "v-lazy directive"
- "custom v-model"
- "directive lifecycle"
- "el._handler"

Keywords: `Vue.directive(`, `directives:`, `v-`, `bind(el`, `inserted(el`

## Directive Overview

Directives are special attributes with the `v-` prefix that provide low-level DOM access.

## Built-in Directives

- `v-text` - Text interpolation
- `v-html` - Raw HTML
- `v-show` - Toggle visibility
- `v-if` / `v-else-if` / `v-else` - Conditional rendering
- `v-for` - List rendering
- `v-on` - Event handling
- `v-bind` - Attribute binding
- `v-model` - Two-way binding
- `v-slot` - Slot content
- `v-pre` - Skip compilation
- `v-once` - Render only once

## Custom Directives

### Basic Directive

```javascript
// Register globally
Vue.directive('focus', {
  inserted(el) {
    el.focus()
  }
})

// Or locally in component
export default {
  directives: {
    focus: {
      inserted(el) {
        el.focus()
      }
    }
  }
}
```

### Directive Hook Functions

| Hook | Description |
|------|-------------|
| `bind` | Once directive is attached (first time) |
| `inserted` | Element inserted into parent node |
| `update` | Component updated (children may not) |
| `componentUpdated` | Component and children updated |
| `unbind` | Directive removed |

```javascript
Vue.directive('my-directive', {
  // Called once when directive is first bound
  bind(el, binding, vnode) {
    console.log('bind', el, binding.name)
  },
  
  // Called once when element is inserted into DOM
  inserted(el, binding, vnode) {
    console.log('inserted', el)
  },
  
  // Called when component updates (may be before children)
  update(el, binding, vnode, oldVnode) {
    console.log('update')
  },
  
  // Called after component and children updated
  componentUpdated(el, binding, vnode, oldVnode) {
    console.log('componentUpdated')
  },
  
  // Called once when directive is removed
  unbind(el, binding, vnode) {
    console.log('unbind')
  }
})
```

## Binding Object

```javascript
el          // DOM element
binding.value        // Value passed to directive (e.g., v-my-directive="value")
binding.oldValue     // Previous value
binding.expression   // String version of binding expression (e.g., "myValue")
binding.arg          // Argument passed (e.g., v-my-directive:arg)
binding.modifiers    // Object with modifiers (e.g., { modifier: true })
binding.name         // Directive name without v- (e.g., "my-directive")
vnode                // Virtual node
oldVnode             // Previous virtual node
```

## Directive Examples

### 1. Focus Directive

```javascript
Vue.directive('focus', {
  inserted(el) {
    el.focus()
  }
})

// Usage
<input v-focus />
```

### 2. Click Outside

```javascript
Vue.directive('click-outside', {
  bind(el, binding) {
    el.clickOutsideEvent = (event) => {
      if (!(el === event.target || el.contains(event.target))) {
        binding.value(event)
      }
    }
    document.body.addEventListener('click', el.clickOutsideEvent)
  },
  
  unbind(el) {
    document.body.removeEventListener('click', el.clickOutsideEvent)
  }
})

// Usage
<div v-click-outside="closeDropdown">
```

### 3. Tooltip

```javascript
Vue.directive('tooltip', {
  bind(el, binding) {
    el.style.position = 'relative'
    
    const tooltip = document.createElement('span')
    tooltip.className = 'tooltip'
    tooltip.textContent = binding.value
    tooltip.style.display = 'none'
    el.appendChild(tooltip)
    
    el.addEventListener('mouseenter', () => {
      tooltip.style.display = 'block'
    })
    
    el.addEventListener('mouseleave', () => {
      tooltip.style.display = 'none'
    })
    
    el.tooltipElement = tooltip
  },
  
  update(el, binding) {
    el.tooltipElement.textContent = binding.value
  },
  
  unbind(el) {
    el.removeEventListener('mouseenter', handler)
    el.removeEventListener('mouseleave', handler)
    if (el.tooltipElement) {
      el.removeChild(el.tooltipElement)
    }
  }
})
```

### 4. Loading State

```javascript
Vue.directive('loading', {
  bind(el, binding) {
    if (binding.value) {
      el.style.position = 'relative'
      
      const loader = document.createElement('div')
      loader.className = 'loader'
      loader.innerHTML = '<span>Loading...</span>'
      el.appendChild(loader)
      
      el.loaderElement = loader
    }
  },
  
  update(el, binding) {
    if (binding.value && !el.loaderElement) {
      // Re-add loader
      const loader = document.createElement('div')
      loader.className = 'loader'
      el.appendChild(loader)
      el.loaderElement = loader
    } else if (!binding.value && el.loaderElement) {
      // Remove loader
      el.removeChild(el.loaderElement)
      el.loaderElement = null
    }
  },
  
  unbind(el) {
    if (el.loaderElement) {
      el.removeChild(el.loaderElement)
    }
  }
})
```

### 5. Scroll Detection

```javascript
Vue.directive('scroll', {
  bind(el, binding) {
    el.scrollHandler = () => {
      binding.value(window.scrollY)
    }
    window.addEventListener('scroll', el.scrollHandler)
  },
  
  unbind(el) {
    window.removeEventListener('scroll', el.scrollHandler)
  }
})

// Usage
<div v-scroll="handleScroll"></div>

export default {
  methods: {
    handleScroll(scrollY) {
      this.isScrolled = scrollY > 100
    }
  }
}
```

## Directive with Arguments

```javascript
// v-highlight:background="color"
// v-highlight:foreground="color"

Vue.directive('highlight', {
  bind(el, binding) {
    if (binding.arg === 'background') {
      el.style.backgroundColor = binding.value
    } else if (binding.arg === 'foreground') {
      el.style.color = binding.value
    }
  },
  
  update(el, binding) {
    if (binding.arg === 'background') {
      el.style.backgroundColor = binding.value
    } else if (binding.arg === 'foreground') {
      el.style.color = binding.value
    }
  }
})
```

## Directive with Modifiers

```javascript
// v-drag.prevent.stop

Vue.directive('drag', {
  bind(el, binding) {
    el.style.position = 'absolute'
    
    let startX, startY, initialX, initialY
    
    el.addEventListener('mousedown', startDrag)
    
    function startDrag(e) {
      if (binding.modifiers.prevent) {
        e.preventDefault()
      }
      if (binding.modifiers.stop) {
        e.stopPropagation()
      }
      
      startX = e.clientX
      startY = e.clientY
      initialX = el.offsetLeft
      initialY = el.offsetTop
      
      document.addEventListener('mousemove', drag)
      document.addEventListener('mouseup', stopDrag)
    }
    
    function drag(e) {
      const dx = e.clientX - startX
      const dy = e.clientY - startY
      el.style.left = `${initialX + dx}px`
      el.style.top = `${initialY + dy}px`
    }
    
    function stopDrag() {
      document.removeEventListener('mousemove', drag)
      document.removeEventListener('mouseup', stopDrag)
    }
    
    el._dragHandlers = { startDrag, drag, stopDrag }
  },
  
  unbind(el) {
    const { startDrag } = el._dragHandlers
    el.removeEventListener('mousedown', startDrag)
  }
})
```

## Object Syntax

```javascript
// v-demo="{ name: 'John', age: 30 }"

Vue.directive('demo', {
  bind(el, binding) {
    console.log(binding.value.name)  // 'John'
    console.log(binding.value.age)   // 30
  }
})
```

## Local vs Global Directives

### Global Registration

```javascript
// main.js
import Vue from 'vue'
import App from './App.vue'

Vue.directive('focus', {
  inserted(el) {
    el.focus()
  }
})

new Vue({
  render: h => h(App)
}).$mount('#app')

// Now available in all components
```

### Local Registration

```javascript
export default {
  directives: {
    focus: {
      inserted(el) {
        el.focus()
      }
    }
  }
}

// Only available in this component and its children
```

## Functional Directives

For simple cases, use functional directives (no state):

```javascript
Vue.directive('bold', {
  functional: true,
  render(h, context) {
    return h('strong', context.data, context.children)
  }
})
```

## Directive Best Practices

### 1. Clean Up Properly

```javascript
// ❌ WRONG - Memory leak
bind(el, binding) {
  el.addEventListener('click', handler)
  // Never removed!
}

// ✅ CORRECT
bind(el, binding) {
  el._clickHandler = handler
  el.addEventListener('click', handler)
},
unbind(el) {
  el.removeEventListener('click', el._clickHandler)
}
```

### 2. Use Unique Variable Names

```javascript
// ❌ WRONG - Can conflict
bind(el, handler) {
  el.handler = handler
}

// ✅ CORRECT
bind(el, binding) {
  el._vMyDirectiveHandler = handler
}
```

### 3. Handle All Hooks When Needed

```javascript
// Only use hooks you need
Vue.directive('simple', {
  inserted(el) {
    el.focus()
  }
})
```

### 4. Document Your Directives

```javascript
/**
 * v-focus
 * 
 * Auto-focuses element on insert
 * 
 * Usage:
 * <input v-focus />
 * 
 * @example
 * <input v-focus placeholder="Search..." />
 */
```

## Common Use Cases

| Directive | Use Case |
|-----------|----------|
| `v-focus` | Auto-focus input |
| `v-click-outside` | Close modal/dropdown |
| `v-lazy-load` | Lazy load images |
| `v-infinite-scroll` | Infinite scrolling |
| `v-tooltip` | Show tooltips |
| `v-permission` | Permission checking |
| `v-analytics` | Analytics tracking |
| `v-drag` | Drag and drop |

## Vue 2 vs Vue 3 Directives

| Feature | Vue 2 | Vue 3 |
|---------|-------|-------|
| Hook names | `bind`, `inserted`, `update`, `unbind` | `beforeMount`, `mounted`, `updated`, `unmounted` |
| Parameters | `binding.arg`, `binding.modifiers` | Same |
| Expression | `binding.expression` | Removed |
| Functional | Supported | Use render functions instead |

## Anti-Patterns

### 1. Don't Use Directives for Component Logic

```javascript
// ❌ WRONG
Vue.directive('admin', {
  bind(el) {
    if (!isAdmin()) {
      el.style.display = 'none'
    }
  }
})

// ✅ CORRECT - Use component logic
<admin-panel v-if="isAdmin" />
```

### 2. Don't Overcomplicate

```javascript
// ❌ WRONG - Directive doing too much
Vue.directive('complex', {
  // 500 lines of code
})

// ✅ CORRECT - Keep directives focused
// Split into multiple directives or use component
```

### 3. Don't Use for Simple Bindings

```javascript
// ❌ WRONG
Vue.directive('color', {
  bind(el, binding) {
    el.style.color = binding.value
  }
})

// Usage
<span v-color="textColor">Text</span>

// ✅ CORRECT - Use binding
<span :style="{ color: textColor }">Text</span>
```

## Testing Directives

```javascript
import { mount } from '@vue/test-utils'
import { createLocalVue } from '@vue/test-utils'

describe('v-focus directive', () => {
  it('focuses element on insert', () => {
    const wrapper = mount({
      template: '<input v-focus />',
      directives: {
        focus: {
          inserted(el) {
            el.focus()
          }
        }
      }
    })
    
    const input = wrapper.find('input')
    expect(document.activeElement).toBe(input.element)
  })
})
```

## References

See the `references/` directory for:

- Directive hook parameters
- Complex directive patterns
- Directive testing strategies
