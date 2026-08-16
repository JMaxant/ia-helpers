# JavaScript / TypeScript Conventions

**Detection triggers**: presence of `package.json`. Presence of `typescript` as a dependency or a `tsconfig.json` → also apply the TypeScript typing points. Version read from `package.json` (`engines.node`) and from framework dependencies where relevant.

- Use destructuring for objects and arrays.
- Prefer `const` and `let` over `var`.
- Use arrow functions for callbacks.
- Avoid side effects in pure functions.
- Type with JSDoc or TypeScript where applicable.
- Handle errors with appropriate try/catch or Promises.

## Performance and scalability

- Avoid `await` inside a loop when the iterations are independent: use `Promise.all`/`Promise.allSettled`, with a concurrency bound if the input size is not controlled.
- Do not block the event loop with heavy synchronous computation; move it off the main thread (worker) or chunk it.

## Vue.js 3

**Detection triggers**: `vue` dependency at major version 3 in `package.json` (or `@vitejs/plugin-vue`/`vue-loader` present), and/or `.vue` files in the diff. Version read from the `vue` dependency constraint.

- Prefer the Composition API with `<script setup>` over the Options API for new components.
- Type props and emits with `defineProps`/`defineEmits` (generics if TypeScript); avoid mutating props directly — emit an event or use a local `ref`/`computed` instead.
- Use `computed` for derived state rather than recomputing it in methods or the template.
- Prefer `ref` for primitives and `reactive` for object-shaped local state; avoid mixing both for the same piece of state.
- Component names: multi-word, PascalCase, to avoid clashing with current and future HTML elements (Vue style guide rule).
- Always provide a `:key` with `v-for`; never combine `v-if` and `v-for` on the same element.
- Scope component styles (`<style scoped>` or CSS modules) to avoid leaking rules globally.
- Extract reusable reactive logic into composables (`useXxx` functions) rather than duplicating it across components or relying on mixins.
- For shared/global state, prefer Pinia over Vuex on new code.
- Two-way binding on a single prop: use `defineModel` (Vue 3.4+) rather than manually wiring `modelValue`/`update:modelValue` when the target version supports it.
- Virtualize long lists rather than rendering thousands of `v-for` nodes at once.
- Use `shallowRef`/`shallowReactive` for large payloads that are replaced wholesale instead of mutated, to avoid deep reactivity conversion on every assignment.
