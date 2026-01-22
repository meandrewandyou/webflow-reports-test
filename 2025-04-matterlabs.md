---
title: Boojum OS
client: MatterLabs
created_at: '2025-04-30'
updated_at: '2025-05-30'
draft: false
---
# About This Audit

## Synopsis

During April 2025, Matter Labs engaged Taran Space to perform a security review of Boojum OS, which is an evolution of ZKsync project aimed at aggregating multiple distinct execution environments within a single network. The review was delivered remotely by 2 auditors with a total effort of 4 weeks for the initial review.

Matter Labs is an engineering team passionate about liberty, blockchain, and math; also known as the inventors of ZKsync: the leading cluster of ZK rollups on Ethereum.

The objectives of the audit are as follows:

1. Determine the correct functioning of the protocol, in accordance with the project specification.
2. Determine possible vulnerabilities, which could be exploited by an attacker.
3. Analyze whether best practices have been applied during development.
4. Make recommendations to improve code safety and readability.

As a reference list of vulnerabilities, this review has included analysis of:

- General code correctness and memory safety
- Resilience to denial of service attacks
- Correctness of transaction validation and processing
- Adhering to Rust security best practices
- Code correctness in unsafe blocks
- Integer overflow or underflow
- Correctness of usage cryptographic primitives
- Security of design

## Scope of the Audit

The audit has been performed on the following target:

- Repository: [zk_ee](https://github.com/matter-labs/zk_ee)
- Commit: [1abfad12dcf9eb1fc71354b3eabf09e89c543010](https://github.com/matter-labs/zk_ee/tree/1abfad12dcf9eb1fc71354b3eabf09e89c543010)

### High-level Scope of the Audit

- <strong>Bootloader</strong> (crate `basic_bootloader`): Only the EOA account model is included in the scope. The modules related to account abstraction are not included in the detailed scope. This means that the `AA_ENABLED` configuration flag can be assumed to be always `false`. The audit includes the bootloader with both the compilation flag `evm-compatibility` enabled and disabled.
- <strong>System Hooks</strong> (crate `system_hooks` and corresponding implementations): precompiles hooks and their underlying implementations in the `basic_system/src/system_functions` and the `crypto` directories.
- <strong>Basic system implementation and system interface</strong> (crate `basic_system`): the focus is on the basic system implementation. To make sense of it, the system interface of some common structures are also included, which are defined in the `zk_ee` and the `storage_models` directories.

The concrete system implementations (`forward_system` and `proof_running_system`) are out of scope themselves, but both running environments and compilation targets should be considered.

Additionally, the crate [`supporting_crates/modexp`](https://github.com/matter-labs/zk_ee/tree/main/supporting_crates/modexp), used by the `crypto` crate, is included into the scope. This is a fork of the [`modexp` crate by Aurora Labs](https://github.com/aurora-is-near/aurora-engine) with modifications to pass the allocator around. As such, only the diff between the upstream and our version is included in the audit.

The following areas are outside the scope of this engagement:

- L1 and L2 smart contracts
- Front-end components
- Infrastructure related to the project
- Key custody

### Detailed Scope

The Rust files included in the scope are:

```text
basic_bootloader
└── src
    ├── bootloader
    │   ├── account_models
    │   │   ├── abstract_account.rs
    │   │   ├── eoa.rs
    │   │   └── mod.rs
    │   ├── constants.rs
    │   ├── errors.rs
    │   ├── gas_helpers.rs
    │   ├── mod.rs
    │   ├── process_transaction.rs
    │   ├── result_keeper.rs
    │   ├── run_single_interaction.rs
    │   ├── runner.rs
    │   ├── supported_ees.rs
    │   └── transaction
    │       ├── mod.rs
    │       ├── rlp.rs
    │       └── u256be_ptr.rs
    └── lib.rs

system_hooks
└── src
    ├── addresses_constants.rs
    ├── l1_messenger.rs
    ├── lib.rs
    ├── mock_precompiles.rs
    └── precompiles.rs

crypto
├── src
│   ├── bigint_riscv.rs
│   ├── blake2s
│   │   ├── delegated.rs
│   │   ├── delegated_extended.rs
│   │   ├── mod.rs
│   │   ├── naive.rs
│   │   ├── test.rs
│   │   ├── test_program
│   │   │   └── src
│   │   │       └── main.rs
│   │   └── unrolled_delegated.rs
│   ├── bls12_381
│   │   ├── curves
│   │   │   ├── g1.rs
│   │   │   ├── g1_swu_iso.rs
│   │   │   ├── g2.rs
│   │   │   ├── g2_swu_iso.rs
│   │   │   ├── mod.rs
│   │   │   ├── pairing_impl.rs
│   │   │   └── util.rs
│   │   ├── fields
│   │   │   ├── fq.rs
│   │   │   ├── fq12.rs
│   │   │   ├── fq2.rs
│   │   │   ├── fq6.rs
│   │   │   ├── fq_alt.rs
│   │   │   ├── fq_ext.rs
│   │   │   └── mod.rs
│   │   └── mod.rs
│   ├── bn254
│   │   ├── curves
│   │   │   ├── g1.rs
│   │   │   ├── g2.rs
│   │   │   ├── mod.rs
│   │   │   └── pairing_impl.rs
│   │   ├── fields
│   │   │   ├── fq.rs
│   │   │   ├── fq12.rs
│   │   │   ├── fq2.rs
│   │   │   ├── fq6.rs
│   │   │   ├── fq_alt.rs
│   │   │   ├── fq_alt_full
│   │   │   │   ├── impls copy.rs
│   │   │   │   ├── impls.rs
│   │   │   │   └── mod.rs
│   │   │   ├── fr.rs
│   │   │   └── mod.rs
│   │   └── mod.rs
│   ├── k256
│   │   └── mod.rs
│   ├── lib.rs
│   ├── modexp
│   │   └── mod.rs
│   ├── p256
│   │   └── mod.rs
│   ├── ripemd160
│   │   └── mod.rs
│   ├── secp256k1
│   │   ├── context.rs
│   │   ├── field
│   │   │   ├── field_10x26.rs
│   │   │   ├── field_5x52.rs
│   │   │   ├── field_8x32.rs
│   │   │   ├── field_impl.rs
│   │   │   ├── mod.rs
│   │   │   ├── mod_inv32.rs
│   │   │   └── mod_inv64.rs
│   │   ├── mod.rs
│   │   ├── points
│   │   │   ├── affine.rs
│   │   │   ├── jacobian.rs
│   │   │   ├── mod.rs
│   │   │   └── storage.rs
│   │   ├── recover.rs
│   │   ├── scalars
│   │   │   ├── invert.rs
│   │   │   ├── mod.rs
│   │   │   ├── scalar32.rs
│   │   │   ├── scalar32_delegation.rs
│   │   │   └── scalar64.rs
│   │   └── test_vectors.rs
│   ├── sha256
│   │   └── mod.rs
│   └── sha3
│       └── mod.rs
└── tests
    └── secp256k1.rs

basic_system
└── src
    ├── cost_constants.rs
    ├── lib.rs
    ├── system_functions
    │   ├── bn254_ecadd.rs
    │   ├── bn254_ecmul.rs
    │   ├── bn254_pairing_check.rs
    │   ├── ecrecover.rs
    │   ├── keccak256.rs
    │   ├── mod.rs
    │   ├── modexp.rs
    │   ├── p256_verify.rs
    │   ├── ripemd160.rs
    │   └── sha256.rs
    └── system_implementation
        ├── io
        │   ├── account_cache.rs
        │   ├── account_cache_entry.rs
        │   ├── history_map.rs
        │   ├── mod.rs
        │   ├── preimage_cache.rs
        │   ├── rollbackable_stack.rs
        │   ├── simple_growable_storage.rs
        │   └── storage_cache.rs
        ├── memory
        │   ├── basic_memory.rs
        │   └── mod.rs
        ├── mod.rs
        └── system
            ├── basic_metadata.rs
            ├── io_subsystem.rs
            └── mod.rs

supporting_crates
└── modexp
    └── src
        ├── arith.rs
        ├── lib.rs
        └── mpnat.rs
```

## Audit Team

- Lead Auditor: [Kirill Taran](https://www.linkedin.com/in/kirill-taran-98103a7b/)
- Auditor: [Tarek Elsayed](https://www.linkedin.com/in/tareknasser360/)

# Summary of the Audit

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Component</th>
  <th style="border: 1px solid white; padding: 8px;">Status</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`basic_bootloader`</td>
  <td style="border: 1px solid white; padding: 8px;">Reviewed by both auditors</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`basic_system`</td>
  <td style="border: 1px solid white; padding: 8px;">Time-boxed review by Kirill Taran</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`supporting_crates/modexp`</td>
  <td style="border: 1px solid white; padding: 8px;">Reviewed by Tarek Elsayed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`zk_ee`</td>
  <td style="border: 1px solid white; padding: 8px;">Partially reviewed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">`system_hooks`</td>
  <td style="border: 1px solid white; padding: 8px;">Partially reviewed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">Fuzzing and unit tests</td>
  <td style="border: 1px solid white; padding: 8px;">Executed</td>
</tr>
</tbody>
</table>

## Review Limitations

During review of the `zk_ee` module, only the code referred from `basic_bootloader` and `basic_system` has been reviewed.

Review of the `basic_system` module has been performed with minor limitations, these parts have been reviewed superficially:

- Module `system_functions`
- File `cost_constants.rs`
- File `simple_growable_storage.rs`
- File `history_map.rs`

## About the `modexp` Module

The differences between [implementation by MatterLabs](https://github.com/matter-labs/zk_ee/tree/main/supporting_crates/modexp) and the original [implementation by AuroraLabs](https://github.com/aurora-is-near/aurora-engine/tree/develop/engine-modexp/src) have been reviewed. The changes introduced in the fork are minimal. The most notable modification is that the crate is now fully parameterized over a custom `Allocator`. This change allows memory allocation to be controlled externally. All vector operations throughout the crate have been updated to explicitly use the provided allocator.

The changes made in this fork are safe in isolation. They do not introduce any direct security concerns. The modifications only make the crate generic over allocation strategies. The most of the risk lies in the allocator implementation itself, which could introduce subtle issues if it behaves incorrectly or fails to meet ZK requirements.

The corresponding fuzz test has been executed and has not revealed any issues.

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
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-1-2-maj2" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">1. Positive-value transfers to the kernel space are not rejected</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #f97316;">High</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-2-2-maj1" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">2. Sender is charged for positive-value self-transfers via CALLCODE</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #f97316;">High</td>
  <td style="border: 1px solid white; padding: 8px; color: #ef4444;">Invalid</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-3-2-min3" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">3. Unimplemented functionality</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Medium</td>
  <td style="border: 1px solid white; padding: 8px; color: #00acc1;">Addressed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-4-2-min1" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">4. Incorrect pubdata size calculation</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Medium</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-5-2-inf1" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">5. AccountProperties difference can be encoded using one byte less</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-6-2-crit1" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">6. Incorrect implementation of `UsizeSerializable` for `BasicBlockMetadataFromOracle`</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-7-2-inf3" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">7. Insufficient overflow protection</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #00acc1;">Addressed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-8-1-inf2" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">8. Missing validations</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-9-2-inf2" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">9. Potential for optimization</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #00acc1;">Addressed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-10-1-min1" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">10. Using EOA account model as a fallback for smart contracts</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #ef4444;">Invalid</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-11-2-min2" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">11. Inconsistent flushing of the caches</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-12-1-inf1" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">12. Inaccuracy in fees calculation</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #84cc16;">Low</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-13-2-qa5" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">13. Code duplication</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-14-1-fuzz" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">14. Crashes in fuzzing test suite</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-15-2-qa1" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">15. Equalities with boolean constants</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-16-2-qa3" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">16. Error handling issues</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #00acc1;">Addressed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-17-2-qa8" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">17. Large function definitions</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #10b981;">Fixed</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-18-2-qa7" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">18. Magic numbers</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-19-1-qa1" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">19. Misleading names and messages</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-20-1-qa9" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">20. Outstanding TODO comments</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-21-1-qa6" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">21. Redundant parameters</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-22-1-qa2" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">22. Some conditions for branching are always `true`</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-23-2-qa2" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">23. Suboptimal types</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-24-1-qa11" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">24. Unit test suite fails in `e2e_proving` mode</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-25-2-qa6" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">25. Unused code</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-26-1-inf3" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">26. Unused error messages</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;"><a href="#issue-27-1-qa3" style="color: white; text-decoration: none; cursor: pointer;" onmouseover="this.style.textDecoration='underline'" onmouseout="this.style.textDecoration='none'">27. Variable can be declared in a more specific scope</a></td>
  <td style="border: 1px solid white; padding: 8px; color: #22c55e;">QA</td>
  <td style="border: 1px solid white; padding: 8px; color: #fbc02d;">Notified</td>
</tr>
</tbody>
</table>


<div id="issue-1-2-maj2" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">1. Positive-value transfers to the kernel space are not rejected</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #f97316; border-radius: 4px; padding: 4px 8px; color: #f97316;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

```rust filepath line=555 highlight=[6]
basic_bootloader/src/bootloader/runner.rs
// Transfers are forbidden to kernel space unless for bootloader or
// explicitly allowed by a feature
let transfer_allowed =
    callee == BOOTLOADER_FORMAL_ADDRESS || cfg!(feature = "transfers_to_kernel_space");
let nominal_token_value = if transfer_allowed {
    call_values.nominal_token_value
} else {
    U256::ZERO
};
```

```rust line=573 highlight=[7]
} = match run_call_preparation(
    callstack,
    system,
    ee_type,
    desired_resources_to_pass,
    &call_values,
    &call_parameters,
    should_finish_callee_frame_on_error,
) {
```

```rust filepath context line=720
basic_bootloader/src/bootloader/runner.rs
fn run_call_preparation<
} = match system.prepare_for_call(
```

```rust filepath context line=585
basic_system/src/system_implementation/system/mod.rs
fn prepare_for_call(
match SystemExt::io(self).transfer_nominal_token_value(
```

The function `handle_requested_external_call_to_special_address_space` is intended to prevent positive-value transfers to kernel space unless the configuration flag `transfers_to_kernel_space` is engaged or the `callee` is explicitly set to `BOOTLOADER_FORMAL_ADDRESS`.

However, the actual callpath to the function `transfer_nominal_token_value` is still performed with the unchanged value `call_values.nominal_token_value`. While the conditions for the transfer are evaluated and variable `nominal_token_value` is correctly set to zero when the conditions are not met, the variable is not actually utilized with the exception at the very end of the handler, after the token transfer is already performed.

```rust filepath line=650
basic_bootloader/src/bootloader/runner.rs
let call_result =
    if nominal_token_value != U256::ZERO && callee != BOOTLOADER_FORMAL_ADDRESS {
        CallResult::Failed { return_values }
    } else {
```

Additionally, the condition to fail the call and return `CallResult::Failed` is incorrect:

- Condition `nominal_token_value != U256::ZERO` can hold true only when `call_values.nominal_token_value` is not zero and also `transfer_allowed` is true, which requires that either `callee == BOOTLOADER_FORMAL_ADDRESS` or `cfg!(feature = "transfers_to_kernel_space")` holds true
- Given the second condition `callee != BOOTLOADER_FORMAL_ADDRESS`, only the `cfg!(feature = "transfers_to_kernel_space")` condition can be satisfied

The conjunction of the two conditions is equivalent to `call_values.nominal_token_value != U256::ZERO && callee != BOOTLOADER_FORMAL_ADDRESS && cfg!(feature = "transfers_to_kernel_space")`, which contradicts the intention behind the function.

<div id="issue-2-2-maj1" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">2. Sender is charged for positive-value self-transfers via CALLCODE</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #f97316; border-radius: 4px; padding: 4px 8px; color: #f97316;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #ef4444; border-radius: 4px; padding: 4px 8px; color: #ef4444;">Invalid</span></span>
  </div>
</div>

```rust filepath line=523
basic_system/src/system_implementation/system/mod.rs
let stipend = if !is_delegate && !nominal_token_value.is_zero() {
```

```rust line=527
resources
	.spendable_part_mut()
	.try_spend_or_floor_self(&positive_value_cost);
```

The opcode `CALLCODE` is allowed to have non-zero `value` parameter, even in static context. However, this is not considered to be a state change since the transfer is to the sender itself. This edge case is not handled by the function `prepare_for_call` currently.

<div id="issue-3-2-min3" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">3. Unimplemented functionality</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #00acc1; border-radius: 4px; padding: 4px 8px; color: #00acc1;">Addressed</span></span>
  </div>
</div>

```rust filepath context line=233 highlight=[1]
zk_ee/src/system/system_trait/system.rs
fn prepare_for_call(
    is_call_to_precompile: bool,
```

```rust filepath context line=474 highlight=[1]
basic_system/src/system_implementation/system/mod.rs
fn prepare_for_call(
    is_call_to_precomile: bool, // typo here
```

```rust filepath line=726
basic_bootloader/src/bootloader/runner.rs
} = match system.prepare_for_call(
    false,
```

Calls to precompiles are not implemented yet, the `is_call_to_precompile` flag is always set to `false`.

```rust filepath context line=260
basic_system/src/system_implementation/system/io_subsystem.rs
fn emit_l1_message(
// TODO: we should charge gas for computation needed to emit: at least to hash log(L2_TO_L1_LOG_SERIALIZE_SIZE) and build tree(~32)
```

Sending L1 messages does not incur charges for the computation spent.

<div id="issue-4-2-min1" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">4. Incorrect pubdata size calculation</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">High</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

```rust filepath line=306
basic_system/src/system_implementation/io/account_cache_entry.rs
AddressDataDiff::Full => 118,
```

In the function `get_encoded_size`, called by the function `net_pubdata_used`, the size of full update is defined as `118`. However, this must coincide with the size of the `AccountProperties` structure which is `124`.

As a consequence, the transaction sender is charged a little bit less than necessary. This is a minor DoS vector.

<div id="issue-5-2-inf1" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">5. AccountProperties difference can be encoded using one byte less</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath highlight=[5] line=308
basic_system/src/system_implementation/io/account_cache_entry.rs
AddressDataDiff::Partial(x) => {
    match (&x.nonce, &x.balance) {
        (None, Some(x)) => x.bytes_touched as usize + 1,
        (Some(x), None) => x.bytes_touched as usize + 1,
        (Some(x), Some(y)) => 2 + x.bytes_touched as usize + y.bytes_touched as usize,
```

It is possible to encode the `AddressDataDiff::Partial` using only one extra byte for all cases. The extra byte should be used to label which of 5 cases is being handled:

- Only nonce has been modified (increased)
- Only balance has been modified (increased)
- Only balance has been modified (decreased)
- Both nonce and balance have been increased
- Nonce has been increased, and balance has been decreased

<div id="issue-6-2-crit1" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">6. Incorrect implementation of `UsizeSerializable` for `BasicBlockMetadataFromOracle`</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

```rust filepath line=111 highlight=[2, 14,16, 28]
basic_system/src/system_implementation/system/basic_metadata.rs
impl UsizeSerializable for BasicBlockMetadataFromOracle {
const USIZE_LEN: usize = <U256 as UsizeSerializable>::USIZE_LEN * (3 + 256)
      + <u64 as UsizeSerializable>::USIZE_LEN * 4
      + <B160 as UsizeDeserializable>::USIZE_LEN;

fn iter(&self) -> impl ExactSizeIterator<Item = usize> {
        ExactSizeChain::new(
        ExactSizeChain::new(
            ExactSizeChain::new(
            ExactSizeChain::new(
                ExactSizeChain::new(
                          ExactSizeChain::new(
                          ExactSizeChain::new(
  UsizeSerializable::iter(&self.eip1559_basefee),

  UsizeSerializable::iter(&self.gas_per_pubdata),
                          ),
                          UsizeSerializable::iter(&self.block_number),
                          ),
                          UsizeSerializable::iter(&self.timestamp),
                ),
                UsizeSerializable::iter(&self.chain_id),
            ),
            UsizeSerializable::iter(&self.gas_limit),
            ),
            UsizeSerializable::iter(&self.coinbase),
        ),
        UsizeSerializable::iter(&self.block_hashes),
        )
    }
}
```

The structure `BasicBlockMetadataFromOracle` implements trait `UsizeSerializable` and its function `iter` returning an iterator or type `ExactSizeIterator`. Such an iterator can provide its consumer with information about its length. While the function is implemented correctly, the related constant `USIZE_LEN` is defined incorrectly.

The value of `USIZE_LEN` currently accounts for 1 value of type `B160`, 4 values of type `u64` and `(3 + 256)` values of type `U256`. However, the actual implementation provides one `U256` value less. There are only two fields typed as `U256` and one fixed-size vector containing `U256` values.

There is currently no consequence of this oversight, since the constant `USIZE_LEN` is not utilized in the current version of the codebase. However, it can become used in a future versions.

<div id="issue-7-2-inf3" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">7. Insufficient overflow protection</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #00acc1; border-radius: 4px; padding: 4px 8px; color: #00acc1;">Addressed</span></span>
  </div>
</div>

Feasibility of these attacks is restricted only by the gas costs. Consider enforcing explicit limits on the amount of events, messages and preimage references that a single transaction can generate in case of non-EVM execution environments.

```rust filepath line=65
zk_ee/src/common_structs/events_storage.rs
self.rollback_depth_in_current_frame += 1;
```

The variable `rollback_depth_in_current_frame` is declared as `usize`, i.e. its maximum value on 32-bit architecture is `4,294,967,295`. This could potentially overflow given that the events storage is initialized only once per block.

```rust filepath line=143
zk_ee/src/common_structs/messages_storage.rs
self.rollback_depth_in_current_frame += 1;
```

Similar to the issue in the events storage with the difference that messages are more expensive than events.

```rust filepath line=23
zk_ee/src/common_structs/new_preimages_publication_storage.rs
pub fn mark_use(&mut self) {
    self.num_uses += 1;
```

Although it is practically very difficult to perform an attack that submits a thousand transactions, where millions of duplicate accounts with different addresses are accessed, theoretically this could be possible in future with non-EVM execution environments. The variable `num_uses` is declared as `usize`, i.e. its maximum value on 32-bit architecture is `4,294,967,295`.

<div id="issue-8-1-inf2" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">8. Missing validations</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

Within the file `basic_bootloader/src/bootloader/transaction/mod.rs`, condition `reserved[1].read().is_zero()` is checked numerous times. But the address is never compared to the constant `SPECIAL_ADDRESS_TO_WASM_DEPLOY`, which is almost unused. Only deployment to EVM is supported right now. Defining an auxiliary function that not only performs the additional validation, but also allows handling different EE types, would ease upgrading later.

```rust filepath line=37
basic_bootloader/src/bootloader/process_transaction.rs
/// We are passing callstack from outside to reuse its memory space between different transactions.
/// It's expected to be empty.
```

Although there is no issue in the current version of the codebase, this assumption should be asserted.

```rust filepath context line=286
basic_bootloader/src/bootloader/account_models/eoa.rs
let result = match reverted {
    false => {
        // Safe to do so by construction.
        match deployed_address {
            DeployedAddress::Address(at) => ExecutionResult::Success {
                output: ExecutionOutput::Create(returndata_region, at),
            },
            _ => ExecutionResult::Success {
                output: ExecutionOutput::Call(returndata_region),
            },
```

As expected, `reverted` is always `true` for `RevertedNoAddress`. However, it would be better to assert that it doesn't leak into the `Success` branch.

<div id="issue-9-2-inf2" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">9. Potential for optimization</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #00acc1; border-radius: 4px; padding: 4px 8px; color: #00acc1;">Addressed</span></span>
  </div>
</div>

```rust filepath line=585
basic_bootloader/src/bootloader/process_transaction.rs
let validation_pubdata = system.net_pubdata_used();`
let ergs_for_pubdata = get_ergs_spent_for_pubdata(system, gas_per_pubdata, None)?;
```

The function `get_ergs_spent_for_pubdata` calls `system.net_pubdata_used()` again, recomputing the value of `validation_pubdata`. Consider passing it as a parameter.

```rust filepath line=315
basic_system/src/system_implementation/io/account_cache.rs
self.cache.for_total_diff_operands(|_, r, addr| {
    // We don't care of the left side, since we're storing the entire snapshot.
```

If the historical values are not required, it is slightly more efficient to define one more auxiliary function that skips calls to `diff_operands_total`:

```rust
&self.btree.iter().for_each(|(k, elem)| {
    let r = unsafe { elem.head.as_ref() };
    // use `k` and `r`
});
```

Similarly, the following snippet can be optimized:

```rust filepath line=152
basic_system/src/system_implementation/io/storage_cache.rs
pub fn net_pubdata_used(&self) -> u64 {
    let mut size = 0;
    self.cache
            .for_total_diff_operands::<_, ()>(|_l, r, _| {
                match r.appearance {
                    Appearance::Retrieved => size += 0,
                    Appearance::Unset => size += 0,
                    Appearance::Updated => size += 32,
                };
```

The optimized code could simply filter only one type of `Appearance` and multiply number of such items by `32`:

```rust
&self.btree.values().filter(|elem| {
    let r = unsafe { elem.head.as_ref() };
    r.value.appearance == Appearance::Updated
}).map(|_| 32)
    .sum();
```

<div id="issue-10-1-min1" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">10. Using EOA account model as a fallback for smart contracts</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #ef4444; border-radius: 4px; padding: 4px 8px; color: #ef4444;">Invalid</span></span>
  </div>
</div>

```rust filepath line=34
basic_bootloader/src/bootloader/account_models/abstract_account.rs
if tx.is_eip_712() && aa_enabled && is_contract {
```

Smart Contracts are implicitly routed to the EOA account model when the Account Abstraction is disabled, i.e. when `AA_ENABLED` is `false`, but `is_contract` is `true`.

Although the `AA_ENABLED` configuration flag is not expected to change between the bootloaders invocations on the same node, there is a scenario where the node operator could disable the Account Abstraction after some time of using it. Such a scenario can arise when a rapidly developing attack, that exploits the Account Abstraction, is discovered.

It would be safer to support such a scenario by immediately rejecting transactions when `!aa_enabled && is_contract` are `true`. Otherwise, if the node running the bootloader switched the config between bootloader invocations, Smart Contracts that were relying on a paymaster would suffer from unexpected charges since the Account Abstraction was disabled but their requests are not rejected.

This issue is reported with Low severity since only the node operator can cause it.

<div id="issue-11-2-min2" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">11. Inconsistent flushing of the caches</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Medium</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath context line=256
basic_bootloader/src/bootloader/mod.rs
pub fn run_prepared<Config: BasicBootloaderExecutionConfig>(
while let Some(next_tx_data_len_bytes) = {
    let mut writable = initial_calldata_buffer.into_writable();
    system
        .try_begin_next_tx(&mut writable)
```

```rust filepath line=399
basic_system/src/system_implementation/system/mod.rs
fn try_begin_next_tx(
    &mut self,
    tx_write_iter: &mut impl zk_ee::oracle::SafeUsizeWritable,
) -> Result<Option<usize>, ()> {
    let next_tx_len_bytes = match self.io.oracle.try_begin_next_tx() {
                None => return Ok(None),
                Some(size) => size.get() as usize,
};
```

```rust filepath context line=385 highlight=[1]
basic_system/src/system_implementation/io/account_cache.rs
fn begin_new_tx(&mut self) {
self.cache.commit();
```

The `NewModelAccountCache` implements the functions `begin_new_tx` and `finish_tx`, similarly to other kinds of cache in the codebase. However, flushing the inner `cache` structure is implemented during the function `begin_new_tx` instead of the function `finish_tx`.

As a consequence, the function `run_prepared` flushes the inner `cache` of `NewModelAccountCache` right after initializing the very first transaction, when it is not needed yet. On the other hand, after processing the last transaction, it does not flush it, because the call to `self.io.oracle.try_begin_next_tx()` returns `None`, so `try_begin_next_tx` does not reach `begin_new_tx`.

```rust filepath context line=127 highlight=[1]
basic_system/src/system_implementation/io/storage_cache.rs
pub fn begin_new_tx(&mut self) {
    self.cache.commit();
```

Similarly, `GenericPubdataAwarePlainStorage` implements flushing the cache in `begin_new_tx`. In contrast to the `NewModelAccountCache` it does not implement the function `finish_tx`.

<div id="issue-12-1-inf1" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">12. Inaccuracy in fees calculation</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #84cc16; border-radius: 4px; padding: 4px 8px; color: #84cc16;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

In `basic_bootloader/src/bootloader/account_models/eoa.rs:355`, the value `transaction.max_fee_per_gas` is used to compute the charged amount. However, the gas price is calculated more precisely in the paymaster flow as `core::cmp::min(max_priority_fee_per_gas, max_fee_per_gas - base_fee)`, see the functions `ensure_payment` and `get_gas_price`.

The impact is not significant since the excess is refunded later, but if both flows are intended to follow the same logic, consider extracting the common code into a function.

<div id="issue-13-2-qa5" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">13. Code duplication</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

These two code snippets are equivalent with variable names as the only exception:

```rust filepath line=744
basic_bootloader/src/bootloader/runner.rs
let mut resources_to_pass = match callstack
    .top()
    .unwrap()
    .clarify_passed_resources(desired_resources_to_pass)
{
    Ok(resources_to_pass) => resources_to_pass,
    Err(SystemError::OutOfResources) => {
        return fail_external_call(
            callstack,
            system,
            should_finish_callee_frame_on_error,
        )
        .map(Either::Right)
    }
    Err(SystemError::Internal(error)) => {
        return Err(error);
    }
};
```

```rust filepath line=792
basic_bootloader/src/bootloader/runner.rs
actual_resources_to_pass = match callstack
    .top()
    .unwrap()
    .clarify_passed_resources(actual_resources_to_pass)
{
    Ok(x) => x,
    Err(SystemError::OutOfResources) => {
        return fail_external_call(
            callstack,
            system,
            should_finish_callee_frame_on_error,
        )
        .map(Either::Right)
    }
    Err(SystemError::Internal(error)) => {
        return Err(error);
    }
}
```

Consider extracting the common code as an auxiliary function.

```rust filepath line=546
basic_system/src/system_implementation/system/mod.rs
ergs: VALUE_TO_EMPTY_ACCOUNT_COST * ERGS_PER_GAS,
```

Code duplication (about 30 lines) can be avoided by moving `assert_eq!` from inside of the `match` expression and extracting the rest as function `fn recompute_hash(preimage_type: PreimageType, buffered: UsizeAlignedByteBox<Global>) -> Bytes32`.

Then there would be something like:

```rust
if PROOF_ENV {
    assert_eq!({
        recompute_hash(preimage_type, buffered.clone()), *hash
    });
} else {
    debug_assert_eq!({
        recompute_hash(preimage_type, buffered.clone()), *hash
    });
}
```

```rust filepath line=546
basic_system/src/system_implementation/system/mod.rs
ergs: VALUE_TO_EMPTY_ACCOUNT_COST * ERGS_PER_GAS,
```

The constant `VALUE_TO_EMPTY_ACCOUNT_COST` is a duplicate of `NEWACCOUNT`. Both are defined in the file `evm_interpreter/src/gas_constants.rs` (out of scope) as `25_000`.

<div id="issue-14-1-fuzz" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">14. Crashes in fuzzing test suite</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

Fuzzing has been performed for all targets, with 32 parallel jobs on a machine with 48 CPU cores. Each fuzzing session was configured with a timeout of 12 hours (43,200 seconds) per target:

```sh
./fuzz.sh parallel --jobs=32 --timeout=43200
```

## Failing assertion in `bootloader_process_transaction`

A crash in the `bootloader_process_transaction` fuzz target has been discovered, which is caused by the assertion placed in `basic_bootloader/src/bootloader/transaction/mod.rs:396`. The crashing input totaling 2KB is saved in Taran Space infrastructure under the filename: `crash-225795e7619398feb58be0939c44029a26cc60a5`.

Here is the abridged output of this crash:

```
$ ./fuzz.sh check
🚨 Crash detected! 🚨
./fuzz/artifacts/bootloader_process_transaction/crash-225795e7619398feb58be0939c44029a26cc60a5
root@fuzz:~/workspace/matter-labs-zk_ee/tests/fuzzer# cargo fuzz run -D bootloader_process_transaction fuzz/artifacts/bootloader_process_transaction/crash-225795e7619398feb58be0939c44029a26cc60a5
...
Running: fuzz/artifacts/bootloader_process_transaction/crash-225795e7619398feb58be0939c44029a26cc60a5

thread '<unnamed>' panicked at /root/workspace/matter-labs-zk_ee/basic_bootloader/src/bootloader/transaction/mod.rs:396:13:
assertion failed: v == 27 || v == 28
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
==66280== ERROR: libFuzzer: deadly signal
...
Error: Fuzz target exited with exit status: 77
```

## Failure of coverage analysis

Running the fuzzing coverage analysis via `./fuzz.sh coverage` failed consistently with the error:

```text
   Compiling fuzzer-fuzz v0.0.0 (/root/workspace/matter-labs-zk_ee/tests/fuzzer/fuzz)
error[E0061]: this method takes 1 argument but 0 arguments were supplied
   --> fuzz/fuzz_targets/bootloader/bootloader_tx_parser.rs:29:29
    |
29  |         let _ = transaction.calculate_hash();
    |                             ^^^^^^^^^^^^^^-- argument #1 of type `u64` is missing
    |
...
error[E0061]: this method takes 1 argument but 0 arguments were supplied
   --> fuzz/fuzz_targets/bootloader/bootloader_tx_parser.rs:34:29
    |
34  |         let _ = transaction.calculate_hash();
    |                             ^^^^^^^^^^^^^^-- argument #1 of type `u64` is missing
    |
...
error[E0599]: no method named `legacy_tx_calculate_signed_hash` found for struct `ZkSyncTransaction` in the current scope
  --> fuzz/fuzz_targets/bootloader/bootloader_tx_parser.rs:47:29
   |
47 |                 transaction.legacy_tx_calculate_signed_hash(chain_id)
   |                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
...
error[E0599]: no method named `eip2930_tx_calculate_signed_hash` found for struct `ZkSyncTransaction` in the current scope
   --> fuzz/fuzz_targets/bootloader/bootloader_tx_parser.rs:50:29
    |
50  |                 transaction.eip2930_tx_calculate_signed_hash(chain_id)
    |                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
...
error[E0599]: no method named `eip1559_tx_calculate_signed_hash` found for struct `ZkSyncTransaction` in the current scope
   --> fuzz/fuzz_targets/bootloader/bootloader_tx_parser.rs:53:29
    |
53  |                 transaction.eip1559_tx_calculate_signed_hash(chain_id)
    |                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    |
...
⚠️ Failed to generate coverage data
```

<div id="issue-15-2-qa1" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">15. Equalities with boolean constants</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath line=74
basic_system/src/system_implementation/system/io_subsystem.rs
assert!(self.storage.storage_cache.has_frames_opened() == false);
assert!(self.storage.account_data_cache.has_frames_opened() == false);
```

```rust filepath line=204
basic_system/src/system_implementation/io/account_cache.rs
if is_warm == false {
    if cold_read_charged == false {
```

Negating predicates is more readable than comparing to `false`.

Examples:

- `assert!(!self.storage.storage_cache.has_frames_opened());`
- `if !is_warm`

<div id="issue-16-2-qa3" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">16. Error handling issues</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #00acc1; border-radius: 4px; padding: 4px 8px; color: #00acc1;">Addressed</span></span>
  </div>
</div>

```rust filepath line=504
basic_system/src/system_implementation/system/mod.rs
return Err(CallPreparationError::System(SystemError::OutOfResources));
```

There can be other kinds of error returned by the `read_account_properties` function from the `account_cache.rs` file, since it calls `self.materialize_element` which can return:

- `InternalError("Unexpected preimage length for AccountProperties")`
- `InternalError("not enough resources")`
- `InternalError("Unsupported EE")`  
  Converting all of them into `OutOfResources` can hinder future crashes investigation.

```rust filepath line=575
basic_system/src/system_implementation/system/mod.rs
return Err(CallPreparationError::System(SystemError::OutOfResources));
```

The error name is misleading since the reason for failure is positive-value transfer with an inappropriate modifier.

```rust filepath context line=73
basic_system/src/system_implementation/io/preimage_cache.rs
fn expose_preimage<const PROOF_ENV: bool>(
) -> Result<&'static [u8], InternalError> {
```

```rust filepath context line=224
basic_system/src/system_implementation/io/preimage_cache.rs
fn get_preimage<const PROOF_ENV: bool>(
) -> Result<&'static [u8], InternalError> {
```

The return types of functions `expose_preimage` and `get_preimage` are declared as the `Result` type, however the `Err` variant of it is never actually returned. On the other hand, both function definitions contain many potential panics.

```rust filepath line=575
basic_system/src/system_implementation/system/io_subsystem.rs
if of {
    return Err(
```

Using `checked_add` and `checked_sub` instead of `overflowing_add` and `overflowing_sub` is more concise and error-proof due to inability to use the wrapped value. There are many other usages like this in the codebase.

```rust filepath line=138
zk_ee/src/common_structs/messages_storage.rs
    .checked_add(total_pubdata as u32 as i32)
    .unwrap();
```

Throughout the codebase, there are many `unwrap()` calls and assertions instead of typed errors. Panics hinder issues investigation when they happen in production.

<div id="issue-17-2-qa8" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">17. Large function definitions</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #10b981; border-radius: 4px; padding: 4px 8px; color: #10b981;">Fixed</span></span>
  </div>
</div>

```rust filepath line=111
basic_system/src/system_implementation/io/account_cache.rs
fn materialize_element<const PROOF_ENV: bool>(
```

This function is too big (100 lines).
```rust filepath context line=585
basic_system/src/system_implementation/system/mod.rs
fn prepare_for_call(
    match SystemExt::io(self).transfer_nominal_token_value(
```

The function `handle_requested_external_call_to_special_address_space` is intended to prevent positive-value transfers to kernel space unless the configuration flag `transfers_to_kernel_space` is engaged or the `callee` is explicitly set to `BOOTLOADER_FORMAL_ADDRESS`.

However, the actual callpath to the function `transfer_nominal_token_value` is still performed with the unchanged value `call_values.nominal_token_value`. While the conditions for the transfer are evaluated and variable `nominal_token_value` is correctly set to zero when the conditions are not met, the variable is not actually utilized with the exception at the very end of the handler, after the token transfer is already performed.

```rust filename line=650
basic_bootloader/src/bootloader/runner.rs
let call_result =
    if nominal_token_value != U256::ZERO && callee != BOOTLOADER_FORMAL_ADDRESS {
        CallResult::Failed { return_values }
    } else {
```

Additionally, the condition to fail the call and return CallResult::Failed is incorrect:

- Condition `nominal_token_value != U256::ZERO` can hold true only when `call_values.nominal_token_value` is not zero and also `transfer_allowed` is true, which requires that either `callee == BOOTLOADER_FORMAL_ADDRESS` or `cfg!(feature = "transfers_to_kernel_space")` holds true.
- Given the second condition `callee != BOOTLOADER_FORMAL_ADDRESS`, only the `cfg!(feature = "transfers_to_kernel_space")` condition can be satisfied

The conjunction of the two conditions is equivalent to `call_values.nominal_token_value != U256::ZERO && callee != BOOTLOADER_FORMAL_ADDRESS && cfg!(feature = "transfers_to_kernel_space")`, which contradicts the intention behind the function.

<div id="issue-18-2-qa7" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">18. Magic numbers</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath line=306
basic_system/src/system_implementation/io/account_cache_entry.rs
AddressDataDiff::Full => 118,
```

Should the constant `ENCODED_SIZE` be used instead?

```rust filepath line=92
basic_system/src/system_implementation/io/account_cache.rs
const COLD_PROPERTIES_ACCESS_EXTRA_COST: BaseResources = BaseResources {
    spendable: BaseComputationalResources {
        ergs: 2500 * ERGS_PER_GAS,
```

```rust filepath line=120
basic_system/src/system_implementation/system/mod.rs
if is_warm_write == false { total_cost + 100 }
```

```rust filepath line=159
basic_system/src/system_implementation/io/storage_cache.rs
Appearance::Updated => size += 32,
```

Constants `2500`, `100` and `32` should be declared and documented better.

```rust filepath line=222
basic_system/src/system_implementation/system/mod.rs
|| block_number < current_block_number.saturating_sub(256)
```

Although the number `256` is clear and correct, it's still better to declare it as a named constant.

```rust filepath line=512
basic_system/src/system_implementation/io/account_cache.rs
Some(Bytes32::from_u256_be(U256::from_limbs([
    0x7bfad8045d85a470,
    0xe500b653ca82273b,
    0x927e7db2dcc703c0,
    0xc5d2460186f7233c,
])));
```

This 32‑byte value is the well‐known “empty code hash” and should be declared as a named constant.

<div id="issue-19-1-qa1" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">19. Misleading names and messages</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath line=21
basic_system/src/system_implementation/io/account_cache_entry.rs
impl<const N: u8> VersioningData<N> {
```

This declaration can be made more readable:

- Use a descriptive name instead of `N`, for example `DEPLOYED` since the parameter is used to denote the deployed state
- The constructor name `empty` can be too ambiguous, it might be suitable to call it `deployed` to contrast it with `non_deployed`

```rust filepath line=47
basic_bootloader/src/bootloader/mod.rs
pub trait BasicBootloaderExecutionConfig: 'static + Clone + Copy + core::fmt::Debug
{
    const SKIP_AA_AND_EXECUTE_CALL: bool;
```

```rust filepath context line=202
basic_system/src/system_implementation/io/mod.rs
fn apply_persistent_changes(
// storage can now write all the changes to tree if needed
if verify_io {
let it = storage_cache.net_diffs_iter();

tree_view
    .verify_and_apply_batch(oracle, it, allocator, logger)
```

The name of `SKIP_AA_AND_EXECUTE_CALL` configuration parameter can be misleading for new developers. When this flag is set to `true`, strictly speaking, only validation and refunding are skipped. Such configuration is used for call simulation.

Additionally, the inverted value of this parameter is passed as the `verify_io` parameter of `fn finish`, it would add clarity if the name of the parameter reflected it, e.g. `ONLY_SIMULATE` or `SKIP_VALIDATION`.

```rust filepath line=423
basic_system/src/system_implementation/io/account_cache.rs
fn read_account_balance_assuming_warm(
```

It would be better to be explicit that it is intended to be used only in the context of the `SELFBALANCE` opcode. Similarly about the `KNOWN_TO_BE_WARM_PROPERTIES_ACCESS_COST` constant.

```rust filepath line=34
basic_bootloader/src/bootloader/account_models/abstract_account.rs
if tx.is_eip_712() && aa_enabled && is_contract {
    AA::Contract(PhantomData)
} else {
    AA::EOA(PhantomData)
}
```

The name `EOA` is misleading here since the EOA is not the only account type it can handle, the `EOA` account model simply represents the “classical” Ethereum account model. Smart contracts are implicitly routed to this account model when the Account Abstraction is disabled, i.e. when `AA_ENABLED` is `false`, but `is_contract` is `true`.

```rust filepath context line=746
basic_bootloader/src/bootloader/runner.rs:746
UpdateQueryError::NumericBoundsError => {
TxError::Validation(InvalidTransaction::LackOfFundForMaxFee {
```

The `NumericBoundsError` can be caused also by an overflow.

```rust filepath line=43
basic_bootloader/src/bootloader/run_single_interaction.rs
InternalError("Insufficient balance while minting").into()
```

The error is actually about overflows, not about insufficient balance.

```rust filepath line=45
basic_bootloader/src/bootloader/runner.rs
/// Helper to revert the caller's frame in case of a failure
/// while preparing to execute an external call.
/// If [callstack] is empty, then we're in the entry frame.
///
fn fail_deployment
```

The commentary by the function `fail_deployment` is a duplicate of the commentary by the function `fail_external_call` in the same file. The words “external call” should be replaced with “deployment”.

```rust filepath line=66
zk_ee/src/common_structs/events_storage.rs
rollback_depths_and_pubdata_stack
```

The structure `EventsStorage` does not use pubdata, so this field should be renamed.

```rust filepath line=39
basic_bootloader/src/bootloader/supported_ees.rs
pub fn clarify_passed_resources
```

This function seems to only apply the 63/64 rule and should be named accordingly.

```rust filepath line=33
basic_bootloader/src/bootloader/supported_ees.rs
pub fn ee_version(&self) -> u8 {
	match self {
    	Self::EVM(..) => ExecutionEnvironmentType::EVM as u8,
```

The function is called as `ee_version` but it actually returns the type identifying the execution environment. The same value is produced by `ee_type`. This can be slightly misleading since usually the term “version” denotes the incremental number used to track evolution of the product.

```rust filepath line=212
basic_bootloader/src/bootloader/account_models/eoa.rs
let to_ee_type = if !transaction.reserved[1].read().is_zero()
```

Slightly misleading name `to_ee_type`, something like `deployment_to_ee_type` would be more understandable.

```rust filepath line=203
basic_system/src/system_implementation/io/preimage_cache.rs
type PreimageType = PreimageRequest;
```

```rust filepath line=11
zk_ee/src/common_structs/new_preimages_publication_storage.rs
pub enum PreimageType {
    Bytecode = 0,
    AccountData = 1,
}
```

The name `PreimageRequest` seems to be perfectly suitable for something that is passed into getter/setter functions.

## Incomprehensible error messages

```rust filepath
basic_bootloader/src/bootloader/process_transaction.rs
.ok_or(InternalError("gp*gl"))?;
.ok_or(InternalError("td-pto"))
.ok_or(InternalError("v+tic"))?;
.ok_or(InternalError("gc*gp"))?;
.ok_or(InternalError("v+pto"))?;
.ok_or(InternalError("td-vpf"))
.ok_or(InternalError("gp*gl"))?;
.ok_or(InternalError("brf-rf"))?;
.ok_or(InternalError("bba-bbb"))?;
.ok_or(InternalError("tgf*gp"))?;
.ok_or(InternalError("fa+v"))
.ok_or(InternalError("mfpg*gl"))?;
```

```rust filepath
basic_bootloader/src/bootloader/account_models/eoa.rs
.ok_or(InternalError("mfpg*gl"))?;
```

```rust filepath
basic_bootloader/src/bootloader/gas_helpers.rs
.ok_or(InternalError("gpp*LTIP"))?;
.ok_or(InternalError("ipo+LTIG"))?;
.ok_or(InternalError("tuo+io"))?;
.ok_or(InternalError("glft*EPF"))?,
.ok_or(InternalError("zb*CZBGC"))?;
.ok_or(InternalError("nzb*CNZBGC"))?;
.ok_or(InternalError("zc+nzc"))
.ok_or(InternalError("gc*gp"))?;
.ok_or(InternalError("cps*epp"))
```

The type `InternalError` is intended to be used only with errors that must never actually happen, and it is not intended to be reported to users. The messages attached to such errors are strictly enough to locate them in case such errors happen.

While these errors are not user-facing, and the error code is exactly enough to locate the place in the codebase, readable error messages would still be valuable for quicker understanding of the codebase by new developers. Ideally, error messages should be informative enough to understand why the program has crashed without reviewing related code.

<div id="issue-20-1-qa9" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">20. Outstanding TODO comments</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

The codebase contains numerous TODO comments. This approach to documenting issues and outstanding work has a few disadvantages:

- TODO comments often become stale compared to properly tracked issues, leading to a lack of progress toward resolution.
- Information in comments must be maintained and updated. Otherwise, it may be misleading for future developers or reviewers.
- These comments highlight weak points in the codebase, which could be exploited by malicious actors if the code is publicly accessible.

Here is the non-exhaustive list of files containing TODO comments:

- `zk_ee/src/system/mod.rs`
- `basic_bootloader/src/bootloader/account_models/eoa.rs`
- `basic_bootloader/src/bootloader/gas_helpers.rs`
- `basic_bootloader/src/bootloader/mod.rs`
- `basic_bootloader/src/bootloader/process_transaction.rs`
- `basic_bootloader/src/bootloader/runner.rs`
- `basic_bootloader/src/bootloader/run_single_interaction.rs`

Some of such comments represent significant features and should be tracked in a dedicated software.

<div id="issue-21-1-qa6" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">21. Redundant parameters</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath
basic_bootloader/src/bootloader/runner.rs
CS: Stack<SupportedEEVMState<S::UsermodeSystem>, S::Allocator>,
...
callstack: &mut CS,
```

In many functions, the `callstack` parameter is used only to query the frame at the top, i.e. as `callstack.top()`. It can be replaced by a minimalistic parameter `topframe: Option<&mut SupportedEEVMState<S::UsermodeSystem>>`. The type parameter `CS` can be removed from such functions, too. Such a refactoring would make code easier to review and maintain.

List of functions that can be improved by such refactoring:

- `fail_external_call`
- `halt_or_continue_after_external_call`
- `halt_or_continue_after_deployment`
- `fail_deployment`
- `handle_requested_external_call_to_special_address_space`
- `run_call_preparation`

```rust filepath line=697
basic_bootloader/src/bootloader/runner.rs
let is_entry_frame = callstack.top().is_none();
```

The boolean `is_entry_frame` is inverse of `should_finish_callee_frame_on_error`, because `callstack` has not been modified since the last time `callstack.top().is_none()` was computed.

The parameter `should_finish_callee_frame_on_error` is redundant.

```rust filepath context line=78
basic_bootloader/src/bootloader/account_models/eoa.rs
fn validate<
    caller_is_code: bool,
```

```rust line=85
// EIP-3607: Reject transactions from senders with deployed code
if caller_is_code {
    return Err(InvalidTransaction::RejectCallerWithCode.into());
}
```

The parameter `caller_is_code` can have the only value `false` since the `Contract` and `EOA` account models are correctly routed from the abstract type. This also means that `Err(InvalidTransaction::RejectCallerWithCode.into())` is never actually thrown.

<div id="issue-22-1-qa2" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">22. Some conditions for branching are always `true`</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath context line=62
basic_bootloader/src/bootloader/runner.rs
fn fail_external_call<
match callstack.top() {
```

```rust line=70
Some(vm) => {
    if finish_callee_frame {
```

Variable `finish_callee_frame` always holds the same value as `callstack.top().is_some()` which is exactly the case already matched before.

```rust filepath context line=184
basic_bootloader/src/bootloader/runner.rs
fn fail_deployment<
match callstack.top() {
```

```rust line=191
Some(vm) => {
    if finish_callee_frame {
```

```rust context line=955
fn handle_requested_deployment<
Err(SystemError::OutOfResources) => fail_deployment(callstack, system, true),
```

The value of the parameter `finish_callee_frame` is explicitly `true` in the current version of the codebase.

<div id="issue-23-2-qa2" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">23. Suboptimal types</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath line=263
basic_bootloader/src/bootloader/account_models/eoa.rs
deployed_address: DeployedAddress::CallNoAddress,
```

Using type `Option<DeployedAddress>`` would be clearer compared to the fake variant of `DeployedAddress`enumeration. Alternatively, consider changing struct`TxExecutionResult` to an enumeration with explicit separation between calls and deployments.

```rust filepath line=148
basic_bootloader/src/bootloader/process_transaction.rs
let spent_on_pubdata = get_ergs_spent_for_pubdata(system, gas_per_pubdata, None)?; // TODO: should it be in ergs or gas?
```

```rust filepath line=906
basic_bootloader/src/bootloader/process_transaction.rs
let spent_on_pubdata = u256_to_u64_saturated(&spent_on_pubdata);
```

The type `U256` is used temporarily for multiplication, this value is being cast into `u64`. It would be better to cast it in the same place, in order to be certain about fitting into the 64 bits.

```rust filepath line=186
basic_system/src/system_implementation/io/preimage_cache.rs
.add_preimage(hash, preimage.len() as u32 as i32, preimage_type)?;
```

```rust filepath line=19
zk_ee/src/common_structs/new_preimages_publication_storage.rs
pub publication_net_bytes: i32,
```

The value `preimage.len()`, of type `usize`, is first cast to `u32` and then to `i32`. Although overflow during these conversions is highly improbable—given that a preimage's size is practically limited and cannot reach 2GB—it is nonetheless unnecessary to perform any conversions.

The type `usize` would be perfectly adequate for the `publication_net_bytes` field. Semantically, this value is never meant to be signed. Moreover, since `usize` can vary across different machines, it is more prudent to mirror the type, unless serialization of the `publication_net_bytes` field is explicitly required.

```rust filepath line=113
basic_bootloader/src/bootloader/run_single_interaction.rs
let call_parameters = CallParameters {
    callers_caller: B160::ZERO, // Fine to use placeholder
```

Using type `Option<Address>` would be clearer and more error-proof, compared to using the default value.

<div id="issue-24-1-qa11" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">24. Unit test suite fails in `e2e_proving` mode</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

The unit test suite is intended to support both forward running and proof-running. The latter is expressed using the `e2e_proving` compiler feature. This mode of testing is important for verifying the ZK proof-related execution paths.

Running the test suite with the `e2e_proving` feature enabled should be possible using the command:

```sh
cargo +nightly llvm-cov --workspace --features e2e_proving --html
```

However, this run has resulted in the failure of the `test_add` test case.

After allowing it to run for 18 hours, it failed with the following output:

```
---- secp256k1::field::field_10x26::tests::test_add stdout ----

thread 'secp256k1::field::field_10x26::tests::test_add' panicked at crypto/src/secp256k1/field/field_10x26.rs:566:9:
attempt to add with overflow
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

failures:
    secp256k1::field::field_10x26::tests::test_add
```

As a consequence, no coverage report has been produced in `e2e_proving` mode.

<div id="issue-25-2-qa6" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">25. Unused code</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath line=21
basic_bootloader/src/bootloader/supported_ees.rs
pub fn needs_scratch_space(&self) -> bool {
```

The function is unused and always returns `false`.

```rust filepath line=756
basic_bootloader/src/bootloader/process_transaction.rs
require!(
    bootloader_received_funds >= required_funds,
```

```rust line=764
let excessive_funds = bootloader_received_funds
    .checked_sub(required_funds)
    .ok_or(InternalError("brf-rf"))?;
```

The error is never thrown because of the assertion.

```rust filepath line=92
basic_system/src/system_implementation/system/basic_metadata.rs
// TODO: gas_limit needed?
pub gas_limit: u64,
```

The field `gas_limit` is initialized during deserialization, but is not practically utilized.

```rust filepath line=220
basic_system/src/system_implementation/memory/basic_memory.rs
unsafe fn clear_returndata_region(&mut self) {
```

The function `clear_returndata_region` is safe, the modifier `unsafe` is redundant.

```rust filepath line=410
basic_system/src/system_implementation/io/account_cache.rs
fn tx_stats(&self) -> Self::TxStats {
```

There are 10 declarations with the name `tx_stats`. None of them is actually called.

```rust filepath context line=254
basic_system/src/system_implementation/io/account_cache_entry.rs
let (delta, op) = match lv.cmp(&rv) {
    core::cmp::Ordering::Equal => (U256::ZERO, CompressionStrategy::Add),
};
if delta == U256::ZERO {
    break 'arm None;
```

Statement `break 'arm None` can be inlined into the pattern matching expression. There is no need to return intermediary value and immediately compare it to already known value.

```rust filepath line=205
basic_system/src/system_implementation/io/account_cache.rs
if cold_read_charged == false {
```

Unreachable since `cold_read_charged` is always `true` after the call to `materialize`. The only case when it is not set by the `materialize` is when the call to `charge_cold_storage_read_extra` results in an error.

The 15 lines inside the branching statement is a duplicate.

```rust filepath line=138
basic_system/src/system_implementation/io/account_cache.rs
let mut cold_read_charged = false;
```

In fact, the variable `cold_read_charged` does not affect anything and can be removed.

```rust filepath line=322
basic_system/src/system_implementation/io/account_cache.rs
let _ = preimages_cache.record_preimage::<false>(
```

Enforcing `PROOF_ENV = false` here, but in fact the function `record_preimage` does not utilize this parameter. It can be removed from its declaration.

```rust filepath line=26
zk_ee/src/common_structs/warm_storage_key.rs
pub struct StorageDiff {
```

```rust filepath line=39
basic_system/src/system_implementation/io/mod.rs
pub type StorageDiff = GenericPlainStorageRollbackData<WarmStorageKey, WarmStorageValue>;
pub type TransientStorageDiff =
```

```rust filepath line=160
basic_system/src/system_implementation/io/mod.rs
fn all_diffs<'a>(&'a self) -> impl Iterator<Item = Self::Diff<'a>> {
    core::iter::empty()
}
```

Declarations `type Diff`, `all_diffs`, `StorageDiff` and `TransientStorageDiff` are not used anywhere.

```rust filepath line=24
zk_ee/src/system/system_trait/execution_environment/environment_state.rs
pub struct SelfDestructParams<S: System> {
```

This structure is not used. Should it be used when a call or deployment is completed?

```rust filepath line=135
zk_ee/src/common_structs/events_storage.rs
pub fn net_diff(
```

```rust filepath line=175
zk_ee/src/common_structs/messages_storage.rs
pub fn net_diff(
```

<div id="issue-26-1-inf3" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">26. Unused error messages</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

The following error messages are not utilized anywhere in the codebase. While no security issues related to these errors have been discovered, they could indicate unhandled errors.

```rust filepath
basic_bootloader/src/bootloader/errors.rs
InvalidStructure,
GasPriceLessThanBasefee,
CallGasCostMoreThanGasLimit,
OverflowPaymentInTransaction,
InvalidChainId,
AccessListNotSupported,
PaymasterReturnDataTooShort,
PaymasterInvalidMagic,
PaymasterContextInvalid,
PaymasterContextOffsetTooLong,
```

<div id="issue-27-1-qa3" style="border: 1px solid #e5e7eb; border-radius: 8px; padding: 16px; margin-top: 2rem; margin-bottom: 16px;">
  <h2 style="color: #e5e7eb; margin: 0 0 12px 0; font-size: 1.25rem; font-weight: 600;">27. Variable can be declared in a more specific scope</h2>
  <div style="display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 12px;">
    <span style="font-size: 0.875rem;"><strong>Severity:</strong> <span style="border: 1px solid #22c55e; border-radius: 4px; padding: 4px 8px; color: #22c55e;">QA</span></span>
    <span style="font-size: 0.875rem;"><strong>Impact:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Likelihood:</strong> <span style="border: 1px solid #e5e7eb; border-radius: 4px; padding: 4px 8px; color: #e5e7eb;">Low</span></span>
    <span style="font-size: 0.875rem;"><strong>Status:</strong> <span style="border: 1px solid #fbc02d; border-radius: 4px; padding: 4px 8px; color: #fbc02d;">Notified</span></span>
  </div>
</div>

```rust filepath line=840
basic_bootloader/src/bootloader/process_transaction.rs
let max_refunded_gas = resources.spendable.ergs.div_floor(ERGS_PER_GAS);
let refund_recipient = if Config::AA_ENABLED && paymaster != B160::ZERO {
    let _succeeded = Self::paymaster_post_op::<_, { Config::SPECIAL_ADDRESS_SPACE_BOUND }>(
        system,
        system_functions,
        callstack,
        transaction,
        tx_hash,
        suggested_signed_hash,
        success,
        max_refunded_gas,
        paymaster,
        gas_per_pubdata,
        validation_pubdata,
        resources,
    )?;
```

The variable `max_refunded_gas` is utilized only in the paymaster flow, and can be moved into that branch. This would improve readability of the function.

# Appendix

# Appendix

## Fuzzing coverage

Generated using the command `./fuzz.sh coverage`.
This run includes the fix for the `bootloader_tx_parser` fuzzing target.

### Module `basic_bootloader`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Lines</th>
  <th style="border: 1px solid white; padding: 8px;">Line Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">account_models/abstract_account.rs</td>
  <td style="border: 1px solid white; padding: 8px;">125/213</td>
  <td style="border: 1px solid white; padding: 8px;">58.7%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">account_models/eoa.rs</td>
  <td style="border: 1px solid white; padding: 8px;">334/676</td>
  <td style="border: 1px solid white; padding: 8px;">49.4%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">account_models/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/38</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">errors.rs</td>
  <td style="border: 1px solid white; padding: 8px;">3/18</td>
  <td style="border: 1px solid white; padding: 8px;">16.7%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">gas_helpers.rs</td>
  <td style="border: 1px solid white; padding: 8px;">74/79</td>
  <td style="border: 1px solid white; padding: 8px;">93.7%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">165/207</td>
  <td style="border: 1px solid white; padding: 8px;">79.7%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">process_transaction.rs</td>
  <td style="border: 1px solid white; padding: 8px;">448/737</td>
  <td style="border: 1px solid white; padding: 8px;">60.8%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">result_keeper.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/2</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">rlp.rs</td>
  <td style="border: 1px solid white; padding: 8px;">51/51</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">run_single_interaction.rs</td>
  <td style="border: 1px solid white; padding: 8px;">111/124</td>
  <td style="border: 1px solid white; padding: 8px;">89.5%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">runner.rs</td>
  <td style="border: 1px solid white; padding: 8px;">502/883</td>
  <td style="border: 1px solid white; padding: 8px;">56.9%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">supported_ees.rs</td>
  <td style="border: 1px solid white; padding: 8px;">39/91</td>
  <td style="border: 1px solid white; padding: 8px;">42.9%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">transaction/abi_utils.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/29</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">transaction/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">561/672</td>
  <td style="border: 1px solid white; padding: 8px;">83.5%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">transaction/u256be_ptr.rs</td>
  <td style="border: 1px solid white; padding: 8px;">46/51</td>
  <td style="border: 1px solid white; padding: 8px;">90.2%</td>
</tr>
</tbody>
</table>

### Module `basic_system`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Lines</th>
  <th style="border: 1px solid white; padding: 8px;">Line Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/bn254_ecadd.rs</td>
  <td style="border: 1px solid white; padding: 8px;">86/86</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/bn254_ecmul.rs</td>
  <td style="border: 1px solid white; padding: 8px;">60/60</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/bn254_pairing_check.rs</td>
  <td style="border: 1px solid white; padding: 8px;">91/94</td>
  <td style="border: 1px solid white; padding: 8px;">96.8%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/ecrecover.rs</td>
  <td style="border: 1px solid white; padding: 8px;">67/68</td>
  <td style="border: 1px solid white; padding: 8px;">98.5%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/keccak256.rs</td>
  <td style="border: 1px solid white; padding: 8px;">26/26</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">7/7</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/modexp.rs</td>
  <td style="border: 1px solid white; padding: 8px;">122/124</td>
  <td style="border: 1px solid white; padding: 8px;">98.4%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/p256_verify.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/68</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/ripemd160.rs</td>
  <td style="border: 1px solid white; padding: 8px;">26/26</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/sha256.rs</td>
  <td style="border: 1px solid white; padding: 8px;">26/26</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/account_cache.rs</td>
  <td style="border: 1px solid white; padding: 8px;">439/610</td>
  <td style="border: 1px solid white; padding: 8px;">72.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/account_cache_entry.rs</td>
  <td style="border: 1px solid white; padding: 8px;">104/180</td>
  <td style="border: 1px solid white; padding: 8px;">57.8</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/history_map.rs</td>
  <td style="border: 1px solid white; padding: 8px;">344/488</td>
  <td style="border: 1px solid white; padding: 8px;">70.5%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">206/270</td>
  <td style="border: 1px solid white; padding: 8px;">76.3%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/preimage_cache.rs</td>
  <td style="border: 1px solid white; padding: 8px;">142/187</td>
  <td style="border: 1px solid white; padding: 8px;">75.9%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/rollbackable_stack.rs</td>
  <td style="border: 1px solid white; padding: 8px;">33/42</td>
  <td style="border: 1px solid white; padding: 8px;">78.6%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/simple_growable_storage.rs</td>
  <td style="border: 1px solid white; padding: 8px;">721/981</td>
  <td style="border: 1px solid white; padding: 8px;">73.5%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/storage_cache.rs</td>
  <td style="border: 1px solid white; padding: 8px;">248/318</td>
  <td style="border: 1px solid white; padding: 8px;">78.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/memory/basic_memory.rs</td>
  <td style="border: 1px solid white; padding: 8px;">162/168</td>
  <td style="border: 1px solid white; padding: 8px;">96.4%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/system/basic_metadata.rs</td>
  <td style="border: 1px solid white; padding: 8px;">72/88</td>
  <td style="border: 1px solid white; padding: 8px;">81.8%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/system/io_subsystem.rs</td>
  <td style="border: 1px solid white; padding: 8px;">212/436</td>
  <td style="border: 1px solid white; padding: 8px;">48.6%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/system/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">264/382</td>
  <td style="border: 1px solid white; padding: 8px;">69.1%</td>
</tr>
</tbody>
</table>

### Module `zk_ee`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Lines</th>
  <th style="border: 1px solid white; padding: 8px;">Line Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/events_storage.rs</td>
  <td style="border: 1px solid white; padding: 8px;">39/68</td>
  <td style="border: 1px solid white; padding: 8px;">57.4%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/generic_ethereum_like_fsm_state.rs</td>
  <td style="border: 1px solid white; padding: 8px;">26/34</td>
  <td style="border: 1px solid white; padding: 8px;">76.5%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/messages_storage.rs</td>
  <td style="border: 1px solid white; padding: 8px;">56/100</td>
  <td style="border: 1px solid white; padding: 8px;">56.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/new_preimages_publication_storage.rs</td>
  <td style="border: 1px solid white; padding: 8px;">84/98</td>
  <td style="border: 1px solid white; padding: 8px;">85.7%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/pubdata_compression.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/49</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/warm_storage_key.rs</td>
  <td style="border: 1px solid white; padding: 8px;">17/18</td>
  <td style="border: 1px solid white; padding: 8px;">94.4%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/warm_storage_value.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/27</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">oracle/usize_rw.rs</td>
  <td style="border: 1px solid white; padding: 8px;">15/71</td>
  <td style="border: 1px solid white; padding: 8px;">21.1%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/kv_markers/kv_impls.rs</td>
  <td style="border: 1px solid white; padding: 8px;">140/201</td>
  <td style="border: 1px solid white; padding: 8px;">69.7%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/kv_markers/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">60/98</td>
  <td style="border: 1px solid white; padding: 8px;">61.2%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/memory/byte_slice.rs</td>
  <td style="border: 1px solid white; padding: 8px;">19/38</td>
  <td style="border: 1px solid white; padding: 8px;">50.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/memory/stack_trait.rs</td>
  <td style="border: 1px solid white; padding: 8px;">42/56</td>
  <td style="border: 1px solid white; padding: 8px;">75.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">4/69</td>
  <td style="border: 1px solid white; padding: 8px;">5.8%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/reference_implementations/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">33/47</td>
  <td style="border: 1px solid white; padding: 8px;">70.2%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_io_oracle/dyn_usize_iterator.rs</td>
  <td style="border: 1px solid white; padding: 8px;">46/49</td>
  <td style="border: 1px solid white; padding: 8px;">93.9%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_io_oracle/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">34/45</td>
  <td style="border: 1px solid white; padding: 8px;">75.6%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/base_system_functions.rs</td>
  <td style="border: 1px solid white; padding: 8px;">8/112</td>
  <td style="border: 1px solid white; padding: 8px;">7.1%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/errors.rs</td>
  <td style="border: 1px solid white; padding: 8px;">3/21</td>
  <td style="border: 1px solid white; padding: 8px;">14.3%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/execution_environment/call_params.rs</td>
  <td style="border: 1px solid white; padding: 8px;">7/99</td>
  <td style="border: 1px solid white; padding: 8px;">7.1%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/execution_environment/interaction_params.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/13</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/execution_environment/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">8/8</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/logger.rs</td>
  <td style="border: 1px solid white; padding: 8px;">6/12</td>
  <td style="border: 1px solid white; padding: 8px;">50.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/resources.rs</td>
  <td style="border: 1px solid white; padding: 8px;">6/6</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/result_keeper.rs</td>
  <td style="border: 1px solid white; padding: 8px;">22/22</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/types_config/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/6</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/aligned_buffer.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/138</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/aligned_vector.rs</td>
  <td style="border: 1px solid white; padding: 8px;">70/97</td>
  <td style="border: 1px solid white; padding: 8px;">72.2%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/bytes32.rs</td>
  <td style="border: 1px solid white; padding: 8px;">73/161</td>
  <td style="border: 1px solid white; padding: 8px;">45.3%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/convenience/memcopy.rs</td>
  <td style="border: 1px solid white; padding: 8px;">5/5</td>
  <td style="border: 1px solid white; padding: 8px;">100.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/integer_utils.rs</td>
  <td style="border: 1px solid white; padding: 8px;">24/85</td>
  <td style="border: 1px solid white; padding: 8px;">28.2%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/13</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/stack_linked_list.rs</td>
  <td style="border: 1px solid white; padding: 8px;">23/29</td>
  <td style="border: 1px solid white; padding: 8px;">79.3%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/type_assert.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/13</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/usize_aligner.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/46</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
</tbody>
</table>

### Other files

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Lines</th>
  <th style="border: 1px solid white; padding: 8px;">Line Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_hooks/src/l1_messenger.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0/168</td>
  <td style="border: 1px solid white; padding: 8px;">0.0%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_hooks/src/lib.rs</td>
  <td style="border: 1px solid white; padding: 8px;">88/89</td>
  <td style="border: 1px solid white; padding: 8px;">98.9%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_hooks/src/precompiles.rs</td>
  <td style="border: 1px solid white; padding: 8px;">118/120</td>
  <td style="border: 1px solid white; padding: 8px;">98.3%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">supporting_crates/modexp/src/arith.rs</td>
  <td style="border: 1px solid white; padding: 8px;">280/309</td>
  <td style="border: 1px solid white; padding: 8px;">90.6%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">supporting_crates/modexp/src/lib.rs</td>
  <td style="border: 1px solid white; padding: 8px;">14/22</td>
  <td style="border: 1px solid white; padding: 8px;">63.6%</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">supporting_crates/modexp/src/mpnat.rs</td>
  <td style="border: 1px solid white; padding: 8px;">427/488</td>
  <td style="border: 1px solid white; padding: 8px;">87.5%</td>
</tr>
</tbody>
</table>

## Unit tests coverage

The unit test suite has been executed with the `e2e_proving` feature disabled since enabling it caused the suite to crash (see the description of this issue in the main part of the report). To collect coverage data, `llvm-cov` has been used:
```sh
cargo llvm-cov --workspace --html
```

### Module `basic_bootloader`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Function Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Region Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">account_models/abstract_account.rs</td>
  <td style="border: 1px solid white; padding: 8px;">87.50% (7/8)</td>
  <td style="border: 1px solid white; padding: 8px;">67.65% (23/34)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">account_models/eoa.rs</td>
  <td style="border: 1px solid white; padding: 8px;">63.64% (7/11)</td>
  <td style="border: 1px solid white; padding: 8px;">43.72% (87/199)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">account_models/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">75.00% (3/4)</td>
  <td style="border: 1px solid white; padding: 8px;">56.25% (9/16)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">errors.rs</td>
  <td style="border: 1px solid white; padding: 8px;">25.00% (1/4)</td>
  <td style="border: 1px solid white; padding: 8px;">10.00% (1/10)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">gas_helpers.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (5/5)</td>
  <td style="border: 1px solid white; padding: 8px;">75.61% (31/41)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">85.71% (6/7)</td>
  <td style="border: 1px solid white; padding: 8px;">70.59% (24/34)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">process_transaction.rs</td>
  <td style="border: 1px solid white; padding: 8px;">52.63% (10/19)</td>
  <td style="border: 1px solid white; padding: 8px;">60.07% (182/303)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">result_keeper.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/2)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/2)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">rlp.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (11/11)</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (29/29)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">run_single_interaction.rs</td>
  <td style="border: 1px solid white; padding: 8px;">33.33% (2/6)</td>
  <td style="border: 1px solid white; padding: 8px;">64.71% (33/51)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">runner.rs</td>
  <td style="border: 1px solid white; padding: 8px;">35.71% (10/28)</td>
  <td style="border: 1px solid white; padding: 8px;">52.05% (152/292)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">supported_ees.rs</td>
  <td style="border: 1px solid white; padding: 8px;">80.00% (8/10)</td>
  <td style="border: 1px solid white; padding: 8px;">77.78% (14/18)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">transaction/abi_utils.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/2)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/44)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">transaction/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">85.29% (29/34)</td>
  <td style="border: 1px solid white; padding: 8px;">75.98% (253/333)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">transaction/u256be_ptr.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (7/7)</td>
  <td style="border: 1px solid white; padding: 8px;">87.23% (41/47)</td>
</tr>
</tbody>
</table>

### Module `basic_system`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Function Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Region Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/bn254_ecadd.rs</td>
  <td style="border: 1px solid white; padding: 8px;">60.00% (6/10)</td>
  <td style="border: 1px solid white; padding: 8px;">82.46% (47/57)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/bn254_ecmul.rs</td>
  <td style="border: 1px solid white; padding: 8px;">50.00% (5/10)</td>
  <td style="border: 1px solid white; padding: 8px;">77.78% (42/54)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/bn254_pairing_check.rs</td>
  <td style="border: 1px solid white; padding: 8px;">26.67% (4/15)</td>
  <td style="border: 1px solid white; padding: 8px;">67.31% (70/104)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/ecrecover.rs</td>
  <td style="border: 1px solid white; padding: 8px;">83.33% (10/12)</td>
  <td style="border: 1px solid white; padding: 8px;">79.37% (50/63)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/keccak256.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (5/5)</td>
  <td style="border: 1px solid white; padding: 8px;">65.00% (13/20)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (1/1)</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (6/6)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/modexp.rs</td>
  <td style="border: 1px solid white; padding: 8px;">50.00% (3/6)</td>
  <td style="border: 1px solid white; padding: 8px;">84.04% (79/94)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/p256_verify.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/5)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/33)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/ripemd160.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (2/2)</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (5/5)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_functions/sha256.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (5/5)</td>
  <td style="border: 1px solid white; padding: 8px;">70.00% (14/20)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/account_cache.rs</td>
  <td style="border: 1px solid white; padding: 8px;">81.25% (26/32)</td>
  <td style="border: 1px solid white; padding: 8px;">60.22% (168/279)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/account_cache_entry.rs</td>
  <td style="border: 1px solid white; padding: 8px;">60.00% (15/25)</td>
  <td style="border: 1px solid white; padding: 8px;">66.67% (36/54)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/history_map.rs</td>
  <td style="border: 1px solid white; padding: 8px;">82.67% (62/75)</td>
  <td style="border: 1px solid white; padding: 8px;">80.00% (204/255)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">80.00% (16/20)</td>
  <td style="border: 1px solid white; padding: 8px;">79.17% (19/24)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/preimage_cache.rs</td>
  <td style="border: 1px solid white; padding: 8px;">90.91% (10/11)</td>
  <td style="border: 1px solid white; padding: 8px;">74.29% (26/35)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/rollbackable_stack.rs</td>
  <td style="border: 1px solid white; padding: 8px;">83.33% (5/6)</td>
  <td style="border: 1px solid white; padding: 8px;">81.82% (9/11)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/simple_growable_storage.rs</td>
  <td style="border: 1px solid white; padding: 8px;">81.58% (62/76)</td>
  <td style="border: 1px solid white; padding: 8px;">80.00% (448/560)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/io/storage_cache.rs</td>
  <td style="border: 1px solid white; padding: 8px;">88.89% (24/27)</td>
  <td style="border: 1px solid white; padding: 8px;">77.00% (77/100)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/memory/basic_memory.rs</td>
  <td style="border: 1px solid white; padding: 8px;">94.44% (17/18)</td>
  <td style="border: 1px solid white; padding: 8px;">87.50% (35/40)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/system/basic_metadata.rs</td>
  <td style="border: 1px solid white; padding: 8px;">75.00% (9/12)</td>
  <td style="border: 1px solid white; padding: 8px;">60.00% (30/50)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/system/io_subsystem.rs</td>
  <td style="border: 1px solid white; padding: 8px;">70.37% (19/27)</td>
  <td style="border: 1px solid white; padding: 8px;">43.80% (53/121)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_implementation/system/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">83.78% (31/37)</td>
  <td style="border: 1px solid white; padding: 8px;">75.00% (114/152)</td>
</tr>
</tbody>
</table>

### Module `zk_ee`

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Function Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Region Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/events_storage.rs</td>
  <td style="border: 1px solid white; padding: 8px;">87.50% (7/8)</td>
  <td style="border: 1px solid white; padding: 8px;">86.67% (13/15)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/generic_ethereum_like_fsm_state.rs</td>
  <td style="border: 1px solid white; padding: 8px;">66.67% (2/3)</td>
  <td style="border: 1px solid white; padding: 8px;">66.67% (12/18)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/messages_storage.rs</td>
  <td style="border: 1px solid white; padding: 8px;">62.50% (5/8)</td>
  <td style="border: 1px solid white; padding: 8px;">73.33% (11/15)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/new_preimages_publication_storage.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (16/16)</td>
  <td style="border: 1px solid white; padding: 8px;">98.48% (65/66)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/pubdata_compression.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/3)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/18)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/warm_storage_key.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (3/3)</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (7/7)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">common_structs/warm_storage_value.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/2)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/10)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">oracle/usize_rw.rs</td>
  <td style="border: 1px solid white; padding: 8px;">17.65% (3/17)</td>
  <td style="border: 1px solid white; padding: 8px;">23.08% (9/39)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/kv_markers/kv_impls.rs</td>
  <td style="border: 1px solid white; padding: 8px;">70.83% (17/24)</td>
  <td style="border: 1px solid white; padding: 8px;">55.00% (55/100)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/kv_markers/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">75.00% (15/20)</td>
  <td style="border: 1px solid white; padding: 8px;">81.82% (54/66)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/memory/byte_slice.rs</td>
  <td style="border: 1px solid white; padding: 8px;">87.50% (7/8)</td>
  <td style="border: 1px solid white; padding: 8px;">90.00% (18/20)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/memory/stack_trait.rs</td>
  <td style="border: 1px solid white; padding: 8px;">73.33% (11/15)</td>
  <td style="border: 1px solid white; padding: 8px;">60.00% (12/20)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">16.67% (1/6)</td>
  <td style="border: 1px solid white; padding: 8px;">6.98% (3/43)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/reference_implementations/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">69.23% (9/13)</td>
  <td style="border: 1px solid white; padding: 8px;">75.00% (12/16)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_io_oracle/dyn_usize_iterator.rs</td>
  <td style="border: 1px solid white; padding: 8px;">85.71% (6/7)</td>
  <td style="border: 1px solid white; padding: 8px;">94.74% (18/19)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_io_oracle/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">66.67% (4/6)</td>
  <td style="border: 1px solid white; padding: 8px;">75.00% (12/16)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/base_system_functions.rs</td>
  <td style="border: 1px solid white; padding: 8px;">14.29% (2/14)</td>
  <td style="border: 1px solid white; padding: 8px;">14.29% (2/14)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/errors.rs</td>
  <td style="border: 1px solid white; padding: 8px;">14.29% (1/7)</td>
  <td style="border: 1px solid white; padding: 8px;">14.29% (1/7)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/execution_environment/call_params.rs</td>
  <td style="border: 1px solid white; padding: 8px;">46.67% (7/15)</td>
  <td style="border: 1px solid white; padding: 8px;">31.25% (15/48)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/execution_environment/interaction_params.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (2/2)</td>
  <td style="border: 1px solid white; padding: 8px;">66.67% (8/12)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/execution_environment/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">50.00% (1/2)</td>
  <td style="border: 1px solid white; padding: 8px;">75.00% (6/8)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/logger.rs</td>
  <td style="border: 1px solid white; padding: 8px;">25.00% (1/4)</td>
  <td style="border: 1px solid white; padding: 8px;">25.00% (1/4)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/resources.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (1/1)</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (6/6)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/system_trait/result_keeper.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/4)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/4)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system/types_config/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/2)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/2)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/aligned_buffer.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/14)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/66)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/aligned_vector.rs</td>
  <td style="border: 1px solid white; padding: 8px;">66.67% (8/12)</td>
  <td style="border: 1px solid white; padding: 8px;">72.73% (16/22)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/bytes32.rs</td>
  <td style="border: 1px solid white; padding: 8px;">60.00% (18/30)</td>
  <td style="border: 1px solid white; padding: 8px;">57.14% (24/42)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/convenience/memcopy.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (1/1)</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (1/1)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/integer_utils.rs</td>
  <td style="border: 1px solid white; padding: 8px;">56.25% (9/16)</td>
  <td style="border: 1px solid white; padding: 8px;">48.84% (21/43)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/mod.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/1)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/1)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/stack_linked_list.rs</td>
  <td style="border: 1px solid white; padding: 8px;">83.33% (5/6)</td>
  <td style="border: 1px solid white; padding: 8px;">88.89% (8/9)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/type_assert.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/3)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/10)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">utils/usize_aligner.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/8)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/15)</td>
</tr>
</tbody>
</table>

### Other files

<table style="border-collapse: collapse; border: 1px solid white;">
<thead>
<tr>
  <th style="border: 1px solid white; padding: 8px;">Filename</th>
  <th style="border: 1px solid white; padding: 8px;">Function Coverage</th>
  <th style="border: 1px solid white; padding: 8px;">Region Coverage</th>
</tr>
</thead>
<tbody>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_hooks/src/l1_messenger.rs</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/2)</td>
  <td style="border: 1px solid white; padding: 8px;">0.00% (0/58)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_hooks/src/lib.rs</td>
  <td style="border: 1px solid white; padding: 8px;">88.89% (8/9)</td>
  <td style="border: 1px solid white; padding: 8px;">83.33% (15/18)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">system_hooks/src/precompiles.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (4/4)</td>
  <td style="border: 1px solid white; padding: 8px;">92.31% (24/26)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">supporting_crates/modexp/src/arith.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (37/37)</td>
  <td style="border: 1px solid white; padding: 8px;">91.74% (222/242)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">supporting_crates/modexp/src/lib.rs</td>
  <td style="border: 1px solid white; padding: 8px;">50.00% (1/2)</td>
  <td style="border: 1px solid white; padding: 8px;">83.33% (5/6)</td>
</tr>
<tr>
  <td style="border: 1px solid white; padding: 8px;">supporting_crates/modexp/src/mpnat.rs</td>
  <td style="border: 1px solid white; padding: 8px;">100.00% (28/28)</td>
  <td style="border: 1px solid white; padding: 8px;">89.84% (230/256)</td>
</tr>
</tbody>
</table>
