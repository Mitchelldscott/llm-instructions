# Instruction Set Summary

---

## FILE MANIFEST

### 1. **system_prompt.md** (Foundation)
- **Purpose:** Establishes the assistant's primary directive and operational mode
- **Content:**
  - PRIMARY DIRECTIVE: Robotic, efficient, uncompromising
  - INSTRUCTION ARCHITECTURE: How to load and structure all modules
  - RESPONSE STRUCTURE: Template for all responses
  - INSTRUCTION ADHERENCE POLICY: How to handle corrections and conflicts
- **Key Rule:** MOST RESTRICTIVE interpretation wins when conflicts exist

### 2. **scope_and_context.md** (Domain Definition)
- **Purpose:** Defines what the assistant WILL and WILL NOT work on
- **Content:**
  - DOMAIN SPECIALIZATION: Bare-metal, expensive math, middleware, autopilot, verification
  - LANGUAGE SCOPE: Rust primary, C/C++ secondary, others restricted
  - CONTEXT PRESERVATION: Project standards and constraints
- **Key Rule:** Never assume context beyond defined scope

### 3. **operational_rules.md** (Behavioral Constraints)
- **Purpose:** Governs how the assistant interacts with users
- **Content:**
  - EXECUTION PRINCIPLES: Compliance over convenience, literal interpretation
  - INSTRUCTION UPDATE PROTOCOL: Exact steps when user corrects the assistant
  - INTERACTION BOUNDARIES: What's prohibited and required
  - ENGAGEMENT SCOPE: What requests to accept/reject
- **Key Rule:** When instructions conflict, request explicit update before proceeding

### 4. **code_generation_standards.md** (Code Quality)
- **Purpose:** Specifies exact code quality requirements
- **Content:**
  - LANGUAGE REQUIREMENTS: Rust compiler settings (#![deny(...)], etc.)
  - CODE STRUCTURE: File organization, function signatures, no magic numbers
  - ERROR HANDLING: Result<T, E> required, no unwrap()
  - UNSAFE CODE POLICY: Justification, encapsulation, testing
  - NUMERIC TYPES: Fixed/floating-point arithmetic rules
- **Key Rule:** Generated code must compile cleanly on first attempt with zero warnings

### 5. **documentation_requirements.md** (Documentation Standards)
- **Purpose:** Mandates comprehensive, verifiable documentation
- **Content:**
  - DOCUMENTATION PHILOSOPHY: Contract between code and user
  - RUSTDOC STANDARD: Mandatory sections in specific order (Summary, Args, Returns, Errors, Safety, Panics, Complexity, Example, Design Rationale, Requirements Traceability)
  - MODULE-LEVEL DOCUMENTATION: //! style with architecture and safety invariants
  - COMMENT STANDARDS: Complex algorithms, non-obvious code, safety-critical sections
  - EXAMPLE DOCUMENTATION: Checklist with 17 verification points
- **Key Rule:** No documentation = code is incomplete and REJECTED

### 6. **verification_and_testing.md** (Test Strategy)
- **Purpose:** Mandates multi-level verification for all code
- **Content:**
  - UNIT TESTS: 100% coverage for critical code, structure with ARRANGE/ACT/ASSERT
  - INTEGRATION TESTS: Cross-module verification
  - BENCHMARKING: WCET analysis using Criterion
  - FORMAL VERIFICATION: Kani, Prusti, Coq for safety-critical code
  - VERIFICATION CHECKLIST: 11-point pre-submission check
  - CI/CD PIPELINE: Enforced checks (cargo check, fmt, clippy, test, doc, audit)
- **Key Rule:** Code without verification is unsafe

### 7. **safety_enforcement.md** (Unsafe Code Policy)
- **Purpose:** Strictest enforcement of unsafe code practices
- **Content:**
  - JUSTIFICATION FRAMEWORK: Every unsafe block needs preconditions, invariants, postconditions, rationale
  - SCOPE MINIMIZATION: Unsafe blocks must be smallest possible
  - ENCAPSULATION REQUIREMENT: Public API never exposes unsafety
  - VERIFICATION REQUIREMENT: Kani proof OR exhaustive testing mandatory
  - UNSAFE CODE CHECKLIST: 10-point verification
  - FORBIDDEN PATTERNS: List of absolutely prohibited patterns
  - MEMORY SAFETY: Critical invariants (alignment, validity, provenance, exclusivity, lifetime)
- **Key Rule:** Violations are safety-critical issues and UNACCEPTABLE

---

## MODULAR ARCHITECTURE

### How It Works

**Layered Design:**
```
system_prompt.md (Meta-rules: How to follow rules)
    ↓
scope_and_context.md (What to work on)
    ↓
operational_rules.md (How to behave when conflicts occur)
    ↓
code_generation_standards.md (Technical requirements)
    ↓
documentation_requirements.md (Documentation contract)
    ↓
verification_and_testing.md (Verification strategy)
    ↓
safety_enforcement.md (Hardest constraints)
```

**Conflict Resolution:**
- No module can override another; conflicts must go to user for clarification

**Extension Points:**
- New modules can be appended (e.g., `performance_analysis.md`, `build_system.md`)
- Existing modules can be extended with new sections
- Changes trigger instruction update protocol (operational_rules.md | SECTION: INSTRUCTION UPDATE PROTOCOL)

## INTEGRATION GUIDE

### Adding as a Git Subtree

```bash
# Add remote (replace URL with your fork)
git remote add llm-instructions https://github.com/username/llm-instructions.git

# Add subtree to 'llm-instructions' directory
git subtree add --prefix llm-instructions llm-instructions master --squash

# Pull updates later
git subtree pull --prefix llm-instructions llm-instructions master --squash
```
