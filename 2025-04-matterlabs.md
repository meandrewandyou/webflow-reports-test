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

- **Bootloader** (crate `basic_bootloader`): Only the EOA account model is included in the scope. The modules related to account abstraction are not included in the detailed scope. This means that the `AA_ENABLED` configuration flag can be assumed to be always `false`. The audit includes the bootloader with both the compilation flag `evm-compatibility` enabled and disabled.
- **System Hooks** (crate `system_hooks` and corresponding implementations): precompiles hooks and their underlying implementations in the `basic_system/src/system_functions` and the `crypto` directories.
- **Basic system implementation and system interface** (crate `basic_system`): the focus is on the basic system implementation. To make sense of it, the system interface of some common structures are also included, which are defined in the `zk_ee` and the `storage_models` directories.

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

| Component                  | Status                            |
| -------------------------- | --------------------------------- |
| `basic_bootloader`         | Reviewed by both auditors         |
| `basic_system`             | Time-boxed review by Kirill Taran |
| `supporting_crates/modexp` | Reviewed by Tarek Elsayed         |
| `zk_ee`                    | Partially reviewed                |
| `system_hooks`             | Partially reviewed                |
| Fuzzing and unit tests     | Executed                          |

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
