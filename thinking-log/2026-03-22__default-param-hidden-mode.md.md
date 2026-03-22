# [2026-03-22]

## Context

Reviewing MR:  
https://github.com/lynx-family/lynx/commit/7e4cc0736a8cb5086ee2deefc64053435543df11

Change introduces a `depth` parameter (default `-1`) to:

```cpp
static Json::Value GetDocumentBodyFromNode(Element* ptr, int depth = -1);
```

And threads it through DevTools so `DOM.getDocument` can return partial DOM trees.

---

## Observation

The addition of `depth` with a default value feels slightly off.

The change is small and practical:
- backward compatible
- minimal code churn
- enables partial DOM retrieval

But it introduces a subtle design smell:

- A new **behavior mode** is being encoded via parameter values (`-1`, `0`, ...)
- The default parameter hides this behavioral choice from call sites
- The function now represents multiple semantics, but its name/signature doesn’t reflect that clearly

This creates a mismatch between **what the API does** and **how it presents itself**

---

## Why

### 1. Behavior mode encoded as value

`depth` is not just data—it controls behavior:

- `-1` → full tree
- `>= 0` → limited depth

This introduces implicit conventions:
- meaning of `-1` is not self-evident
- callers depend on shared understanding rather than explicit API

Feels like:
> a semantic concept is being compressed into a primitive value

---

### 2. Default parameter hides intent

Call sites like:

```cpp
GetDocumentBodyFromNode(root);
```

do not communicate:
- whether “full tree” is intentional
- or just inherited from default

This weakens readability and makes future changes risky:
- hard to tell which callers depend on current default
- changing default behavior becomes dangerous

---

### 3. “约定即负担” (convention is burden)

Using sentinel values (`-1`, `0`) creates implicit agreements:
- cheap now
- costly later

They:
- increase cognitive load
- constrain future evolution
- require every reader to learn hidden rules

---

### 4. Time trade-off (technical debt)

The use of default parameter is mainly for:
- backward compatibility
- avoiding widespread changes

Which effectively means:
> deferring proper modeling of behavior modes

This is a classic trade:
- save time now
- pay complexity later

---

### 5. Mismatch with existing style / taste

My general preference:

- Different behavior modes → different API shapes  
  (overloads, distinct functions, or explicit options)

This:
- makes intent visible
- keeps semantics aligned with structure
- scales better over time

The current approach feels like a **tactical patch**, not a consistent abstraction

---

## Decision

- Left feedback as **non-blocking**
- Highlighted:
  - hidden convention (`-1`)
  - implicit behavior via default parameter
  - inconsistency with existing API style
  - long-term cost vs short-term convenience

- Suggested direction:
  - separate APIs or clearer abstraction for different modes

Did not push for immediate refactor due to:
- small scope
- practical benefit
- cost of changing existing call sites

---

## Confidence

Medium

Confident in:
- the underlying design concerns
- long-term risks of this pattern

Less certain about:
- whether this change justifies stronger enforcement in this context
