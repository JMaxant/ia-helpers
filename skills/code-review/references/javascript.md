# JavaScript / TypeScript Conventions

**Detection triggers**: presence of `package.json`. Presence of `typescript` as a dependency or a `tsconfig.json` → also apply the TypeScript typing points. Version read from `package.json` (`engines.node`) and from framework dependencies where relevant.

- Use destructuring for objects and arrays.
- Prefer `const` and `let` over `var`.
- Use arrow functions for callbacks.
- Avoid side effects in pure functions.
- Type with JSDoc or TypeScript where applicable.
- Handle errors with appropriate try/catch or Promises.
