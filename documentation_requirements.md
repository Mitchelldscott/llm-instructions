# Documentation Requirements

## DOCUMENTATION PHILOSOPHY

All documentation is a **contract** between code and user. It must be:
- **Accurate:** Reflects implementation exactly
- **Complete:** Covers all cases (normal, error, edge)
- **Verifiable:** Checkable by reader and formal tools
- **Traceable:** Links to requirements and design

## RUSTDOC STANDARD (Primary)

### A. MANDATORY SECTIONS (In Order)

Every public item (`pub fn`, `pub struct`, `pub trait`, `pub mod`) requires:

```rust
/// Single-sentence summary of purpose.
///
/// Detailed explanation of what this does, when to use it, and any assumptions.
/// Reference the invariants, pre/post-conditions, and failure modes.
///
/// # Generic Arguments
/// * `T`: Description of generic type parameter
///
/// # Arguments
/// * `arg_name`: Description of purpose and expected constraints
///
/// # Returns
/// * `Ok(value)`: Description of success case and value semantics
/// * `Err(error)`: Each error variant with handling guidance
///
/// # Errors
/// * `ErrorVariant::Case1`: When this occurs and how to handle it
/// * `ErrorVariant::Case2`: When this occurs and how to handle it
///
/// # Safety
/// [For safe functions: leave blank or state "This function is safe."]
/// [For unsafe functions: see safety_critical_enforcement.md]
///
/// # Panics
/// [List all panic conditions or state "Never panics."]
///
/// # Complexity
/// Time: O(n log n) where n = [explain]
/// Space: O(1) or O(n) where n = [explain]
///
/// # Example
/// ```
/// # use crate::*;
/// let result = my_function(42)?;
/// assert_eq!(result, expected_value);
/// # Ok::<(), MyError>(())
/// ```
///
/// # Design Rationale
/// This function uses [algorithm] because [justification]. Alternative [X]
/// was rejected because [reason].
///
/// # Requirements Traceability
/// - REQ-ABC-001: [requirement description]
/// - REQ-ABC-002: [requirement description]
///
/// > **Author:** [Leave blank—filled by user]
```

### B. MODULE-LEVEL DOCUMENTATION

Every module must start with:

```rust
//! Module name and one-line purpose.
//!
//! Detailed description of the module's role, scope, and responsibilities.
//! What problems does it solve? What invariants does it maintain?
//!
//! # Architecture
//! [If complex, describe internal organization]
//!
//! # Safety Invariants
//! [If applicable, list all safety properties this module guarantees]
//!
//! # Example
//! ```
//! # use crate::my_module::*;
//! // Demonstrate typical usage pattern
//! ```
```

### C. CONSTANTS AND TYPES

```rust
/// Represents the maximum velocity of the vehicle.
/// Derived from: Hardware acceleration limit ÷ Control loop frequency
const VEHICLE_MAX_VELOCITY_M_S: f32 = 25.0;

/// A transfer function in standard form: H(z) = N(z) / D(z)
///
/// Generic parameters:
/// * `T`: Numeric type (f32, f64, fixed-point)
/// * `M`: Numerator order (length of `numerator` array)
/// * `N`: Denominator order (length of `denominator` array)
///
/// Invariants:
/// - denominator[0] ≠ 0 (leading coefficient nonzero)
pub struct TransferFunction<T, const M: usize, const N: usize> {
    pub numerator: [T; M],
    pub denominator: [T; N],
}
```

### D. TEST DOCUMENTATION

Every test must explain:
- **What:** What property or behavior is tested?
- **Why:** Why is this test necessary? (coverage, safety, regression)
- **How:** What inputs and expected outputs?

```rust
#[test]
fn test_saturate_clamps_to_upper_bound() {
    // PROPERTY: Values exceeding max are clamped, not wrapped.
    // RATIONALE: Overflow in control law is safety-critical; must be explicit.
    let result = saturate(150.0, 0.0, 100.0);
    assert_eq!(result, 100.0);
}
```

## COMMENT STANDARDS

### For Complex Algorithms:

```rust
// Algorithm: Recursive Least Squares (RLS)
// Reference: "Adaptive Control Systems" (Narendra, K.S., 2005)
// 
// The covariance matrix P(k) is updated via:
//   P(k) = [P(k-1) - P(k-1)φ(k)φ(k)ᵀP(k-1)] / [λ + φ(k)ᵀP(k-1)φ(k)]
//
// Where:
//   λ ∈ (0, 1]: Forgetting factor (0.99 typical for tracking)
//   φ(k): Regression vector [y(k-1), ..., y(k-na); u(k-1), ..., u(k-nb)]
//
// WCET: O(n²) where n = na + nb (order of system)
// STABILITY: Guaranteed if λ > 0 and P(k) > 0 ∀k
```

### For Non-Obvious Code:

```rust
// INSIGHT: We use 2's complement saturation rather than wrapping to detect
// overflow in control law. Wrapping would silently corrupt the control signal.
// Saturation preserves sign and limits magnitude, which is safer.
let saturated = value.saturating_add(delta);
```

### For Safety-Critical Sections:

```rust
// SAFETY: This register write enables interrupts globally.
// PRECONDITION: All interrupt handlers must be registered before this point.
// INVARIANT: sp (stack pointer) must point to valid memory throughout.
// POSTCONDITION: Interrupts are enabled; CPU can be preempted at any instruction.
// 
// VERIFICATION: Kani formal verification proof in kani/enable_interrupts_proof.rs
unsafe { interrupts::enable() }
```

## DOCUMENTATION CHECKLIST

Before publishing new code, verify:

- [ ] Every public function has Rustdoc with all sections
- [ ] Summary sentence is one line, imperative mood
- [ ] Generic arguments documented
- [ ] All parameters documented with constraints
- [ ] Returns section documents all cases (Ok and Err)
- [ ] Errors section lists all possible variants
- [ ] Safety section filled (even if "This function is safe.")
- [ ] Panics section lists all panic conditions or states "Never panics."
- [ ] Example compiles and demonstrates correct usage
- [ ] Complexity analysis included (time and space)
- [ ] Design rationale explains algorithm choice
- [ ] Requirements traceability included
- [ ] Module-level documentation present
- [ ] Constants have units and derivation
- [ ] Unsafe code has SAFETY comments with proof sketch
- [ ] Formal verification references included (if applicable)

---

**All code without complete documentation is considered incomplete and will be rejected.**
