# Pattern: Preference vs Effective State

## Signals

- A single field or API represents both:
  - **user preference / intent**
  - **runtime capability / system state**
- Getters modify stored values based on lifecycle or environment
- Same value may mean:
  - “user configured false”
  - OR “temporarily unavailable”
- Callers cannot distinguish *cause* from *result*
- Data layer depends on runtime conditions
- Temporary system constraints overwrite persistent intent
- String-keyed maps or generic channels are used to carry both concepts

---

## Heuristic

> If two dimensions differ in **source, lifecycle, or meaning**,
> they must not share the same representation.

Especially separate:

- **Preference**: persistent, user-driven, source of truth
- **Effective State**: derived, runtime-dependent, may change without user input

A single value should never represent both.

---

## Core Formula

```text
EffectiveState = f(Preference, SystemState)
```

---

## Action

1. Keep preference as source of truth
2. Model system state independently
3. Compute effective state at composition boundary
4. Avoid collapsing through weak channels

---

## Rationale

- Prevents loss of information
- Preserves source of truth
- Prevents lying data
- Keeps layer boundaries clean

---

## Trade-offs

- Slightly more verbose APIs
- Requires explicit composition layer
- More objects / state channels

---

## Examples

```java
boolean isLogBoxEnabled() {
  return userSetting && lifecycle.isEnabled();
}
```

```java
boolean isLogBoxPreference();
boolean isLogBoxEffective();
```

---

## References

- 2026-03-24__preference-vs-effective-state.md
- 2026-03-24__intent-vs-system-state-loss-of-information.md
