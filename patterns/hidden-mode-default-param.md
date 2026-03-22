# Pattern: Hidden Mode via Default Parameter

## Signals

- A function parameter uses sentinel values (e.g. `-1`, `0`) to encode different behaviors  
- A default parameter represents a **non-trivial behavioral choice**  
- The function name does not reflect multiple behavior modes  
- Call sites omit the parameter and rely on the default  
- Understanding behavior requires knowledge of implicit conventions  

---

## Heuristic

If a parameter changes **behavioral mode** (not just data),  
and especially if it relies on sentinel values or defaults →  
it should likely be modeled as an explicit API or type.

---

## Action

Prefer one of:

1. **Separate APIs**
   - `GetFullDocument(...)`
   - `GetDocumentWithDepth(...)`

2. **Explicit options object**
   ```cpp
   struct SnapshotOptions {
     int depth;
     // potentially more fields in the future
   };
   ```

3. **Avoid default parameters for behavior**
   - Require callers to explicitly specify behavior when it matters  

---

## Rationale

- Makes intent visible at call sites  
- Avoids hidden conventions (“约定即负担”)  
- Reduces risk when changing defaults  
- Improves long-term evolvability of the API  

---

## Trade-offs

- Higher short-term cost (more explicit code, more changes required)  
- May not be worth enforcing in small or low-impact changes  

---

## Examples

```cpp
GetDocumentBodyFromNode(node);        // ❌ intent unclear
GetDocumentBodyFromNode(node, -1);    // ❌ relies on convention

GetFullDocument(node);                // ✅ explicit
GetDocumentWithDepth(node, 2);        // ✅ explicit
```

---

## References

- 2026-03-22__default-param-hidden-mode.md
