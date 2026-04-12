---
description: "Use when implementing features, writing code, or creating new source files. Enforces test-first development — tests must be written before implementation code."
applyTo: '{src,lib,packages}/**'
---

# Test-First Development

When creating or modifying any source file:

1. **Write or update tests FIRST** that describe the intended behavior based on success criteria
2. **Run tests** to confirm they fail for the right reason (the feature isn't implemented yet)
3. **Implement** the minimum code to make all tests pass
4. **Run tests again** to confirm they pass
5. **Do not refactor** unrelated code in the same change

Tests should describe behavior, not implementation details. A good test breaks when behavior changes, not when internals are refactored.
