---
title: ZKsync OS - Crypto
client: Matter Labs
created_at: '2025-11-19'
updated_at: '2025-12-23'
draft: false
---
<script>
  import Levels from '$lib/components/levels.svelte';
  import LevelText from '$lib/components/colored-level.svelte';
  import StatusText from '$lib/components/colored-status.svelte';
</script>

# Synopsis

In October 2025, Matter Labs engaged Taran Space to conduct an additional security assessment of their new product, ZKsync OS, with a focus on the cryptographic modules. The audit was conducted over a single 4-week phase.

Following the assessment, Matter Labs addressed the most severe security issues, and the implemented fixes were verified in December 2025.

## About ZKsync OS

ZKsync OS, developed by Matter Labs, is a generalized RISC-V–based state transition function used within the ZKsync protocol. Further details are available in our previous report: https://github.com/taran-space/audit-reports/blob/main/2025-08-ZKsync-OS.pdf

This audit focused on ECC and field arithmetic in ZKsync OS: Secp256k1 and Secp256r1 implementations, BN254 and BLS12-381 fields and pairings, big-integer delegation on RISC-V, normalization and magnitude invariants, and point/field operations used in proof verification.

## Audit Team

The audit of ZKsync OS was conducted by a team of two full-time auditors:
- Lead Auditor: [Kirill Taran](https://www.linkedin.com/in/kirill-taran-98103a7b/)
- Auditor: [Ilia Imeteo](https://www.self.so/ilia-imeteo)

Fuzz-testing assistance was provided by:
- Auditor: [Tarek Elsayed](https://www.linkedin.com/in/tareknasser360/)

# Scope and Timeline

## Repository

The codebase is located in the repository by the following link:

- https://github.com/matter-labs/zksync-os

## Modules in Scope

The scope is limited to specific modules:
- `callable_oracles`
- `crypto`

## Timeline

Duration: 4 weeks

Dates: October 20, 2025 - November 19, 2025

Commit hash:
- `f32e78b857f699cdfa30c3e46a35ccb177f67a89`

# Findings Classification

## Severity Rating

The severity rating is assigned to each issue after evaluating two factors: "likelihood" and "impact".

### Likelihood

Likelihood reflects the ease of exploitation, indicating how readily attackers could exploit a finding. This assessment considers the required level of access, the availability of exploitation information, the necessity of social engineering, the presence of race conditions or brute-forcing opportunities, and any additional barriers to exploitation.

This factor is categorized into one of three levels:

- **<span class="border rounded px-1">High</span>** Exploitation is almost certain to occur, straightforward to execute, or challenging but highly incentivized. The issue can be exploited by virtually anyone under almost any condition. Attackers can exploit the finding unilaterally without needing special permissions or encountering significant obstacles.
- **<span class="border rounded px-1">Medium</span>** Exploitation of the issue requires non-trivial preconditions. Once these preconditions are met, the issue is either easy to exploit or well incentivized. Attackers may need to leverage a third party, gain access to non-public information, exploit a race condition, or overcome moderate challenges to exploit the finding.
- **<span class="border rounded px-1">Low</span>** Exploitation requires stringent preconditions, possibly requiring the alignment of several unlikely factors or a preceding attack. It might involve implausible social engineering, exploiting a challenging race condition, or guessing difficult-to-predict data, making it unlikely. There is little or no incentive to exploit the issue. Centralization issues, exploitable only by operators, on-chain governance, or development teams, fall into this category as well.

### Impact

Impact considers the consequences of successful exploitation on the target system. It accounts for potential loss of confidentiality, integrity, and availability, as well as potential reputational damage.

This factor is classified into one of three levels:

- **<span class="border rounded px-1">High</span>** Entails the loss of a significant portion of assets within the protocol, or causes substantial harm to the majority of users. Attackers can access or modify all data in a system, execute arbitrary code, escalate their privileges to superuser level, or disrupt the system's ability to serve its users.
- **<span class="border rounded px-1">Medium</span>** Leads to global losses of assets in moderate amounts or affects only a subset of users, which is still deemed unacceptable. Attackers can access or modify some unauthorized data, affect availability or restrict access to the system, or obtain significant internal technical insights.
- **<span class="border rounded px-1">Low</span>** Losses are inconvenient but manageable. This applies to scenarios like easily remediable griefing attacks or gas inefficiencies. Attackers may access limited amounts of unauthorized information or marginally degrade system performance. This category also includes code quality concerns and non-exploitable issues that may still negatively impact public perception of security.

### Final Severity

Upon assessing the issue's likelihood and impact, its severity is determined using the table below:

|                           | <Levels level="High" />     | <Levels level="Medium" />   | <Levels level="Low" />      |
| ------------------------- | --------------------------- | --------------------------- | --------------------------- |
| <Levels level="High" />   | **<LevelText level={4} />** | **<LevelText level={3} />** | **<LevelText level={2} />** |
| <Levels level="Medium" /> | **<LevelText level={3} />** | **<LevelText level={2} />** | **<LevelText level={1} />** |
| <Levels level="Low" />    | **<LevelText level={2} />** | **<LevelText level={1} />** | **<LevelText level={0} />** |

In exceptional cases, the total severity may be elevated to highlight the significance of the issue. Whenever this occurs, the rationale for applying an increased severity level is detailed in the report.

## Issue Statuses

The status of an issue can be one of the following:

- <StatusText status="Notified" /> indicates that the client has been informed of the issue but has either not addressed it yet or has acknowledged the behavior without planning to remediate it soon. This inaction may stem from the client not viewing the behavior as problematic, lacking a feasible solution, or intending to address it at a later date.
- <StatusText status="Addressed" /> is used when the client has made an effort to address the issue with partial success. In the case of complex issues, the provided fix may only resolve one of several points described.
- <StatusText status="Fixed" /> means that the client has implemented a solution that completely resolves the described issue.

# Incorrect identity predicates for affine and Jacobian points

In the `secp256r1` module, incorrect predicates are used to determine whether a point represents the identity (the point at infinity):

```rust filepath line=282
src/secp256r1/points/jacobian.rs
pub(crate) const fn is_infinity_const(&self) -> bool {
    self.z.is_zero() || (self.x.is_zero() || self.y.is_zero())
}
```

```rust filepath line=21
src/secp256r1/points/affine.rs
pub(crate) fn is_infinity(&self) -> bool {
    self.infinity || (self.x.is_zero() || self.y.is_zero())
}
```

In Jacobian coordinates, a point represents the identity if and only if `z` is zero. In affine coordinates, the identity is typically represented only using an explicit flag, not by coordinate values.

While interpreting the `(0, 0)` as the identity could be intended as a simplification of the implementation, the aforementioned functions also consider points `(0, y)` and `(x, 0)` as the identity. This is logically incorrect: valid curve points may legitimately have `x` or `y` equal to zero, and must not be treated as the identity.

This flaw breaks fundamental elliptic-curve invariants and has critical impacts:
- Valid signatures may be rejected
- Invalid signatures may be accepted

This behavior diverges from standard `secp256r1` implementations and can lead to inconsistent public keys, hashes, and verification results. Arithmetic operations may return incorrect results, spuriously produce the identity, or terminate early: addition, doubling and, overall, scalar multiplication.

[page-break]

The following functions rely directly or indirectly on the incorrect identity predicates:
- Function `odd_multiples`, defined in `context.rs:29-41`
- Function `double`, defined in `jacobian.rs:297-298`
- Function `add`, defined in `jacobian.rs:320-324`
- Function `reject_identity()`, defined in `affine.rs:54-59`
- Function `double_assign`, defined in `jacobian.rs:36-81`
- Function `add_ge_assign`, defined in `jacobian.rs:159-220`

## Recommendations

1. As an immediate mitigation, replace the second `||` operator with `&&`, so that points with exactly one zero coordinate are no longer misclassified as the identity. This reduces incorrect behavior but does not fully restore the correct mathematical definition of the point at infinity.
2. A mathematically correct approach is to define the identity exclusively via the coordinate system invariant: `z` is zero for Jacobian points, or the explicit `infinity` flag for affine points, and never determine identity from `(x, y)` coordinate values.
3. Centralize identity checks into a single helper function shared by affine and Jacobian representations to prevent future inconsistencies.

[page-break]

# DoS and corrupted state in BN254 and BLS12-381 pairings checks

## The issue in the `bn254` module

The function `bn254_pairing_check_inner` parses the buffer `src` into G1 and G2 affine points and constructs `(G1Affine, G2Affine)` pairs. The buffer `src` is supplied by the caller function `execute` and may contain arbitrary data defined during a smart contract execution. The `(G1Affine, G2Affine)` pairs are then passed into the function `Bn254::multi_pairing` on line `154`:
```rust filepath line=66
basic_system/src/system_functions/bn254_pairing_check.rs
fn bn254_pairing_check_inner<A: Allocator>(
    num_pairs: usize,
    src: &[u8],
    allocator: A,
) -> Result<bool, ()> {
```

```rust line=152
let g1_iter = pairs.iter().map(|(g1, _)| g1);
let g2_iter = pairs.iter().map(|(_, g2)| g2);
let result = Bn254::multi_pairing(g1_iter, g2_iter);
```

The function `Bn254::multi_pairing` includes a call to the `Bn254::multi_miller_loop` function, which processes each _(G1,G2)_ pair by converting the _G1_ input into `G1Prepared` and the `G2Affine` input into `G2Prepared`. After the conversions, both results are checked for representing the point-at-infinity, using the `is_zero` function. If either point is the point-at-infinity, the pair is skipped and the loop continues with the next pair:
```rust filepath line=109
crypto/src/bn254/curves/pairing_impl.rs
let q: Self::G2Prepared = q.into();
if q.is_zero() {
    continue;
}
```

[page-break]

The conversion `into()` for `G2Affine` type is implemented via `G2PreparedNoAlloc::from`, which handles the point-at-infinity case as follows:
```rust context line=217 highlight=[4]
impl From<G2Affine> for G2PreparedNoAlloc
if q.infinity {
    #[allow(invalid_value)]
    G2PreparedNoAlloc {
        ell_coeffs: unsafe { core::mem::MaybeUninit::uninit().assume_init() }, // unused/filtered above
        infinity: true,
    }
} else {
```

Both `q.is_zero()` and `q.infinity` implement the check for point-at-infinity:
```rust line=357
impl G2PreparedNoAlloc {
    pub fn is_zero(&self) -> bool {
        self.infinity
    }
}
```

This means that both lines `110` and `217` check for the same condition, so there is a scenario when the block on lines `218-222` is executed. The line `220` contains an `unsafe` expression, accompanied by the comment `// unused/filtered above`. Apparently, the idea here is that the `unsafe` section would never actually execute if the field `ell_coeffs` is not accessed. However, struct fields are eagerly initialized in Rust, so all fields of a struct literal are evaluated before the value exists, and the `unsafe` section is executed every time the control flow reaches lines `218-222`.

In other words, the `unsafe` section on line `220` is always entered before the filtering on line `110` occurs.

This `unsafe` section immediately triggers an Undefined Behavior, even if the field `ell_coeffs` is never accessed later, because the expression `MaybeUninit::uninit().assume_init()` creates an uninitialized array and treats it as initialized. This is confirmed to be an invalid use by the documentation of the `assume_init` function:
```rust
/// # Safety
///
/// It is up to the caller to guarantee that the `MaybeUninit<T>` really is in an initialized
/// state. Calling this when the content is not yet fully initialized causes immediate undefined
/// behavior. The [type-level documentation][inv] contains more information about
/// this initialization invariant.
```

[page-break]

## Similar issue in the `bls12_381` module

The same issue is present in the BLS12-381 implementation, due to code duplication.

```rust filepath context line=112
crypto/src/bls12_381/curves/pairing_impl.rs
fn multi_miller_loop
let q: Self::G2Prepared = q.into();
if q.is_zero() {
    continue;
}
```

```rust context line=217
impl From<G2Affine> for G2PreparedNoAlloc {
if q.infinity {
    #[allow(invalid_value)]
    G2PreparedNoAlloc {
        ell_coeffs: unsafe { core::mem::MaybeUninit::uninit().assume_init() }, // unused/filtered above
        infinity: true,
    }
} else {
```

## Severity

The impact of Undefined Behavior in a cryptographic setting is considered High, because it invalidates all guarantees about correctness, soundness, reproducibility, and security. Undefined Behavior can affect any code surrounding the line triggering it, since it permits the compiler to apply otherwise invalid optimizations.

Practically, UB may allow the attacker to:
- bypass cryptographic verification completely
- corrupt other transactions in any blocks within the same execution context

Since input points are derived from attacker-controlled smart contract calldata, triggering the UB is trivial. Exploitability also depends on compiler behavior but we classify the likelihood of this issue as High.

Even if the corrupted state is eventually rejected during verification, the issue still equips attackers with an extremely cheap DoS vector.

[page-break]

## Recommendations

Apply these recommendations to both BN254 and BLS12-381 implementations:

1. Reject invalid points, including the point-at-infinity, early — before invoking any conversions on them.
2. Construct a concrete, fully-initialized dummy `EllCoeff` (zeros) in the array initializer.

# Insufficient magnitude update in scalar multiplication

In the `secp256k1` module, several incorrect magnitude updates have been identified, specifically in the implementation of scalar multiplication.

The function `mul_int_in_place`, which takes a field element `self` and a scalar `rhs`, increases the current magnitude by `rhs`. This is incorrect: every limb of the field element is multiplied by `rhs`, so if a limb is already inflated by a factor of `k`, the resulting inflation factor is `k * rhs`, not `k + rhs`.

```rust filepath context line=79 highlight=[1]
src/secp256k1/field/field_impl.rs
pub(super) fn mul_int_in_place(&mut self, rhs: u32)
self.magnitude += rhs;
debug_assert!(self.magnitude <= Self::max_magnitude());

self.value.mul_int_in_place(rhs);
self.normalized = false;
```

The same incorrect update is present in `mul_int`:

```rust context line=159 highlight=[1]
pub(super) const fn mul_int(&self, rhs: u32)
let new_magnitude = self.magnitude + rhs;
debug_assert!(new_magnitude <= Self::max_magnitude());

let value = self.value.mul_int(rhs);
Self::new(value, new_magnitude)
```

Both `mul_int_in_place` and `mul_int` underestimate the resulting magnitude of the field element. At a minimum, this invalidates assumptions relied upon by routines such as `negate_in_place`.

In this codebase, the magnitude invariants are ensured only by debug assertions, and no runtime validations exist in release mode. As a consequence, silent field-arithmetic failures may occur. In the context of signature verification, this may lead to:
- invalid signatures being accepted,
- valid signatures being rejected.

If the same code were used in a decentralized environment (e.g. an L1 network), an additional impact would be the risk of consensus faults.

## Recommendation

Correct the magnitude update so that the resulting magnitude is computed as the product of the previous magnitude and the scalar, rather than their sum.

# Failed assertion in the Fused Multiply-Add operation

During fuzzing test runs, an assertion failed:
```
thread '<unnamed>' panicked at /zksync-os/basic_system/src/system_functions/modexp/delegation/bigint.rs:686:25:
assertion `left == right` failed
  left: 1
 right: 0
```

See the _Appendix_ section for details on how fuzzing was performed.

The assertion is located in the function `fma`, which implements a Fused Multiply-Add (FMA) operation for the `BigintRepr` structure.

[page-break]

The function `bigint_op_delegation_raw`, defined in the in-scope crate `crypto`, returns a non-zero value to signal a limb overflow. The assertion assumes that no overflow can occur at this stage of the algorithm:

```rust filepath context line=681 highlight=[1,6]
basic_system/src/system_functions/modexp/delegation/bigint.rs
unsafe fn fma
let of = bigint_op_delegation_raw(
    dst_scratch_capacity[dst_digit].as_mut_ptr().cast(),
    carry_scratch.cast(),
    BigIntOps::Add,
);
assert_eq!(of, 0);
```

However, this assumption is incorrect. During accumulation, under certain operand combinations, a carry produced by earlier partial products can overflow the destination digit, requiring propagation into higher digits.

As a consequence, this creates a limited potential for panic-based DoS during modular exponentiation. Because this is a non-debug assertion, the failure is a deterministic panic in release builds rather than silent truncation. The function `fma` is callable from functions `mul_step`, `square_step`, and `reduce_initially`. These functions are invoked by the `modpow` function, which is reachable from the top-level `execute` function when processing the `modexp` system function. This system function can be used by both honest and malicious transactions. If the assertion is triggered by an honest transaction, the failure will appear as a DoS to the user.

At the same time, the likelihood of exploitation is low. The failing input is not trivially constructible.

# Unchecked field-arithmetic invariants

Regular CI test runs execute the test suite only in Release mode, where debug assertions and integer overflow checks are disabled. The codebase provides no runtime validation of field-arithmetic invariants in Release builds. Correctness of the field operations depends entirely on debug-mode assertions and overflow checks, which are never exercised by CI. If these invariants are violated, the failures remain silent in Release mode, allowing field-arithmetic errors to persist undetected.

Consider the file `.github/workflows/ci.yml`:
- The only `cargo test` commands without `--release` flag used in this file are located on lines `71` and `155`, where the scope of their action is limited to modules `transactions` and `binary_checker` correspondingly.
- All the other invocations of `cargo test` run in Release profiles: lines `43`, `45`, `178`, `201` and `202`.

As a consequence, overflows and debug assertions during test suite execution are not reported.

When running the tests specifically on the `crypto` crate, with overflow checks and debug assertions enabled, multiple sporadious issues have been discovered. These issues do not reproduce consistently due to randomized nature of property testing involved.

## Proof of concept

After running `./dump_bin.sh` as usually, enable the extra checks using the `RUSTFLAGS` and build the project including the test code of the `crypto` crate:
```sh
export RUSTFLAGS="-C overflow-checks=on -C debug-assertions=on"
cargo build --release
cargo test --release -p crypto
```

Then run the property tests multiple times, using a loop:
```sh
$ for i in `seq 1 1000`
do
    cargo test --release -p crypto 2>&1 | grep FAILED | tee -a errors.txt
done
```

During manual exploration, it is practical to limit scope of each pass to only one submodule or one test case, for easier interpretation of results. Additionally, parallelization should be enabled in order to save total computation time:
```sh
seq 1 1000 \
| xargs -n1 -P48 sh -c 'cargo test --release -p crypto secp256k1::field::field_10x26 2>&1 | grep FAILED || true' _ \
| tee -a errors.txt
```

[page-break]

## Examples of unreported issues

Although it is recommended to ensure overflow checks and debug assertions in all test cases, actual violations have been discovered only in the `secp256k1::field` module.

### Violated assertions

These test cases have crashed on a `debug_assert!` statement:
- `secp256k1::field::field_10x26`: `test_add`, `test_invert_var`, `test_mul`, `to_bytes_round`, `to_signed30_round`
- `secp256k1::field::field_5x52`: `test_add`, `test_invert`, `test_mul`, `to_bytes_round`, `to_signed62_round`, `to_storage_round`
- `secp256k1::field::tests`: `storage_round_trip`, `test_add`, `test_add_const`, `test_invert`, `test_invert_const`, `test_mul`, `test_mul_const`, `test_square`, `test_square_const`, `to_bytes_round_trip`
- `secp256k1::points::jacobian`: `test_add_zinv`

For instance, one of violated debug assertions is located in the `normalize_in_place` function:
```rust filepath context line=589 highlight=[5]
src/secp256k1/field/field_10x26.rs
pub(super) const fn normalize_in_place(&mut self) {
self.0[8] &= 0x3FFFFFF;
m &= self.0[8];

/* ... except for a possible carry at bit 22 of self.0[9] (i.e. bit 256 of the field element) */
debug_assert!(self.0[9] >> 23 == 0);

// At most a single final reduction is needed; check if the value is >= the field characteristic
x = (self.0[9] >> 22)
    | ((self.0[9] == 0x03FFFFF)
        & (m == 0x3FFFFFF)
        & ((self.0[1] + 0x40 + ((self.0[0] + 0x3D1) >> 26)) > 0x3FFFFFF))
        as u32;
```

### Missed overflows

These test cases have crashed with overflow error:
- `secp256k1::field::field_10x26`: `test_add`, `test_invert_var`, `test_mul`, `to_bytes_round`, `to_signed30_round`
- `secp256k1::field::field_5x52`: `test_add`
- `secp256k1::field::tests`: `test_add_const`
- `secp256k1::points::affine`: `jacobian_round_trip`
- `secp256k1::points::jacobian`: `test_add_zinv`

For instance, the overflows error in the `field_10x26::test_mul` test case is thrown from the `normalize_in_place` function:
```rust filepath context line=559 highlight=[7]
src/secp256k1/field/field_10x26.rs
pub(super) const fn normalize_in_place(&mut self) {
// Reduce self.0[9] at the start so there will be at most a single carry from the first pass
let mut x = self.0[9] >> 22;
self.0[9] &= 0x03FFFFF;

// The first pass ensures the magnitude is 1, ...
self.0[0] += x * 0x3D1;
self.0[1] += x << 6;
self.0[1] += self.0[0] >> 26;
self.0[0] &= 0x3FFFFFF;
```

It is possible that some of the overflows are caused by broken invariants, indicated by the violated assertions.

# Recommendation

Refactor CI workflows and run all tests with all possible checks since running the test suite on local developer's machine is not straightforward and time consuming. In security context, this means that the test suite is never actually executed to the full extent.

Additionally, run the test suite multiple times. Property-based tests do not guarantee full coverage in a single pass, and repeated execution significantly increases the likelihood of catching regressions.

# Magnitude limit 2047 cannot detect arithmetic issues in 5×52 backend

The magnitude limit for the `5x52` storage is defined as `2047`:

```rust filepath line=89
crypto/src/secp256k1/field/field_5x52.rs
pub(super) const fn max_magnitude() -> u32 {
    2047u32
}
```

Although `2047` is the mathematical upper bound for keeping all limbs within 63 bits, it is not compatible with the actual algorithmic invariants in the `mul_in_place` implementation.

The `mul_in_place` implementation contains tight limb-width debug assertions:
```rust filepath context line=162
crypto/src/secp256k1/field/field_5x52.rs
pub(super) const fn mul_in_place
debug_assert!(a0 >> 56 == 0);
debug_assert!(a1 >> 56 == 0);
debug_assert!(a2 >> 56 == 0);
debug_assert!(a3 >> 56 == 0);
debug_assert!(a4 >> 52 == 0);
```

These constraints imply:
- Limbs 0–3 may temporarily grow from 52 bits up to 56 bits.
- Limb 4 may expand only to 52 bits.
- No limb is ever allowed to approach the theoretical 63-bit capacity that would justify a magnitude limit of `2047`.

Thus, a magnitude of `2047` can never occur during legal operation, and using `2047` as `max_magnitude()` renders magnitude-based invariants validation ineffective. Internal arithmetic faults may pass unnoticed.

[page-break]

### Estimation based on `mul_in_place` invariants

A normalized limb less than `2^52` corresponds to the magnitude of 1. The largest permitted limb before entering `mul_in_place` is less than `2^56`. Thus the true maximum magnitude allowed by the `mul_in_place` function is:

```
(2^56) / (2^52) = 2^4 = 16
```

Any magnitude above `16` cannot be produced by correct arithmetic and cannot be safely consumed by `mul_in_place`.

## Recommendation

Set the magnitude limit to `16`.

# Incorrect validation of wNAF table digits

In `secp256k1`, the function `table_verify` is used in debug assertions to validate wNAF digits:

```rust filepath line=328 highlight=[2]
crypto/src/secp256k1/recover.rs
fn table_get_ge(pre: &[Affine], n: i32, w: usize) -> Affine {
    debug_assert!(table_verify(n, w));

    if n > 0 {
        pre[(n - 1) as usize / 2]
    } else {
        let mut r = pre[(-n - 1) as usize / 2];
        r.y.negate_in_place(1);
        r
    }
}
```

[page-break]

However, it is implemented incorrectly:

```rust line=373 highlight=[2,4]
fn table_verify(n: i32, w: usize) -> bool {
    let n = n as usize;

    ((n & 1) == 1) || (n >= !(1 << ((w - 1) - 1))) || (n <= (1 << ((w - 1) - 1)))
}
```

This definition contains several issues:

## 1. Incorrect cast to the `usize` type

The cast from `i32` to the `usize` type appears to serve as the `abs(n)` expression. However, casting a negative value reinterprets bits as large unsigned integer (two’s complement), not an absolute value of `n`.

The intended range check should enforce symmetric bounds `max` and `-max` for the `n`.

## 2. Incorrect logical junctions

The predicates are combined with the `||` operator, but the `&&` operator must be used. All constraints must hold simultaneously.

## 3. Incorrect order of operations in bit shift

The intended limit is `(1 << (w - 1)) - 1`, but the current code computes `1 << (w - 2)`.

## 4. Incorrect arithmetic negation

The unary operator `!` is a bitwise "NOT", not arithmetic negation. It does not represent the negative bound, which should be symmetric to the positive bound.

## 5. Lack of robustness for allowed values of `w`

The current implementation relies on `w` staying within a narrow range. If `w` is ever changed to be outside this range, the shift or subtraction can underflow or become invalid for the chosen integer type:

- Underflow when `w < 2`
- Overflow when `w > 31`

The aforementioned flaws do not introduce incorrect values of wNAF digits, however they make the function `table_verify` significantly more permissive. This makes it not effective in preventing incorrect values of wNAF digits via debug assertions which is its only purpose.

This is especially important because there are no runtime checks in release mode due to performance reasons. If invalid wNAF digits reach table indexing expressions like `pre[(n - 1) as usize / 2]` or `pre[(-n - 1) as usize / 2]`, they can go out of bounds or select wrong points. This would panic in debug mode or produce incorrect scalar multiplication results in release mode.

Additionally, no equivalent validations of wNAF table digits have been identified in the corresponding secp256r1 implementation.

## Recommendation

Consider translating the [similar function in Bitcoin](https://github.com/bitcoin/bitcoin/blob/master/src/secp256k1/src/ecmult_impl.h#L117-L123) from C to Rust:

```rust
fn table_verify(n: i32, w: usize) -> bool {
    // `w` is constant, but this check makes the function future-proof
    // it would underflow/overflow when `w` is out of range
    debug_assert!((2..=31).contains(&w));

    // n must be odd
    if (n & 1) == 0 {
        return false;
    }

    let max: i32 = (1_i32 << (w - 1)) - 1;
    n >= -max && n <= max
}
```

Additionally, implement similar validation in `secp256r1`.

[page-break]

# Miscellaneous validation issues

## Imprecise range check includes the modulus

In the `8x32` backend, the `from_bytes` function performs an incorrect range check when validating field elements parsed from raw bytes:

```rust filepath context line=58
crypto/src/secp256k1/field_8x32.rs
pub(super) fn from_bytes
let value = Self::from_bytes_unchecked(bytes);

if u256::leq(&value.0, &Self::MODULUS.0) {
    Some(value)
} else {
    None
}
```

This condition `u256::leq` accepts values equal to _P_, whereas valid field elements must satisfy _0 ≤ x < P_. As a result, the modulus _P_ itself is incorrectly accepted as a valid field element.

## Missing defensive magnitude check in `normalize_in_place`

The `normalize_in_place` function assumes that magnitude of the internal field element `self.magnitude` does not exceed `31`, but this invariant is not enforced locally:

```rust filepath context line=559
crypto/src/secp256k1/field/field_10x26.rs
pub(super) const fn normalize_in_place
// Reduce self.0[9] at the start so there will be at most a single carry from the first pass
let mut x = self.0[9] >> 22;
self.0[9] &= 0x03FFFFF;
```

If `self.magnitude > 31`, intermediate arithmetic could overflow, leading to incorrect field normalization. A debug assertion should ensure this assumption.

In the current version of the codebase, this condition is not reachable: the maximum observed magnitude at this point is `10`. However, future changes (e.g., repeated add() or negate() operations) could introduce code paths where the magnitude exceeds `31`.

## Missing defensive overflow checks

```rust filepath context line=316 highlight=[8]
crypto/src/secp256k1/scalars/scalar64.rs
fn muladd(a: u64, b: u64, c0: u64, c1: u64, c2: u64)
let t = (a as u128) * (b as u128);
let th = (t >> 64) as u64; // at most 0xFFFFFFFFFFFFFFFE
let tl = t as u64;

let (new_c0, carry0) = c0.overflowing_add(tl);
let new_th = th.wrapping_add(carry0 as u64); // at most 0xFFFFFFFFFFFFFFFF
let (new_c1, carry1) = c1.overflowing_add(new_th);
let new_c2 = c2 + (carry1 as u64);

(new_c0, new_c1, new_c2)
```

Similarly in:
- `crypto/src/secp256k1/scalars/scalar64.rs`
	- `muladd_fast` on lines 328–340
	- `sumadd_fast` on lines 342–347
- `crypto/src/secp256k1/scalars/scalar32.rs`
	- Corresponding 32-bit implementations

## Unenforced input bound in `sub_mod_with_carry`

The `sub_mod_with_carry` function relies on the precondition _a < 2·P_, but this bound is not enforced. If the invariant is violated, the result is not guaranteed to lie in _[0, P)_.

```rust filepath line=142 highlight=[1]
crypto/src/bigint_delegation/u256.rs
/// Note: we assume `self < 2*modulus`, otherwise the result might not be in the range
/// # Safety
/// `DelegationModParams` should only provide references to mutable statics.
/// It is the responsibility of the caller to make sure that is the case
pub unsafe fn sub_mod_with_carry<T: DelegatedModParams<4>>(a: &mut U256, carry: bool) {
    let borrow = delegation::sub(a, T::modulus()) != 0;

    if borrow && !carry {
        delegation::add(a, T::modulus());
    }
}
```

Same issue in `crypto/src/bigint_delegation/u512.rs`.

In the current version of the codebase, all callers respect the required bounds.

## Unsafe buffer bounds assumption

Invoking the function `assert_unchecked` is immediate Undefined Behavior if the condition does not actually hold:

```rust filepath context highlight=[10]
crypto/src/blake2s/delegated_extended.rs
fn finalize_impl
unsafe {
    // write zeroes
    let start = self
        .state
        .input_buffer
        .as_mut_ptr()
        .cast::<u8>()
        .add(self.buffer_filled_bytes);
    let end = self.state.input_buffer.as_mut_ptr_range().end.cast::<u8>();
    core::hint::assert_unchecked(start <= end);
    core::ptr::write_bytes(start, 0, end.offset_from_unsigned(start));
    // and run round function
    self.state
        .run_round_function_with_byte_len::<false>(self.buffer_filled_bytes, true);

    core::mem::transmute_copy::<_, [u8; 32]>(self.state.read_state_for_output_ref())
}
```

If `start <= end` is not guaranteed to hold and this can be influenced by user input, then `assert_unchecked` must be removed.

Even if it can be proven, add `debug_assert!(start <= end)` to catch invariant breaks during test suite runs or fuzzing.

## Unchecked internal invariant in pairing Miller loop

Deterministic panic if curve configuration constants and precomputation logic diverge.

In BLS12-381:

```rust filepath context line=120 highlight=[3,5]
crypto/src/bls12_381/curves/pairing_impl.rs
fn multi_miller_loop
for i in BitIteratorBE::without_leading_zeros(Config::X).skip(1) {
    f.square_in_place();
    Self::ell(&mut f, &ell_coeffs.next().unwrap(), &p.0);
    if i {
        Self::ell(&mut f, &ell_coeffs.next().unwrap(), &p.0);
    }
}
```

In BN254:

```rust filepath context line=115 highlight=[8,12]
crypto/src/bn254/curves/pairing_impl.rs
fn multi_miller_loop
let mut ell_coeffs = q.ell_coeffs.iter();

for i in (1..Config::ATE_LOOP_COUNT.len()).rev() {
    if i != Config::ATE_LOOP_COUNT.len() - 1 {
        f.square_in_place();
    }

    Self::ell(&mut f, ell_coeffs.next().unwrap(), &p.0);

    let bit = Config::ATE_LOOP_COUNT[i - 1];
    if bit == 1 || bit == -1 {
        Self::ell(&mut f, &ell_coeffs.next().unwrap(), &p.0);
    }
}
```

Validate `ell_coeffs.len()` against `EXPECTED_ELL_COEFFS` using a debug assertion. This would improve diagnosticability.

# Concurrent usage is not supported

The library is designed using global static variables that must be initialized before calling the functions defined by the library:

```rust filepath context line=82
crypto/src/lib.rs
pub fn init_lib
bn254::fields::init();
bls12_381::fields::init();
secp256k1::init();
bigint_delegation::init();
secp256r1::init();
```

[page-break]

The crates themselves contain multiple global static variables:
```rust filepath line=7
crypto/src/bigint_delegation/u256.rs
static mut COPY_PLACE_0: MaybeUninit<U256> = MaybeUninit::uninit();
static mut COPY_PLACE_1: MaybeUninit<U256> = MaybeUninit::uninit();
static mut COPY_PLACE_2: MaybeUninit<U256> = MaybeUninit::uninit();
static mut COPY_PLACE_3: MaybeUninit<U256> = MaybeUninit::uninit();
static mut ONE: MaybeUninit<U256> = MaybeUninit::uninit();
static mut ZERO: MaybeUninit<U256> = MaybeUninit::uninit();
static mut SCRATCH: MaybeUninit<U256> = MaybeUninit::uninit();
```

Such declarations have been identified in modules `secp256k1`, `secp256r1`, `bigint_delegation`, `bls12_381` and `bn254`.

Beyond the requirement to initialize the library before use, a more significant consequence of using global static variables is that the library cannot be safely used concurrently or in a multi-threaded environment.

Usage within ZKsync OS has been analyzed and is safe since no concurrency is intended. Additionally, some test cases in the test suite are disabled with the comment `"requires single threaded runner"`.

However, this limitation should be kept in mind by any third-party developers building on top of the `crypto` library. Not only multi-threaded execution, but also concurrent usage may silently cause data races and potentially unsafe behavior if accessed concurrently without external synchronization.

Such usages could be:
- Async runtimes running on a single OS thread but interleaving execution
- Interrupt-driven execution

If the library needs to be used concurrently by community developers, it should be forked and modified to include synchronization.

## Recommendation

Document the initialization and concurrency constraints for using the library for potential community developers.

# Variable-time scalar operations should not be used with secret data

Several functions involved in core cryptographic operations are not constant-time. If these functions are ever used with secret inputs, attackers may exploit timing variability to gather statistical information and potentially recover secret data.

This observation is relevant for both `secp256k1` and `secp256r1` modules. See the "Code Duplicates" issue for identified instances of functions `mul_shift_384_vartime`.

## Function `mul_shift_384_vartime` and scalar decomposition

The argument-dependent timing of the `mul_shift_384_vartime` function comes from the conditional branch at the end. The variable `l` is derived from the parameter `b`. The `if` statement decides whether to execute `self.add_in_place(&Self::ONE)`based on that bit. This creates data-dependent control flow: one path does the addition, the other does not.

```rust filepath line=149 highlight=[6]
src/secp256k1/scalars/scalar64.rs
pub(super) fn mul_shift_384_vartime(&mut self, b: &Self)
let words = b.0.as_words();
let l = words[1];
self.0 = U256::from_words([words[2], words[3], 0, 0]);

if (l >> 63) & 1 != 0 {
    self.add_in_place(&Self::ONE);
}
```

The function is used in the `decompose` function, which implements the scalar decomposition operation, which in its turn affects the scalar multiplication operation.

[page-break]

## Function `pow_vartime`, scalar inversion and affine conversion

Scalar inversion, implemented by the `invert_assign` function, uses the function `pow_vartime` which performs variable-time exponentiation, creating potential timing side-channels:

```rust filepath context line=86 highlight=[5]
src/secp256r1/scalar/mod.rs
pub fn pow_vartime(&self, exp: &[u64])
while j > 0 {
    j -= 1;
    res.square_assign();

    if ((exp[i] >> j) & 1) == 1 {
        res.mul_assign(self);
    }
}
```

Timing variability affects the functions relying on scalar inversion:
- `invert_assign` calls `pow_vartime` in `src/secp256r1/scalar/mod.rs:68`
- `verify` calls `invert_assign` in `src/secp256r1/verify.rs:25`

[page-break]

# Normalization check is not strong enough

In module `secp256k1`, the function `normalize_in_place` is defined, conditioning on values of the `normalized` flag and `magnitude` tracker:

```rust filepath context line=129
crypto/src/secp256k1/field/field_impl.rs
pub(super) fn normalize_in_place
if !self.normalized || self.magnitude > 1 {
    self.value.normalize_in_place();
    self.magnitude = 1;
    self.normalized = true;
}
```

However, this condition has a minor logical flaw — it assumes that it is potentially possible that `self.normalized` is true but `self.magnitude` is not `1`. When a value is normalized, its magnitude must always be `1`.

## Recommendation

Replace this code with:

```rust
if self.normalized {
    debug_assert!(self.magnitude == 1);
} else {
    debug_assert!(self.magnitude > 1);

    self.value.normalize_in_place();
    self.magnitude = 1;
    self.normalized = true;
}
```

[page-break]

# Unit tests issues

### Outdated instructions in the `README`

Running the tests as described in the `README` results in several failures, all throwing error `ZKsync OS bin file missing: /.../zksync_os/for_tests.bin"`.

Examples of such test cases:
- `test_0_gas_limit`
- `fibish_sol`

After inspecting the project’s CI workflows, we have discovered that the script `dump_bin.sh` should be executed with `--type for-tests` flag now, in order to include new test cases into the coverage:
```sh
./dump_bin.sh --type for-tests
```

### Some of the unit tests are not executed during CI

The following modules are not executed during regular CI runs:
- `secp256k1::field::field_8x32`
- `secp256k1::scalars::scalar32_delegation`

Additional workflow should be created that runs the test suite in single-threaded mode, filtering only those modules that require this:
```sh
cargo test --release -p crypto secp256k1::field::field_8x32 -- --test-threads=1 --ignored
cargo test --release -p crypto secp256k1::scalars::scalar32_delegation -- --test-threads=1 --ignored
```

[page-break]

### Some of the unit tests require larger stack

The test case `secp256k1::field::field_8x32::tests::test_invert` overflows the stack of standard size (8MB):
```sh
$ cargo test -p crypto secp256k1::field::field_8x32::tests::test_invert -- --test-threads=1 --ignored
...
test secp256k1::field::field_8x32::tests::test_invert ... 
thread '<unknown>' has overflowed its stack
fatal runtime error: stack overflow, aborting
```

It does succeed in case of bigger stack:
```sh
$ RUST_MIN_STACK=33554432 cargo test -p crypto secp256k1::field::field_8x32::tests::test_invert -- --test-threads=1 --ignored
...
running 1 test
test secp256k1::field::field_8x32::tests::test_invert ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 110 filtered out; finished in 0.57s

     Running tests/secp256k1.rs
```

### Insufficient Test Coverage in `callable_oracles` Crate

- The `callable_oracles` crate has
  - no dedicated fuzzing targets
  - minimal Unit Testing:
    - Only 1 unit test in the entire `callable_oracles` crate (`try_compute_hash_to_prime`)
    - No property-based tests
    - No integration tests

### Limitations of Fuzzing Coverage in `crypto` Crate

- The `crypto` crate has only one fuzzing gap:
  - BLS12-381:
    - Only 1 proptest regression
    - No dedicated fuzz targets

# Excessive magnitude increment

In the `secp256k1` module, the function `add_int_in_place` overestimates the magnitude increase of a field element:

```rust filepath context line=114 highlight=[1]
src/secp256k1/field/field_impl.rs
pub(super) fn add_int_in_place(&mut self, rhs: u32)
self.magnitude += rhs;
debug_assert!(self.magnitude <= Self::max_magnitude());

self.value.add_int_in_place(rhs);
self.normalized = false;
```

Here, the field element is incremented by a small scalar `rhs`. However, the correct magnitude increment should be `ceil(rhs / radix)`, where the radix depends on the limb representation: 2^52 for the 5x52 backend, 2^26 for 10x26 backend, etc.

In practice, for the 5x52 representation, the radix is large enough that the increment is always exactly `1`, even for the maximum `u32` input. For the 10x26 representation, the correct increment may range from `1` up to `2^(32 - 26) = 64`, depending on how close `rhs` is to `u32::MAX`.

This issue does not affect correctness or security; it only potentially reduces performance.

## Recommendation

Adjust the magnitude update so that the increment is always `1` in the 5x52 backend, and between `1` and `64` (inclusive) in the 10x26 backend, depending on the value of `rhs`.

[page-break]

# Magic numbers decrease maintainability

Throughout the codebase, hard-coded number literals without context or a description are used. Using such “magic numbers” goes against best practices as they reduce code readability and maintenance as developers are unable to easily understand their use and may make inconsistent changes across the codebase.

Instances of magic numbers have been discovered in the following locations.

## File `crypto/src/secp256k1/field/field_impl.rs`

Hard-coded literal `8` is used as the limit for magnitudes.

```rust line=69 highlight=[2,3]
pub(super) fn mul_in_place(&mut self, rhs: &Self) {
    debug_assert!(self.magnitude <= 8);
    debug_assert!(rhs.magnitude <= 8);

    self.value.mul_in_place(&rhs.value);
    self.magnitude = 1;
    self.normalized = false;
}
```

```rust line=86 highlight=[2]
pub(super) fn square_in_place(&mut self) {
    debug_assert!(self.magnitude <= 8);
```

```rust line=150 highlight=[2,3]
pub(super) const fn mul(&self, rhs: &Self) -> Self {
    debug_assert!(self.magnitude <= 8);
    debug_assert!(rhs.magnitude <= 8);
```

```rust line=166 highlight=[2]
pub(super) const fn square(&self) -> Self {
    debug_assert!(self.magnitude <= 8);
```

[page-break]

## File `crypto/src/secp256k1/points/affine.rs`

Hard-coded literal `32` is used as the limit for magnitudes, but also it is redundant code since `Self::X_MAGNITUDE_MAX` and `Self::Y_MAGNITUDE_MAX` are defined as a lesser number `4`.

```rust filepath line=41 highlight=[5,7]
crypto/src/secp256k1/points/affine.rs
pub(crate) const fn assert_verify(&self) {
    #[cfg(all(debug_assertions, not(feature = "bigint_ops")))]
    {
        debug_assert!(self.x.0.magnitude <= Self::X_MAGNITUDE_MAX);
        debug_assert!(self.x.0.magnitude <= 32);
        debug_assert!(self.y.0.magnitude <= Self::Y_MAGNITUDE_MAX);
        debug_assert!(self.y.0.magnitude <= 32);
    }
}
```

Same code snippet is also implemented in lines 122-130 in the same file.

## File `crypto/src/secp256k1/points/jacobian.rs`

Similarly, same literal `32` is hard-coded and also has no effect because of stronger equations involving `X_MAGNITUDE_MAX`, `Y_MAGNITUDE_MAX` and `Z_MAGNITUDE_MAX`:

```rust filepath line=28  highlight=[5,7,9]
crypto/src/secp256k1/points/jacobian.rs
pub(super) const fn assert_verify(&self) {
    #[cfg(all(debug_assertions, not(feature = "bigint_ops")))]
    {
        debug_assert!(self.x.0.magnitude <= Self::X_MAGNITUDE_MAX);
        debug_assert!(self.x.0.magnitude <= 32);
        debug_assert!(self.y.0.magnitude <= Self::Y_MAGNITUDE_MAX);
        debug_assert!(self.y.0.magnitude <= 32);
        debug_assert!(self.z.0.magnitude <= Self::Z_MAGNITUDE_MAX);
        debug_assert!(self.z.0.magnitude <= 32);
    }
}
```

Same code snippet is also implemented in lines 195-205 in the same file.

## Impact

Using magic numbers leads to maintainability issues and confusion, especially when they need to be updated or dynamically configured.

## Recommendation

Declare named constants or implement configuration values instead of directly embedding literals to improve clarity and reduce potential errors in future development.

# Code duplicates

In the codebase, several code duplicates have been discovered:
- The function `bits_var` has several exact copies:
  - `crypto/src/secp256r1/scalar/mod.rs:132-144`
  - `crypto/src/secp256k1/scalars/scalar32_delegation.rs:169-184`
  - `crypto/src/secp256k1/scalars/scalar64.rs:94-107`
  - Additionally, there is one slightly modified copy:
  - `crypto/src/secp256k1/scalars/scalar32.rs:92-104`
- The function `mul_shift_384_vartime` is duplicated in:
  - `crypto/src/secp256k1/scalars/scalar64.rs:145-156`
  - `crypto/src/secp256k1/scalars/scalar32_delegation.rs:228-239`
  - `crypto/src/secp256k1/scalars/scalar32.rs:126-138`
- The affine equality `eq` is also duplicated in:
  - `src/secp256r1/points/affine.rs:82`
  - `src/secp256k1/points/affine.rs:81`
  - `src/secp256k1/points/affine.rs:249`

Code duplication increases security risks, as it hinders maintainability and also reviewability. Additionally, all future issue fixes must be manually applied and reviewed in all areas of duplication.

## Recommendation

We recommend extracting common code into auxiliary functions in order to improve maintainability and reviewability of the codebase.

[page-break]

# Redundant `else` branch

In the function `mul_assign`, the "schoolbook" algorithm is implemented. Right after iterating `j` up to `4`, the `if` splits execution depending on the value of `i + j`: 

```rust filepath context line=101
crypto/src/secp256r1/scalar/scalar64.rs
pub(super) fn mul_assign
while j < 4 {
    let k = i + j;

    ...
```

```rust line=114
    j += 1;
}

if i + j >= 4 {
    hi[i + j - 4] = carry;
} else {
    lo[i + j] = carry;
}
```

If `i + j < 4`, the `else` branch would execute. However, this condition is never satisfied because after the `while` loop, `j` is guaranteed to equal `4`, and `i` is always greater than or equal to zero.

# Appendix

# Appendix

## Fuzzing parameters

The fuzzing was executed with the following parameters and targets:

- Duration
  - **48 hours**
- Machine:
  - CPU: Virtual AMD EPYC-Milan
    - **2 GHz, 48 Threads**
    - 32 MiB L3 Cache
  - RAM: 184 GiB

### Fuzzing targets

|                         |                         |
|:------------------------|:------------------------|
| `blake2s`               | `bn254_ecadd`           |
| `bn254_ecmul`           | `bn254_pairing_check`   |
| `ecrecover`             | `p256_verify`           |
| `keccak256`             | `sha256`                |
| `ripemd160`             | `modexp`                |
| `modexp_delegation`     | `precompiles_ecadd`     |
| `precompiles_ecmul`     | `precompiles_ecpairing` |
| `precompiles_ecrecover` | `precompiles_p256`      |
| `precompiles_sha256`    | `precompiles_ripemd160` |
| `precompiles_modexp`    | `precompiles_modexplen` |

[page-break]

## Fuzzing coverage

Generated using the command `./fuzz.sh coverage`.

### Crate `callable_oracles`

| File / Module | Lines Covered | Line Coverage | Functions Covered | Function Coverage |
|-----|----:|----:|----:|----:|
| <span style="color:orange">**callable_oracles**</span> | 114 / 650 | <span style="color:orange">17.5%</span> | 13 / 78 | <span style="color:orange">16.7%</span> |
| <span style="color:#6BD36B">src/arithmetic/mod.rs</span> | 53 / 54 | <span style="color:#6BD36B">98.1%</span> | 7 / 17 | <span style="color:#4DA3FF">41.2%</span> |
| <span style="color:red">src/hash_to_prime/common.rs</span> | 0 / 247 | <span style="color:red">0.0%</span> | 0 / 34 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/hash_to_prime/compute.rs</span> | 0 / 75 | <span style="color:red">0.0%</span> | 0 / 5 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/hash_to_prime/evaluate.rs</span> | 0 / 28 | <span style="color:red">0.0%</span> | 0 / 2 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/hash_to_prime/verify.rs</span> | 0 / 119 | <span style="color:red">0.0%</span> | 0 / 5 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/hash_to_prime/lib.rs</span> | 0 / 11 | <span style="color:red">0.0%</span> | 0 / 2 | <span style="color:red">0.0%</span> |
| <span style="color:#4DA3FF">src/utils/evaluate.rs</span> | 42 / 96 | <span style="color:#4DA3FF">43.8%</span> | 2 / 9 | <span style="color:orange">22.2%</span> |
| <span style="color:#6BD36B">src/utils/usize_slice_iterator.rs</span> | 19 / 20 | <span style="color:#6BD36B">95.0%</span> | 4 / 4 | <span style="color:#6BD36B">100.0%</span> |
| <span style="color:orange"><b>Total</b></span> | 114 / 650 | <span style="color:orange">17.5%</span> | 13 / 78 | <span style="color:orange">16.7%</span> |

### Crate `crypto`

| File / Module | Lines Covered | Line Coverage | Functions Covered | Function Coverage |
|-----|----:|----:|----:|----:|
| <span style="color:red">src/ark_ff_delegation</span> | 0 / 1080 | <span style="color:red">0.0%</span> | 0 / 210 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/biginteger/mod.rs</span> | 0 / 387 | <span style="color:red">0.0%</span> | 0 / 74 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/biginteger/const_helpers.rs</span> | 0 / 117 | <span style="color:red">0.0%</span> | 0 / 19 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/fp/mod.rs</span> | 0 / 414 | <span style="color:red">0.0%</span> | 0 / 91 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/fp/montgomery_backend.rs</span> | 0 / 162 | <span style="color:red">0.0%</span> | 0 / 26 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bigint_delegation/delegation.rs</span> | 0 / 82 | <span style="color:red">0.0%</span> | 0 / 16 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bigint_delegation/mod.rs</span> | 0 / 4 | <span style="color:red">0.0%</span> | 0 / 1 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bigint_delegation/u256.rs</span> | 0 / 209 | <span style="color:red">0.0%</span> | 0 / 29 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bigint_delegation/u512.rs</span> | 0 / 189 | <span style="color:red">0.0%</span> | 0 / 14 | <span style="color:red">0.0%</span> |
| <span style="color:#6BD36B">src/blake2s/naive.rs</span> | 20 / 30 | <span style="color:#6BD36B">66.7%</span> | 19 / 29 | <span style="color:#4DA3FF">65.5%</span> |
| <span style="color:red">**crypto/bls12_381**</span> | 0 / 591 | <span style="color:red">0.0%</span> | 0 / 65 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bls12_381/curves/g1.rs</span> | 0 / 92 | <span style="color:red">0.0%</span> | 0 / 14 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bls12_381/curves/g2.rs</span> | 0 / 105 | <span style="color:red">0.0%</span> | 0 / 11 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bls12_381/curves/pairing_impl.rs</span> | 0 / 209 | <span style="color:red">0.0%</span> | 0 / 21 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bls12_381/curves/util.rs</span> | 0 / 166 | <span style="color:red">0.0%</span> | 0 / 14 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bls12_381/fields/fq2.rs</span> | 0 / 13 | <span style="color:red">0.0%</span> | 0 / 4 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bls12_381/fields/fq6.rs</span> | 0 / 6 | <span style="color:red">0.0%</span> | 0 / 1 | <span style="color:red">0.0%</span> |
| <span style="color:red">**crypto/bn254**</span> | 0 / 306 | <span style="color:red">0.0%</span> | 0 / 36 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bn254/curves/g1.rs</span> | 0 / 31 | <span style="color:red">0.0%</span> | 0 / 7 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bn254/curves/g2.rs</span> | 0 / 26 | <span style="color:red">0.0%</span> | 0 / 5 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bn254/curves/pairing_impl.rs</span> | 0 / 235 | <span style="color:red">0.0%</span> | 0 / 22 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bn254/fields/fq2.rs</span> | 0 / 3 | <span style="color:red">0.0%</span> | 0 / 1 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bn254/fields/fq6.rs</span> | 0 / 11 | <span style="color:red">0.0%</span> | 0 / 1 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bn254/glv_decomposition.rs</span> | 0 / 93 | <span style="color:red">0.0%</span> | 0 / 12 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bn254/lib.rs</span> | 0 / 2 | <span style="color:red">0.0%</span> | 0 / 1 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/bn254/raw_delegation_interface.rs</span> | 0 / 18 | <span style="color:red">0.0%</span> | 0 / 2 | <span style="color:red">0.0%</span> |
| <span style="color:#6BD36B">**crypto/secp256k1**</span> | 1663 / 2105 | <span style="color:#6BD36B">79.0%</span> | 136 / 188 | <span style="color:#6BD36B">72.3%</span> |
| <span style="color:red">src/secp256k1/mod.rs</span> | 0 / 34 | <span style="color:red">0.0%</span> | 0 / 2 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256k1/context.rs</span> | 0 / 34 | <span style="color:red">0.0%</span> | 0 / 4 | <span style="color:red">0.0%</span> |
| <span style="color:#6BD36B">src/secp256k1/field/field_5x52.rs</span> | 332 / 384 | <span style="color:#6BD36B">86.5%</span> | 20 / 27 | <span style="color:#6BD36B">74.1%</span> |
| <span style="color:#4DA3FF">src/secp256k1/field/field_impl.rs</span> | 88 / 144 | <span style="color:#4DA3FF">61.1%</span> | 17 / 27 | <span style="color:#4DA3FF">63.0%</span> |
| <span style="color:#6BD36B">src/secp256k1/field/mod.rs</span> | 114 / 169 | <span style="color:#6BD36B">67.5%</span> | 24 / 35 | <span style="color:#6BD36B">68.6%</span> |
| <span style="color:#6BD36B">src/secp256k1/field/mod_inv64.rs</span> | 321 / 336 | <span style="color:#6BD36B">95.5%</span> | 16 / 17 | <span style="color:#6BD36B">94.1%</span> |
| <span style="color:#6BD36B">src/secp256k1/points/affine.rs</span> | 80 / 111 | <span style="color:#6BD36B">72.1%</span> | 10 / 15 | <span style="color:#6BD36B">66.7%</span> |
| <span style="color:#4DA3FF">src/secp256k1/points/jacobian.rs</span> | 170 / 300 | <span style="color:#4DA3FF">56.7%</span> | 6 / 12 | <span style="color:#4DA3FF">50.0%</span> |
| <span style="color:#6BD36B">src/secp256k1/points/storage.rs</span> | 7 / 7 | <span style="color:#6BD36B">100.0%</span> | 1 / 1 | <span style="color:#6BD36B">100.0%</span> |
| <span style="color:#6BD36B">src/secp256k1/recover.rs</span> | 218 / 229 | <span style="color:#6BD36B">95.2%</span> | 11 / 11 | <span style="color:#6BD36B">100.0%</span> |
| <span style="color:#6BD36B">src/secp256k1/scalars/invert.rs</span> | 89 / 89 | <span style="color:#6BD36B">100.0%</span> | 2 / 2 | <span style="color:#6BD36B">100.0%</span> |
| <span style="color:#6BD36B">src/secp256k1/scalars/mod.rs</span> | 33 / 42 | <span style="color:#6BD36B">78.6%</span> | 10 / 13 | <span style="color:#6BD36B">76.9%</span> |
| <span style="color:#6BD36B">src/secp256k1/scalars/scalar64.rs</span> | 211 / 226 | <span style="color:#6BD36B">93.4%</span> | 19 / 22 | <span style="color:#6BD36B">86.4%</span> |
| <span style="color:red">**crypto/secp256r1**</span> | 0 / 904 | <span style="color:red">0.0%</span> | 0 / 92 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/context.rs</span> | 0 / 20 | <span style="color:red">0.0%</span> | 0 / 3 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/field/fe64.rs</span> | 0 / 173 | <span style="color:red">0.0%</span> | 0 / 28 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/field/mod.rs</span> | 0 / 68 | <span style="color:red">0.0%</span> | 0 / 6 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/mod.rs</span> | 0 / 9 | <span style="color:red">0.0%</span> | 0 / 1 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/points/affine.rs</span> | 0 / 42 | <span style="color:red">0.0%</span> | 0 / 6 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/points/jacobian.rs</span> | 0 / 203 | <span style="color:red">0.0%</span> | 0 / 10 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/points/storage.rs</span> | 0 / 7 | <span style="color:red">0.0%</span> | 0 / 1 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/scalar/mod.rs</span> | 0 / 56 | <span style="color:red">0.0%</span> | 0 / 8 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/scalar/scalar64.rs</span> | 0 / 198 | <span style="color:red">0.0%</span> | 0 / 18 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/u64_arithmatic.rs</span> | 0 / 12 | <span style="color:red">0.0%</span> | 0 / 3 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/verify.rs</span> | 0 / 65 | <span style="color:red">0.0%</span> | 0 / 4 | <span style="color:red">0.0%</span> |
| <span style="color:red">src/secp256r1/wnaf.rs</span> | 0 / 51 | <span style="color:red">0.0%</span> | 0 / 4 | <span style="color:red">0.0%</span> |
| <span style="color:#6BD36B">src/sha3/mod.rs</span> | 20 / 20 | <span style="color:#6BD36B">100.0%</span> | 9 / 23 | <span style="color:#4DA3FF">39.1%</span> |
| <span style="color:orange"><b>Total</b></span> | 1703 / 5633 | <span style="color:orange">30.2%</span> | 164 / 718 | <span style="color:orange">22.8%</span> |

## Unit tests coverage

The unit test suite has been executed with the `e2e_proving` feature disabled since enabling it caused the suite to crash (see the description of this issue in the main part of the report). To collect coverage data, `llvm-cov` has been used.

Since the test suite of the `crypto` crate is built in "property testing" fashion, each new pass can result in slightly different coverage. This means that to compute realistic coverage, it is necessary to aggregate results from multiple runs. Multiple `llvm-cov` passes have been performed in parallel, on same machine used for fuzzing (48 cores). Each pass has been performed in single-threaded mode since some of the test cases require it:
```sh
# first, we ensure that all test code is compiled
cargo llvm-cov -p crypto --no-report -- --list

# bigger stack is needed for `field_8x32::tests::test_invert`
export RUST_MIN_STACK=33554432

seq 1 96 \
| xargs -P48 -n1 sh -c '
  cargo llvm-cov -p crypto --no-report -- --test-threads=1 --ignored || true
'

cargo llvm-cov report -p crypto --html
```

Following are the coverage statistics produced by the commands above.

### Crate `callable_oracles`

| Filename | Function Coverage | Line Coverage | Region Coverage |
|-----------|------------------:|---------------:|----------------:|
| <span style="color:#6BD36B">src/arithmetic/mod.rs</span> | <span style="color:#4DA3FF">41.2%</span> | <span style="color:#6BD36B">98.1%</span> | <span style="color:red">0.0%</span> |
| <span style="color:red">src/hash_to_prime/common.rs</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> |
| <span style="color:red">src/hash_to_prime/compute.rs</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> |
| <span style="color:red">src/hash_to_prime/evaluate.rs</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> |
| <span style="color:red">src/hash_to_prime/verify.rs</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> |
| <span style="color:red">src/hash_to_prime/lib.rs</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> | <span style="color:red">0.0%</span> |
| <span style="color:#4DA3FF">src/utils/evaluate.rs</span> | <span style="color:orange">22.2%</span> | <span style="color:#4DA3FF">43.8%</span> | <span style="color:red">0.0%</span> |
| <span style="color:#6BD36B">src/utils/usize_slice_iterator.rs</span> | <span style="color:#6BD36B">100.0%</span> | <span style="color:#6BD36B">95.0%</span> | <span style="color:red">0.0%</span> |
| <span style="color:orange"><b>Total</b></span> | <span style="color:orange">16.7%</span> | <span style="color:orange">17.5%</span> | <span style="color:red">0.0%</span> |

### Crate `crypto`

| Filename | Function Coverage | Line Coverage | Region Coverage |
|-----------|------------------:|---------------:|----------------:|
| <span style="color:orange">crypto/src/ark_ff_delegation/biginteger/mod.rs</span> | <span style="color:orange">29.73% (22/74)</span> | <span style="color:orange">23.92% (94/393)</span> | <span style="color:orange">24.40% (133/545)</span> |
| <span style="color:orange">crypto/src/ark_ff_delegation/const_helpers.rs</span> | <span style="color:#4DA3FF">40.00% (10/25)</span> | <span style="color:#4DA3FF">34.94% (58/166)</span> | <span style="color:orange">29.02% (74/255)</span> |
| <span style="color:#4DA3FF">crypto/src/ark_ff_delegation/fp/mod.rs</span> | <span style="color:#4DA3FF">54.95% (50/91)</span> | <span style="color:#4DA3FF">52.87% (221/418)</span> | <span style="color:#4DA3FF">50.98% (311/610)</span> |
| <span style="color:#4DA3FF">crypto/src/ark_ff_delegation/fp/montgomery_backend.rs</span> | <span style="color:#4DA3FF">65.52% (19/29)</span> | <span style="color:#4DA3FF">54.45% (104/191)</span> | <span style="color:#4DA3FF">60.43% (168/278)</span> |
| <span style="color:#6BD36B">crypto/src/bigint_delegation/delegation.rs</span> | <span style="color:#6BD36B">100.00% (15/15)</span> | <span style="color:#6BD36B">100.00% (83/83)</span> | <span style="color:#6BD36B">100.00% (169/169)</span> |
| <span style="color:#6BD36B">crypto/src/bigint_delegation/mod.rs</span> | <span style="color:#6BD36B">100.00% (1/1)</span> | <span style="color:#6BD36B">100.00% (4/4)</span> | <span style="color:#6BD36B">100.00% (4/4)</span> |
| <span style="color:#6BD36B">crypto/src/bigint_delegation/u256.rs</span> | <span style="color:#6BD36B">87.88% (29/33)</span> | <span style="color:#6BD36B">89.87% (213/237)</span> | <span style="color:#6BD36B">89.04% (406/456)</span> |
| <span style="color:#6BD36B">crypto/src/bigint_delegation/u512.rs</span> | <span style="color:#6BD36B">100.00% (14/14)</span> | <span style="color:#6BD36B">98.43% (188/191)</span> | <span style="color:#6BD36B">98.70% (456/462)</span> |
| <span style="color:red">crypto/src/blake2s/naive.rs</span> | <span style="color:red">0.00% (0/9)</span> | <span style="color:red">0.00% (0/30)</span> | <span style="color:red">0.00% (0/43)</span> |
| <span style="color:red">crypto/src/blake2s/test.rs</span> | <span style="color:red">0.00% (0/2)</span> | <span style="color:red">0.00% (0/18)</span> | <span style="color:red">0.00% (0/26)</span> |
| <span style="color:orange">crypto/src/bls12_381/curves/g1.rs</span> | <span style="color:orange">25.00% (4/16)</span> | <span style="color:orange">20.16% (26/129)</span> | <span style="color:orange">12.12% (24/198)</span> |
| <span style="color:#6BD36B">crypto/src/bls12_381/curves/g1_swu_iso.rs</span> | <span style="color:#6BD36B">100.00% (1/1)</span> | <span style="color:#6BD36B">100.00% (6/6)</span> | <span style="color:#6BD36B">100.00% (12/12)</span> |
| <span style="color:red">crypto/src/bls12_381/curves/g2.rs</span> | <span style="color:red">0.00% (0/10)</span> | <span style="color:red">0.00% (0/105)</span> | <span style="color:red">0.00% (0/176)</span> |
| <span style="color:#6BD36B">crypto/src/bls12_381/curves/g2_swu_iso.rs</span> | <span style="color:#6BD36B">100.00% (1/1)</span> | <span style="color:#6BD36B">100.00% (6/6)</span> | <span style="color:#6BD36B">100.00% (12/12)</span> |
| <span style="color:#6BD36B">crypto/src/bls12_381/curves/pairing_impl.rs</span> | <span style="color:#6BD36B">66.67% (14/21)</span> | <span style="color:#6BD36B">80.56% (174/216)</span> | <span style="color:#6BD36B">85.32% (308/361)</span> |
| <span style="color:red">crypto/src/bls12_381/curves/util.rs</span> | <span style="color:red">0.00% (0/10)</span> | <span style="color:red">0.00% (0/166)</span> | <span style="color:red">0.00% (0/320)</span> |
| <span style="color:#6BD36B">crypto/src/bls12_381/fields/fq.rs</span> | <span style="color:#6BD36B">100.00% (26/26)</span> | <span style="color:#6BD36B">98.01% (246/251)</span> | <span style="color:#6BD36B">93.78% (422/450)</span> |
| <span style="color:#4DA3FF">crypto/src/bls12_381/fields/fq2.rs</span> | <span style="color:#4DA3FF">50.00% (2/4)</span> | <span style="color:#4DA3FF">46.15% (6/13)</span> | <span style="color:#4DA3FF">42.86% (6/14)</span> |
| <span style="color:#6BD36B">crypto/src/bls12_381/fields/fq6.rs</span> | <span style="color:#6BD36B">100.00% (1/1)</span> | <span style="color:#6BD36B">100.00% (6/6)</span> | <span style="color:#6BD36B">100.00% (7/7)</span> |
| <span style="color:#6BD36B">crypto/src/bls12_381/fields/fr.rs</span> | <span style="color:#6BD36B">78.95% (15/19)</span> | <span style="color:#6BD36B">84.29% (118/140)</span> | <span style="color:#6BD36B">85.61% (113/132)</span> |
| <span style="color:#6BD36B">crypto/src/bls12_381/fields/mod.rs</span> | <span style="color:#6BD36B">100.00% (1/1)</span> | <span style="color:#6BD36B">100.00% (4/4)</span> | <span style="color:#6BD36B">100.00% (4/4)</span> |
| <span style="color:orange">crypto/src/bn254/curves/g1.rs</span> | <span style="color:#4DA3FF">40.00% (4/10)</span> | <span style="color:orange">29.85% (20/67)</span> | <span style="color:orange">25.51% (25/98)</span> |
| <span style="color:red">crypto/src/bn254/curves/g2.rs</span> | <span style="color:red">0.00% (0/5)</span> | <span style="color:red">0.00% (0/26)</span> | <span style="color:red">0.00% (0/41)</span> |
| <span style="color:#6BD36B">crypto/src/bn254/curves/pairing_impl.rs</span> | <span style="color:#6BD36B">68.18% (15/22)</span> | <span style="color:#6BD36B">81.82% (198/242)</span> | <span style="color:#6BD36B">87.38% (374/428)</span> |
| <span style="color:#6BD36B">crypto/src/bn254/fields/fq.rs</span> | <span style="color:#6BD36B">100.00% (19/19)</span> | <span style="color:#6BD36B">97.06% (198/204)</span> | <span style="color:#6BD36B">92.41% (353/382)</span> |
| <span style="color:#6BD36B">crypto/src/bn254/fields/fq2.rs</span> | <span style="color:#6BD36B">100.00% (1/1)</span> | <span style="color:#6BD36B">100.00% (3/3)</span> | <span style="color:#6BD36B">100.00% (3/3)</span> |
| <span style="color:#6BD36B">crypto/src/bn254/fields/fq6.rs</span> | <span style="color:#6BD36B">100.00% (1/1)</span> | <span style="color:#6BD36B">100.00% (11/11)</span> | <span style="color:#6BD36B">100.00% (17/17)</span> |
| <span style="color:#6BD36B">crypto/src/glv_decomposition.rs</span> | <span style="color:#6BD36B">91.67% (11/12)</span> | <span style="color:#6BD36B">96.35% (132/137)</span> | <span style="color:#6BD36B">94.52% (207/219)</span> |
| <span style="color:#6BD36B">crypto/src/lib.rs</span> | <span style="color:#6BD36B">100.00% (1/1)</span> | <span style="color:#6BD36B">100.00% (9/9)</span> | <span style="color:#6BD36B">100.00% (6/6)</span> |
| <span style="color:red">crypto/src/raw_delegation_interface.rs</span> | <span style="color:red">0.00% (0/2)</span> | <span style="color:red">0.00% (0/18)</span> | <span style="color:red">0.00% (0/17)</span> |
| <span style="color:red">crypto/src/secp256k1/context.rs</span> | <span style="color:red">0.00% (0/4)</span> | <span style="color:red">0.00% (0/34)</span> | <span style="color:red">0.00% (0/45)</span> |
| <span style="color:red">crypto/src/secp256k1/field/field_10x26.rs</span> | <span style="color:red">0.00% (0/36)</span> | <span style="color:red">0.00% (0/624)</span> | <span style="color:red">0.00% (0/876)</span> |
| <span style="color:red">crypto/src/secp256k1/field/field_5x52.rs</span> | <span style="color:red">0.00% (0/37)</span> | <span style="color:red">0.00% (0/448)</span> | <span style="color:red">0.00% (0/562)</span> |
| <span style="color:#6BD36B">crypto/src/secp256k1/field/field_8x32.rs</span> | <span style="color:#6BD36B">70.00% (21/30)</span> | <span style="color:#6BD36B">82.42% (211/256)</span> | <span style="color:#6BD36B">77.27% (170/220)</span> |
| <span style="color:red">crypto/src/secp256k1/field/field_impl.rs</span> | <span style="color:red">0.00% (0/30)</span> | <span style="color:red">0.00% (0/152)</span> | <span style="color:red">0.00% (0/203)</span> |
| <span style="color:red">crypto/src/secp256k1/field/mod.rs</span> | <span style="color:red">0.00% (0/52)</span> | <span style="color:red">0.00% (0/347)</span> | <span style="color:red">0.00% (0/323)</span> |
| <span style="color:red">crypto/src/secp256k1/field/mod_inv32.rs</span> | <span style="color:red">0.00% (0/22)</span> | <span style="color:red">0.00% (0/353)</span> | <span style="color:red">0.00% (0/533)</span> |
| <span style="color:red">crypto/src/secp256k1/field/mod_inv64.rs</span> | <span style="color:red">0.00% (0/18)</span> | <span style="color:red">0.00% (0/348)</span> | <span style="color:red">0.00% (0/550)</span> |
| <span style="color:red">crypto/src/secp256k1/mod.rs</span> | <span style="color:orange">25.00% (1/4)</span> | <span style="color:red">9.76% (4/41)</span> | <span style="color:red">5.97% (4/67)</span> |
| <span style="color:red">crypto/src/secp256k1/points/affine.rs</span> | <span style="color:red">0.00% (0/21)</span> | <span style="color:red">0.00% (0/139)</span> | <span style="color:red">0.00% (0/186)</span> |
| <span style="color:red">crypto/src/secp256k1/points/jacobian.rs</span> | <span style="color:red">0.00% (0/20)</span> | <span style="color:red">0.00% (0/384)</span> | <span style="color:red">0.00% (0/660)</span> |
| <span style="color:red">crypto/src/secp256k1/points/storage.rs</span> | <span style="color:red">0.00% (0/1)</span> | <span style="color:red">0.00% (0/7)</span> | <span style="color:red">0.00% (0/5)</span> |
| <span style="color:red">crypto/src/secp256k1/recover.rs</span> | <span style="color:red">0.00% (0/24)</span> | <span style="color:red">0.00% (0/411)</span> | <span style="color:red">0.00% (0/543)</span> |
| <span style="color:red">crypto/src/secp256k1/scalars/invert.rs</span> | <span style="color:red">0.00% (0/3)</span> | <span style="color:red">0.00% (0/107)</span> | <span style="color:red">0.00% (0/136)</span> |
| <span style="color:red">crypto/src/secp256k1/scalars/mod.rs</span> | <span style="color:red">0.00% (0/30)</span> | <span style="color:red">0.00% (0/137)</span> | <span style="color:red">0.00% (0/148)</span> |
| <span style="color:#4DA3FF">crypto/src/secp256k1/scalars/scalar32_delegation.rs</span> | <span style="color:#4DA3FF">63.16% (24/38)</span> | <span style="color:#4DA3FF">61.03% (166/272)</span> | <span style="color:#4DA3FF">39.93% (119/298)</span> |
| <span style="color:red">crypto/src/secp256k1/scalars/scalar64.rs</span> | <span style="color:red">0.00% (0/28)</span> | <span style="color:red">0.00% (0/247)</span> | <span style="color:red">0.00% (0/746)</span> |
| <span style="color:red">crypto/src/secp256r1/context.rs</span> | <span style="color:red">0.00% (0/3)</span> | <span style="color:red">0.00% (0/20)</span> | <span style="color:red">0.00% (0/32)</span> |
| <span style="color:red">crypto/src/secp256r1/field/fe32_delegation.rs</span> | <span style="color:red">4.35% (1/23)</span> | <span style="color:red">7.00% (7/100)</span> | <span style="color:red">8.33% (10/120)</span> |
| <span style="color:red">crypto/src/secp256r1/field/fe64.rs</span> | <span style="color:red">0.00% (0/30)</span> | <span style="color:red">0.00% (0/185)</span> | <span style="color:red">0.00% (0/480)</span> |
| <span style="color:red">crypto/src/secp256r1/field/mod.rs</span> | <span style="color:red">0.00% (0/15)</span> | <span style="color:red">0.00% (0/256)</span> | <span style="color:red">0.00% (0/177)</span> |
| <span style="color:orange">crypto/src/secp256r1/mod.rs</span> | <span style="color:#4DA3FF">50.00% (1/2)</span> | <span style="color:orange">30.77% (4/13)</span> | <span style="color:orange">21.05% (4/19)</span> |
| <span style="color:red">crypto/src/secp256r1/points/affine.rs</span> | <span style="color:red">0.00% (0/6)</span> | <span style="color:red">0.00% (0/42)</span> | <span style="color:red">0.00% (0/62)</span> |
| <span style="color:red">crypto/src/secp256r1/points/jacobian.rs</span> | <span style="color:red">0.00% (0/14)</span> | <span style="color:red">0.00% (0/240)</span> | <span style="color:red">0.00% (0/447)</span> |
| <span style="color:red">crypto/src/secp256r1/points/storage.rs</span> | <span style="color:red">0.00% (0/1)</span> | <span style="color:red">0.00% (0/7)</span> | <span style="color:red">0.00% (0/3)</span> |
| <span style="color:red">crypto/src/secp256r1/scalar/mod.rs</span> | <span style="color:red">0.00% (0/10)</span> | <span style="color:red">0.00% (0/80)</span> | <span style="color:red">0.00% (0/88)</span> |
| <span style="color:red">crypto/src/secp256r1/scalar/scalar64.rs</span> | <span style="color:red">0.00% (0/21)</span> | <span style="color:red">0.00% (0/209)</span> | <span style="color:red">0.00% (0/564)</span> |
| <span style="color:orange">crypto/src/secp256r1/scalar/scalar_delegation.rs</span> | <span style="color:red">5.56% (1/18)</span> | <span style="color:red">9.59% (7/73)</span> | <span style="color:orange">11.76% (10/85)</span> |
| <span style="color:red">crypto/src/secp256r1/u64_arithmatic.rs</span> | <span style="color:red">0.00% (0/3)</span> | <span style="color:red">0.00% (0/12)</span> | <span style="color:red">0.00% (0/18)</span> |
| <span style="color:red">crypto/src/secp256r1/verify.rs</span> | <span style="color:red">0.00% (0/6)</span> | <span style="color:red">0.00% (0/120)</span> | <span style="color:red">0.00% (0/208)</span> |
| <span style="color:red">crypto/src/secp256r1/wnaf.rs</span> | <span style="color:red">0.00% (0/4)</span> | <span style="color:red">0.00% (0/51)</span> | <span style="color:red">0.00% (0/69)</span> |
| <span style="color:red">crypto/src/sha3/mod.rs</span> | <span style="color:red">0.00% (0/5)</span> | <span style="color:red">0.00% (0/20)</span> | <span style="color:red">0.00% (0/34)</span> |
| <span style="color:red">zksync_os_runner/src/lib.rs</span> | <span style="color:red">0.00% (0/3)</span> | <span style="color:red">0.00% (0/37)</span> | <span style="color:red">0.00% (0/36)</span> |
| <span style="color:orange"><b>Total</b></span> | <span style="color:orange"><b>31.41% (326/1038)</b></span> | <span style="color:orange"><b>27.28% (2527/9262)</b></span> | <span style="color:orange"><b>27.51% (3931/14288)</b></span> |