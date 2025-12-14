# Safety-Critical Enforcement

## UNSAFE CODE POLICY (CRITICAL)

This section applies to all unsafe code in the system. Violations are **UNACCEPTABLE**.

### A. JUSTIFICATION FRAMEWORK

**Every unsafe block must have:**

1. **SAFETY comment** (required)
```rust
// SAFETY: [Detailed invariant explanation]
unsafe {
    // Unsafe code goes here
}
```

2. **Preconditions** (required)
- What must be true before entering the unsafe block?
- How does the caller guarantee this?
- What inputs are invalid?

3. **Invariants** (required)
- What memory/logical properties must hold during execution?
- Why is the unsafe operation sound given these invariants?

4. **Postconditions** (required)
- What state is guaranteed after the block?
- Are there residual effects or side effects?

5. **Unsafe Rationale** (required)
- Why is unsafe necessary here?
- Was a safe alternative considered? Why rejected?
- What is the performance/memory justification?

**Example (CORRECT):**
```rust
/// # Safety
/// [Precondition] The caller must guarantee:
/// - `ptr` points to valid, initialized memory containing an array of at least `len` elements
/// - `ptr` is properly aligned for type `T`
/// - The memory was allocated by the global allocator or compatible allocator
///
/// [Invariant] This function maintains:
/// - The memory referenced by `ptr` is not accessed by other code during execution
/// - No uninitialized memory is read
/// - The lifetime of the returned slice does not outlive the allocation
///
/// [Postcondition] After this function returns:
/// - The returned slice is valid for the lifetime parameter
/// - The underlying memory is unchanged
/// - The slice can be safely dereferenced
///
/// [Unsafe Rationale] A safe alternative (bounds-checked indexing) would add
/// runtime overhead (~5 cycles) critical in the real-time control loop.
/// This unsafe block is used in inner loop, executed 10,000 times per second.
/// The function name ends with _unchecked to indicate the lack of bounds checking.
unsafe fn slice_from_raw_parts_unchecked<'a, T>(ptr: *const T, len: usize) -> &'a [T] {
    core::slice::from_raw_parts(ptr, len)
}
```

**Example (WRONG):**
```rust
// ✗ NO COMMENT - UNACCEPTABLE
unsafe { ptr.read() }

// ✗ INCOMPLETE - Missing preconditions
// SAFETY: Reading from pointer
unsafe { ptr.read() }

// ✗ VAGUE - No specifics
// SAFETY: The pointer is valid.
unsafe { ptr.read() }
```

### B. SCOPE MINIMIZATION

**Rule:** Unsafe block must be the smallest possible.

```rust
// ✗ WRONG: Entire function is unsafe
unsafe fn process_buffer(ptr: *mut u8, len: usize) -> bool {
    for i in 0..len {
        *ptr.add(i) = transform(*ptr.add(i));
    }
    validate_result()  // This is safe! Don't include in unsafe block
}

// ✓ CORRECT: Only the necessary part
fn process_buffer(ptr: *mut u8, len: usize) -> bool {
    for i in 0..len {
        // SAFETY: i < len (from loop invariant)
        unsafe {
            *ptr.add(i) = transform(*ptr.add(i));
        }
    }
    validate_result()  // Outside unsafe block
}
```

### C. ENCAPSULATION REQUIREMENT

**Rule:** Public API must NEVER expose unsafety directly.

```rust
// ✗ WRONG: Public API is unsafe
pub unsafe fn write_register(addr: usize, value: u32) {
    *(addr as *mut u32) = value;
}

// ✓ CORRECT: Safe wrapper provides contract
pub fn write_register(offset: RegisterOffset, value: u32) -> Result<(), RegError> {
    if !offset.is_valid() {
        return Err(RegError::InvalidOffset);
    }
    
    let addr = REGISTER_BASE + offset.as_usize();
    // SAFETY: offset.is_valid() guarantees addr is in valid register region
    unsafe {
        *(addr as *mut u32) = value;
    }
    Ok(())
}
```

### D. VERIFICATION REQUIREMENT

**Every unsafe block must have formal verification or exhaustive testing.**

**Option 1: Kani Proof**
```rust
#[cfg(kani)]
mod proofs {
    #[kani::proof]
    fn verify_dereference_safety() {
        let ptr = kani::any::<*const u32>();
        // Kani checks: no UB, no null dereference, alignment correct
        let value = unsafe { *ptr };  // MUST be sound
    }
}
```

**Option 2: Exhaustive Testing**
```rust
#[test]
fn test_pointer_dereference_all_cases() {
    // Test valid pointer
    let value: u32 = 42;
    let ptr = &value as *const u32;
    assert_eq!(unsafe { *ptr }, 42);
    
    // Test edge cases: zero, max, alignment
    // ... additional test cases ...
}
```

### E. UNSAFE CODE CHECKLIST

Before submitting code with unsafe blocks:

- [ ] SAFETY comment present with all sections
- [ ] Preconditions documented and verifiable by caller
- [ ] Invariants are sound and provable
- [ ] Postconditions documented
- [ ] Unsafe rationale (performance/memory) justified
- [ ] Unsafe block is minimal scope
- [ ] Public API does not expose unsafety
- [ ] Formal proof OR exhaustive testing present
- [ ] All unsafe operations covered by SAFETY comment
- [ ] Code review by senior engineer completed (2+ reviewers recommended)

### F. FORBIDDEN PATTERNS

**These patterns are NEVER acceptable:**

```rust
// ✗ FORBIDDEN: No safety comment
unsafe { some_operation() }

// ✗ FORBIDDEN: Justification by assumption
unsafe { /* It's probably safe */ some_operation() }

// ✗ FORBIDDEN: Unsafe in public function signature
pub unsafe fn public_api() { ... }

// ✗ FORBIDDEN: Cast away const/mut without justification
unsafe { *(ptr as *mut T) }

// ✗ FORBIDDEN: Transmute without soundness proof
unsafe { core::mem::transmute::<T, U>(value) }

// ✗ FORBIDDEN: Pointer arithmetic without bounds check
unsafe { ptr.add(user_provided_offset).read() }
```

### G. REAL-TIME CONSTRAINTS

For real-time systems, unsafe optimization must:

1. **Quantify benefit:**
   ```
   WCET reduction: 5μs → 3μs (40% improvement)
   Control loop frequency: 1kHz (1ms period)
   Slack margin after optimization: 950μs
   Safety margin: 95% of budget remaining ✓
   ```

2. **Prove soundness formally** (Kani required for critical paths)

3. **Document fallback safety mechanism:**
   ```
   If unsafe code fails (e.g., hardware fault):
   System enters safe state via watchdog timer (10ms timeout)
   ```

### H. MEMORY SAFETY

**Critical invariants for any unsafe code accessing memory:**

1. **Alignment:** `addr % align_of::<T>() == 0`
2. **Validity:** Pointer references initialized data (no dangling references)
3. **Provenance:** Pointer was created from valid source (allocation, stack, etc.)
4. **Exclusivity:** No other code accesses same memory simultaneously (for writes)
5. **Lifetime:** Referenced data lives long enough (documented in comments)

---

**Any unsafe code without complete SAFETY documentation will be REJECTED.**

**Violations of this policy are safety-critical issues and must be escalated immediately.**
