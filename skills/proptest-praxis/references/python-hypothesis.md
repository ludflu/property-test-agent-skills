# Property testing in Python (Hypothesis)

Use this reference when the target repository is Python. Prefer **Hypothesis** with **pytest** unless the project already standardizes on another property-testing library.

## Framework choice and commands

- Install and run the way the repo already does (`uv`, `poetry`, `pip`, etc.). Typical dev dependency: `hypothesis`.
- Run focused tests with `pytest` paths and `-k` filters; use `pytest --hypothesis-show-statistics` when tuning generators.
- Reproduce a case with `@example(...)`, `hypothesis.reproduce_failure`, or the seed Hypothesis prints on failure.

## Mapping core techniques

| Concept in the skill | Hypothesis |
| --- | --- |
| Compositional strategies | `hypothesis.strategies` (`st.integers`, `st.lists`, `st.dictionaries`, `st.text`, `st.binary`, `st.one_of`, `st.builds`, `st.fixed_dictionaries`) |
| map / flat_map | `st.map`, `st.flatmap` |
| Recursive trees / ASTs | `st.recursive` |
| Assumptions (avoid reject-heavy filters) | `hypothesis.assume` sparingly; prefer `st.filter` or generation that builds valid values |
| Stateful operation sequences | `hypothesis.stateful`: `RuleBasedStateMachine`, `@rule`, `@invariant`, `@initialize`; compare production vs reference in shared rules |
| Draw inside a test | `@given(st.data())` and `data.draw(...)` for dependent generation |
| Shrinking | Automatic; tune with `max_examples`, `@settings`, or `@example` for regression locks |
| Failure persistence | Checked-in `@example` or `hypothesis.settings(deadline=..., print_blob=True)`; store minimal examples in regression tests |

## Model-based stateful differential testing

1. Define an explicit operation representation (dataclass, enum, or small AST) if replay outside Hypothesis helps debugging; otherwise encode operations as `@rule` methods on a `RuleBasedStateMachine`.
2. Subclass `RuleBasedStateMachine` with two instances in `@invariant` or after each rule: **production** (system under test) and **reference model** (simple dict, list, or pure functions).
3. Include mutations, reads, iteration, bulk ops, clear/reset, and copy/snapshot where the API exposes them. Generate initial configuration in `@initialize`.
4. Map handles or IDs in the reference; select operands from reference state, not from possibly wrong production outputs.
5. Use `Bundle` to draw operands tied to prior operations when the API returns handles.

## Generators and coverage

- **Near-valid inputs:** build valid structures with `st.builds` / `st.recursive`, then `st.one_of(valid, invalid_variant)` for single-constraint violations.
- **Partitioning:** combine `st.sampled_from` dimensions (e.g. warmed cache × overwrite × boundary size) via `st.tuples` or `st.fixed_dictionaries`.
- **Swarm-style mixes:** multiple test classes or `@settings(max_examples=...)` profiles with different rule weights using `RuleBasedStateMachine.settings` and targeted `@rule` subsets in separate machines.
- **Determinism:** avoid unseeded `random` in test bodies; use Hypothesis draws only.

## Oracle and failure workflow

- Prefer independent reference code in the test module or a small `reference_*` helper package on the discovery branch—not imports of production algorithms you are trying to validate.
- On failure, minimize with Hypothesis shrinking, then freeze as a plain pytest test with explicit inputs and assertions.
- Async code: `@given` with async tests requires pytest-asyncio patterns; keep oracles synchronous when possible.

## Python-specific cautions

- Respect actual API contracts for exceptions (`pytest.raises`) vs invalid internal state.
- Mutable default arguments, shared globals, and process-level caches can cause history-dependent flakiness; isolate state per example (`@settings(...)` `database=None` or fresh fixtures).
- For numeric code, use `st.integers`, `st.floats(allow_nan=..., allow_infinity=...)`, or decimals explicitly matching the implementation’s domain.
