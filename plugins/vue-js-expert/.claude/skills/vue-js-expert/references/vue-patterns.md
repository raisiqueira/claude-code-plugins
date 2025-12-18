# Vue 3 Patterns and Anti-Patterns

This document provides common patterns and anti-patterns for Vue 3 development based on real-world experience and best practices.

## Common Patterns

### 1. Composable Pattern

Extract reusable logic into composables following the VueUse guidelines.

**Good:**
```typescript
// composables/useAuth.ts
import { ref, computed } from 'vue'

export function useAuth() {
  const user = ref<User | null>(null)
  const isAuthenticated = computed(() => user.value !== null)
  const isAdmin = computed(() => user.value?.role === 'admin')

  async function login(credentials: Credentials) {
    const response = await authApi.login(credentials)
    user.value = response.user
  }

  async function logout() {
    await authApi.logout()
    user.value = null
  }

  return {
    user: readonly(user),
    isAuthenticated,
    isAdmin,
    login,
    logout
  }
}
```

**Usage:**
```vue
<script setup lang="ts">
import { useAuth } from '@/composables/useAuth'

const { user, isAuthenticated, login, logout } = useAuth()
</script>
```

### 2. defineModel Pattern

Use `defineModel` for v-model bindings instead of manual prop + emit.

**Good:**
```vue
<script setup lang="ts">
// Automatically creates modelValue prop and update:modelValue emit
const modelValue = defineModel<string>({ required: true })

// Multiple v-models
const isOpen = defineModel<boolean>('open', { default: false })
const selectedId = defineModel<number>('selected')

// With getter/setter
const internalValue = defineModel<string>({
  get(value) {
    return value.toUpperCase()
  },
  set(value) {
    return value.toLowerCase()
  }
})
</script>

<template>
  <input v-model="modelValue" />
</template>
```

**Avoid:**
```vue
<script setup lang="ts">
// Manual prop + emit (more boilerplate)
const props = defineProps<{ modelValue: string }>()
const emit = defineEmits<{ 'update:modelValue': [value: string] }>()

const updateValue = (value: string) => {
  emit('update:modelValue', value)
}
</script>
```

### 3. Readonly Pattern

Expose readonly refs to prevent external mutations.

**Good:**
```typescript
import { ref, readonly } from 'vue'

export function useCounter() {
  const count = ref(0)

  const increment = () => count.value++
  const decrement = () => count.value--

  return {
    count: readonly(count),  // External code can't mutate
    increment,
    decrement
  }
}
```

**Avoid:**
```typescript
export function useCounter() {
  const count = ref(0)

  return {
    count,  // External code can mutate directly
    increment: () => count.value++
  }
}

// Bad usage
const { count } = useCounter()
count.value = 999  // Should use increment() instead
```

### 4. Computed with Setter

Use computed with getter/setter for derived state that can be modified.

```typescript
const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  get: () => `${firstName.value} ${lastName.value}`,
  set: (value: string) => {
    const [first, last] = value.split(' ')
    firstName.value = first
    lastName.value = last
  }
})

// Usage
fullName.value = 'Jane Smith'  // Updates firstName and lastName
```

### 5. Provide/Inject Pattern

Share state across component hierarchy without prop drilling.

**Parent Component:**
```typescript
import { provide, ref } from 'vue'

const theme = ref('light')

provide('theme', {
  theme: readonly(theme),
  toggleTheme: () => {
    theme.value = theme.value === 'light' ? 'dark' : 'light'
  }
})
```

**Child Component:**
```typescript
import { inject } from 'vue'

const themeContext = inject('theme')

// With type safety
interface ThemeContext {
  theme: Readonly<Ref<'light' | 'dark'>>
  toggleTheme: () => void
}

const themeContext = inject<ThemeContext>('theme')
```

### 6. Async Component Pattern

Lazy load heavy components for better performance.

```typescript
import { defineAsyncComponent } from 'vue'

const HeavyChart = defineAsyncComponent({
  loader: () => import('./components/HeavyChart.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorDisplay,
  delay: 200,  // Show loading after 200ms
  timeout: 3000  // Show error after 3s
})
```

## Anti-Patterns

### 1. Destructuring Reactive

**Avoid:**
```typescript
import { reactive } from 'vue'

const state = reactive({ count: 0, message: 'Hello' })

// Loses reactivity!
const { count, message } = state

count++  // Doesn't update UI
```

**Good:**
```typescript
import { reactive, toRefs } from 'vue'

const state = reactive({ count: 0, message: 'Hello' })

// Preserves reactivity
const { count, message } = toRefs(state)

count.value++  // Updates UI
```

**Better - Use ref instead:**
```typescript
import { ref } from 'vue'

const count = ref(0)
const message = ref('Hello')

// Can destructure because they're already separate refs
```

### 2. Mutating Props

**Avoid:**
```vue
<script setup lang="ts">
const props = defineProps<{ items: Item[] }>()

// Never mutate props directly!
props.items.push(newItem)
</script>
```

**Good:**
```vue
<script setup lang="ts">
const props = defineProps<{ items: Item[] }>()
const emit = defineEmits<{ 'update:items': [items: Item[]] }>()

// Or use defineModel
const items = defineModel<Item[]>('items', { required: true })

const addItem = (item: Item) => {
  items.value = [...items.value, item]
}
</script>
```

### 3. Using Options API in New Code

**Avoid:**
```vue
<script lang="ts">
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  }
}
</script>
```

**Good:**
```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
const increment = () => count.value++
</script>
```

### 4. Not Using shallowRef for Large Data

**Avoid:**
```typescript
import { ref } from 'vue'

// Deep reactivity on 10,000 items - performance issue!
const largeDataset = ref(Array(10000).fill({ /* complex object */ }))
```

**Good:**
```typescript
import { shallowRef } from 'vue'

// Only triggers on array reference change - much faster
const largeDataset = shallowRef(Array(10000).fill({ /* complex object */ }))

// Update by replacing reference
largeDataset.value = [...largeDataset.value, newItem]
```

### 5. Accessing Refs in Same Tick

**Avoid:**
```typescript
const message = ref('Hello')

message.value = 'World'
console.log(message.value)  // Logs "World" but DOM not updated yet

const el = ref<HTMLElement>()
el.value?.scrollIntoView()  // Might not work - DOM not updated
```

**Good:**
```typescript
import { ref, nextTick } from 'vue'

const message = ref('Hello')

message.value = 'World'
await nextTick()  // Wait for DOM update
console.log(message.value)  // DOM is now updated

const el = ref<HTMLElement>()
await nextTick()
el.value?.scrollIntoView()  // Works - DOM updated
```

### 6. Not Cleaning Up Side Effects

**Avoid:**
```typescript
// Memory leak - interval never cleaned up!
function useInterval() {
  const count = ref(0)

  setInterval(() => {
    count.value++
  }, 1000)

  return count
}
```

**Good:**
```typescript
import { ref, onUnmounted } from 'vue'

function useInterval() {
  const count = ref(0)

  const intervalId = setInterval(() => {
    count.value++
  }, 1000)

  onUnmounted(() => {
    clearInterval(intervalId)
  })

  return count
}
```

**Better - Use tryOnScopeDispose:**
```typescript
import { ref } from 'vue'
import { tryOnScopeDispose } from '@vueuse/core'

function useInterval() {
  const count = ref(0)

  const intervalId = setInterval(() => {
    count.value++
  }, 1000)

  tryOnScopeDispose(() => {
    clearInterval(intervalId)
  })

  return count
}
```

### 7. Default Exports

**Avoid:**
```typescript
// Hard to refactor, rename, and find usage
export default function useCounter() {
  // ...
}

export default defineComponent({
  // ...
})
```

**Good:**
```typescript
// Easy to search, refactor, and tree-shake
export function useCounter() {
  // ...
}

export const UserCard = defineComponent({
  name: 'UserCard',
  // ...
})
```

### 8. watch Without Cleanup

**Avoid:**
```typescript
watch(source, async (value) => {
  // Race condition - multiple requests in flight!
  const result = await fetchData(value)
  data.value = result
})
```

**Good:**
```typescript
watch(source, async (value, oldValue, onCleanup) => {
  let cancelled = false

  onCleanup(() => {
    cancelled = true
  })

  const result = await fetchData(value)

  if (!cancelled) {
    data.value = result
  }
})
```

## Testing Patterns

### Component Testing

**Good test structure using Testing Library:**
```typescript
import { render, screen, waitFor } from '@testing-library/vue'
import { describe, it, expect, vi } from 'vitest'
import userEvent from '@testing-library/user-event'
import UserCard from './UserCard.vue'

describe('UserCard', () => {
  const defaultProps = {
    user: { id: 1, name: 'John Doe', email: 'john@example.com' }
  }

  it('renders user information', () => {
    render(UserCard, {
      props: defaultProps
    })

    expect(screen.getByText('John Doe')).toBeInTheDocument()
    expect(screen.getByText('john@example.com')).toBeInTheDocument()
  })

  it('emits update event when editing', async () => {
    const user = userEvent.setup()
    const onUpdate = vi.fn()

    render(UserCard, {
      props: {
        ...defaultProps,
        onUpdate
      }
    })

    const editButton = screen.getByRole('button', { name: /edit/i })
    await user.click(editButton)

    expect(onUpdate).toHaveBeenCalledWith(defaultProps.user)
  })

  it('handles async data loading', async () => {
    render(UserCard, {
      props: { userId: 1 }
    })

    expect(screen.getByText(/loading/i)).toBeInTheDocument()

    await waitFor(() => {
      expect(screen.queryByText(/loading/i)).not.toBeInTheDocument()
    })

    expect(screen.getByText('John Doe')).toBeInTheDocument()
  })
})
```

### Composable Testing

**Good composable test:**
```typescript
import { describe, it, expect } from 'vitest'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { count } = useCounter()
    expect(count.value).toBe(0)
  })

  it('initializes with custom value', () => {
    const { count } = useCounter(10)
    expect(count.value).toBe(10)
  })

  it('increments count', () => {
    const { count, increment } = useCounter()
    increment()
    expect(count.value).toBe(1)
  })

  it('respects max value', () => {
    const { count, increment } = useCounter(0, { max: 5 })

    for (let i = 0; i < 10; i++) {
      increment()
    }

    expect(count.value).toBe(5)
  })
})
```

## Performance Patterns

### 1. v-once for Static Content

```vue
<template>
  <!-- Only renders once, never updates -->
  <div v-once>
    <h1>{{ staticTitle }}</h1>
    <p>{{ staticDescription }}</p>
  </div>
</template>
```

### 2. v-memo for Expensive Lists

```vue
<template>
  <div v-for="item in items" :key="item.id" v-memo="[item.id, item.name]">
    <!-- Only re-renders if item.id or item.name changes -->
    <ExpensiveComponent :item="item" />
  </div>
</template>
```

### 3. Virtual Scrolling

For very long lists (1000+ items), use virtual scrolling:

```vue
<script setup lang="ts">
import { useVirtualList } from '@vueuse/core'

const { list, containerProps, wrapperProps } = useVirtualList(
  allItems,
  { itemHeight: 50 }
)
</script>

<template>
  <div v-bind="containerProps" style="height: 400px">
    <div v-bind="wrapperProps">
      <div v-for="{ data, index } in list" :key="index">
        {{ data }}
      </div>
    </div>
  </div>
</template>
```

### 4. Debouncing Expensive Operations

```typescript
import { ref, watch } from 'vue'
import { useDebounceFn } from '@vueuse/core'

const searchQuery = ref('')
const results = ref([])

const debouncedSearch = useDebounceFn(async (query: string) => {
  results.value = await searchApi.search(query)
}, 300)

watch(searchQuery, (newQuery) => {
  debouncedSearch(newQuery)
})
```

## Summary

**Do:**
- Use Composition API with `<script setup>`
- Prefer `ref` over `reactive`
- Use `defineModel` for v-model
- Clean up side effects with `tryOnScopeDispose`
- Use named exports
- Test components and composables
- Optimize performance with `shallowRef`, `v-memo`, lazy loading

**Don't:**
- Mutate props directly
- Destructure reactive without `toRefs`
- Use Options API in new code
- Forget to clean up intervals/listeners
- Use default exports
- Deep observe large datasets
- Access refs before `nextTick` when DOM update is needed
