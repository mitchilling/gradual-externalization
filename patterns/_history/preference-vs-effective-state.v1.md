# Pattern: Preference vs Effective State

## Signals

- A value represents both **user configuration** and **runtime capability**
- Getters modify returned values based on system state (e.g. lifecycle, environment)
- The same field returns different meanings depending on context
- Callers cannot tell whether a value is:
  - “what the user set”, or
  - “what is currently possible”
- Data/persistence layer depends on runtime or lifecycle state

## Heuristic

> If a value can represent both:
>
> - **"what is configured (source of truth), and"**
> - **"what is currently effective (derived state)"**
>
> → these must be modeled separately.

A single field should not carry both meanings.

## Action

### 1. Separate the concepts explicitly

- **Preference**
  - user-defined
  - persisted
  - stable
- **Effective State**
  - computed
  - depends on runtime conditions
  - may change without user input

### 2. Keep the data layer pure

- Settings / storage should return **raw configured values**
- No conditional logic based on runtime state

### 3. Compute effective state at composition layer

Use a higher-level abstraction (e.g. facade) to combine:

- preference (`Settings`)
- runtime state (`Lifecycle`, environment, etc.)

## Rationale

- Prevents “lying data” (values that no longer reflect user intent)
- Preserves a clear **source of truth**
- Makes semantics explicit and predictable
- Avoids hidden coupling between layers
- Improves debuggability:
  - can inspect preference and effective state independently

## Trade-offs

- Slightly more verbose API (two concepts instead of one)
- Requires an additional composition layer
- Callers must choose which value they need

## Examples

```java
// ❌ Mixed meaning (preference + runtime state)
boolean isLogBoxEnabled() {
  return userSetting && lifecycle.isEnabled();
}
```

```java
// ✅ Separated concepts
boolean isLogBoxPreference();   // what user configured
boolean isLogBoxEffective();    // derived from preference + runtime
```

## References

2026-03-24__preference-vs-effective-state.md
