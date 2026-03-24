# [2026-03-24]

## Context

Relevant code:

- `DevToolState`
- `DevToolLifecycle`
- `DevToolSettings`
- `LynxEnv`
in [Lynx](https://github.com/lynx-family/lynx)

Question:
Whether `DevToolSettings` should check `DevToolLifecycle.isEnabled()` before returning values.

## Observation

There was an attempt to make `DevToolSettings` conditionally return values based on runtime state (e.g. engine enabled/disabled).

We decided not to do this, and keep `DevToolSettings` as a pure data layer.

The key tension:

> Should a settings layer return what the user configured,
> or what is currently effective in the system?

## Why

### 1. Mixing preference with runtime state leads to “lying data”

Example:

- User enables LogBox → preference = `true`
- Engine is disabled → LogBox cannot actually run

If `DevToolSettings` returns `false` in this case:

- it no longer reflects the user’s intent
- it reflects a *derived constraint* instead

Now the same field is representing:

- sometimes preference
- sometimes effective behavior

This makes the data unreliable.

### 2. Loss of semantic clarity

Two distinct concepts are being collapsed:

- **User Preference** → what is configured / persisted
- **Effective State** → what the system can actually do at runtime

They have different:

- lifecycles
- sources of truth
- responsibilities

Merging them creates ambiguity:

- what does this value actually mean?
- can I trust it?

### 3. Hidden coupling between layers

If `DevToolSettings` depends on `DevToolLifecycle`:

- data layer becomes aware of runtime state
- responsibilities bleed across layers

This introduces:

- tighter coupling
- harder reasoning
- more fragile evolution

### 4. Facade is the correct composition point

The “effective” behavior is a function of:

- user preference
- runtime conditions (lifecycle, environment)

This combination is inherently **cross-cutting logic**.

Placing it in a facade (`LynxEnv`) makes:

- composition explicit
- responsibilities clearer
- behavior easier to reason about

## Decision

- Keep `DevToolSettings` as a **pure preference / persistence layer**
- Do not mix runtime checks into its getters
- Compute **effective state** at a higher layer (`LynxEnv`) by combining:
  - preference (`DevToolSettings`)
  - runtime state (`DevToolLifecycle`)

This preserves:

- correctness of stored data
- clarity of responsibility boundaries

## Confidence

High

The separation between:

- “what is configured”
- “what is currently possible”

is fundamental and widely applicable.

Mixing them consistently leads to:

- ambiguity
- coupling
- incorrect assumptions by callers
