# Requirements Document Template

<!--
  Instructions for use:
  1. Fill in this template
  2. Send to PM Agent via Telegram/Discord/WebChat:
     "Build from requirements: /path/to/this/file.md"
  3. The pipeline runs autonomously — you only need to approve push at the end
-->

---

## Project Context

**Project**: [Name of the project or module]
**Target layer**: [L0 Core / L1 Compute / L2 MCP Server / L3 Skill / L4 Agent]
**Estimated scope**: [Small (1-3 tasks) / Medium (4-8 tasks) / Large (9-15 tasks)]

---

## Problem Statement

<!-- What problem does this solve? Why does it need to exist? -->

---

## Functional Requirements

<!-- List each capability the implementation must provide.
     Be specific: inputs, outputs, expected behavior. -->

### FR-1: [Feature name]
- **Input**: [what goes in]
- **Output**: [what comes out]
- **Behavior**: [precise description of what it does]
- **Edge cases**: [invalid input, boundary conditions]

### FR-2: [Feature name]
- **Input**:
- **Output**:
- **Behavior**:
- **Edge cases**:

<!-- Add more FRs as needed -->

---

## Non-Functional Requirements

### Performance
- [ ] [Specific performance requirement, e.g., "must process 1000 samples in < 100ms"]

### Compatibility
- [ ] Must work with Python 3.11+
- [ ] Must not break existing imports
- [ ] [Other compatibility constraints]

### Code quality
- [ ] All new functions must have docstrings
- [ ] Type annotations required for all function signatures
- [ ] Test coverage must be ≥ 80% for new code

---

## Interface Constraints

<!-- If this must integrate with existing code, specify exactly how -->

### Existing code it must call:
```python
# Example: the new code must use this existing function
from core.simulation import run_simulation
```

### Existing code that must call it:
```python
# Example: this existing code will call the new function
from core.NEW_MODULE import new_function
result = new_function(param1, param2)
```

---

## Acceptance Criteria

<!-- Binary conditions — either pass or fail, no ambiguity -->

- [ ] AC-1: `pytest tests/test_NEW_MODULE.py` passes with 0 failures
- [ ] AC-2: `from core.NEW_MODULE import new_function` works without error
- [ ] AC-3: [Specific behavioral assertion]
- [ ] AC-4: No existing tests in `pytest tests/` are broken

---

## Out of Scope

<!-- Explicitly list what should NOT be implemented -->

- [Thing that might seem related but is excluded]
- [Another exclusion]

---

## Example Usage

```python
# Concrete example of how the implemented feature will be used
from core.new_module import NewClass

obj = NewClass(param=value)
result = obj.process(data)
print(result)  # Expected: {...}
```

---

## Reference Files

<!-- Files the planner and coder agents should read for context -->

- `core/simulation/__init__.py` — existing patterns to follow
- `data/tank_config.json` — configuration format reference
- `tests/test_core/test_simulation.py` — test structure to mimic
