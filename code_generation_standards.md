# Code Generation Standards

## CODE QUALITY BASELINE (MANDATORY)

### A. LANGUAGE: RUST (Primary)

**Compiler Settings (crate root):**
```rust
#![no_std]
#![forbid(unsafe_code)]  // OR justify all unsafe blocks
#![deny(
    unused,
    missing_docs,
    clippy::all,
    clippy::pedantic,
    clippy::nursery,
    clippy::cargo,
    clippy::suspicious,
    clippy::complexity,
    clippy::unwrap_used,
    clippy::expect_used,
    clippy::panic,
    clippy::unimplemented,
    clippy::todo,
    clippy::indexing_slicing,
    clippy::big_endian_bytes,
)]
#![warn(rust_2018_idioms)]
```

**Formatting:**
- ALL code must pass `cargo fmt` without modification
- Indentation: 4 spaces
- Line length: 100 characters (soft), 120 characters (hard limit)
- Use project's `rustfmt.toml` configuration

**Linting:**
- ZERO clippy warnings
- ZERO compiler warnings
- All unsafe blocks require explicit justification (see safety_enforcement.md)

### B. CODE STRUCTURE

**File Organization:**
```
src/
  lib.rs (or main.rs)           // Module declarations and module-level docs
  module_name/
    mod.rs                      // Module documentation, re-exports
    submodule.rs                // Implementation
    tests.rs                    // #[cfg(test)] tests
benches/
  benchmark_name.rs             // Criterion benchmarks for WCET analysis
tests/
  integration_test.rs           // Cross-module integration tests
```

**Function Signature Requirements:**
- Explicit return types (no inference for public APIs)
- Explicit parameter types (no inference)
- Generic constraints fully specified (`where` clauses required)
- Error types explicitly defined (use Result<T, E>, not implicit unwrap)

**No Magic Numbers:**
- All constants defined as `const` or associated constants
- Reasons for constant values documented
- Physical units specified in comments (e.g., `// meters per second`)

### C. ERROR HANDLING

**Mandatory Requirements:**
- Use Result<T, E> for fallible operations
- Define explicit error enums per module
- Never use unwrap() or expect() in library code
- Never use panic!() for control flow

**Error Propagation:**
```rust
// ✓ CORRECT: Propagate errors up the call stack
pub fn operation() -> Result<T, MyError> {
    let value = fallible_operation()?;
    Ok(process(value))
}

// ✗ WRONG: Silences errors
pub fn operation() -> T {
    fallible_operation().unwrap_or_default()
}
```

### D. UNSAFE CODE POLICY

**If forbid(unsafe_code) is NOT set:**

1. Every unsafe block requires:
   - Explicit `// SAFETY:` comment
   - Detailed explanation of invariants maintained
   - Proof sketch of soundness
   - References to formal verification if applicable

2. Unsafe code must be:
   - Minimal (smallest possible scope)
   - Encapsulated (not exposed in public API)
   - Tested exhaustively (including edge cases)
   - Formally verified (for critical functions)

Example:
```rust
/// # Safety
/// The caller must guarantee:
/// - [Precondition] `ptr` points to valid, initialized memory
/// - [Invariant] Memory is properly aligned for type T
/// - [Postcondition] Dereference produces valid T instance
unsafe fn dereference_ptr<T>(ptr: *const T) -> T {
    std::ptr::read(ptr)
}
```

### E. NUMERIC TYPES AND ARITHMETIC

**Fixed-Point Arithmetic:**
- No automatic conversions between types
- Overflow/underflow must be detected or prevented
- Document scaling factors and precision loss

**Floating-Point Arithmetic:**
- Comply with IEEE-754
- Bound round-off errors in critical algorithms
- Document precision requirements (e.g., ±1e-6)
- Use checked operations where overflow is possible

**Constraints:**
- No mixed-type arithmetic without explicit cast
- Matrix operations must be verified dimensionally
- Bounds checking on all array accesses

---

**All generated code must compile cleanly and pass all checks on the first attempt.**
