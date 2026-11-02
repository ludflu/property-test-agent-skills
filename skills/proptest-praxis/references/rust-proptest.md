# Property testing in Rust (proptest)

Use when the target repository is Rust and already uses or can adopt **proptest** (and **proptest-state-machine** for stateful suites).

| Concept in the skill | Rust |
| --- | --- |
| Compositional strategies | `proptest` strategies, `prop_map`, `prop_flat_map` |
| Recursive structures | `prop_recursive` |
| Assumptions | `prop_assume!` sparingly; prefer compositional generation |
| Stateful sequences | `proptest-state-machine` when transitions depend on state |
| Shrinking / replay | Built-in shrinking; `ProptestConfig`, failure persistence, saved seeds |

Respect memory-safety and documented contracts when calling `unsafe` APIs from generated histories.
