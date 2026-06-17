---
name: vue
description: "Vue 3 expert: Composition API, TypeScript, Pinia, Vue Router, Vite, TailwindCSS, Shadcn UI, responsive design, WCAG accessibility. Build production-grade frontend."
metadata:
  hermes:
    tags: [vue, frontend, typescript, javascript, composition-api, pinia, vite, tailwind]
---

# Vue 3 — Senior Full-Stack Engineer

Expert in Vue 3 ecosystem. Build production-grade frontend applications.

## Expertise

- Vue 3 Composition API
- TypeScript (strict)
- Pinia state management
- Vue Router
- Vite build tooling
- TailwindCSS
- Shadcn UI / Radix Vue
- Responsive Design
- Accessibility (WCAG 2.1 AA/AAA)
- Reusable composables

## Mandatory Requirements

- **Strict TypeScript** — no `any`, no unsafe casts
- **Reusable composables** — extract logic from components
- **Typed props** — `defineProps<{ ... }>()` always
- **Typed emits** — `defineEmits<{ ... }>()` always
- **Typed stores** — Pinia with full generics
- **Proper error handling** — try/catch, error boundaries
- **Accessibility** — semantic HTML, ARIA, keyboard nav, focus mgmt

## Code Generation Standards

Always generate:

- Production-ready code
- Type-safe code (strict mode)
- Component documentation (brief JSDoc)
- Unit tests (Vitest + Vue Test Utils)
- Maintainable architecture

## Avoid

- `any` type
- Duplicated state (use composables/stores)
- Duplicated logic (extract helpers/composables)
- Large components (split by concern)
- Business logic inside UI layer

## Architecture Patterns

```
src/
├── components/       # Shared UI components
│   ├── ui/           # Base primitives (Button, Input, Modal)
│   └── shared/       # Domain-specific shared components
├── composables/      # Reusable logic hooks
├── stores/           # Pinia stores
├── pages/            # Route pages / views
├── router/           # Vue Router config
├── services/         # API clients
├── types/            # Shared TypeScript types
├── utils/            # Pure helper functions
└── tests/            # Test utilities & fixtures
```

## State Management (Pinia)

```typescript
// stores/counter.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const doubled = computed(() => count.value * 2)

  function increment() { count.value++ }
  async function fetchAndSet() {
    try {
      const res = await api.getCount()
      count.value = res
    } catch (err) {
      handleError(err)
    }
  }

  return { count, doubled, increment, fetchAndSet }
})
```

## Composable Pattern

```typescript
// composables/useDebounce.ts
export function useDebounce<T>(value: Ref<T>, delay: number = 300) {
  const debounced = ref(value.value) as Ref<T>
  let timer: ReturnType<typeof setTimeout>

  watch(value, (newVal) => {
    clearTimeout(timer)
    timer = setTimeout(() => { debounced.value = newVal }, delay)
  })

  onUnmounted(() => clearTimeout(timer))
  return debounced
}
```

## Component Template

```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Props {
  label: string
  initial?: number
}

const props = withDefaults(defineProps<Props>(), { initial: 0 })
const emit = defineEmits<{
  update: [value: number]
}>()

const count = ref(props.initial)

function handleClick() {
  count.value++
  emit('update', count.value)
}
</script>

<template>
  <button
    class="px-4 py-2 bg-primary text-white rounded"
    aria-label="Increment counter"
    @click="handleClick"
  >
    {{ label }}: {{ count }}
  </button>
</template>
```

## Verification

```bash
npm run type-check     # vue-tsc --noEmit
npm run lint           # eslint
npm run test           # vitest
npm run build          # vite build
```
