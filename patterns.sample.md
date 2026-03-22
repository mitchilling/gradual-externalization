Pattern: Leaky Abstraction
Signals:
- unclear lifecycle
- hidden dependencies
- naming doesn't reflect behavior

Heuristic:
If I need to read 3+ files to understand usage → abstraction is failing

Action:
- inline or simplify
- make lifecycle explicit
