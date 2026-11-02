# Property testing in TypeScript (fast-check)

Use this reference when the target repository is TypeScript or JavaScript. Prefer **fast-check** with the repo’s test runner (**Vitest**, **Jest**, or **node:test**) unless the project already standardizes on another property-testing library.

## Framework choice and commands

- Typical dependency: `fast-check`; runner integration often via `@fast-check/vitest` or `fc.configureGlobal` for Jest.
- Run the repo’s usual test command (`npm test`, `pnpm vitest`, etc.) with a path or name filter on property tests.
- Reproduce with `{ seed: number }` in `fc.assert` / runner options, or `fc.sample` while developing generators.

## Mapping core techniques

| Concept in the skill | fast-check |
| --- | --- |
| Compositional strategies | `fc.integer`, `fc.string`, `fc.array`, `fc.dictionary`, `fc.record`, `fc.oneof`, `fc.constant`, `fc.constantFrom`, `fc.uuid`, `fc.webUrl`, etc. |
| map / chain | `fc.map`, `fc.chain` |
| Recursive trees / ASTs | `fc.letrec` / `fc.memo` for recursive structures |
| Assumptions | `fc.pre(condition)`; prefer building valid inputs over filtering |
| Stateful operation sequences | `fc.modelBasedTesting` / command sequences (`fc.commands`, `command`) comparing SUT vs model |
| Properties | `fc.property(...args, (args) => { ... })`, `fc.asyncProperty` for async |
| Assert / shrink | `fc.assert(property, { numRuns, seed, endOnFailure })` |
| Failure persistence | Seed + minimal counterexample in a committed regression test; optional `fc.readablePathMapper` for debugging |

## Model-based stateful differential testing

1. Define a reference model (plain objects, Maps, arrays) updated in parallel with the production API.
2. Use **model-based testing**: generate a command list that runs the same logical operations against both model and implementation; compare observable results after each step or at invariants.
3. Include mutations, reads, iteration, bulk operations, clear/reset, and clone/snapshot when exposed. Generate initial config as part of the setup or first commands.
4. Track logical IDs for handles; choose operands from the reference model’s state.
5. For UI or async services, use `fc.asyncProperty` and await both paths; keep timeouts explicit.

## Generators and coverage

- **Near-valid inputs:** `fc.oneof(validArbitrary, invalidArbitrary)` or `fc.map` from a valid base with a single field corrupted.
- **Partitioning:** `fc.tuple` / `fc.record` over `fc.constantFrom` enums for operation class, cache state, and size bucket.
- **Swarm-style mixes:** separate `fc.assert` blocks or command arbitraries with different `maxCommands` / weighting via custom `fc.oneof` frequency arrays.
- **Determinism:** do not use `Math.random()` in properties; rely on fast-check’s RNG.

## Oracle and failure workflow

- Keep reference models in test helpers or a discovery-branch module that does not reuse the optimized production algorithm under test.
- After shrink, extract a deterministic unit test with fixed inputs (no `fc.assert`) for the PR regression.
- Strict equality: use `JSON.stringify` only when semantics match; prefer domain comparators for order-insensitive structures.

## TypeScript-specific cautions

- Distinguish `undefined`, missing keys, and `null` when the API cares; use typed `fc.record` and exact optional fields.
- Date, BigInt, and typed arrays need matching arbitraries (`fc.date`, custom bigint strategy, `fc.uint8Array`).
- ESM vs CJS and mock timers (Jest/Vitest fake timers) can change behavior; document timer mode in the harness.
- For unsafe casts (`as any`) in tests, treat them as test-only; production APIs should still be exercised through their public types.
