---
title: ZKsync OS - Crypto
client: Matter Labs
created_at: '2025-11-19'
updated_at: '2025-12-23'
draft: false
---
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

- <strong><span class="border rounded px-1">High</span></strong> Exploitation is almost certain to occur, straightforward to execute, or challenging but highly incentivized. The issue can be exploited by virtually anyone under almost any condition. Attackers can exploit the finding unilaterally without needing special permissions or encountering significant obstacles.
- <strong><span class="border rounded px-1">Medium</span></strong> Exploitation of the issue requires non-trivial preconditions. Once these preconditions are met, the issue is either easy to exploit or well incentivized. Attackers may need to leverage a third party, gain access to non-public information, exploit a race condition, or overcome moderate challenges to exploit the finding.
- <strong><span class="border rounded px-1">Low</span></strong> Exploitation requires stringent preconditions, possibly requiring the alignment of several unlikely factors or a preceding attack. It might involve implausible social engineering, exploiting a challenging race condition, or guessing difficult-to-predict data, making it unlikely. There is little or no incentive to exploit the issue. Centralization issues, exploitable only by operators, on-chain governance, or development teams, fall into this category as well.

### Impact

Impact considers the consequences of successful exploitation on the target system. It accounts for potential loss of confidentiality, integrity, and availability, as well as potential reputational damage.

This factor is classified into one of three levels:

- <strong><span class="border rounded px-1">High</span></strong> Entails the loss of a significant portion of assets within the protocol, or causes substantial harm to the majority of users. Attackers can access or modify all data in a system, execute arbitrary code, escalate their privileges to superuser level, or disrupt the system's ability to serve its users.
- <strong><span class="border rounded px-1">Medium</span></strong> Leads to global losses of assets in moderate amounts or affects only a subset of users, which is still deemed unacceptable. Attackers can access or modify some unauthorized data, affect availability or restrict access to the system, or obtain significant internal technical insights.
- <strong><span class="border rounded px-1">Low</span></strong> Losses are inconvenient but manageable. This applies to scenarios like easily remediable griefing attacks or gas inefficiencies. Attackers may access limited amounts of unauthorized information or marginally degrade system performance. This category also includes code quality concerns and non-exploitable issues that may still negatively impact public perception of security.

### Final Severity

Upon assessing the issue's likelihood and impact, its severity is determined using the table below:

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;"></th>
  <th style="border: 1px solid white; padding: 8px;"><span class="border rounded px-1">High</span></th>
  <th style="border: 1px solid white; padding: 8px;"><span class="border rounded px-1">Medium</span></th>
  <th style="border: 1px solid white; padding: 8px;"><span class="border rounded px-1">Low</span></th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span class="border rounded px-1">High</span></td>
  <td style="border: 1px solid white; padding: 8px;"><strong><span style="color: #ef4444; font-weight: 700;">Critical</span></strong></td>
  <td style="border: 1px solid white; padding: 8px;"><strong><span style="color: #f97316; font-weight: 700;">High</span></strong></td>
  <td style="border: 1px solid white; padding: 8px;"><strong><span style="color: #fbc02d; font-weight: 700;">Medium</span></strong></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span class="border rounded px-1">Medium</span></td>
  <td style="border: 1px solid white; padding: 8px;"><strong><span style="color: #f97316; font-weight: 700;">High</span></strong></td>
  <td style="border: 1px solid white; padding: 8px;"><strong><span style="color: #fbc02d; font-weight: 700;">Medium</span></strong></td>
  <td style="border: 1px solid white; padding: 8px;"><strong><span style="color: #84cc16; font-weight: 700;">Low</span></strong></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span class="border rounded px-1">Low</span></td>
  <td style="border: 1px solid white; padding: 8px;"><strong><span style="color: #fbc02d; font-weight: 700;">Medium</span></strong></td>
  <td style="border: 1px solid white; padding: 8px;"><strong><span style="color: #84cc16; font-weight: 700;">Low</span></strong></td>
  <td style="border: 1px solid white; padding: 8px;"><strong><span style="color: #22c55e; font-weight: 700;">QA</span></strong></td>
</tr>
</tbody>
</table>

In exceptional cases, the total severity may be elevated to highlight the significance of the issue. Whenever this occurs, the rationale for applying an increased severity level is detailed in the report.

## Issue Statuses

The status of an issue can be one of the following:

- <span style="color: #fbc02d; margin: 0px;">Notified</span> indicates that the client has been informed of the issue but has either not addressed it yet or has acknowledged the behavior without planning to remediate it soon. This inaction may stem from the client not viewing the behavior as problematic, lacking a feasible solution, or intending to address it at a later date.
- <span style="color: #00acc1; margin: 0px;">Addressed</span> is used when the client has made an effort to address the issue with partial success. In the case of complex issues, the provided fix may only resolve one of several points described.
- <span style="color: #10b981; margin: 0px;">Fixed</span> means that the client has implemented a solution that completely resolves the described issue.

<h1 id="summary-of-issues">Summary of Issues</h1>

<table style="border-collapse: collapse; border: 1px solid white; width: 100%;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px; text-align: left;">Title</th>
  <th style="border: 1px solid white; padding: 8px; text-align: left;">Severity</th>
  <th style="border: 1px solid white; padding: 8px; text-align: left;">Status</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-1-affine-and-jacobian-points" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">1. Incorrect identity predicates for affine and Jacobian points</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #ef4444;">Critical</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-2-undefined-behavior-in-pairings-check" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">2. DoS and corrupted state in BN254 and BLS12-381 pairings checks</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #ef4444;">Critical</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-3-insufficient-magnitude" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">3. Insufficient magnitude update in scalar multiplication</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #f97316;">High</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-4-failed-assertion-in-fma" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">4. Failed assertion in the Fused Multiply-Add operation</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Medium</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-5-unchecked-field-arithmetic" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">5. Unchecked field-arithmetic invariants</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Medium</td>
  <td style="border: 1px solid white; padding: 8px; color: #00acc1;">Addressed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-6-magnitude-limit-2047" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">6. Magnitude limit 2047 cannot detect arithmetic issues in 5×52 backend</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-7-incorrect-validation-table_verify" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">7. Incorrect validation of wNAF table digits</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-8-miscellaneous-validations" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">8. Miscellaneous validation issues</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #00acc1;">Addressed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-9-QA-thread-safety" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">9. Concurrent usage is not supported</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-10-QA-variable-timing" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">10. Variable-time scalar operations should not be used with secret data</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #00acc1;">Addressed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-11-QA-normalization-check" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">11. Normalization check is not strong enough</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-12-unit_tests" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">12. Unit tests issues</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #00acc1;">Addressed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-13-QA-excessive-magnitude" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">13. Excessive magnitude increment</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-14-QA-magic-numbers" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">14. Magic numbers decrease maintainability</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #00acc1;">Addressed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-15-QA-code-duplicates" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">15. Code duplicates</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-16-QA-redundant-code" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">16. Redundant `else` branch</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
</tbody>
</table>


<div id="issue-1-affine-and-jacobian-points" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">1. Incorrect identity predicates for affine and Jacobian points</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #ef4444; border-radius: 4px; padding: 4px 8px; color: #ef4444;">Critical</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

In the `secp256r1` module, incorrect predicates are used to determine whether a point represents the identity (the point at infinity):

<pre class="language-rust" data-attributes="filepath line=282"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">src/secp256r1/points/jacobian.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">282</span><span style="font-family: monospace !important;">pub(crate) const fn is_infinity_const(&amp;self) -&gt; bool {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">283</span><span style="font-family: monospace !important;">    self.z.is_zero() || (self.x.is_zero() || self.y.is_zero())</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">284</span><span style="font-family: monospace !important;">}</span></span></code></pre>

<pre class="language-rust" data-attributes="filepath line=21"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">src/secp256r1/points/affine.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">21</span><span style="font-family: monospace !important;">pub(crate) fn is_infinity(&amp;self) -&gt; bool {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">22</span><span style="font-family: monospace !important;">    self.infinity || (self.x.is_zero() || self.y.is_zero())</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">23</span><span style="font-family: monospace !important;">}</span></span></code></pre>

In Jacobian coordinates, a point represents the identity if and only if `z` is zero. In affine coordinates, the identity is typically represented only using an explicit flag, not by coordinate values.

While interpreting the `(0, 0)` as the identity could be intended as a simplification of the implementation, the aforementioned functions also consider points `(0, y)` and `(x, 0)` as the identity. This is logically incorrect: valid curve points may legitimately have `x` or `y` equal to zero, and must not be treated as the identity.

This flaw breaks fundamental elliptic-curve invariants and has critical impacts:
- Valid signatures may be rejected
- Invalid signatures may be accepted

This behavior diverges from standard `secp256r1` implementations and can lead to inconsistent public keys, hashes, and verification results. Arithmetic operations may return incorrect results, spuriously produce the identity, or terminate early: addition, doubling and, overall, scalar multiplication.



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

<div id="issue-2-undefined-behavior-in-pairings-check" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">2. DoS and corrupted state in BN254 and BLS12-381 pairings checks</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #ef4444; border-radius: 4px; padding: 4px 8px; color: #ef4444;">Critical</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

## The issue in the `bn254` module

The function `bn254_pairing_check_inner` parses the buffer `src` into G1 and G2 affine points and constructs `(G1Affine, G2Affine)` pairs. The buffer `src` is supplied by the caller function `execute` and may contain arbitrary data defined during a smart contract execution. The `(G1Affine, G2Affine)` pairs are then passed into the function `Bn254::multi_pairing` on line `154`:
<pre class="language-rust" data-attributes="filepath line=66"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">basic_system/src/system_functions/bn254_pairing_check.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">66</span><span style="font-family: monospace !important;">fn bn254_pairing_check_inner&lt;A: Allocator&gt;(</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">67</span><span style="font-family: monospace !important;">    num_pairs: usize,</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">68</span><span style="font-family: monospace !important;">    src: &amp;[u8],</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">69</span><span style="font-family: monospace !important;">    allocator: A,</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">70</span><span style="font-family: monospace !important;">) -&gt; Result&lt;bool, ()&gt; {</span></span></code></pre>

<pre class="language-rust" data-attributes="line=152"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">152</span><span style="font-family: monospace !important;">let g1_iter = pairs.iter().map(|(g1, _)| g1);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">153</span><span style="font-family: monospace !important;">let g2_iter = pairs.iter().map(|(_, g2)| g2);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">154</span><span style="font-family: monospace !important;">let result = Bn254::multi_pairing(g1_iter, g2_iter);</span></span></code></pre>

The function `Bn254::multi_pairing` includes a call to the `Bn254::multi_miller_loop` function, which processes each _(G1,G2)_ pair by converting the _G1_ input into `G1Prepared` and the `G2Affine` input into `G2Prepared`. After the conversions, both results are checked for representing the point-at-infinity, using the `is_zero` function. If either point is the point-at-infinity, the pair is skipped and the loop continues with the next pair:
<pre class="language-rust" data-attributes="filepath line=109"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/bn254/curves/pairing_impl.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">109</span><span style="font-family: monospace !important;">let q: Self::G2Prepared = q.into();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">110</span><span style="font-family: monospace !important;">if q.is_zero() {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">111</span><span style="font-family: monospace !important;">    continue;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">112</span><span style="font-family: monospace !important;">}</span></span></code></pre>



The conversion `into()` for `G2Affine` type is implemented via `G2PreparedNoAlloc::from`, which handles the point-at-infinity case as follows:
<pre class="language-rust" data-attributes="context line=217 highlight=[4]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">217</span><span style="font-family: monospace !important;">impl From&lt;G2Affine&gt; for G2PreparedNoAlloc</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">218</span><span style="font-family: monospace !important;">if q.infinity {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">219</span><span style="font-family: monospace !important;">    #[allow(invalid_value)]</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">220</span><span style="font-family: monospace !important;">    G2PreparedNoAlloc {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">221</span><span style="font-family: monospace !important;">        ell_coeffs: unsafe { core::mem::MaybeUninit::uninit().assume_init() }, // unused/filtered above</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">222</span><span style="font-family: monospace !important;">        infinity: true,</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">223</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">224</span><span style="font-family: monospace !important;">} else {</span></span></code></pre>

Both `q.is_zero()` and `q.infinity` implement the check for point-at-infinity:
<pre class="language-rust" data-attributes="line=357"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">357</span><span style="font-family: monospace !important;">impl G2PreparedNoAlloc {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">358</span><span style="font-family: monospace !important;">    pub fn is_zero(&amp;self) -&gt; bool {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">359</span><span style="font-family: monospace !important;">        self.infinity</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">360</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">361</span><span style="font-family: monospace !important;">}</span></span></code></pre>

This means that both lines `110` and `217` check for the same condition, so there is a scenario when the block on lines `218-222` is executed. The line `220` contains an `unsafe` expression, accompanied by the comment `// unused/filtered above`. Apparently, the idea here is that the `unsafe` section would never actually execute if the field `ell_coeffs` is not accessed. However, struct fields are eagerly initialized in Rust, so all fields of a struct literal are evaluated before the value exists, and the `unsafe` section is executed every time the control flow reaches lines `218-222`.

In other words, the `unsafe` section on line `220` is always entered before the filtering on line `110` occurs.

This `unsafe` section immediately triggers an Undefined Behavior, even if the field `ell_coeffs` is never accessed later, because the expression `MaybeUninit::uninit().assume_init()` creates an uninitialized array and treats it as initialized. This is confirmed to be an invalid use by the documentation of the `assume_init` function:
<pre class="language-rust"><code>/// # Safety
///
/// It is up to the caller to guarantee that the `MaybeUninit&lt;T&gt;` really is in an initialized
/// state. Calling this when the content is not yet fully initialized causes immediate undefined
/// behavior. The [type-level documentation][inv] contains more information about
/// this initialization invariant.
</code></pre>



## Similar issue in the `bls12_381` module

The same issue is present in the BLS12-381 implementation, due to code duplication.

<pre class="language-rust" data-attributes="filepath context line=112"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/bls12_381/curves/pairing_impl.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">112</span><span style="font-family: monospace !important;">fn multi_miller_loop</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">113</span><span style="font-family: monospace !important;">let q: Self::G2Prepared = q.into();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">114</span><span style="font-family: monospace !important;">if q.is_zero() {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">115</span><span style="font-family: monospace !important;">    continue;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">116</span><span style="font-family: monospace !important;">}</span></span></code></pre>

<pre class="language-rust" data-attributes="context line=217"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">217</span><span style="font-family: monospace !important;">impl From&lt;G2Affine&gt; for G2PreparedNoAlloc {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">218</span><span style="font-family: monospace !important;">if q.infinity {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">219</span><span style="font-family: monospace !important;">    #[allow(invalid_value)]</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">220</span><span style="font-family: monospace !important;">    G2PreparedNoAlloc {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">221</span><span style="font-family: monospace !important;">        ell_coeffs: unsafe { core::mem::MaybeUninit::uninit().assume_init() }, // unused/filtered above</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">222</span><span style="font-family: monospace !important;">        infinity: true,</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">223</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">224</span><span style="font-family: monospace !important;">} else {</span></span></code></pre>

## Severity

The impact of Undefined Behavior in a cryptographic setting is considered High, because it invalidates all guarantees about correctness, soundness, reproducibility, and security. Undefined Behavior can affect any code surrounding the line triggering it, since it permits the compiler to apply otherwise invalid optimizations.

Practically, UB may allow the attacker to:
- bypass cryptographic verification completely
- corrupt other transactions in any blocks within the same execution context

Since input points are derived from attacker-controlled smart contract calldata, triggering the UB is trivial. Exploitability also depends on compiler behavior but we classify the likelihood of this issue as High.

Even if the corrupted state is eventually rejected during verification, the issue still equips attackers with an extremely cheap DoS vector.



## Recommendations

Apply these recommendations to both BN254 and BLS12-381 implementations:

1. Reject invalid points, including the point-at-infinity, early — before invoking any conversions on them.
2. Construct a concrete, fully-initialized dummy `EllCoeff` (zeros) in the array initializer.

<div id="issue-3-insufficient-magnitude" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">3. Insufficient magnitude update in scalar multiplication</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #f97316; border-radius: 4px; padding: 4px 8px; color: #f97316;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

In the `secp256k1` module, several incorrect magnitude updates have been identified, specifically in the implementation of scalar multiplication.

The function `mul_int_in_place`, which takes a field element `self` and a scalar `rhs`, increases the current magnitude by `rhs`. This is incorrect: every limb of the field element is multiplied by `rhs`, so if a limb is already inflated by a factor of `k`, the resulting inflation factor is `k * rhs`, not `k + rhs`.

<pre class="language-rust" data-attributes="filepath context line=79 highlight=[1]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">src/secp256k1/field/field_impl.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">79</span><span style="font-family: monospace !important;">pub(super) fn mul_int_in_place(&amp;mut self, rhs: u32)</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">80</span><span style="font-family: monospace !important;">self.magnitude += rhs;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">81</span><span style="font-family: monospace !important;">debug_assert!(self.magnitude &lt;= Self::max_magnitude());</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">82</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">83</span><span style="font-family: monospace !important;">self.value.mul_int_in_place(rhs);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">84</span><span style="font-family: monospace !important;">self.normalized = false;</span></span></code></pre>

The same incorrect update is present in `mul_int`:

<pre class="language-rust" data-attributes="context line=159 highlight=[1]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">159</span><span style="font-family: monospace !important;">pub(super) const fn mul_int(&amp;self, rhs: u32)</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">160</span><span style="font-family: monospace !important;">let new_magnitude = self.magnitude + rhs;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">161</span><span style="font-family: monospace !important;">debug_assert!(new_magnitude &lt;= Self::max_magnitude());</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">162</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">163</span><span style="font-family: monospace !important;">let value = self.value.mul_int(rhs);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">164</span><span style="font-family: monospace !important;">Self::new(value, new_magnitude)</span></span></code></pre>

Both `mul_int_in_place` and `mul_int` underestimate the resulting magnitude of the field element. At a minimum, this invalidates assumptions relied upon by routines such as `negate_in_place`.

In this codebase, the magnitude invariants are ensured only by debug assertions, and no runtime validations exist in release mode. As a consequence, silent field-arithmetic failures may occur. In the context of signature verification, this may lead to:
- invalid signatures being accepted,
- valid signatures being rejected.

If the same code were used in a decentralized environment (e.g. an L1 network), an additional impact would be the risk of consensus faults.

## Recommendation

Correct the magnitude update so that the resulting magnitude is computed as the product of the previous magnitude and the scalar, rather than their sum.

<div id="issue-4-failed-assertion-in-fma" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">4. Failed assertion in the Fused Multiply-Add operation</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

During fuzzing test runs, an assertion failed:
<pre><code>thread '&lt;unnamed&gt;' panicked at /zksync-os/basic_system/src/system_functions/modexp/delegation/bigint.rs:686:25:
assertion `left == right` failed
  left: 1
 right: 0
</code></pre>

See the _Appendix_ section for details on how fuzzing was performed.

The assertion is located in the function `fma`, which implements a Fused Multiply-Add (FMA) operation for the `BigintRepr` structure.



The function `bigint_op_delegation_raw`, defined in the in-scope crate `crypto`, returns a non-zero value to signal a limb overflow. The assertion assumes that no overflow can occur at this stage of the algorithm:

<pre class="language-rust" data-attributes="filepath context line=681 highlight=[1,6]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">basic_system/src/system_functions/modexp/delegation/bigint.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">681</span><span style="font-family: monospace !important;">unsafe fn fma</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">682</span><span style="font-family: monospace !important;">let of = bigint_op_delegation_raw(</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">683</span><span style="font-family: monospace !important;">    dst_scratch_capacity[dst_digit].as_mut_ptr().cast(),</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">684</span><span style="font-family: monospace !important;">    carry_scratch.cast(),</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">685</span><span style="font-family: monospace !important;">    BigIntOps::Add,</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">686</span><span style="font-family: monospace !important;">);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">687</span><span style="font-family: monospace !important;">assert_eq!(of, 0);</span></span></code></pre>

However, this assumption is incorrect. During accumulation, under certain operand combinations, a carry produced by earlier partial products can overflow the destination digit, requiring propagation into higher digits.

As a consequence, this creates a limited potential for panic-based DoS during modular exponentiation. Because this is a non-debug assertion, the failure is a deterministic panic in release builds rather than silent truncation. The function `fma` is callable from functions `mul_step`, `square_step`, and `reduce_initially`. These functions are invoked by the `modpow` function, which is reachable from the top-level `execute` function when processing the `modexp` system function. This system function can be used by both honest and malicious transactions. If the assertion is triggered by an honest transaction, the failure will appear as a DoS to the user.

At the same time, the likelihood of exploitation is low. The failing input is not trivially constructible.

<div id="issue-5-unchecked-field-arithmetic" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">5. Unchecked field-arithmetic invariants</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #00acc1; border-radius: 4px; padding: 4px 8px; color: #00acc1;">Addressed</span></span>
  </div>
</div>

Regular CI test runs execute the test suite only in Release mode, where debug assertions and integer overflow checks are disabled. The codebase provides no runtime validation of field-arithmetic invariants in Release builds. Correctness of the field operations depends entirely on debug-mode assertions and overflow checks, which are never exercised by CI. If these invariants are violated, the failures remain silent in Release mode, allowing field-arithmetic errors to persist undetected.

Consider the file `.github/workflows/ci.yml`:
- The only `cargo test` commands without `--release` flag used in this file are located on lines `71` and `155`, where the scope of their action is limited to modules `transactions` and `binary_checker` correspondingly.
- All the other invocations of `cargo test` run in Release profiles: lines `43`, `45`, `178`, `201` and `202`.

As a consequence, overflows and debug assertions during test suite execution are not reported.

When running the tests specifically on the `crypto` crate, with overflow checks and debug assertions enabled, multiple sporadious issues have been discovered. These issues do not reproduce consistently due to randomized nature of property testing involved.

## Proof of concept

After running `./dump_bin.sh` as usually, enable the extra checks using the `RUSTFLAGS` and build the project including the test code of the `crypto` crate:
<pre class="language-sh"><code>export RUSTFLAGS="-C overflow-checks=on -C debug-assertions=on"
cargo build --release
cargo test --release -p crypto
</code></pre>

Then run the property tests multiple times, using a loop:
<pre class="language-sh"><code>$ for i in `seq 1 1000`
do
    cargo test --release -p crypto 2&gt;&amp;1 | grep FAILED | tee -a errors.txt
done
</code></pre>

During manual exploration, it is practical to limit scope of each pass to only one submodule or one test case, for easier interpretation of results. Additionally, parallelization should be enabled in order to save total computation time:
<pre class="language-sh"><code>seq 1 1000 \
| xargs -n1 -P48 sh -c 'cargo test --release -p crypto secp256k1::field::field_10x26 2&gt;&amp;1 | grep FAILED || true' _ \
| tee -a errors.txt
</code></pre>



## Examples of unreported issues

Although it is recommended to ensure overflow checks and debug assertions in all test cases, actual violations have been discovered only in the `secp256k1::field` module.

### Violated assertions

These test cases have crashed on a `debug_assert!` statement:
- `secp256k1::field::field_10x26`: `test_add`, `test_invert_var`, `test_mul`, `to_bytes_round`, `to_signed30_round`
- `secp256k1::field::field_5x52`: `test_add`, `test_invert`, `test_mul`, `to_bytes_round`, `to_signed62_round`, `to_storage_round`
- `secp256k1::field::tests`: `storage_round_trip`, `test_add`, `test_add_const`, `test_invert`, `test_invert_const`, `test_mul`, `test_mul_const`, `test_square`, `test_square_const`, `to_bytes_round_trip`
- `secp256k1::points::jacobian`: `test_add_zinv`

For instance, one of violated debug assertions is located in the `normalize_in_place` function:
<pre class="language-rust" data-attributes="filepath context line=589 highlight=[5]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">src/secp256k1/field/field_10x26.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">589</span><span style="font-family: monospace !important;">pub(super) const fn normalize_in_place(&amp;mut self) {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">590</span><span style="font-family: monospace !important;">self.0[8] &amp;= 0x3FFFFFF;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">591</span><span style="font-family: monospace !important;">m &amp;= self.0[8];</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">592</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">593</span><span style="font-family: monospace !important;">/* ... except for a possible carry at bit 22 of self.0[9] (i.e. bit 256 of the field element) */</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">594</span><span style="font-family: monospace !important;">debug_assert!(self.0[9] &gt;&gt; 23 == 0);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">595</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">596</span><span style="font-family: monospace !important;">// At most a single final reduction is needed; check if the value is &gt;= the field characteristic</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">597</span><span style="font-family: monospace !important;">x = (self.0[9] &gt;&gt; 22)</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">598</span><span style="font-family: monospace !important;">    | ((self.0[9] == 0x03FFFFF)</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">599</span><span style="font-family: monospace !important;">        &amp; (m == 0x3FFFFFF)</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">600</span><span style="font-family: monospace !important;">        &amp; ((self.0[1] + 0x40 + ((self.0[0] + 0x3D1) &gt;&gt; 26)) &gt; 0x3FFFFFF))</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">601</span><span style="font-family: monospace !important;">        as u32;</span></span></code></pre>

### Missed overflows

These test cases have crashed with overflow error:
- `secp256k1::field::field_10x26`: `test_add`, `test_invert_var`, `test_mul`, `to_bytes_round`, `to_signed30_round`
- `secp256k1::field::field_5x52`: `test_add`
- `secp256k1::field::tests`: `test_add_const`
- `secp256k1::points::affine`: `jacobian_round_trip`
- `secp256k1::points::jacobian`: `test_add_zinv`

For instance, the overflows error in the `field_10x26::test_mul` test case is thrown from the `normalize_in_place` function:
<pre class="language-rust" data-attributes="filepath context line=559 highlight=[7]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">src/secp256k1/field/field_10x26.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">559</span><span style="font-family: monospace !important;">pub(super) const fn normalize_in_place(&amp;mut self) {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">560</span><span style="font-family: monospace !important;">// Reduce self.0[9] at the start so there will be at most a single carry from the first pass</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">561</span><span style="font-family: monospace !important;">let mut x = self.0[9] &gt;&gt; 22;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">562</span><span style="font-family: monospace !important;">self.0[9] &amp;= 0x03FFFFF;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">563</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">564</span><span style="font-family: monospace !important;">// The first pass ensures the magnitude is 1, ...</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">565</span><span style="font-family: monospace !important;">self.0[0] += x * 0x3D1;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">566</span><span style="font-family: monospace !important;">self.0[1] += x &lt;&lt; 6;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">567</span><span style="font-family: monospace !important;">self.0[1] += self.0[0] &gt;&gt; 26;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">568</span><span style="font-family: monospace !important;">self.0[0] &amp;= 0x3FFFFFF;</span></span></code></pre>

It is possible that some of the overflows are caused by broken invariants, indicated by the violated assertions.

# Recommendation

Refactor CI workflows and run all tests with all possible checks since running the test suite on local developer's machine is not straightforward and time consuming. In security context, this means that the test suite is never actually executed to the full extent.

Additionally, run the test suite multiple times. Property-based tests do not guarantee full coverage in a single pass, and repeated execution significantly increases the likelihood of catching regressions.

<div id="issue-6-magnitude-limit-2047" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">6. Magnitude limit 2047 cannot detect arithmetic issues in 5×52 backend</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

The magnitude limit for the `5x52` storage is defined as `2047`:

<pre class="language-rust" data-attributes="filepath line=89"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256k1/field/field_5x52.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">89</span><span style="font-family: monospace !important;">pub(super) const fn max_magnitude() -&gt; u32 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">90</span><span style="font-family: monospace !important;">    2047u32</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">91</span><span style="font-family: monospace !important;">}</span></span></code></pre>

Although `2047` is the mathematical upper bound for keeping all limbs within 63 bits, it is not compatible with the actual algorithmic invariants in the `mul_in_place` implementation.

The `mul_in_place` implementation contains tight limb-width debug assertions:
<pre class="language-rust" data-attributes="filepath context line=162"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256k1/field/field_5x52.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">162</span><span style="font-family: monospace !important;">pub(super) const fn mul_in_place</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">163</span><span style="font-family: monospace !important;">debug_assert!(a0 &gt;&gt; 56 == 0);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">164</span><span style="font-family: monospace !important;">debug_assert!(a1 &gt;&gt; 56 == 0);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">165</span><span style="font-family: monospace !important;">debug_assert!(a2 &gt;&gt; 56 == 0);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">166</span><span style="font-family: monospace !important;">debug_assert!(a3 &gt;&gt; 56 == 0);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">167</span><span style="font-family: monospace !important;">debug_assert!(a4 &gt;&gt; 52 == 0);</span></span></code></pre>

These constraints imply:
- Limbs 0–3 may temporarily grow from 52 bits up to 56 bits.
- Limb 4 may expand only to 52 bits.
- No limb is ever allowed to approach the theoretical 63-bit capacity that would justify a magnitude limit of `2047`.

Thus, a magnitude of `2047` can never occur during legal operation, and using `2047` as `max_magnitude()` renders magnitude-based invariants validation ineffective. Internal arithmetic faults may pass unnoticed.



### Estimation based on `mul_in_place` invariants

A normalized limb less than `2^52` corresponds to the magnitude of 1. The largest permitted limb before entering `mul_in_place` is less than `2^56`. Thus the true maximum magnitude allowed by the `mul_in_place` function is:

<pre><code>(2^56) / (2^52) = 2^4 = 16
</code></pre>

Any magnitude above `16` cannot be produced by correct arithmetic and cannot be safely consumed by `mul_in_place`.

## Recommendation

Set the magnitude limit to `16`.

<div id="issue-7-incorrect-validation-table_verify" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">7. Incorrect validation of wNAF table digits</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

In `secp256k1`, the function `table_verify` is used in debug assertions to validate wNAF digits:

<pre class="language-rust" data-attributes="filepath line=328 highlight=[2]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256k1/recover.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">328</span><span style="font-family: monospace !important;">fn table_get_ge(pre: &amp;[Affine], n: i32, w: usize) -&gt; Affine {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">329</span><span style="font-family: monospace !important;">    debug_assert!(table_verify(n, w));</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">330</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">331</span><span style="font-family: monospace !important;">    if n &gt; 0 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">332</span><span style="font-family: monospace !important;">        pre[(n - 1) as usize / 2]</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">333</span><span style="font-family: monospace !important;">    } else {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">334</span><span style="font-family: monospace !important;">        let mut r = pre[(-n - 1) as usize / 2];</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">335</span><span style="font-family: monospace !important;">        r.y.negate_in_place(1);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">336</span><span style="font-family: monospace !important;">        r</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">337</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">338</span><span style="font-family: monospace !important;">}</span></span></code></pre>



However, it is implemented incorrectly:

<pre class="language-rust" data-attributes="line=373 highlight=[2,4]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">373</span><span style="font-family: monospace !important;">fn table_verify(n: i32, w: usize) -&gt; bool {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">374</span><span style="font-family: monospace !important;">    let n = n as usize;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">375</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">376</span><span style="font-family: monospace !important;">    ((n &amp; 1) == 1) || (n &gt;= !(1 &lt;&lt; ((w - 1) - 1))) || (n &lt;= (1 &lt;&lt; ((w - 1) - 1)))</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">377</span><span style="font-family: monospace !important;">}</span></span></code></pre>

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

<pre class="language-rust"><code>fn table_verify(n: i32, w: usize) -&gt; bool {
    // `w` is constant, but this check makes the function future-proof
    // it would underflow/overflow when `w` is out of range
    debug_assert!((2..=31).contains(&amp;w));

    // n must be odd
    if (n &amp; 1) == 0 {
        return false;
    }

    let max: i32 = (1_i32 &lt;&lt; (w - 1)) - 1;
    n &gt;= -max &amp;&amp; n &lt;= max
}
</code></pre>

Additionally, implement similar validation in `secp256r1`.

<div id="issue-8-miscellaneous-validations" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">8. Miscellaneous validation issues</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #00acc1; border-radius: 4px; padding: 4px 8px; color: #00acc1;">Addressed</span></span>
  </div>
</div>

## Imprecise range check includes the modulus

In the `8x32` backend, the `from_bytes` function performs an incorrect range check when validating field elements parsed from raw bytes:

<pre class="language-rust" data-attributes="filepath context line=58"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256k1/field_8x32.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">58</span><span style="font-family: monospace !important;">pub(super) fn from_bytes</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">59</span><span style="font-family: monospace !important;">let value = Self::from_bytes_unchecked(bytes);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">60</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">61</span><span style="font-family: monospace !important;">if u256::leq(&amp;value.0, &amp;Self::MODULUS.0) {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">62</span><span style="font-family: monospace !important;">    Some(value)</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">63</span><span style="font-family: monospace !important;">} else {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">64</span><span style="font-family: monospace !important;">    None</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">65</span><span style="font-family: monospace !important;">}</span></span></code></pre>

This condition `u256::leq` accepts values equal to _P_, whereas valid field elements must satisfy _0 ≤ x < P_. As a result, the modulus _P_ itself is incorrectly accepted as a valid field element.

## Missing defensive magnitude check in `normalize_in_place`

The `normalize_in_place` function assumes that magnitude of the internal field element `self.magnitude` does not exceed `31`, but this invariant is not enforced locally:

<pre class="language-rust" data-attributes="filepath context line=559"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256k1/field/field_10x26.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">559</span><span style="font-family: monospace !important;">pub(super) const fn normalize_in_place</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">560</span><span style="font-family: monospace !important;">// Reduce self.0[9] at the start so there will be at most a single carry from the first pass</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">561</span><span style="font-family: monospace !important;">let mut x = self.0[9] &gt;&gt; 22;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">562</span><span style="font-family: monospace !important;">self.0[9] &amp;= 0x03FFFFF;</span></span></code></pre>

If `self.magnitude > 31`, intermediate arithmetic could overflow, leading to incorrect field normalization. A debug assertion should ensure this assumption.

In the current version of the codebase, this condition is not reachable: the maximum observed magnitude at this point is `10`. However, future changes (e.g., repeated add() or negate() operations) could introduce code paths where the magnitude exceeds `31`.

## Missing defensive overflow checks

<pre class="language-rust" data-attributes="filepath context line=316 highlight=[8]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256k1/scalars/scalar64.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">316</span><span style="font-family: monospace !important;">fn muladd(a: u64, b: u64, c0: u64, c1: u64, c2: u64)</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">317</span><span style="font-family: monospace !important;">let t = (a as u128) * (b as u128);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">318</span><span style="font-family: monospace !important;">let th = (t &gt;&gt; 64) as u64; // at most 0xFFFFFFFFFFFFFFFE</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">319</span><span style="font-family: monospace !important;">let tl = t as u64;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">320</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">321</span><span style="font-family: monospace !important;">let (new_c0, carry0) = c0.overflowing_add(tl);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">322</span><span style="font-family: monospace !important;">let new_th = th.wrapping_add(carry0 as u64); // at most 0xFFFFFFFFFFFFFFFF</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">323</span><span style="font-family: monospace !important;">let (new_c1, carry1) = c1.overflowing_add(new_th);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">324</span><span style="font-family: monospace !important;">let new_c2 = c2 + (carry1 as u64);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">325</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">326</span><span style="font-family: monospace !important;">(new_c0, new_c1, new_c2)</span></span></code></pre>

Similarly in:
- `crypto/src/secp256k1/scalars/scalar64.rs`
	- `muladd_fast` on lines 328–340
	- `sumadd_fast` on lines 342–347
- `crypto/src/secp256k1/scalars/scalar32.rs`
	- Corresponding 32-bit implementations

## Unenforced input bound in `sub_mod_with_carry`

The `sub_mod_with_carry` function relies on the precondition _a < 2·P_, but this bound is not enforced. If the invariant is violated, the result is not guaranteed to lie in _[0, P)_.

<pre class="language-rust" data-attributes="filepath line=142 highlight=[1]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/bigint_delegation/u256.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">142</span><span style="font-family: monospace !important;">/// Note: we assume `self &lt; 2*modulus`, otherwise the result might not be in the range</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">143</span><span style="font-family: monospace !important;">/// # Safety</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">144</span><span style="font-family: monospace !important;">/// `DelegationModParams` should only provide references to mutable statics.</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">145</span><span style="font-family: monospace !important;">/// It is the responsibility of the caller to make sure that is the case</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">146</span><span style="font-family: monospace !important;">pub unsafe fn sub_mod_with_carry&lt;T: DelegatedModParams&lt;4&gt;&gt;(a: &amp;mut U256, carry: bool) {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">147</span><span style="font-family: monospace !important;">    let borrow = delegation::sub(a, T::modulus()) != 0;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">148</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">149</span><span style="font-family: monospace !important;">    if borrow &amp;&amp; !carry {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">150</span><span style="font-family: monospace !important;">        delegation::add(a, T::modulus());</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">151</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">152</span><span style="font-family: monospace !important;">}</span></span></code></pre>

Same issue in `crypto/src/bigint_delegation/u512.rs`.

In the current version of the codebase, all callers respect the required bounds.

## Unsafe buffer bounds assumption

Invoking the function `assert_unchecked` is immediate Undefined Behavior if the condition does not actually hold:

<pre class="language-rust" data-attributes="filepath context highlight=[10]"><code>crypto/src/blake2s/delegated_extended.rs
fn finalize_impl
unsafe {
    // write zeroes
    let start = self
        .state
        .input_buffer
        .as_mut_ptr()
        .cast::&lt;u8&gt;()
        .add(self.buffer_filled_bytes);
    let end = self.state.input_buffer.as_mut_ptr_range().end.cast::&lt;u8&gt;();
    core::hint::assert_unchecked(start &lt;= end);
    core::ptr::write_bytes(start, 0, end.offset_from_unsigned(start));
    // and run round function
    self.state
        .run_round_function_with_byte_len::&lt;false&gt;(self.buffer_filled_bytes, true);

    core::mem::transmute_copy::&lt;_, [u8; 32]&gt;(self.state.read_state_for_output_ref())
}
</code></pre>

If `start <= end` is not guaranteed to hold and this can be influenced by user input, then `assert_unchecked` must be removed.

Even if it can be proven, add `debug_assert!(start <= end)` to catch invariant breaks during test suite runs or fuzzing.

## Unchecked internal invariant in pairing Miller loop

Deterministic panic if curve configuration constants and precomputation logic diverge.

In BLS12-381:

<pre class="language-rust" data-attributes="filepath context line=120 highlight=[3,5]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/bls12_381/curves/pairing_impl.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">120</span><span style="font-family: monospace !important;">fn multi_miller_loop</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">121</span><span style="font-family: monospace !important;">for i in BitIteratorBE::without_leading_zeros(Config::X).skip(1) {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">122</span><span style="font-family: monospace !important;">    f.square_in_place();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">123</span><span style="font-family: monospace !important;">    Self::ell(&amp;mut f, &amp;ell_coeffs.next().unwrap(), &amp;p.0);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">124</span><span style="font-family: monospace !important;">    if i {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">125</span><span style="font-family: monospace !important;">        Self::ell(&amp;mut f, &amp;ell_coeffs.next().unwrap(), &amp;p.0);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">126</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">127</span><span style="font-family: monospace !important;">}</span></span></code></pre>

In BN254:

<pre class="language-rust" data-attributes="filepath context line=115 highlight=[8,12]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/bn254/curves/pairing_impl.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">115</span><span style="font-family: monospace !important;">fn multi_miller_loop</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">116</span><span style="font-family: monospace !important;">let mut ell_coeffs = q.ell_coeffs.iter();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">117</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">118</span><span style="font-family: monospace !important;">for i in (1..Config::ATE_LOOP_COUNT.len()).rev() {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">119</span><span style="font-family: monospace !important;">    if i != Config::ATE_LOOP_COUNT.len() - 1 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">120</span><span style="font-family: monospace !important;">        f.square_in_place();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">121</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">122</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">123</span><span style="font-family: monospace !important;">    Self::ell(&amp;mut f, ell_coeffs.next().unwrap(), &amp;p.0);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">124</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">125</span><span style="font-family: monospace !important;">    let bit = Config::ATE_LOOP_COUNT[i - 1];</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">126</span><span style="font-family: monospace !important;">    if bit == 1 || bit == -1 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">127</span><span style="font-family: monospace !important;">        Self::ell(&amp;mut f, &amp;ell_coeffs.next().unwrap(), &amp;p.0);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">128</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">129</span><span style="font-family: monospace !important;">}</span></span></code></pre>

Validate `ell_coeffs.len()` against `EXPECTED_ELL_COEFFS` using a debug assertion. This would improve diagnosticability.

<div id="issue-9-QA-thread-safety" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">9. Concurrent usage is not supported</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

The library is designed using global static variables that must be initialized before calling the functions defined by the library:

<pre class="language-rust" data-attributes="filepath context line=82"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/lib.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">82</span><span style="font-family: monospace !important;">pub fn init_lib</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">83</span><span style="font-family: monospace !important;">bn254::fields::init();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">84</span><span style="font-family: monospace !important;">bls12_381::fields::init();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">85</span><span style="font-family: monospace !important;">secp256k1::init();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">86</span><span style="font-family: monospace !important;">bigint_delegation::init();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">87</span><span style="font-family: monospace !important;">secp256r1::init();</span></span></code></pre>



The crates themselves contain multiple global static variables:
<pre class="language-rust" data-attributes="filepath line=7"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/bigint_delegation/u256.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">7</span><span style="font-family: monospace !important;">static mut COPY_PLACE_0: MaybeUninit&lt;U256&gt; = MaybeUninit::uninit();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">8</span><span style="font-family: monospace !important;">static mut COPY_PLACE_1: MaybeUninit&lt;U256&gt; = MaybeUninit::uninit();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">9</span><span style="font-family: monospace !important;">static mut COPY_PLACE_2: MaybeUninit&lt;U256&gt; = MaybeUninit::uninit();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">10</span><span style="font-family: monospace !important;">static mut COPY_PLACE_3: MaybeUninit&lt;U256&gt; = MaybeUninit::uninit();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">11</span><span style="font-family: monospace !important;">static mut ONE: MaybeUninit&lt;U256&gt; = MaybeUninit::uninit();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">12</span><span style="font-family: monospace !important;">static mut ZERO: MaybeUninit&lt;U256&gt; = MaybeUninit::uninit();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">13</span><span style="font-family: monospace !important;">static mut SCRATCH: MaybeUninit&lt;U256&gt; = MaybeUninit::uninit();</span></span></code></pre>

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

<div id="issue-10-QA-variable-timing" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">10. Variable-time scalar operations should not be used with secret data</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #00acc1; border-radius: 4px; padding: 4px 8px; color: #00acc1;">Addressed</span></span>
  </div>
</div>

Several functions involved in core cryptographic operations are not constant-time. If these functions are ever used with secret inputs, attackers may exploit timing variability to gather statistical information and potentially recover secret data.

This observation is relevant for both `secp256k1` and `secp256r1` modules. See the "Code Duplicates" issue for identified instances of functions `mul_shift_384_vartime`.

## Function `mul_shift_384_vartime` and scalar decomposition

The argument-dependent timing of the `mul_shift_384_vartime` function comes from the conditional branch at the end. The variable `l` is derived from the parameter `b`. The `if` statement decides whether to execute `self.add_in_place(&Self::ONE)`based on that bit. This creates data-dependent control flow: one path does the addition, the other does not.

<pre class="language-rust" data-attributes="filepath line=149 highlight=[6]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">src/secp256k1/scalars/scalar64.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">149</span><span style="font-family: monospace !important;">pub(super) fn mul_shift_384_vartime(&amp;mut self, b: &amp;Self)</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">150</span><span style="font-family: monospace !important;">let words = b.0.as_words();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">151</span><span style="font-family: monospace !important;">let l = words[1];</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">152</span><span style="font-family: monospace !important;">self.0 = U256::from_words([words[2], words[3], 0, 0]);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">153</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">154</span><span style="font-family: monospace !important;">if (l &gt;&gt; 63) &amp; 1 != 0 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">155</span><span style="font-family: monospace !important;">    self.add_in_place(&amp;Self::ONE);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">156</span><span style="font-family: monospace !important;">}</span></span></code></pre>

The function is used in the `decompose` function, which implements the scalar decomposition operation, which in its turn affects the scalar multiplication operation.



## Function `pow_vartime`, scalar inversion and affine conversion

Scalar inversion, implemented by the `invert_assign` function, uses the function `pow_vartime` which performs variable-time exponentiation, creating potential timing side-channels:

<pre class="language-rust" data-attributes="filepath context line=86 highlight=[5]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">src/secp256r1/scalar/mod.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">86</span><span style="font-family: monospace !important;">pub fn pow_vartime(&amp;self, exp: &amp;[u64])</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">87</span><span style="font-family: monospace !important;">while j &gt; 0 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">88</span><span style="font-family: monospace !important;">    j -= 1;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">89</span><span style="font-family: monospace !important;">    res.square_assign();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">90</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">91</span><span style="font-family: monospace !important;">    if ((exp[i] &gt;&gt; j) &amp; 1) == 1 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">92</span><span style="font-family: monospace !important;">        res.mul_assign(self);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">93</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">94</span><span style="font-family: monospace !important;">}</span></span></code></pre>

Timing variability affects the functions relying on scalar inversion:
- `invert_assign` calls `pow_vartime` in `src/secp256r1/scalar/mod.rs:68`
- `verify` calls `invert_assign` in `src/secp256r1/verify.rs:25`

<div id="issue-11-QA-normalization-check" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">11. Normalization check is not strong enough</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

In module `secp256k1`, the function `normalize_in_place` is defined, conditioning on values of the `normalized` flag and `magnitude` tracker:

<pre class="language-rust" data-attributes="filepath context line=129"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256k1/field/field_impl.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">129</span><span style="font-family: monospace !important;">pub(super) fn normalize_in_place</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">130</span><span style="font-family: monospace !important;">if !self.normalized || self.magnitude &gt; 1 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">131</span><span style="font-family: monospace !important;">    self.value.normalize_in_place();</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">132</span><span style="font-family: monospace !important;">    self.magnitude = 1;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">133</span><span style="font-family: monospace !important;">    self.normalized = true;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">134</span><span style="font-family: monospace !important;">}</span></span></code></pre>

However, this condition has a minor logical flaw — it assumes that it is potentially possible that `self.normalized` is true but `self.magnitude` is not `1`. When a value is normalized, its magnitude must always be `1`.

## Recommendation

Replace this code with:

<pre class="language-rust"><code>if self.normalized {
    debug_assert!(self.magnitude == 1);
} else {
    debug_assert!(self.magnitude &gt; 1);

    self.value.normalize_in_place();
    self.magnitude = 1;
    self.normalized = true;
}
</code></pre>

<div id="issue-12-unit_tests" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">12. Unit tests issues</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #00acc1; border-radius: 4px; padding: 4px 8px; color: #00acc1;">Addressed</span></span>
  </div>
</div>

### Outdated instructions in the `README`

Running the tests as described in the `README` results in several failures, all throwing error `ZKsync OS bin file missing: /.../zksync_os/for_tests.bin"`.

Examples of such test cases:
- `test_0_gas_limit`
- `fibish_sol`

After inspecting the project’s CI workflows, we have discovered that the script `dump_bin.sh` should be executed with `--type for-tests` flag now, in order to include new test cases into the coverage:
<pre class="language-sh"><code>./dump_bin.sh --type for-tests
</code></pre>

### Some of the unit tests are not executed during CI

The following modules are not executed during regular CI runs:
- `secp256k1::field::field_8x32`
- `secp256k1::scalars::scalar32_delegation`

Additional workflow should be created that runs the test suite in single-threaded mode, filtering only those modules that require this:
<pre class="language-sh"><code>cargo test --release -p crypto secp256k1::field::field_8x32 -- --test-threads=1 --ignored
cargo test --release -p crypto secp256k1::scalars::scalar32_delegation -- --test-threads=1 --ignored
</code></pre>



### Some of the unit tests require larger stack

The test case `secp256k1::field::field_8x32::tests::test_invert` overflows the stack of standard size (8MB):
<pre class="language-sh"><code>$ cargo test -p crypto secp256k1::field::field_8x32::tests::test_invert -- --test-threads=1 --ignored
...
test secp256k1::field::field_8x32::tests::test_invert ... 
thread '&lt;unknown&gt;' has overflowed its stack
fatal runtime error: stack overflow, aborting
</code></pre>

It does succeed in case of bigger stack:
<pre class="language-sh"><code>$ RUST_MIN_STACK=33554432 cargo test -p crypto secp256k1::field::field_8x32::tests::test_invert -- --test-threads=1 --ignored
...
running 1 test
test secp256k1::field::field_8x32::tests::test_invert ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 110 filtered out; finished in 0.57s

     Running tests/secp256k1.rs
</code></pre>

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

<div id="issue-13-QA-excessive-magnitude" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">13. Excessive magnitude increment</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

In the `secp256k1` module, the function `add_int_in_place` overestimates the magnitude increase of a field element:

<pre class="language-rust" data-attributes="filepath context line=114 highlight=[1]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">src/secp256k1/field/field_impl.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">114</span><span style="font-family: monospace !important;">pub(super) fn add_int_in_place(&amp;mut self, rhs: u32)</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">115</span><span style="font-family: monospace !important;">self.magnitude += rhs;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">116</span><span style="font-family: monospace !important;">debug_assert!(self.magnitude &lt;= Self::max_magnitude());</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">117</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">118</span><span style="font-family: monospace !important;">self.value.add_int_in_place(rhs);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">119</span><span style="font-family: monospace !important;">self.normalized = false;</span></span></code></pre>

Here, the field element is incremented by a small scalar `rhs`. However, the correct magnitude increment should be `ceil(rhs / radix)`, where the radix depends on the limb representation: 2^52 for the 5x52 backend, 2^26 for 10x26 backend, etc.

In practice, for the 5x52 representation, the radix is large enough that the increment is always exactly `1`, even for the maximum `u32` input. For the 10x26 representation, the correct increment may range from `1` up to `2^(32 - 26) = 64`, depending on how close `rhs` is to `u32::MAX`.

This issue does not affect correctness or security; it only potentially reduces performance.

## Recommendation

Adjust the magnitude update so that the increment is always `1` in the 5x52 backend, and between `1` and `64` (inclusive) in the 10x26 backend, depending on the value of `rhs`.

<div id="issue-14-QA-magic-numbers" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">14. Magic numbers decrease maintainability</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #00acc1; border-radius: 4px; padding: 4px 8px; color: #00acc1;">Addressed</span></span>
  </div>
</div>

Throughout the codebase, hard-coded number literals without context or a description are used. Using such “magic numbers” goes against best practices as they reduce code readability and maintenance as developers are unable to easily understand their use and may make inconsistent changes across the codebase.

Instances of magic numbers have been discovered in the following locations.

## File `crypto/src/secp256k1/field/field_impl.rs`

Hard-coded literal `8` is used as the limit for magnitudes.

<pre class="language-rust" data-attributes="line=69 highlight=[2,3]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">69</span><span style="font-family: monospace !important;">pub(super) fn mul_in_place(&amp;mut self, rhs: &amp;Self) {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">70</span><span style="font-family: monospace !important;">    debug_assert!(self.magnitude &lt;= 8);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">71</span><span style="font-family: monospace !important;">    debug_assert!(rhs.magnitude &lt;= 8);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">72</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">73</span><span style="font-family: monospace !important;">    self.value.mul_in_place(&amp;rhs.value);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">74</span><span style="font-family: monospace !important;">    self.magnitude = 1;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">75</span><span style="font-family: monospace !important;">    self.normalized = false;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">76</span><span style="font-family: monospace !important;">}</span></span></code></pre>

<pre class="language-rust" data-attributes="line=86 highlight=[2]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">86</span><span style="font-family: monospace !important;">pub(super) fn square_in_place(&amp;mut self) {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">87</span><span style="font-family: monospace !important;">    debug_assert!(self.magnitude &lt;= 8);</span></span></code></pre>

<pre class="language-rust" data-attributes="line=150 highlight=[2,3]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">150</span><span style="font-family: monospace !important;">pub(super) const fn mul(&amp;self, rhs: &amp;Self) -&gt; Self {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">151</span><span style="font-family: monospace !important;">    debug_assert!(self.magnitude &lt;= 8);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">152</span><span style="font-family: monospace !important;">    debug_assert!(rhs.magnitude &lt;= 8);</span></span></code></pre>

<pre class="language-rust" data-attributes="line=166 highlight=[2]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">166</span><span style="font-family: monospace !important;">pub(super) const fn square(&amp;self) -&gt; Self {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">167</span><span style="font-family: monospace !important;">    debug_assert!(self.magnitude &lt;= 8);</span></span></code></pre>



## File `crypto/src/secp256k1/points/affine.rs`

Hard-coded literal `32` is used as the limit for magnitudes, but also it is redundant code since `Self::X_MAGNITUDE_MAX` and `Self::Y_MAGNITUDE_MAX` are defined as a lesser number `4`.

<pre class="language-rust" data-attributes="filepath line=41 highlight=[5,7]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256k1/points/affine.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">41</span><span style="font-family: monospace !important;">pub(crate) const fn assert_verify(&amp;self) {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">42</span><span style="font-family: monospace !important;">    #[cfg(all(debug_assertions, not(feature = "bigint_ops")))]</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">43</span><span style="font-family: monospace !important;">    {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">44</span><span style="font-family: monospace !important;">        debug_assert!(self.x.0.magnitude &lt;= Self::X_MAGNITUDE_MAX);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">45</span><span style="font-family: monospace !important;">        debug_assert!(self.x.0.magnitude &lt;= 32);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">46</span><span style="font-family: monospace !important;">        debug_assert!(self.y.0.magnitude &lt;= Self::Y_MAGNITUDE_MAX);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">47</span><span style="font-family: monospace !important;">        debug_assert!(self.y.0.magnitude &lt;= 32);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">48</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">49</span><span style="font-family: monospace !important;">}</span></span></code></pre>

Same code snippet is also implemented in lines 122-130 in the same file.

## File `crypto/src/secp256k1/points/jacobian.rs`

Similarly, same literal `32` is hard-coded and also has no effect because of stronger equations involving `X_MAGNITUDE_MAX`, `Y_MAGNITUDE_MAX` and `Z_MAGNITUDE_MAX`:

<pre class="language-rust" data-attributes="filepath line=28 highlight=[5,7,9]"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256k1/points/jacobian.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">28</span><span style="font-family: monospace !important;">pub(super) const fn assert_verify(&amp;self) {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">29</span><span style="font-family: monospace !important;">    #[cfg(all(debug_assertions, not(feature = "bigint_ops")))]</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">30</span><span style="font-family: monospace !important;">    {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">31</span><span style="font-family: monospace !important;">        debug_assert!(self.x.0.magnitude &lt;= Self::X_MAGNITUDE_MAX);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">32</span><span style="font-family: monospace !important;">        debug_assert!(self.x.0.magnitude &lt;= 32);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">33</span><span style="font-family: monospace !important;">        debug_assert!(self.y.0.magnitude &lt;= Self::Y_MAGNITUDE_MAX);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">34</span><span style="font-family: monospace !important;">        debug_assert!(self.y.0.magnitude &lt;= 32);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">35</span><span style="font-family: monospace !important;">        debug_assert!(self.z.0.magnitude &lt;= Self::Z_MAGNITUDE_MAX);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">36</span><span style="font-family: monospace !important;">        debug_assert!(self.z.0.magnitude &lt;= 32);</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">37</span><span style="font-family: monospace !important;">    }</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">38</span><span style="font-family: monospace !important;">}</span></span></code></pre>

Same code snippet is also implemented in lines 195-205 in the same file.

## Impact

Using magic numbers leads to maintainability issues and confusion, especially when they need to be updated or dynamically configured.

## Recommendation

Declare named constants or implement configuration values instead of directly embedding literals to improve clarity and reduce potential errors in future development.

<div id="issue-15-QA-code-duplicates" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">15. Code duplicates</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

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

<div id="issue-16-QA-redundant-code" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">16. Redundant `else` branch</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

In the function `mul_assign`, the "schoolbook" algorithm is implemented. Right after iterating `j` up to `4`, the `if` splits execution depending on the value of `i + j`: 

<pre class="language-rust" data-attributes="filepath context line=101"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; margin-right: 1rem !important; box-sizing: border-box !important;"></span><span style="font-family: monospace !important;">crypto/src/secp256r1/scalar/scalar64.rs</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">101</span><span style="font-family: monospace !important;">pub(super) fn mul_assign</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">102</span><span style="font-family: monospace !important;">while j &lt; 4 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">103</span><span style="font-family: monospace !important;">    let k = i + j;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">104</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">105</span><span style="font-family: monospace !important;">    ...</span></span></code></pre>

<pre class="language-rust" data-attributes="line=114"><code><span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">114</span><span style="font-family: monospace !important;">    j += 1;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">115</span><span style="font-family: monospace !important;">}</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">116</span><span style="font-family: monospace !important;"> </span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">117</span><span style="font-family: monospace !important;">if i + j &gt;= 4 {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">118</span><span style="font-family: monospace !important;">    hi[i + j - 4] = carry;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">119</span><span style="font-family: monospace !important;">} else {</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">120</span><span style="font-family: monospace !important;">    lo[i + j] = carry;</span></span>
<span style="display: block !important;"><span style="display: inline-block !important; width: 4em !important; color: #6b7280 !important; text-align: right !important; margin-right: 1rem !important; user-select: none !important; font-family: monospace !important; box-sizing: border-box !important;">121</span><span style="font-family: monospace !important;">}</span></span></code></pre>

If `i + j < 4`, the `else` branch would execute. However, this condition is never satisfied because after the `while` loop, `j` is guaranteed to equal `4`, and `i` is always greater than or equal to zero.

# Appendix

# Appendix

## Fuzzing parameters

The fuzzing was executed with the following parameters and targets:

- Duration
  - <strong>48 hours</strong>
- Machine:
  - CPU: Virtual AMD EPYC-Milan
    - <strong>2 GHz, 48 Threads</strong>
    - 32 MiB L3 Cache
  - RAM: 184 GiB

### Fuzzing targets

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;"></th>
  <th style="border: 1px solid white; padding: 8px;"></th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`blake2s`</td>
  <td style="border: 1px solid white; padding: 8px;">`bn254_ecadd`</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`bn254_ecmul`</td>
  <td style="border: 1px solid white; padding: 8px;">`bn254_pairing_check`</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`ecrecover`</td>
  <td style="border: 1px solid white; padding: 8px;">`p256_verify`</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`keccak256`</td>
  <td style="border: 1px solid white; padding: 8px;">`sha256`</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`ripemd160`</td>
  <td style="border: 1px solid white; padding: 8px;">`modexp`</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`modexp_delegation`</td>
  <td style="border: 1px solid white; padding: 8px;">`precompiles_ecadd`</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`precompiles_ecmul`</td>
  <td style="border: 1px solid white; padding: 8px;">`precompiles_ecpairing`</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`precompiles_ecrecover`</td>
  <td style="border: 1px solid white; padding: 8px;">`precompiles_p256`</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`precompiles_sha256`</td>
  <td style="border: 1px solid white; padding: 8px;">`precompiles_ripemd160`</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`precompiles_modexp`</td>
  <td style="border: 1px solid white; padding: 8px;">`precompiles_modexplen`</td>
</tr>
</tbody>
</table>



## Fuzzing coverage

Generated using the command `./fuzz.sh coverage`.

### Crate `callable_oracles`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">File / Module</th>
  <th style="border: 1px solid white; padding: 8px;">Lines Covered</th>
  <th style="border: 1px solid white; padding: 8px;">Line Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Functions Covered</th>
  <th style="border: 1px solid white; padding: 8px;">Function Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange"><strong>callable_oracles</strong></span></td>
  <td style="border: 1px solid white; padding: 8px;">114 / 650</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">17.5%</span></td>
  <td style="border: 1px solid white; padding: 8px;">13 / 78</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">16.7%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/arithmetic/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">53 / 54</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">98.1%</span></td>
  <td style="border: 1px solid white; padding: 8px;">7 / 17</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">41.2%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/common.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 247</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 34</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/compute.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 75</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 5</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/evaluate.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 28</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 2</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/verify.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 119</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 5</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/lib.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 11</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 2</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">src/utils/evaluate.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">42 / 96</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">43.8%</span></td>
  <td style="border: 1px solid white; padding: 8px;">2 / 9</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">22.2%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/utils/usize_slice_iterator.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">19 / 20</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">95.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">4 / 4</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange"><b>Total</b></span></td>
  <td style="border: 1px solid white; padding: 8px;">114 / 650</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">17.5%</span></td>
  <td style="border: 1px solid white; padding: 8px;">13 / 78</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">16.7%</span></td>
</tr>
</tbody>
</table>

### Crate `crypto`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">File / Module</th>
  <th style="border: 1px solid white; padding: 8px;">Lines Covered</th>
  <th style="border: 1px solid white; padding: 8px;">Line Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Functions Covered</th>
  <th style="border: 1px solid white; padding: 8px;">Function Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/ark_ff_delegation</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 1080</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 210</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/biginteger/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 387</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 74</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/biginteger/const_helpers.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 117</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 19</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/fp/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 414</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 91</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/fp/montgomery_backend.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 162</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 26</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bigint_delegation/delegation.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 82</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 16</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bigint_delegation/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 4</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 1</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bigint_delegation/u256.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 209</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 29</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bigint_delegation/u512.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 189</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 14</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/blake2s/naive.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">20 / 30</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">66.7%</span></td>
  <td style="border: 1px solid white; padding: 8px;">19 / 29</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">65.5%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red"><strong>crypto/bls12_381</strong></span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 591</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 65</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bls12_381/curves/g1.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 92</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 14</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bls12_381/curves/g2.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 105</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 11</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bls12_381/curves/pairing_impl.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 209</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 21</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bls12_381/curves/util.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 166</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 14</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bls12_381/fields/fq2.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 13</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 4</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bls12_381/fields/fq6.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 6</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 1</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red"><strong>crypto/bn254</strong></span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 306</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 36</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bn254/curves/g1.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 31</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 7</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bn254/curves/g2.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 26</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 5</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bn254/curves/pairing_impl.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 235</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 22</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bn254/fields/fq2.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 3</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 1</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bn254/fields/fq6.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 11</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 1</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bn254/glv_decomposition.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 93</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 12</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bn254/lib.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 2</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 1</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/bn254/raw_delegation_interface.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 18</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 2</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B"><strong>crypto/secp256k1</strong></span></td>
  <td style="border: 1px solid white; padding: 8px;">1663 / 2105</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">79.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">136 / 188</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">72.3%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256k1/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 34</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 2</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256k1/context.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 34</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 4</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/secp256k1/field/field_5x52.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">332 / 384</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">86.5%</span></td>
  <td style="border: 1px solid white; padding: 8px;">20 / 27</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">74.1%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">src/secp256k1/field/field_impl.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">88 / 144</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">61.1%</span></td>
  <td style="border: 1px solid white; padding: 8px;">17 / 27</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">63.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/secp256k1/field/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">114 / 169</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">67.5%</span></td>
  <td style="border: 1px solid white; padding: 8px;">24 / 35</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">68.6%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/secp256k1/field/mod_inv64.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">321 / 336</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">95.5%</span></td>
  <td style="border: 1px solid white; padding: 8px;">16 / 17</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">94.1%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/secp256k1/points/affine.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">80 / 111</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">72.1%</span></td>
  <td style="border: 1px solid white; padding: 8px;">10 / 15</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">66.7%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">src/secp256k1/points/jacobian.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">170 / 300</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">56.7%</span></td>
  <td style="border: 1px solid white; padding: 8px;">6 / 12</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">50.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/secp256k1/points/storage.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">7 / 7</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">1 / 1</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/secp256k1/recover.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">218 / 229</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">95.2%</span></td>
  <td style="border: 1px solid white; padding: 8px;">11 / 11</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/secp256k1/scalars/invert.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">89 / 89</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">2 / 2</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/secp256k1/scalars/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">33 / 42</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">78.6%</span></td>
  <td style="border: 1px solid white; padding: 8px;">10 / 13</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">76.9%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/secp256k1/scalars/scalar64.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">211 / 226</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">93.4%</span></td>
  <td style="border: 1px solid white; padding: 8px;">19 / 22</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">86.4%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red"><strong>crypto/secp256r1</strong></span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 904</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 92</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/context.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 20</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 3</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/field/fe64.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 173</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 28</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/field/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 68</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 6</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 9</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 1</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/points/affine.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 42</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 6</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/points/jacobian.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 203</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 10</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/points/storage.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 7</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 1</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/scalar/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 56</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 8</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/scalar/scalar64.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 198</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 18</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/u64_arithmatic.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 12</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 3</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/verify.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 65</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 4</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/secp256r1/wnaf.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 51</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">0 / 4</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/sha3/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;">20 / 20</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;">9 / 23</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">39.1%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange"><b>Total</b></span></td>
  <td style="border: 1px solid white; padding: 8px;">1703 / 5633</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">30.2%</span></td>
  <td style="border: 1px solid white; padding: 8px;">164 / 718</td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">22.8%</span></td>
</tr>
</tbody>
</table>

## Unit tests coverage

The unit test suite has been executed with the `e2e_proving` feature disabled since enabling it caused the suite to crash (see the description of this issue in the main part of the report). To collect coverage data, `llvm-cov` has been used.

Since the test suite of the `crypto` crate is built in "property testing" fashion, each new pass can result in slightly different coverage. This means that to compute realistic coverage, it is necessary to aggregate results from multiple runs. Multiple `llvm-cov` passes have been performed in parallel, on same machine used for fuzzing (48 cores). Each pass has been performed in single-threaded mode since some of the test cases require it:
<pre class="language-sh"><code># first, we ensure that all test code is compiled
cargo llvm-cov -p crypto --no-report -- --list

# bigger stack is needed for `field_8x32::tests::test_invert`
export RUST_MIN_STACK=33554432

seq 1 96 \
| xargs -P48 -n1 sh -c '
  cargo llvm-cov -p crypto --no-report -- --test-threads=1 --ignored || true
'

cargo llvm-cov report -p crypto --html
</code></pre>

Following are the coverage statistics produced by the commands above.

### Crate `callable_oracles`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Function Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Line Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Region Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/arithmetic/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">41.2%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">98.1%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/common.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/compute.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/evaluate.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/verify.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">src/hash_to_prime/lib.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">src/utils/evaluate.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">22.2%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">43.8%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">src/utils/usize_slice_iterator.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">95.0%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange"><b>Total</b></span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">16.7%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">17.5%</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.0%</span></td>
</tr>
</tbody>
</table>

### Crate `crypto`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Function Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Line Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Region Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">crypto/src/ark_ff_delegation/biginteger/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">29.73% (22/74)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">23.92% (94/393)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">24.40% (133/545)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">crypto/src/ark_ff_delegation/const_helpers.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">40.00% (10/25)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">34.94% (58/166)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">29.02% (74/255)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">crypto/src/ark_ff_delegation/fp/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">54.95% (50/91)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">52.87% (221/418)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">50.98% (311/610)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">crypto/src/ark_ff_delegation/fp/montgomery_backend.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">65.52% (19/29)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">54.45% (104/191)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">60.43% (168/278)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bigint_delegation/delegation.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (15/15)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (83/83)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (169/169)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bigint_delegation/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (1/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (4/4)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (4/4)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bigint_delegation/u256.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">87.88% (29/33)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">89.87% (213/237)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">89.04% (406/456)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bigint_delegation/u512.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (14/14)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">98.43% (188/191)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">98.70% (456/462)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/blake2s/naive.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/9)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/30)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/43)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/blake2s/test.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/2)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/18)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/26)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">crypto/src/bls12_381/curves/g1.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">25.00% (4/16)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">20.16% (26/129)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">12.12% (24/198)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bls12_381/curves/g1_swu_iso.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (1/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (6/6)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (12/12)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/bls12_381/curves/g2.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/10)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/105)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/176)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bls12_381/curves/g2_swu_iso.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (1/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (6/6)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (12/12)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bls12_381/curves/pairing_impl.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">66.67% (14/21)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">80.56% (174/216)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">85.32% (308/361)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/bls12_381/curves/util.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/10)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/166)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/320)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bls12_381/fields/fq.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (26/26)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">98.01% (246/251)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">93.78% (422/450)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">crypto/src/bls12_381/fields/fq2.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">50.00% (2/4)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">46.15% (6/13)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">42.86% (6/14)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bls12_381/fields/fq6.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (1/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (6/6)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (7/7)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bls12_381/fields/fr.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">78.95% (15/19)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">84.29% (118/140)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">85.61% (113/132)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bls12_381/fields/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (1/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (4/4)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (4/4)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">crypto/src/bn254/curves/g1.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">40.00% (4/10)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">29.85% (20/67)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">25.51% (25/98)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/bn254/curves/g2.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/5)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/26)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/41)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bn254/curves/pairing_impl.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">68.18% (15/22)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">81.82% (198/242)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">87.38% (374/428)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bn254/fields/fq.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (19/19)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">97.06% (198/204)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">92.41% (353/382)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bn254/fields/fq2.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (1/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (3/3)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (3/3)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/bn254/fields/fq6.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (1/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (11/11)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (17/17)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/glv_decomposition.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">91.67% (11/12)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">96.35% (132/137)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">94.52% (207/219)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/lib.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (1/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (9/9)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">100.00% (6/6)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/raw_delegation_interface.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/2)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/18)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/17)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/context.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/4)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/34)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/45)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/field/field_10x26.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/36)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/624)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/876)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/field/field_5x52.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/37)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/448)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/562)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">crypto/src/secp256k1/field/field_8x32.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">70.00% (21/30)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">82.42% (211/256)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#6BD36B">77.27% (170/220)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/field/field_impl.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/30)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/152)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/203)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/field/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/52)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/347)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/323)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/field/mod_inv32.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/22)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/353)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/533)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/field/mod_inv64.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/18)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/348)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/550)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">25.00% (1/4)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">9.76% (4/41)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">5.97% (4/67)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/points/affine.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/21)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/139)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/186)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/points/jacobian.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/20)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/384)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/660)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/points/storage.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/7)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/5)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/recover.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/24)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/411)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/543)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/scalars/invert.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/3)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/107)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/136)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/scalars/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/30)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/137)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/148)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">crypto/src/secp256k1/scalars/scalar32_delegation.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">63.16% (24/38)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">61.03% (166/272)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">39.93% (119/298)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256k1/scalars/scalar64.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/28)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/247)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/746)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/context.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/3)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/20)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/32)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/field/fe32_delegation.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">4.35% (1/23)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">7.00% (7/100)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">8.33% (10/120)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/field/fe64.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/30)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/185)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/480)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/field/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/15)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/256)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/177)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">crypto/src/secp256r1/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:#4DA3FF">50.00% (1/2)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">30.77% (4/13)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">21.05% (4/19)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/points/affine.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/6)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/42)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/62)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/points/jacobian.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/14)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/240)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/447)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/points/storage.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/1)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/7)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/3)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/scalar/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/10)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/80)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/88)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/scalar/scalar64.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/21)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/209)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/564)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">crypto/src/secp256r1/scalar/scalar_delegation.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">5.56% (1/18)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">9.59% (7/73)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange">11.76% (10/85)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/u64_arithmatic.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/3)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/12)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/18)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/verify.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/6)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/120)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/208)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/secp256r1/wnaf.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/4)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/51)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/69)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">crypto/src/sha3/mod.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/5)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/20)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/34)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">zksync_os_runner/src/lib.rs</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/3)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/37)</span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:red">0.00% (0/36)</span></td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange"><b>Total</b></span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange"><b>31.41% (326/1038)</b></span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange"><b>27.28% (2527/9262)</b></span></td>
  <td style="border: 1px solid white; padding: 8px;"><span style="color:orange"><b>27.51% (3931/14288)</b></span></td>
</tr>
</tbody>
</table>
