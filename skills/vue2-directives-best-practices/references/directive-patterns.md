# Vue 2 Directive Patterns Reference

## Directive Hooks

```javascript
Vue.directive('my-directive', {
  // 1. Called once when directive is first bound
  bind(el, binding, vnode) {
    console.log('bind')
  },
  
  // 2. Called once when element is inserted into DOM
  inserted(el, binding, vnode) {
    console.log('inserted')
  },
  
  // 3. Called when component updates (may be before children)
  update(el, binding, vnode, oldVnode) {
    console.log('update')
  },
  
  // 4. Called after component and children updated
  componentUpdated(el, binding, vnode, oldVnode) {
    console.log('componentUpdated')
  },
  
  // 5. Called once when directive is removed
  unbind(el, binding, vnode) {
    console.log('unbind')
  }
})
```

## Binding Object Properties

```javascript
el                 // DOM element
binding.value       // Value passed to directive
binding.oldValue   // Previous value
binding.expression // String expression (e.g., "myVar")
binding.arg        // Argument (e.g., "color" in v-my-directive:color)
binding.modifiers  // Object with modifiers
binding.name       // Directive name without v-
vnode              // Virtual node
oldVnode           // Previous virtual node
```

## Common Directive Patterns

### 1. Click Outside

```javascript
Vue.directive('click-outside', {
  bind(el, binding) {
    el._clickOutside = (event) => {
      if (!(el === event.target || el.contains(event.target))) {
        binding.value(event)
      }
    }
    document.body.addEventListener('click', el._clickOutside)
  },
  
  unbind(el) {
    document.body.removeEventListener('click', el._clickOutside)
  }
})
```

### 2. Focus

```javascript
Vue.directive('focus', {
  inserted(el) {
    el.focus()
  },
  
  update(el, binding) {
    if (binding.value) {
      el.focus()
    }
  }
})
```

### 3. Intersection Observer

```javascript
Vue.directive('intersect', {
  inserted(el, binding) {
    const observer = new IntersectionObserver((entries) => {
      if (entries[0].isIntersecting) {
        binding.value()
      }
    })
    
    el._observer = observer
    observer.observe(el)
  },
  
  unbind(el) {
    el._observer.disconnect()
  }
})
```

### 4. Resize

```javascript
Vue.directive('resize', {
  inserted(el, binding) {
    el._resizeHandler = () => {
      binding.value({
        width: el.offsetWidth,
        height: el.offsetHeight
      })
    }
    
    window.addEventListener('resize', el._resizeHandler)
    el._resizeHandler()
  },
  
  unbind(el) {
    window.removeEventListener('resize', el._resizeHandler)
  }
})
```

### 5. Long Press

```javascript
Vue.directive('long-press', {
  bind(el, binding) {
    let timeout
    
    const start = () => {
      timeout = setTimeout(() => {
        binding.value()
      }, binding.arg || 500)
    }
    
    const cancel = () => {
      if (timeout) {
        clearTimeout(timeout)
      }
    }
    
    el.addEventListener('mousedown', start)
    el.addEventListener('touchstart', start)
    el.addEventListener('mouseup', cancel)
    el.addEventListener('mouseleave', cancel)
    el.addEventListener('touchend', cancel)
    
    el._longPressHandlers = { start, cancel }
  },
  
  unbind(el) {
    const { start, cancel } = el._longPressHandlers
    el.removeEventListener('mousedown', start)
    el.removeEventListener('touchstart', start)
    el.removeEventListener('mouseup', cancel)
    el.removeEventListener('mouseleave', cancel)
    el.removeEventListener('touchend', cancel)
  }
})
```

### 6. Debounce

```javascript
Vue.directive('debounce', {
  bind(el, binding) {
    let timeout
    
    const handler = () => {
      clearTimeout(timeout)
      timeout = setTimeout(() => {
        binding.value.handler()
      }, binding.arg || 300)
    }
    
    el.addEventListener('input', handler)
    
    el._debounceHandler = handler
    el._debounceTimeout = timeout
  },
  
  unbind(el) {
    clearTimeout(el._debounceTimeout)
    el.removeEventListener('input', el._debounceHandler)
  }
})
```

### 7.权限检查

```javascript
Vue.directive('permission', {
  bind(el, binding) {
    const hasPermission = checkPermission(binding.value)
    
    if (!hasPermission) {
      el.style.display = 'none'
    }
  },
  
  update(el, binding) {
    const hasPermission = checkPermission(binding.value)
    
    el.style.display = hasPermission ? '' : 'none'
  }
})
```

### 8. Lazy Load Image

```javascript
Vue.directive('lazy', {
  bind(el, binding) {
    el._src = binding.value
    
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          el.src = el._src
          observer.unobserve(el)
        }
      })
    })
    
    el._observer = observer
    observer.observe(el)
  },
  
  unbind(el) {
    el._observer.disconnect()
  }
})
```

## Directive with Arguments

```javascript
// v-my-directive:argument
Vue.directive('style', {
  bind(el, binding) {
    if (binding.arg === 'color') {
      el.style.color = binding.value
    } else if (binding.arg === 'background') {
      el.style.backgroundColor = binding.value
    }
  },
  
  update(el, binding) {
    if (binding.arg === 'color') {
      el.style.color = binding.value
    } else if (binding.arg === 'background') {
      el.style.backgroundColor = binding.value
    }
  }
})
```

## Directive with Modifiers

```javascript
// v-my-directive.modifier1.modifier2
Vue.directive('drag', {
  bind(el, binding) {
    let startX, startY
    let isDragging = false
    
    const onStart = (e) => {
      if (binding.modifiers.handle && !e.target.matches('.handle')) {
        return
      }
      
      startX = e.clientX || e.touches[0].clientX
      startY = e.clientY || e.touches[0].clientY
      isDragging = true
      
      if (binding.modifiers.stop) {
        e.stopPropagation()
      }
      
      document.addEventListener('mousemove', onMove)
      document.addEventListener('mouseup', onEnd)
      document.addEventListener('touchmove', onMove)
      document.addEventListener('touchend', onEnd)
    }
    
    const onMove = (e) => {
      if (!isDragging) return
      
      const clientX = e.clientX || e.touches[0].clientX
      const clientY = e.clientY || e.touches[0].clientY
      
      el.style.left = `${clientX - startX}px`
      el.style.top = `${clientY - startY}px`
    }
    
    const onEnd = () => {
      isDragging = false
      document.removeEventListener('mousemove', onMove)
      document.removeEventListener('mouseup', onEnd)
      document.removeEventListener('touchmove', onMove)
      document.removeEventListener('touchend', onEnd)
    }
    
    el.addEventListener('mousedown', onStart)
    el.addEventListener('touchstart', onStart)
    
    el._dragHandlers = { onStart, onMove, onEnd }
  },
  
  unbind(el) {
    const { onStart, onMove, onEnd } = el._dragHandlers
    el.removeEventListener('mousedown', onStart)
    el.removeEventListener('touchstart', onStart)
    document.removeEventListener('mousemove', onMove)
    document.removeEventListener('mouseup', onEnd)
    document.removeEventListener('touchmove', onMove)
    document.removeEventListener('touchend', onEnd)
  }
})
```

## Functional Directives

```javascript
// No state, stateless component
Vue.directive('uppercase', {
  functional: true,
  bind(el, binding) {
    el.textContent = binding.value.toUpperCase()
  },
  update(el, binding) {
    el.textContent = binding.value.toUpperCase()
  }
})

// Or with render function
Vue.directive('bold', {
  functional: true,
  render(h, context) {
    return h('strong', context.data, context.children)
  }
})
```

## Local Registration

```javascript
export default {
  directives: {
    focus: {
      inserted(el) {
        el.focus()
      }
    },
    'click-outside': {
      inserted(el, binding) {
        // Implementation
      }
    }
  }
}
```

## Best Practices

1. **Always clean up** - Remove event listeners, disconnect observers
2. **Use unique names** - Prefix with underscore: `el._handler`
3. **Handle all hooks** - Use only hooks you need
4. **Document** - Explain what the directive does
5. **Test** - Ensure proper cleanup and edge cases
