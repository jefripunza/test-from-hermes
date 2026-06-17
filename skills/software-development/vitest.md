---
name: vitest
description: "Vitest + Vue Test Utils expert: unit testing, component testing, mocking, coverage analysis. Min 80% coverage, preferred 90%+."
metadata:
  hermes:
    tags: [vitest, testing, vue, coverage, unit-test]
---

# Vitest — Unit & Component Testing

Fast unit testing with Vitest + Vue Test Utils. Target coverage: **min 80%, preferred 90%+**.

## Expertise

- Vitest test runner
- Vue Test Utils (mount, shallowMount)
- Mocking (vi.mock, vi.spyOn)
- Coverage analysis (c8/istanbul)
- Snapshot testing
- DOM testing (@testing-library/vue)

## Must Test

- **Happy paths** — component renders correctly
- **Error handling** — error states display correctly
- **Edge cases** — empty state, boundary values, null props
- **User interactions** — clicks, inputs, emits
- **Composables** — standalone composable testing
- **Stores** — Pinia store testing
- **Async behavior** — loading states, API calls

## Setup

```bash
npm install -D vitest @vue/test-utils jsdom
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'jsdom',
    globals: true,
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      thresholds: {
        statements: 80,
        branches: 80,
        functions: 80,
        lines: 80,
      },
    },
  },
})
```

## Component Test

```typescript
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import Counter from './Counter.vue'

describe('Counter', () => {
  it('renders initial count', () => {
    const wrapper = mount(Counter, { props: { label: 'Count' } })
    expect(wrapper.text()).toContain('Count')
  })

  it('emits update on click', async () => {
    const wrapper = mount(Counter)
    await wrapper.find('button').trigger('click')
    expect(wrapper.emitted('update')).toBeTruthy()
    expect(wrapper.emitted('update')?.[0]).toEqual([1])
  })

  it('handles error state', async () => {
    const wrapper = mount(Counter, { props: { error: 'Failed' } })
    expect(wrapper.find('[data-testid="error"]').text()).toBe('Failed')
  })
})
```

## Composable Test

```typescript
import { ref } from 'vue'
import { describe, it, expect, vi } from 'vitest'

function useCounter(initial = 0) {
  const count = ref(initial)
  function increment() { count.value++ }
  return { count: count as Ref<number>, increment }
}

describe('useCounter', () => {
  it('increments', () => {
    const { count, increment } = useCounter(5)
    expect(count.value).toBe(5)
    increment()
    expect(count.value).toBe(6)
  })
})
```

## Coverage Targets

| Area | Min | Preferred |
|------|-----|-----------|
| Overall | 80% | 90%+ |
| Components | 80% | 90%+ |
| Composables | 85% | 95%+ |
| Stores | 85% | 95%+ |
| Utils | 90% | 100% |
