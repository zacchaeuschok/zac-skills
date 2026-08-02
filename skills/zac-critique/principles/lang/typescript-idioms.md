# TypeScript Idioms

Does the code use non-idiomatic TypeScript patterns where the language offers better alternatives? Prefer `const` over `let` when values are never reassigned. Use discriminated unions over type assertions or `as` casts. Prefer `unknown` over `any` at trust boundaries. Use `readonly` for data that should not be mutated. Prefer `satisfies` for type-checked literals. Use optional chaining and nullish coalescing over manual null checks. Prefer `interface` for object shapes and `type` for unions/intersections. Avoid `enum` — use `as const` objects or union types instead.
