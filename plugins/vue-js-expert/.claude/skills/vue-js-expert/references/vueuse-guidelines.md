# VueUse Development Guidelines

This document contains official VueUse guidelines for building Vue composables and functions. Reference these patterns when creating reusable composables.

Source: https://vueuse.org/guidelines.html

## General Principles

### Vue API Imports
Import all Vue APIs directly from `"vue"` rather than other sources.

```typescript
// Good
import { ref, computed, watch } from 'vue'

// Avoid
import { ref } from '@vue/reactivity'
```

### Ref Over Reactive
Prefer `ref` to `reactive` whenever feasible for improved flexibility and predictability.

```typescript
// Preferred
const count = ref(0)
const user = ref({ name: 'John', age: 30 })

// Less flexible
const state = reactive({ count: 0, user: { name: 'John', age: 30 } })
```

**Why ref is preferred:**
- More flexible (can be reassigned)
- Works better with TypeScript inference
- Easier to pass around and compose
- Clear .value access makes reactivity explicit

### Shallow References
Use `shallowRef` instead of `ref` when managing large data volumes to avoid unnecessary deep reactivity overhead.

```typescript
// For large datasets (1000+ items)
const largeDataset = shallowRef([/* thousands of items */])

// For nested objects where only top-level changes matter
const config = shallowRef({
  nested: { deep: { values: {} } }
})

// Trigger update by replacing entire reference
config.value = { ...config.value, newProp: 'value' }
```

### Deep Reactivity
When deep reactivity is required, use explicitly named `deepRef` rather than standard `ref` for clarity.

```typescript
// Makes intent clear
const deepRef = <T>(value: T) => ref(value)

const nestedState = deepRef({
  level1: {
    level2: {
      level3: 'value'
    }
  }
})

// Deep mutations work
nestedState.value.level1.level2.level3 = 'new value'
```

## Function Design Patterns

### Options Objects
Accept options as objects rather than positional parameters to allow flexible future extensions without breaking changes.

```typescript
// Good - extensible
function useFeature(target: Ref<Element>, options?: {
  immediate?: boolean
  flush?: WatchOptions['flush']
  debounce?: number
}) {
  // Implementation
}

// Avoid - hard to extend
function useFeature(
  target: Ref<Element>,
  immediate?: boolean,
  flush?: WatchOptions['flush']
) {
  // Implementation
}
```

### Configurable Globals
Support `configurableWindow` and `configurableDocument` options to accommodate multi-window scenarios, testing environments, and server-side rendering contexts.

```typescript
import { ConfigurableWindow, defaultWindow } from '@vueuse/core'

interface UseEventListenerOptions extends ConfigurableWindow {
  passive?: boolean
  capture?: boolean
}

function useEventListener(
  target: Ref<EventTarget>,
  event: string,
  handler: EventListener,
  options?: UseEventListenerOptions
) {
  const { window = defaultWindow } = options || {}

  // Use window from options or default
  if (!window) return

  // Implementation
}
```

**Benefits:**
- Works in multi-window applications (iframes, popups)
- Testable with mock window/document objects
- SSR-safe (window may not exist)

### Watch Configuration
Make `immediate` and `flush` options configurable when using `watch` or `watchEffect` internally.

```typescript
interface UseWatcherOptions {
  immediate?: boolean
  flush?: 'pre' | 'post' | 'sync'
  deep?: boolean
}

function useWatcher(
  source: Ref<any>,
  callback: () => void,
  options?: UseWatcherOptions
) {
  const { immediate = false, flush = 'pre', deep = false } = options || {}

  watch(source, callback, { immediate, flush, deep })
}
```

### Cleanup Management
Employ `tryOnScopeDispose` to gracefully handle side-effect cleanup.

```typescript
import { tryOnScopeDispose } from '@vueuse/core'

function useEventListener(target: Ref<EventTarget>, event: string, handler: EventListener) {
  const cleanup = () => {
    target.value?.removeEventListener(event, handler)
  }

  target.value?.addEventListener(event, handler)

  // Automatically cleanup when component unmounts
  tryOnScopeDispose(cleanup)

  // Also return cleanup for manual control
  return cleanup
}
```

**Why tryOnScopeDispose:**
- Works both inside and outside component setup
- Gracefully handles edge cases (not in active effect scope)
- Prevents memory leaks

## API Design

### Feature Detection
Output an `isSupported` flag when functions involve Web APIs with limited browser support.

```typescript
import { ref, computed } from 'vue'

function useGeolocation() {
  const isSupported = computed(() =>
    'geolocation' in navigator
  )

  const coords = ref<GeolocationCoordinates | null>(null)
  const error = ref<GeolocationPositionError | null>(null)

  if (isSupported.value) {
    // Implementation
  }

  return {
    isSupported,
    coords,
    error
  }
}
```

### Asynchronous Composables
Return PromiseLike objects from async functions to enable usage with Vue's `<Suspense>` component.

```typescript
export function useAsyncData<T>(
  fetcher: () => Promise<T>
): PromiseLike<{ data: Ref<T | null>, error: Ref<Error | null> }> {
  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)

  const promise = fetcher()
    .then(result => {
      data.value = result
      return { data, error }
    })
    .catch(err => {
      error.value = err
      return { data, error }
    })

  // Return object with promise methods
  return {
    data,
    error,
    then: promise.then.bind(promise),
    catch: promise.catch.bind(promise)
  }
}

// Usage with Suspense
// <Suspense>
//   <template #default>
//     <Component />
//   </template>
//   <template #fallback>
//     <Loading />
//   </template>
// </Suspense>
```

### Controls Option
Provide a `controls` option for functions commonly used with single returns, allowing users to choose between simple or advanced modes.

```typescript
// Simple mode - just returns the value
const counter = useCounter()
// counter is Ref<number>

// Advanced mode - returns value + controls
const { count, inc, dec, reset } = useCounter({ controls: true })
// Full control object with methods
```

**Implementation:**
```typescript
function useCounter(initialValue = 0, options?: { controls?: boolean }) {
  const count = ref(initialValue)

  const inc = () => count.value++
  const dec = () => count.value--
  const reset = () => count.value = initialValue

  if (options?.controls) {
    return { count, inc, dec, reset }
  }

  return count
}
```

## Code Quality

### Logging
Avoid console logging in library code.

```typescript
// Avoid
function useDebug() {
  console.log('Debug info')  // Don't do this in libraries
}

// If debugging is needed, use a debug option
function useFeature(options?: { debug?: boolean }) {
  if (options?.debug) {
    // User explicitly enabled debugging
  }
}
```

### Naming Consistency
Follow established naming patterns across the codebase for discoverability and predictability.

**Common prefixes:**
- `use*` - Composables that use reactivity (useCounter, useMouse)
- `create*` - Factory functions (createEventHook, createGlobalState)
- `on*` - Event listeners (onClickOutside, onKeyStroke)
- `try*` - Safe versions that won't throw (tryOnMounted, tryOnScopeDispose)

**Common suffixes:**
- `*Ref` - Returns a ref (computedRef, syncRef)
- `*Reactive` - Returns reactive object (reactiveState)

## Renderless Components

### Render Functions
Use render functions rather than Single File Components for wrapper components.

```typescript
import { defineComponent, h } from 'vue'

export const MyWrapper = defineComponent({
  name: 'MyWrapper',
  setup(props, { slots }) {
    // Logic here

    return () => h('div', { class: 'wrapper' }, slots.default?.())
  }
})
```

**Why render functions:**
- Smaller bundle size
- More flexible
- Better TypeScript support
- No template compilation needed

### Props Reactivity
Wrap component props in `reactive` for seamless prop forwarding to slots.

```typescript
import { defineComponent, reactive, toRefs } from 'vue'

export const PropsForwarder = defineComponent({
  props: {
    count: Number,
    message: String
  },
  setup(props, { slots }) {
    // Make props reactive for slot binding
    const reactiveProps = reactive(props)

    return () => slots.default?.(reactiveProps)
  }
})
```

### Type Reuse
Leverage function option types as component prop types rather than duplicating type definitions.

```typescript
// Composable with options
interface UseCounterOptions {
  min?: number
  max?: number
  step?: number
}

function useCounter(initialValue: number, options?: UseCounterOptions) {
  // Implementation
}

// Component reuses the same type
import { defineComponent, PropType } from 'vue'

export const Counter = defineComponent({
  props: {
    initialValue: Number,
    options: Object as PropType<UseCounterOptions>
  },
  setup(props) {
    const counter = useCounter(props.initialValue, props.options)
    return { counter }
  }
})
```

## Summary Checklist

When creating a new composable, ensure:

- [ ] Import Vue APIs from `"vue"`
- [ ] Prefer `ref` over `reactive`
- [ ] Use `shallowRef` for large data
- [ ] Accept options as objects, not positional params
- [ ] Include `configurableWindow`/`configurableDocument` if using browser APIs
- [ ] Make `immediate` and `flush` configurable for internal watchers
- [ ] Use `tryOnScopeDispose` for cleanup
- [ ] Return `isSupported` for Web APIs
- [ ] Return PromiseLike for async composables
- [ ] Provide `controls` option for common single-return functions
- [ ] No console logging
- [ ] Follow naming conventions
- [ ] Use render functions for components
- [ ] Reuse types between functions and components
