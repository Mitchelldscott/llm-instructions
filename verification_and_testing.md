# Verification and Testing

**MANDATORY TESTING STANDARDS:**

1.  **UNIT TESTS (Module-Level)**
    - **Requirement:** 100% line coverage for critical algorithms.
    - **Structure:** Must follow `ARRANGE` -> `ACT` -> `ASSERT` pattern.
    - **Scope:** Verify normal paths, error handling, boundaries, and panic conditions.
    - **Location:** `src/module_name/tests.rs` or `#[cfg(test)]`.

2.  **INTEGRATION TESTS (System-Level)**
    - **Requirement:** Verify cross-module interactions and stability invariants.
    - **Location:** `tests/` directory.

3.  **PERFORMANCE VERIFICATION (Real-Time)**
    - **Requirement:** WCET (Worst-Case Execution Time) analysis using Criterion.
    - **Constraint:** Must detect regressions and quantify execution time against budget.

4.  **FORMAL VERIFICATION (Safety-Critical)**
    - **Requirement:** Kani, Prusti, or Coq proofs required for:
        - Core control loops
        - State estimators
        - Unsafe code blocks
    - **Output:** Provide proof stubs if full formal verification is not possible immediately.

5.  **CI/CD ENFORCEMENT**
    - Every commit must pass:
```bash
cargo check
cargo fmt --check
cargo clippy -- -D warnings
cargo test --all-targets
cargo test --doc
cargo doc --no-deps
cargo bench --no-run
cargo audit
```

## VERIFICATION CHECKLIST

Before declaring code complete:

- [ ] All unit tests pass
- [ ] Integration tests pass
- [ ] 100% line coverage for critical code
- [ ] All error paths tested
- [ ] All panic conditions documented and tested
- [ ] Benchmarks show no regressions
- [ ] WCET analysis complete (if real-time)
- [ ] Formal proofs or verification stubs present (if critical)
- [ ] Code reviews completed
- [ ] CI pipeline green (all checks pass)

**Failure in any check blocks merge.**

**OPERATIONAL RULE:**
You may generate code for the user without generating tests, but always remind 
the user that it is required. If any code (that is not under development) is 
not covered by tests, it is a critical error and must be reported to the 
developer immediately. 

---