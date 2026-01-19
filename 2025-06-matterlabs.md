---
title: ZKsync OS
client: Matter Labs
created_at: "2025-08-04"
updated_at: "2025-10-15"
draft: false
---
# Synopsis

In March 2025, Matter Labs engaged Taran Space to conduct a security assessment of their new product, ZKsync OS. Given the extensive and complex codebase, the audit was carried out in multiple phases. An initial four-week phase was conducted in April 2025 and resulted in private report. Following the remediation of the issues, Matter Labs decided to continue with the updated codebase and longer duration in order to facilitate a more thorough examination, discover as many potential issues as possible and to produce the public audit report.

This document summarizes the results of the next phase: a two-month audit conducted between June 9, 2025, and August 4, 2025. Following the audit, Matter Labs addressed the most severe security issues, and the implemented solutions were verified in September 2025.

## Audit Team

The audit of ZKsync OS was conducted by a team comprising two full-time auditors:

- Lead Auditor: [Kirill Taran](https://www.linkedin.com/in/kirill-taran-98103a7b/)
- Auditor: [Tarek Elsayed](https://www.linkedin.com/in/tareknasser360/)

For specialized areas of the audit, particularly the EVM implementation and unsafe Rust techniques, additional experts were involved:

- Senior Auditor: [Vitali Gorgut](https://www.linkedin.com/in/gorgut/)
- Auditor: [Ilia Imeteo](https://www.self.so/ilia-imeteo)

# About ZKsync OS

ZKsync OS, developed by Matter Labs, is a generalized RISC-V based state transition function for the ZKsync protocol. Implemented in Rust and compiled into a RISC-V binary, it can be proven using the `zksync-airbender`. This framework is designed to enhance Ethereum's scalability and composability through zero-knowledge rollup (zkRollup) technology, efficiently facilitating multiple execution environments (EVM, EraVM, Wasm, etc.) within a unified ecosystem.

Building upon the groundwork laid by zkSync Era, ZKsync OS features a modular architecture that not only supports the seamless operation of EVM and EraVM but enables the future integration of WebAssembly. This design includes a bootloader for aggregating these environments and increases the system’s adaptability, enabling the customization of the framework to accommodate new execution environments and various storage models as per specific project needs.

## General Architecture

The architecture of ZKsync OS can be considered as two main layers:

- <strong>Operation layer</strong> is a Rust-based program that can be upgraded independently of the proving circuits, enabling seamless upgrades and customization for app-specific chains.

- <strong>Proving layer</strong> can be fully optimized or replaced without affecting the operation layer, ensuring long-term flexibility and innovation.

## Operation Layer

The operation layer serves as the system-level implementation of the chain’s state transition function. It manages memory and I/O operations to maximize performance and efficiency. Additionally, it supports multiple virtual machines, each tailored to different use cases:

- <strong>EVM (Ethereum Virtual Machine):</strong> Ensures full compatibility with Ethereum.
- <strong>EraVM:</strong> Maintains backward compatibility with existing applications.
- <strong>WASM (WebAssembly):</strong> Expands possibilities by allowing smart contracts in various programming languages.
- <strong>Native RISC-V Contracts:</strong> Delivers maximum performance for high-efficiency use cases.

# Scope and Timeline

## Repository

The codebase is located in the repository by the following link:

- https://github.com/matter-labs/zksync-os

## Modules in Scope

The scope is limited to specific modules of the operation layer:

- `basic_bootloader`
- `basic_system`
- `evm_interpreter`
- `zk_ee`

The proving layer is out-of-scope.

## Phase A

Duration: 5 weeks

Dates: June 9, 2025 - July 14, 2025

Commit hash:

- `96d9d3701a5f5b7c47b42337ff4e70db2bd04223`
- coincides with the branch `taran-audit-phase-1`

Focus on general safety:

- State consistency, data integrity and validity
- EVM correctness and compatibility
- Funds safety, gas accounting
- Network security, performance and DoS vectors

## Phase B

Duration: 3 weeks

Dates: July 14, 2025 - August 4, 2025

Commit hash:

- `5e69d4483243bb0d166b7e2137fa54a8bb05925a`
- coincides with the tag `v0.0.5`

Focus on L1 integration aspects as:

- L1 messaging
- L1 transactions
- L2 base token withdrawals
- Pubdata publishing and compression
- System upgrades

## EVM Compatibility Requirements

For ZKsync OS, strict compatibility with the Ethereum Cancun virtual machine is essential. Compatibility deviations are classified as bugs unless they are pre-listed in the known issues document. Currently, Cancun's documentation is the main reference point for any virtual machine-related queries or clarifications.

## Scope Limitations

During the audit, Account Abstraction (AA) was determined to be outside the audit's scope. The auditing team proceeded with the assumption that the `AA_ENABLED` configuration would be set to `false`, meaning that Account Abstraction features were not reviewed or considered during the review.

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

# Access list decoding can be exploited to submit invalid transactions

The access list parser converts every ABI-encoded value of type `uint256` to the type `usize`, silently discarding the upper bits:

```rust filepath context line=40
basic_bootloader/src/bootloader/transaction/access_list_parser.rs
fn new(slice: &'a [u8], offset: usize)
let bytestring_len = Self::parse_u256(slice, offset)?.as_limbs()[0] as usize;
```

```rust line=52
// For now, it only has the access list
let outer_offset = Self::parse_u256(slice, offset)?.as_limbs()[0] as usize;
let outer_base = offset + outer_offset;
let outer_len = Self::parse_u256(slice, outer_base)?.as_limbs()[0] as usize;
```

This truncation affects 8 critical variables: `access_list_rel_offset`, `bytestring_len`, `outer_offset`, `outer_len`, `item_ptr_offset`, `keys_ptr_offset`, `keys_len` and `count`. Transaction hash calculation in the `apply_access_list_encoding_to_hash()` function uses these truncated values instead of the original bytes.

As a consequence, attackers can craft valid signatures for invalid transactions. For example, a transaction with `keys_len` set to `0x10000000000000001` is processed and verified as if `keys_len` had value `1`. However, the transaction retains the full value, a discrepancy that can be exploited for malicious purposes. Since the transaction hash is calculated using the truncated value, the signature verification succeeds.

Impacts include:
- Hash collisions between legitimate and malicious transactions
- Potential memory corruption if subsequent processing expects the actual data size to match the truncated count
- Potential DoS caused by non-determinism, if other protocol participants handle the truncation differently

[page-break]

## Recommendation

Implement bounds checking in the `parse_u256()` function to reject any value exceeding `u32::MAX`. Add validation to ensure all access list length and offset values fit within expected bounds before processing and fail explicitly when this condition is not satisfied.

# Returndata buffer overflow can cause DoS or exploitable memory corruption

The function `copy_into_return_memory` extends the `returndata_buffer` without validating its final size against the `MAX_RETURNDATA_BUFFER_SIZE` constant, which is currently defined as 128 MB.

```rust filepath context line=246 highlight=[8]
basic_system/src/system_implementation/memory/basic_memory.rs
fn copy_into_return_memory
let new_returndata_start = self.returndata_buffer.len();

let Some(new_returndata_len) = new_returndata_start.checked_add(source.len()) else {
    return Err(InternalError("OOM"));
};
let new_returndata_len = new_returndata_len.next_multiple_of(USIZE_SIZE);
self.returndata_buffer.extend_from_slice(source);
self.returndata_buffer.resize(new_returndata_len, 0);
```

```rust filepath line=15
basic_system/src/system_implementation/memory/basic_memory.rs
pub const MAX_RETURNDATA_BUFFER_SIZE: usize = 1 << 27; // 128 MB
```

Further, we will abbreviate the `MAX_RETURNDATA_BUFFER_SIZE` constant as _"128 MB."_

## Impact of the issue

In the simplest scenario, a malicious actor can execute a smart contract that pushes unbounded data. This will exhaust memory and result in a Denial-of-Service condition.

Additionally, a more intricate attack is possible that forces reallocations and results in memory corruption, because the `copy_into_return_memory` function returns a structure of type `OSManagedResizableSlice`, which refers to the `returndata_buffer` before its expansion:

```rust filepath context line=250 highlight=[1]
basic_system/src/system_implementation/memory/basic_memory.rs
fn copy_into_return_memory
self.returndata_buffer.extend_from_slice(source);
```

```rust line=255 highlight=[7]
unsafe {
    let start = self
        .returndata_buffer
        .as_mut_ptr()
        .add(new_returndata_start);
    let slice = OSManagedResizableSlice {
        ptr: NonNull::new_unchecked(start),
        len: source.len(),
    };

    Ok(slice)
}
```

The `returndata_buffer` variable is initialized as follows:

```rust filepath context line=188 highlight=[2]
basic_system/src/system_implementation/memory/basic_memory.rs
fn new(allocator: Self::Allocator)
let returndata_buffer =
    allocate_vec_usize_aligned(MAX_RETURNDATA_BUFFER_SIZE, allocator.clone());
```

When an attempt is made to grow the heap larger than _128 MB_, a call to `Vec::extend_from_slice` reallocates the data of the buffer, rendering all previously returned pointers as dangling. These dangling pointers can still be read later in the same transaction, e.g., when the VM passes the `returndata_buffer` to the caller. Dereferencing the dangling pointers leads to an immediate Use-After-Free condition, resulting in memory corruption potentially under the attacker's control.

## Minimalistic Proof-of-Concept

The following test case demonstrates that the `returndata_buffer` variable is not protected from growing larger than `MAX_RETURNDATA_BUFFER_SIZE`:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use alloc::alloc::Global;

    #[test]
    fn returndata_buffer_can_grow_past_limit() {
        let mut mem = MemoryImpl::new(Global);

        // 150 MB payload (more than MAX_RETURNDATA_BUFFER_SIZE)
        let payload = vec![0u8; 150 << 20];

        // first call already exceeds the cap
        mem.copy_into_return_memory(&payload).unwrap();
        // the buffer keeps growing
        mem.copy_into_return_memory(&payload).unwrap();

        assert!(
            mem.returndata_buffer.len() <= MAX_RETURNDATA_BUFFER_SIZE,
            "returndata_buffer overflowed: {} bytes",
            mem.returndata_buffer.len()
        );
    }
}
```

To execute this test case, place it in the `basic_system/src/system_implementation/memory/basic_memory.rs` file.

## Affected call chains

The `copy_into_return_memory` function is heavily utilized across the system, posing challenges for developers in ensuring its invocations do not push the `returndata_buffer` size over the `MAX_RETURNDATA_BUFFER_SIZE` limit.

Entry points leading to the `copy_into_return_memory` function include:

- The `l1_messenger_hook_inner` function, which could be called multiple times, accumulating message hashes in the `returndata_buffer`.
- The `pure_system_function_hook_impl` entry point, which could execute various system functions, two of which might request the allocation of huge data:
  - `IdentityPrecompile`, used for copying an arbitrary number of bytes.
  - `ModExp`, writing `mod_len - output.len()` bytes into the `returndata_buffer`.
- Multiple other functions reachable from the `run_prepared` entry point.

The function `run_prepared` initializes the `returndata_buffer` using `System::init_from_oracle(oracle)` before executing a batch of transactions and clears the buffer using `self.clear_returndata_region()` before executing every transaction. Thus, a single value of `returndata_buffer` never spans multiple transactions.

However, the buffer is not cleared between calls made within a single transaction. The `returndata_buffer` is only cleared after the transaction ends.

## Difficulty of exploitation

A single malicious transaction could expand the `returndata_buffer` beyond _128 MB_ without running into the Out-of-Gas condition via multiple attack paths. The most straightforward attack scenario is via the `IdentityPrecompile` system function. The attacker can choose a slice of call data to copy, and the `copy_into_return_memory` function appends it verbatim. For instance, copying 150 MB, or 4,710,912 words, would incur a gas cost of approximately 14.1M, according to the formula _15 + 3 × ⌈len/32⌉_. This operation could be executed in one go or through multiple smaller segments, such as 10 chunks of 15 MB each.

## Recommendation

Implementing size validation for the `returndata_buffer` or utilizing a pre-allocated fixed buffer would fully mitigate the risk.

# Incorrect native cost calculation causes DoS

The `very_low_copy_cost` function miscalculates the native costs for copy operations. It incorrectly applies the constant `COPY_BASE_NATIVE_COST`, which is `80`, for both the per-byte multiplication and the base addition. The correct approach should utilize `COPY_BYTE_NATIVE_COST`, valued at `1`, for the per-byte cost.

```rust filepath context line=306 highlight=[1]
evm_interpreter/src/utils.rs
pub fn very_low_copy_cost
let native = crate::native_resource_constants::COPY_BASE_NATIVE_COST
    .checked_mul(len)?
    .checked_add(crate::native_resource_constants::COPY_BASE_NATIVE_COST)?;
```

This results in significantly inflated native resource costs than intended:

- The expected cost for a copy operation is `1 * len + 80`
- However, the actual cost is calculated as `80 * len + 80`

Where `len` represents the length parameter in the `very_low_copy_cost` function.

For example, a 1 kilobyte copy operation would incur a charge of 82,000 native units instead of the anticipated 1,104 units --- about 74 times more than expected.

The intended calculation method is evident from the naming convention and patterns observed elsewhere in the codebase. For instance, heap expansion costs follow a `base + (per_byte * bytes)` pattern, as it is observed in the same file:

```rust filepath context line=258
evm_interpreter/src/utils.rsg
pub(crate) fn resize_heap
let net_cost_native = HEAP_EXPANSION_BASE_NATIVE_COST.saturating_add(
    HEAP_EXPANSION_PER_BYTE_NATIVE_COST.saturating_mul(net_byte_increase as u64),
```

This miscalculation causes transactions with large copy operations to consume excessive native resources, risking transaction failure even with ample gas provisioned.

As a consequence, users can experience unexpected transaction failures for operations that should succeed. For instance, when deploying contracts with substantial bytecode or manipulating large data arrays.

While native resources are not directly deducted from the Ethereum balance of the sender, this overcharging leads to a higher rate of transactions failing without an apparent reason. This could be perceived as a Denial of Service (DoS) from the user's perspective.

## Recommendation

Correct the pricing formula by replacing the first occurrence of `COPY_BASE_NATIVE_COST` with `COPY_BYTE_NATIVE_COST` to follow the anticipated `base + (per_byte * length)` pattern.

# Non-deterministic decoding allows effective DoS attacks

This issue has been identified in two distinct system hooks, each with varying degrees of likelihood. First, we describe the Medium-severity instance, followed by an analysis of the High-severity case.

## In the L1 Messenger hook

In the `l1_messenger_hook_inner` function, the variables `message_offset` and `length` are parsed into the type `usize`:

```rust filepath context line=154
system_hooks/src/l1_messenger.rs
fn l1_messenger_hook_inner
let message_offset: usize = match U256::from_be_slice(&calldata[4..36]).try_into() {
```

```rust line=187
let length =
    match U256::from_be_slice(&calldata[length_encoding_end - 32..length_encoding_end])
        .try_into()
```

Both conversions are explicitly validated to fit into the `usize` type, and the hook errors out with the message `"L1 messenger failure: sendToL1 called with invalid calldata"` if they do not.

Furthermore, the variables `length_encoding_end` and `message_end`, both of type `usize`, are introduced. The value of `length_encoding_end` is determined by validating `message_offset` to be `32` and then adding `36` to it. The value of `message_end` is calculated as the sum of two `usize` values, with potential overflow being addressed:

```rust filepath context line=198
system_hooks/src/l1_messenger.rs
fn l1_messenger_hook_inner
let message_end = match length_encoding_end.checked_add(length) {
```

The snippets mentioned each have a potential failure related to the `usize` type: two stem from the `try_into` call and one from the `checked_add` call.

These failures are managed, but the use of `usize` introduces non-determinism into the protocol. If any of the `message_offset`, `length`, or `message_end` variables exceed `u32::MAX`—which would still be accommodated by the `u64` type—the validation and execution via the `forward_system` would proceed successfully.

However, values exceeding `u32::MAX` would cause a failure in a node running the `proof_running_system`. Consequently, a single invalid transaction could fail the entire batch during the proving stage, long after resources have been expended on its validation.

## In the L2 Base Token hook

The `l2_base_token.rs` file is subject to similar issues as well, particularly the `WITHDRAW_WITH_MESSAGE_SELECTOR` case. Similarly to the `l1_messenger_hook`, there are 3 failures related to the `usize` type:

```rust filepath context line=201
system_hooks/src/l2_base_token.rs
fn l2_base_token_hook_inner
let message_offset: usize =
    match U256::from_be_slice(&calldata[36..68]).try_into() {
```

```rust line=222
let length =
    match U256::from_be_slice(&calldata[length_encoding_end - 32..length_encoding_end])
        .try_into()
```

```rust line=232
let message_end =
    match length_encoding_end.checked_add(length) {
```

For the L1 messenger, the likelihood of exploitation is reduced since the L1 contracts and governance must have validated all transactions before they are processed by the L2 nodes.

However, the `WITHDRAW_WITH_MESSAGE_SELECTOR` case of the L2 base token hook is a permissionless function, allowing any user to attach a message of dynamic size and trigger the issue. Unlike the transactions handled by the L1 messenger, these transactions do not undergo validation on L1 before being processed on L2.

## Impact of the issue

This vulnerability opens a pathway for malicious actors to conduct a Denial-of-Service (DoS) attack at a very low cost.

An attacker could generate transactions that, while appearing valid, lead to failures during the L2 proving stage. This results in the rejection of entire blocks, containing not only the malicious transactions but also those of other users.

## Recommendation

Consider using a fixed-sized type like `u32` for message-related variables and explicitly setting an even lower limit for the message size.

In L1 contracts, it is crucial to ensure that L1 transactions and system upgrade requests are properly validated. This validation should ensure that messages are not excessively long and that message parameters can comfortably fit within the `u32` range.

# Flawed addresses validation can lead to token loss

In `l2_base_token.rs`, the `l2_base_token_hook_inner` function interprets the `calldata[4..36]` slice, consisting of 32 bytes, as an Ethereum address by taking the lowest 20 bytes. The slice is correctly validated, in the `WITHDRAW_SELECTOR` case, to not have non-zero bytes among the 12 highest bytes:

```rust filepath context line=170
system_hooks/src/l2_base_token.rs
fn l2_base_token_hook_inner
// check that first 12 bytes in address encoding are zero
if calldata[4..4 + 12].iter().any(|byte| *byte != 0) {
    return Ok(Err(
        "Contract deployer failure: withdraw called with invalid calldata",
```

However, the alternative branch `WITHDRAW_WITH_MESSAGE_SELECTOR` of `l2_base_token_hook_inner` does not include a similar validation:

```rust line=270 highlight=[4]
let mut message: alloc::vec::Vec<u8, S::Allocator> =
    alloc::vec::Vec::with_capacity_in(message_length, system.get_allocator());
message.extend_from_slice(FINALIZE_ETH_WITHDRAWAL_SELECTOR);
message.extend_from_slice(&calldata[16..36]);
message.extend_from_slice(&nominal_token_value.to_be_bytes::<32>());
message.extend_from_slice(&caller.to_be_bytes::<20>());
message.extend_from_slice(additional_data);
```

The presence of non-zero bytes in the 12 highest bytes of an address could stem from errors in 3rd-party applications built on the rollup, or simply from a user's mistake. The corresponding Solidity smart contracts of the settlement layer would ignore these bytes, using only the lowest 20 bytes to determine the address. Such an inadvertent mismatch, whether it's an off-by-some offset or residual data in the upper 12 bytes, could result in the interpretation of a 20-byte address that doesn't align with the caller's intentions.

As a consequence of such a mistake, the user's tokens will be silently transferred to an invalid address, effectively resulting in the loss of those tokens without any immediate indication of the mistake to the user.

# Merkle proofs serialization can produce malformed data

The structure `FlatStorageLeaf` implements trait `UsizeSerializable` and its function `iter` returning an iterator or type `ExactSizeIterator`. Such an iterator can provide its consumer with information about its length:

```rust filepath line=52 highlight=[3]
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
impl<const N: usize> UsizeSerializable for FlatStorageLeaf<N> {
    const USIZE_LEN: usize =
        <Bytes32 as UsizeSerializable>::USIZE_LEN * 2 + <u64 as UsizeSerializable>::USIZE_LEN * 2;

    fn iter(&self) -> impl ExactSizeIterator<Item = usize> {
        ExactSizeChain::new(
            UsizeSerializable::iter(&self.key),
            ExactSizeChain::new(
                UsizeSerializable::iter(&self.value),
                UsizeSerializable::iter(&self.next),
```

While the function `iter` is implemented correctly, the related constant `USIZE_LEN` is defined incorrectly.

The value of `USIZE_LEN` currently accounts for 2 values of type `Bytes32` and 2 values of type `u64`. However, the actual implementation provides one `u64` value less. The structure `FlatStorageLeaf` declares only one field typed as `u64`, named `next`.

## Severity of the issue

In other implementations of the `UsizeSerializable` trait, the `iter` function validates the number of bytes consumed during serialization against the constant `USIZE_LEN`. This constant acts as a safeguard against invalid serialization, ensuring the integrity of the data being processed.

However, this validation step leveraging the `USIZE_LEN` constant is not implemented for `FlatStorageLeaf`, meaning there is no direct impact associated with the incorrect definition of it.

Still, the `USIZE_LEN` constant related to the structure `FlatStorageLeaf` is used to define similar constants for other structures:

- `ExistingReadProof`
- `ExistingWriteProof`
- `NewReadProof`
- `NewWriteProof`

These structures, in turn, influence the serialization of even more structures:

- `ValueAtIndexProof`
- `ReadValueWithProof`
- `WriteValueWithProof`

Data within these 7 data structures is critical for the correct functioning of Merkle tree-based storage. However, it is at risk of becoming malformed during the serialization process due to the potential influence of `USIZE_LEN`. For example, an incorrect definition of `USIZE_LEN` might lead to premature termination of serialization, among other potential issues.

_Only because the incorrect definition is involved into serialization of 7 other critical proof-related structures, the likelihood and overall severity of the issue are both considered "Medium." This classification reflects a significantly broader attack surface compared to a similar issue reported in the previous audit, which was rated as "Low."_

# Opcode `SELFBALANCE` is not guaranteed to succeed

```rust filepath context line=188 highlight=[3]
evm_interpreter/src/interpreter.rs
pub fn run
opcodes::ADDRESS => self.address(),
opcodes::BALANCE => self.balance(system),
opcodes::SELFBALANCE => self.selfbalance(system),
opcodes::CODESIZE => self.codesize(),
opcodes::CODECOPY => self.codecopy(system),
```

The function `get_selfbalance`, which handles the opcode `SELFBALANCE`, silently delegates execution to `read_account_balance_assuming_warm`. This function implements only one specific scenario where the account balance is already cached.

```rust filepath line=329 highlight=[8]
basic_system/src/system_implementation/flat_storage_model/mod.rs
fn get_selfbalance(
    &mut self,
    ee_type: ExecutionEnvironmentType,
    resources: &mut Self::Resources,
    address: &<Self::IOTypes as SystemIOTypesConfig>::Address,
) -> Result<<Self::IOTypes as SystemIOTypesConfig>::NominalTokenValue, SystemError> {
    self.account_data_cache
        .read_account_balance_assuming_warm(ee_type, resources, address)
}
```

In the alternative scenario where the cache is cold, the function does not proceed and returns an `InternalError` instead:

```rust filepath context line=417 highlight=[3]
basic_system/src/system_implementation/flat_storage_model/account_cache.rs
pub fn read_account_balance_assuming_warm
match self.cache.get(address.into()) {
    Some(cache_item) => Ok(cache_item.current().value().balance),
    None => Err(InternalError("Balance assumed warm but not in cache").into()),
}
```

Other locations in the codebase do not insert the balance of the contract before executing its code, so the cache cannot be safely assumed to be warm. Executing `SELFBALANCE` is not guaranteed to succeed, diverging from the EVM specification.

## Recommendation

Consider explicitly implementing balance retrieval in the event of a cache miss, or ensure the contract's balance is inserted into the cache before executing its code.

# Overflow during memory heap expansion

The function `grow_heap` aims to increase the size of the specified region to match the parameter `new_size`, aligned to `usize`.

The function verifies if the region is managed by the `MemoryImpl` instance and positioned at the top of the managed heap; otherwise, it returns `InternalError`. It then aligns `new_size` to a multiple of `USIZE_SIZE` and checks that this aligned `new_size` does not exceed the `MAX_HEAP_BUFFER_SIZE`. If the capacity is exceeded, it returns `Ok(None)`.

```rust filepath context line=134
basic_system/src/system_implementation/memory/basic_memory.rs
fn grow_heap
    new_size: usize,
```

```rust line=150 highlight=[1]
let new_size = new_size.next_multiple_of(USIZE_SIZE);

let current_heap_capacity = unsafe {
    self.heap_buffer.len()
        - self
            .current_heap_start
            .offset_from_unsigned(self.heap_buffer.cast())
};
if new_size > current_heap_capacity {
    return Ok(None);
}
```

However, the validation against the heap's maximum capacity is ineffective due to the absence of a bounds check on the `new_size` parameter before aligning it to `next_multiple_of(USIZE_SIZE)`. This oversight can cause overflow and silent wrapping to zero in release mode when compiled with `overflow-checks=false`.

Overflow occurs when the caller-provided `new_size` exceeds `usize::MAX - (USIZE_SIZE - 1)`.

```rust filepath context line=171 highlight=[3]
basic_system/src/system_implementation/memory/basic_memory.rs
fn grow_heap
Ok(Some(OSManagedResizableSlice {
    ptr: self.current_heap_start,
    len: new_size,
```

As a consequence of `new_size` wrapping to zero, heap expansion does not occur, and an `OSManagedResizableSlice` is returned with `len` set to zero. Any caller expecting the returned buffer to match the specified size would then run into an "Out-of-Bounds" write.

## Minimalistic Proof-of-Concept

The following code snippet serves as an example of vulnerable usage of the `grow_heap` function:

```rust
let mut memory = ...;
let empty_region = memory.empty_managed_region();

// Returns a buffer with length 0
let mut resized_region = memory
  .grow_heap(empty_region, usize::MAX)
  .unwrap().unwrap();

let data = b"deadbeef";

// Panic: Range end index 8 out of range for slice of length 0
resized_region[..data.len()].copy_from_slice(data);
```

## Actual usages in the codebase

However, there is only one potentially unsafe call chain in the codebase where the function `grow_heap` is called with arbitrary value of the `new_size` parameter.

In `evm_interpreter/src/utils.rs`, the `offset` parameter of the `resize_heap` function is directly influenced by certain opcodes, including `MSTORE`, `MLOAD` and `MCOPY`. After some manipulations, this parameter is utilized to call the `grow_heap` function at line 272.

This call path, though influenced by user input from a smart contract, is deemed safe in the specific case of the EVM execution environment, because the EVM interpreter constrains `new_size` values to `u32::MAX - 31`. However, the context can differ for other execution environments.

### Recommendation

Return a well-typed error when the parameter `new_size` exceeds `usize::MAX - (USIZE_SIZE - 1)`.

# Potential "Use-After-Free" issues in the preimage cache

Several preimage cache operations bypass the lifetime limitations of the data stored in the `storage` field of type `BTreeMap`. This is done by employing the unsafe operation `core::mem::transmute`, which extends the lifetime to `'static` without guarantees. If the underlying `BTreeMap` reallocates, or if the actual data is freed, the `'static` reference will become dangling. Subsequent access to such a reference will cause a Use-After-Free condition, potentially leading to memory corruption, arbitrary code execution, or system crashes.

```rust filepath context line=82 highlight=[3]
basic_system/src/system_implementation/flat_storage_model/preimage_cache.rs
fn expose_preimage
if let Some(cached) = self.storage.get(hash) {
    unsafe {
        let cached: &'static [u8] = core::mem::transmute(cached.as_slice());
```

```rust line=180 highlight=[5]
let inserted = self.storage.entry(*hash).or_insert(buffered);
// Safety: IO implementer that will use it is expected to live beoynd any frame (as it's part of the OS),
// so we can extend the lifetime
unsafe {
    let cached: &'static [u8] = core::mem::transmute(inserted.as_slice());
```

```rust context line=199 highlight=[4]
fn insert_verified_preimage(
let inserted = self.storage.entry(*hash).or_insert(preimage);

unsafe {
    let cached: &'static [u8] = core::mem::transmute(inserted.as_slice());
```

In general, such approach can be considered a bad practice because it circumvents the borrow checker, which is a compiler mechanism designed to catch memory issues.

In the file `basic_system/src/system_implementation/flat_storage_model/preimage_cache.rs`, the primary entry points to this attack vector are the functions `get_preimage` and `record_preimage`.

The result of the `record_preimage` function is utilized only by `deploy_code`, where it lives for a short time and does not leave the function's scope. Although it is formally returned from the function, the actual callers discard the value.

A more detailed analysis is required to assess the risk posed by the `get_preimage` function. The value returned by this function is not only used by a few functions but is also stored in the `AccountData::bytecode` field during the execution of the `read_account_properties` function, which is called by multiple other functions during the transaction validation stage. Later, the value stored in `AccountData::bytecode` is retrieved by several functions, including the `evm_interpreter` module and execution handlers for both L1 and L2 transactions.

# Invalid safety assumption when deserializing data from oracle

The safety guarantee in the `materialize_element` function is dependent on the assumption that the `init_from_iter` function is correct and that `dst.assume_init()` is safe to call as long as `init_from_iter` has completed successfully:

```rust filepath context line=177 highlight=[9,12]
basic_system/src/system_implementation/flat_storage_model/storage_cache.rs
fn materialize_element<'a>(
let mut dst =
    core::mem::MaybeUninit::<InitialStorageSlotData<EthereumIOTypesConfig>>::uninit(
    );
let mut it = oracle
    .create_oracle_access_iterator::<InitialStorageSlotDataIterator<EthereumIOTypesConfig>>(
        *address,
    )
    .expect("must make an iterator");
unsafe { UsizeDeserializable::init_from_iter(&mut dst, &mut it).expect("must initialize") };
assert!(it.next().is_none());

// Safety: Since the `init_from_iter` has completed successfully and there's no
// outstanding data as per line before, we can assume that the value was read
// correctly.
let data_from_oracle = unsafe { dst.assume_init() } ;
```

[page-break]

The `init_from_iter` function, in its turn, relies on the correct implementation of the `from_iter` method from the trait `UsizeDeserializable`:

```rust filepath context line=50
zk_ee/src/kv_markers/mod.rs
pub trait UsizeDeserializable
unsafe fn init_from_iter(
    this: &mut MaybeUninit<Self>,                                        
    src: &mut impl ExactSizeIterator<Item = usize>,                      
) -> Result<(), InternalError> {                                         
    let new = UsizeDeserializable::from_iter(src)?;
```

This trait is implemented for multiple types but not all implementations enforce length of the output to match the corresponding `USIZE_LEN` constant. Specifically, the `materialize_element` function relies on the correct trait implementation for the `InitialStorageSlotData` structure.

## Implementation of `UsizeDeserializable` for `InitialStorageSlotData`

In this particular case, the implementation does not validate the data being deserialized against the expected length defined by `USIZE_LEN`:

```rust filepath line=168
zk_ee/src/system_io_oracle/mod.rs
impl<IOTypes: SystemIOTypesConfig> UsizeSerializable for InitialStorageSlotData<IOTypes> {
    const USIZE_LEN: usize = <bool as UsizeSerializable>::USIZE_LEN
        + <u8 as UsizeSerializable>::USIZE_LEN
        + <IOTypes::StorageValue as UsizeSerializable>::USIZE_LEN;
    fn iter(&self) -> impl ExactSizeIterator<Item = usize> {
        ExactSizeChain::new(
            UsizeSerializable::iter(&self.is_new_storage_slot),
            UsizeSerializable::iter(&self.initial_value),
        )
    }
}

impl<IOTypes: SystemIOTypesConfig> UsizeDeserializable for InitialStorageSlotData<IOTypes> {
    const USIZE_LEN: usize = <Self as UsizeSerializable>::USIZE_LEN;

    fn from_iter(src: &mut impl ExactSizeIterator<Item = usize>) -> Result<...
        let is_new_storage_slot = UsizeDeserializable::from_iter(src)?;
        let initial_value = UsizeDeserializable::from_iter(src)?;

        let new = Self {
            is_new_storage_slot,
            initial_value,
        };

        Ok(new)
```

## Incomplete `UsizeDeserializable` implementations

The `UsizeDeserializable` trait, utilizing the `USIZE_LEN` constant, offers additional protection against malformed data during deserialization by enabling developers to validate the number of bytes read. The constant `USIZE_LEN` is also usually shared with the `UsizeSerializable` trait to ensure data integrity during serialization.

```rust filepath line=268
zk_ee/src/kv_markers/kv_impls.rs
impl UsizeDeserializable for B160 {
    const USIZE_LEN: usize = <B160 as UsizeSerializable>::USIZE_LEN;

    fn from_iter(src: &mut impl ExactSizeIterator<Item = usize>) -> Result<Self, InternalError> {
        if src.len() < <Self as UsizeDeserializable>::USIZE_LEN {
            return Err(InternalError("b160 deserialization failed: too short"));
        }
```

Throughout the codebase, many types implement `UsizeSerializable` and `UsizeDeserializable`, including manually maintained values for the `USIZE_LEN` parameter. However, only two of such types implement the validation according to the aforementioned pattern: `B160` and `Bytes32` from the file `zk_ee/src/kv_markers/kv_impls.rs`.

<br>

Constants `USIZE_LEN` in all of the other types are not actually utilized, yet require being updated according to any internal structure changes.

<br>

## Recommendation

Complete all `UsizeDeserializable` implementations by enforcing output lengths to match the corresponding `USIZE_LEN` constants. Ideally, these constants should not be maintained manually but an automated mechanism should derive them from the actual structure layouts.

# L1 transactions can pose strong DoS potential

L1 transactions are initiated on the settlement layer: users can request "priority transactions" on the settlement layer, as well as the governance can schedule a system upgrade on L1. Hashes of such transactions are committed on-chain, so a batch containing them could be verified after publishing.

Such transactions are labeled using the `L1_L2_TX_TYPE` and `UPGRADE_TX_TYPE` constants and are processed using the `process_l1_transaction` function:

```rust filepath context line=61 highlight=[8,18]
basic_bootloader/src/bootloader/process_transaction.rs
pub fn process_transaction
let tx_type = transaction.tx_type.read();

match tx_type {
    ZkSyncTransaction::UPGRADE_TX_TYPE => {
        if !is_first_tx {
            Err(Validation(InvalidTransaction::UpgradeTxNotFirst))
        } else {
            Self::process_l1_transaction(
                system,
                system_functions,
                memories,
                transaction,
                false,
            )
        }
    }
    ZkSyncTransaction::L1_L2_TX_TYPE => {
        Self::process_l1_transaction(system, system_functions, memories, transaction, true)
    }

    _ => Self::process_l2_transaction::<Config>(
```

For regular L2 transactions, the `process_l2_transaction` function ensures signature verification is performed via the calls to functions `transaction_validation`, `AA::validate` and `EOA::validate`. The latter function is defined in the file `basic_bootloader/src/bootloader/account_models/eoa.rs`.

However, the `process_l1_transaction` function does not include signature verification. Any user can attempt submitting a transaction in the L2 network which includes `L1_L2_TX_TYPE` or `UPGRADE_TX_TYPE` as the value of `tx_type`. Such values of the `tx_type` field would be invalid from the standard EVM perspective. If sequencer does not filter invalid values of the `tx_type` field, bogus L1 transactions could be validated, executed, proved and included into a batch for publication.

Since the settlement layer verifies the L1 transactions against the hashes, committed before, only authorized L1 transactions can actually be published. Unauthorized L1 transactions will be rejected together with all the other transactions included into the same batch. This leads to significant amount of computation lost and presents a strong DoS vector. An attacker needs to submit only one bogus L1 transaction to fail multiple legit transactions.

_While the impact is significant, the likelihood of this issue depends on the specific implementation of the sequencer node, which is out of scope of this audit. From the bootloader and ZKsync OS perspective, this is a trust assumption that the sequencer rejects transactions with non-standard values of the `tx_type` field. It is not possible to mitigate this issue by means of the ZKsync OS since the L1 signers cannot be preliminary whitelisted._

## Recommendation

Ensure that users cannot submit transactions with values of the `tx_type` equal to `L1_L2_TX_TYPE` or `UPGRADE_TX_TYPE` by rejecting such by the sequencer node.

# Merkle tree implementation concerns

## Insufficient separation between testing and simulation code

One of the data structures related to the Merkle tree implementation is `FlatStorageBacking`, serving as a multi-purpose index and incorporating features for testing, such as the `RANDOMIZED` parameter:

```rust filepath line=901 highlight=[4]
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
pub struct FlatStorageBacking<
    const N: usize,
    H: FlatStorageHasher,
    const RANDOMIZED: bool,
```

<br>

This structure is utilized to define the `TestingTree` type, intended solely for testing purposes:

```rust filepath line=1667
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
pub type TestingTree<const RANDOMIZED: bool> =
    FlatStorageBacking<TESTING_TREE_HEIGHT, Blake2sStorageHasher, RANDOMIZED>;
```

However, it also underpins the definition of the `InMemoryTree` type, employed in production code for transaction simulation:

```rust filepath context line=17
api/src/lib.rs
pub fn run_batch_generate_witness
tree: InMemoryTree,
```

```rust filepath line=8
forward_system/src/run/test_impl/tree.rs
pub struct InMemoryTree<const RANDOMIZED_TREE: bool = false> {
    /// Hash map from a pair of Address, slot into values.
    pub cold_storage: HashMap<Bytes32, Bytes32>,
    pub storage_tree: TestingTree<RANDOMIZED_TREE>,
}
```

Beyond concerns over code quality, there is a potential future issue given that the production structure `InMemoryTree` relies on the testing constant `TESTING_TREE_HEIGHT` instead of the production constant `TREE_HEIGHT`. Although both constants are currently set to `64`, this could change in the future and go unnoticed. Simulations of transactions should run against the most current, real state.

```rust filepath line=55
basic_system/src/system_implementation/flat_storage_model/mod.rs
pub const TREE_HEIGHT: usize = 64;
```

```rust filepath line=1666
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
pub const TESTING_TREE_HEIGHT: usize = 64;
```

### Recommendation

1. Rename the `TestingTree` structure and relocate the `InMemoryTree` definition out of the `test_impl` folder to clearly indicate the production usage of both types.
2. In the `TestingTree` definition, use the `TREE_HEIGHT` constant instead of the `TESTING_TREE_HEIGHT` constant.

<br>

## Large function definitions are difficult to maintain

The single function definition `verify_and_apply_batch`, located in the `simple_growable_storage.rs` file, occupies 600 lines of code, which poses challenges for review and refactoring.

Besides merely size of the function, the definition also includes several bad practices, e.g. panicking instead of proper error handling, one-letter variable names and using tuples instead of dedicated structures. However, some of the code quality issues deserve to be reported separately.

## Complex handling of iterators instead of relying on the standard `chunks` operation

While iterating over the `current_hashes_buffer` vector, overlapping windows are used together with custom skipping of every second window. This behaviour can be implemented using the standard iterator operation `chunks`:

```rust filepath context line=692
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
fn verify_and_apply_batch
for (idx, [a, b]) in current_hashes_buffer.array_windows::<2>().enumerate() {
    if next_merged {
        next_merged = false;
        continue;
    }
```

```rust line=746
next_hashes_buffer.push((merged_index, a.1, read_hash, write_hash));
next_merged = true;
```

Using the `chunks` operation would also render the auxiliary function `can_merge` redundant, since it only determines that two leaves belong to the same chunk by validating the `a = b + 1` equation:

```rust line=628
fn can_merge(pair: (u64, u64)) -> bool {
    let (a, b) = pair;
    debug_assert_ne!(a, b);
    a & !1 == b & !1
```

## Assertion on the number of writes can be strengthened

After verifying the state changes in a transaction batch, if the root hash remains unchanged, it is asserted that `num_new_writes` is zero. However, the `num_new_writes` variable represents only "non-existing writes." The assertion should instead target the `num_total_writes` variable, as writes to already existing cells must result in a changed root hash, too.

```rust context filepath line=775 highlight=[2,10]
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
fn verify_and_apply_batch
if let Some(new_root) = current_hashes_buffer[0].3 {
    if num_total_writes > 0 {
        assert!(
            new_root != self.root,
            "hash collision on state root with non-zero number of writes"
        );
    }
    self.root = new_root;
} else {
    assert_eq!(num_new_writes, 0);
    // root should not change in such case
}
```

## Miscellaneous code quality concerns

```rust filepath line=230
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
let num_new_writes = source
    .clone()
    .filter(|(_, v)| v.current_value != v.initial_value && v.is_new_storage_slot)
```

Instead of cloning the `source` iterator and filtering out reads again, it would be more efficient to clone the `writes_iter` which contains only writes.

```rust filepath line=772
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
// if we have new root - use it
if let Some(new_root) = current_hashes_buffer[0].3 {
```

In multiple locations, getters `.2` and `.3` are used. In mission-critical code such should be replaced with a named field to improve maintainability.

# Missing validation of token transfer during tokens burning

The implementation of withdrawals from L2 to L1 relies on the implicit invariant that `nominal_token_value` has been transferred from the `caller` to `L2_BASE_TOKEN_ADDRESS`. Otherwise, any user could crash the node by attempting to withdraw non-existing tokens:

```rust filepath context lines=147 highlight=[11]
system_hooks/src/l2_base_token.rs
fn l2_base_token_hook_inner
// Burn nominal_token_value
match system.io.update_account_nominal_token_balance(
    ExecutionEnvironmentType::parse_ee_version_byte(caller_ee)
        .map_err(SystemError::LeafDefect)?,
    resources,
    &L2_BASE_TOKEN_ADDRESS,
    &nominal_token_value,
    true,
) {
    Ok(_) => Ok(()),
    Err(UpdateQueryError::NumericBoundsError) => Err(SystemError::LeafDefect(
        internal_error!("L2 base token must have withdrawal amount"),
```

The prerequisite token transfer is ensured earlier in the withdrawal flow, by another function `external_call_before_vm`, which invokes the `transfer_nominal_token_value_inner` function. This is happening right before the `handle_requested_external_call_to_special_address_space` function passes execution to `try_intercept`, which calls the `l2_base_token_hook` handler.

```rust filepath context lines=425 highlight=[1,3]
system_hooks/src/l2_base_token.rs
fn external_call_before_vm
if let Some(TransferInfo { value, target }) = transfer_to_perform {
    match actual_resources_to_pass.with_infinite_ergs(|inf_resources| {
        self.system.io.transfer_nominal_token_value(
            ExecutionEnvironmentType::NoEE,
            inf_resources,
            &call_request.caller,
            &target,
            &value,
```

However, when `transfer_to_perform` is `None`, no action is taken, despite certain call modifiers potentially setting it to this value. System hooks currently treat such modifiers as errors, thereby preventing `None` values. The concern here is that this invariant is not explicitly enforced in the code.

## Recommendation

While the analysis revealed that this scenario is safe, it is still recommended to validate consistency between the `external_call_before_vm` and `l2_base_token_hook` functions:

- Explicitly handle the `None` case for the `transfer_to_perform` parameter.
- Additionally, verify that the transaction has adjusted the balances of the `caller` and the `L2_BASE_TOKEN_ADDRESS` by the burned amount.

# Missing validations during system upgrades

_The following two issues are reported with "Low" severity since only the governance or operator could cause them, most probably as a consequence of deployment mistake. Given the importance of system upgrade operation, extra checks are advisable._

## Observable bytecode hash is not verified

During system upgrades, the function `set_bytecode_details` is reached from the `contract_deployer_hook` function. The function utilizes parameters `observable_bytecode_hash` and `observable_bytecode_len` to set the corresponding fields of the contract's account:

```rust filepath context line=795
basic_system/src/system_implementation/flat_storage_model/account_cache.rs
pub fn set_bytecode_details
observable_bytecode_hash: Bytes32,
observable_bytecode_len: u32,
```

```rust line=876 highlight=[3]
account_data.update(|cache_record| {
    cache_record.update(|v, m| {
        v.observable_bytecode_hash = observable_bytecode_hash;
        v.observable_bytecode_len = observable_bytecode_len;
```

These parameters receive values retrieved in the `contract_deployer_hook_inner` function as the result of parsing the calldata. The `observable_bytecode_len` parameter corresponds to the exact length of the byte code received, since the same value is used in the `expose_preimage` function, where the buffer is truncated at this length, and hash of the buffer would not match if the length was incorrect.

However, the `observable_bytecode_hash` is not verified against the byte code:

```rust filepath context line=168
system_hooks/src/contract_deployer.rs
let observable_bytecode_hash =
    Bytes32::from_array(calldata[96..128].try_into().expect("Always valid"));
```

```rust line=182 highlight=[8]
system.set_bytecode_details(
    resources,
    &address,
    ExecutionEnvironmentType::EVM,
    bytecode_hash,
    bytecode_length,
    0,
    observable_bytecode_hash,
    bytecode_length,
)?;
```

Note that the `bytecode_hash` is a value computed using the _Blake2s_ hash function and is verified in the `expose_preimage` function, while the `observable_bytecode_hash` should be a _Keccak256_ hash but passed into the function `set_bytecode_details` without verification. As a consequence, potential governance mistakes can cause inconcistency in the final account data.

## Validation of EIP-3541

```rust filepath context line=179
system_hooks/src/contract_deployer.rs
fn contract_deployer_hook_inner
// Also EIP-3541(reject code starting with 0xEF) should be validated by governance.
```

Although it is required to be validated by the L1 governance, the extra check would not impose significant performance penalty. Given the rareness and importance of such operation as system upgrade, the extra check is advisable.

Additionally, the validations required by governance should be documented.

# Potential extra validations

## Validate receiver when reading the EVM deployment flag

```rust filepath context line=485
basic_bootloader/src/bootloader/account_models/eoa.rs
fn charge_additional_intrinsic_gas
let to = transaction.to.read();
let is_deployment =
    !transaction.reserved[1].read().is_zero() || to == SPECIAL_ADDRESS_TO_WASM_DEPLOY;
```

Validate that `transaction.to == null` when the EVM deployment transaction flag is set. Auxiliary predicate `isDeployment` could be defined that examines value of the `transaction.reserved[1]` flag and also validates the `transaction.to` field.

```rust filepath context line=65 highlight=[2]
basic_bootloader/src/bootloader/transaction/mod.rs
pub struct ZkSyncTransaction
/// Now `reserved[0]` is used as a flag to distinguish EIP-155(with chain id) legacy transactions.
/// `reserved[1]` is used as EVM deployment transaction flag(`to` == null in such case).
pub reserved: [ParsedValue<U256>; 4],
```

## Unsafe function is called without validating preconditions

The function `grow_heap` includes call to the `offset_from_unsigned` function:

```rust filepath context line=152 highlight=[5]
basic_system/src/system_implementation/memory/basic_memory.rs
fn grow_heap
let current_heap_capacity = unsafe {
    self.heap_buffer.len()
        - self
            .current_heap_start
            .offset_from_unsigned(self.heap_buffer.cast())
};
```

[page-break]

However, the function `offset_from_unsigned` is unsafe and requires the following condition to be validated before the call:

> The distance between the pointers must be non-negative (self >= origin)

In this particular case, this means that `self.current_heap_start` must be ensured to be not lesser than `self.heap_buffer`.

Otherwise, the call can lead to an Unsafe Behaviour.

# Unchecked `base_fee_per_gas` downcast

The bootloader performs an unchecked downcast of a block-level parameter `base_fee_per_gas` from type `U256` to type `u64` using `try_into().unwrap()`:

```rust filepath context line=341
basic_bootloader/src/bootloader/mod.rs
pub fn run_prepared
let block_header = BlockHeader::new(
```
```rust line=350 highlight=[1]
    base_fee_per_gas.try_into().unwrap(),
```

The oracle can set arbitrary values without validation, as confirmed by the comment:

```rust filepath line=117
basic_bootloader/src/bootloader/block_header.rs
pub fn new
// currently operator can set any base_fee_per_gas, in practice it's usually constant
base_fee_per_gas,
```

If the oracle supplies a `base_fee_per_gas` value exceeding `u64::MAX`, the bootloader will panic and halt all transaction processing. While economically impractical under normal conditions, this creates a real DoS vector if the oracle is misconfigured or compromised.

The value `base_fee_per_gas` is initialized using the call `system.get_eip1559_basefee()` that extracts the field `eip1559_basefee` from the structure `BlockMetadataFromOracle`.

A panic in this context halts the network until the oracle issue is resolved.

## Recommendation

Handle gracefully the case of `base_fee_per_gas` value exceeding `u64::MAX` by returning an error of type `TxError::Internal`.

# Unsafe assumption that `pubdata` cannot decrease after validation

The function `get_resources_to_charge_for_pubdata` computes a fair amount of native resources to charge for the pubdata used by a transaction. This function iterates through all state elements, comparing initial and final values to calculate the state difference and determine the amount of resources to charge, proportional to this difference. It is parameterized by an optional `base_pubdata` parameter. When `base_pubdata` is `None`, the result of the state iteration is returned as is.

However, when a value for `base_pubdata` is provided, it is used to adjust the result of the state iteration:

```rust filepath context line=138 highlight=[1]
basic_bootloader/src/bootloader/gas_helpers.rs
pub fn get_resources_to_charge_for_pubdata
let current_pubdata_spent = system.net_pubdata_used()? - base_pubdata.unwrap_or(0);
let native_per_pubdata = u256_to_u64_saturated(&native_per_pubdata);
let native = current_pubdata_spent
    .checked_mul(native_per_pubdata)
    .ok_or(InternalError("cps*epp"))?;
let native = <S::Resources as zk_ee::system::Resources>::Native::from_computational(native);
Ok((current_pubdata_spent, S::Resources::from_native(native)))
```

Since unchecked arithmetic is employed, there is a potential risk of underflow.

Initially, the function is utilized by `transaction_validation` to compute `validation_pubdata`. This call is safe because no value is passed as `base_pubdata` yet:

```rust filepath context line=704
basic_bootloader/src/bootloader/process_transaction.rs
fn transaction_validation
let (validation_pubdata, to_charge_for_pubdata) =
    get_resources_to_charge_for_pubdata(system, native_per_pubdata, None)?;
```

The second call, which is more concerning, occurs during the gas refund process and passes the result of the first call, referred to as `validation_pubdata`, as the value for the `base_pubdata` parameter:

```rust filepath context line=964 highlight=[5]
basic_bootloader/src/bootloader/process_transaction.rs
fn refund_transaction
let (_pubdata_spent, to_charge_for_pubdata) = get_resources_to_charge_for_pubdata(
    system,
    native_per_pubdata,
    Some(validation_pubdata),
```

## Severity of the issue

The impact of the underflow is that the `current_pubdata_spent` variable receives a value close to `u64::MAX`, leading to native resource exhaustion and an unexplained transaction reversal for the user.

However, triggering the underflow is unlikely with the currently implemented execution environments. The `pubdata` value post-execution does not decrease compared to its post-validation value. Yet, this assumption might not hold in other execution environments that could charge a conservative estimate of pubdata during validation, intending to refund any excess later.

## Recommendation

To ensure a future-proof design, consider two approaches:

- Implement checked arithmetic to prevent underflow and reject scenarios where the post-execution pubdata value is less than the post-validation value. Additionally, document this behaviour as a requirement for future execution environments.
- Alternatively, explicitly permit a reduction in pubdata during later processing stages, and incorporate this difference into the gas refund calculation.

# Decreased performance due to redundant `pubdata` calculation

The function `get_resources_to_charge_for_pubdata` serves as the entry point for computationally intensive calculations defined by the function `net_pubdata_used` and by functions implemented in several types of storage, all named `calculate_pubdata_used_by_tx`. It iterates through all versions of state elements and calculates the amount of change created by the transaction.

```rust filepath context line=138
basic_bootloader/src/bootloader/gas_helpers.rs
pub fn get_resources_to_charge_for_pubdata
let current_pubdata_spent = system.net_pubdata_used()? - base_pubdata.unwrap_or(0);
```

```rust filepath context line=357
basic_system/src/system_implementation/system/io_subsystem.rs
fn net_pubdata_used
Ok(self.storage.pubdata_used_by_tx() as u64
    + self.logs_storage.calculate_pubdata_used_by_tx()? as u64)
}
```

Since the `pubdata` calculation involves multiple iterations through a potentially large set of changes, it negatively affects the performance of the node and should be used only when absolutely necessary. However, besides the lightweight `pubdata` calculation during the validation stage and the proper `pubdata` calculation during the refund stage, the `get_resources_to_charge_for_pubdata` function is also called during transaction execution:

```rust filepath context line=751 highlight=[1]
basic_bootloader/src/bootloader/process_transaction.rs
fn transaction_execution
if !check_enough_resources_for_pubdata(
    system,
    native_per_pubdata,
    resources,
    Some(validation_pubdata),
)? {
    let _ = system
        .get_logger()
        .write_fmt(format_args!("Not enough gas for pubdata after execution\n"));
    Ok(execution_result.reverted())
```

[page-break]

```rust filepath context line=160 highlight=[2]
basic_bootloader/src/bootloader/gas_helpers.rs
pub fn check_enough_resources_for_pubdata
    let (_, resources_for_pubdata) =
        get_resources_to_charge_for_pubdata(system, native_per_pubdata, base_pubdata)?;
```

The `pubdata` calculation is performed at the end of the actual transaction execution, so the result does not change until the refund stage:

```rust filepath context line=964
basic_bootloader/src/bootloader/process_transaction.rs
fn refund_transaction
let (_pubdata_spent, to_charge_for_pubdata) = get_resources_to_charge_for_pubdata(
    system,
    native_per_pubdata,
    Some(validation_pubdata),
)?;
```

## Recommendation

Avoid excessive calculation by passing the result of the `pubdata` calculation from the execution stage to the refund stage. Alternatively, a more sophisticated but universal memoization technique could be implemented.

# Double resource accounting is a deviation from EVM semantics

ZKsync OS implements double-accounting for computation resources. One accounting subsystem tracks, charges and refunds "ergs" which are trivially translated into "gas", so this accounting subsystem ensures full compliance with EVM semantics when the node executes transactions. Another subsystem accounts "native" resources which represent cycles spent in the proving mode.

Each subsystem declares its own error type for situations when the node runs out of the corresponding resource:

- `OutOfErgs`, representing the "Out Of Gas" condition in the EVM
- `OutOfNativeResources`, representing the condition when the proving or publishing would exceed the allowed amount of computation resources

However, since the `OutOfNativeResources` can be reported during transaction validation or execution, this presents deviation from pure EVM semantics:

```rust filepath context line=131 highlight=[7,10]
system_hooks/src/precompiles.rs
pub fn pure_system_function_hook_impl
Err(SystemFunctionError::System(SystemError::OutOfErgs))
| Err(SystemFunctionError::InvalidInput) => {
    let _ = system
        .get_logger()
        .write_fmt(format_args!("Out of gas during system hook\n"));
    resources.exhaust_ergs();
    Ok(make_error_return_state(system, resources))
}
Err(SystemFunctionError::System(SystemError::OutOfNativeResources)) => {
    Err(FatalError::OutOfNativeResources)
}
```

As a consequence, a transaction that executes correctly in EVM, without causing the "Out Of Gas" condition, can still revert or be rejected due to excessive native resources cost. This is expected, according to [the docs](https://github.com/matter-labs/zksync-os/blob/taran-audit-phase-1/docs/double_resource_accounting.md):

> If a transaction execution runs out of native resources, the entire transaction is reverted. If the same happens during transaction validation, the transaction is considered invalid.

However, this situation could look like a spontaneous Denial-of-Service to the user.

Similar issues have been observed in other locations:

- `basic_bootloader/src/bootloader/account_models/eoa.rs`, line 361
- `basic_bootloader/src/bootloader/account_models/eoa.rs`, line 503
- `basic_bootloader/src/bootloader/account_models/eoa.rs`, line 556

# Inconsistent developer documentation

## Opcode `DIFFICULTY` is not consistent with the documentation

The `DIFFICULTY`/`PREVRANDAO` opcode in the EVM interpreter returns zero instead of the constant value `2500000000000000` specified in the [documentation](https://github.com/matter-labs/zksync-docs/blob/6046caaf61ac52a33cb94791a857addaaf9ae734/content/20.zksync-protocol/70.differences/10.evm-instructions.md?plain=1#L180-L182).

In the `difficulty` handler, the `U256::ZERO` value is pushed onto the stack:

```rust filepath line=32 highlight=[3]
evm_interpreter/src/instructions/environment.rs
pub fn difficulty(&mut self) -> InstructionResult {
    self.spend_gas_and_native(gas_constants::BASE, DIFFICULTY_NATIVE_COST)?;
    self.push_values(&[U256::ZERO])?;
    Ok(())
}
```

This deviation from documented behavior could cause compatibility issues for contracts that check specific difficulty values or use it in non-critical calculations.

## Documentation incorrectly lists `p256Verify` as precompile at address `0x100`

The [documentation](https://github.com/matter-labs/zksync-docs/blob/6046caaf61ac52a33cb94791a857addaaf9ae734/content/20.zksync-protocol/70.differences/40.pre-compiles.md?plain=1#L23) lists `p256Verify` as a precompile at address `0x100`. However, the auxiliary function `is_precompile` only recognizes addresses from 1 to 10 as the precompile addresses.

```rust filepath line=415
evm_interpreter/src/utils.rs
/// Helper to check if an address is an ethereum precompile
#[inline(always)]
pub fn is_precompile(address: &B160) -> bool {
    let highest_precompile_address = 10;
    let limbs = address.as_limbs();
    if limbs[1] != 0u64 || limbs[2] != 0u64 {
        return false;
    }
    limbs[0] > 0 && limbs[0] <= highest_precompile_address
}
```

The `p256Verify` function is actually implemented as a system function rather than an EVM precompile, making the documentation incorrect.

This discrepancy could mislead developers expecting standard precompile behavior at address `0x100`.

# Incorrect opcode name in debug output

The opcode `TSTORE` (transient storage) is defined as `0x5d` as it is observed in the `opcodes.rs` file:

```rust filepath line=52
evm_interpreter/src/opcodes.rs
pub const TSTORE: u8 = 0x5d;
```

However, the `OPCODE_JUMPMAP` array, defined in the same file and used for debug string mapping, contains a typo where the opcode `0x5d` is incorrectly labeled as `"STORE"`.

```rust filepath context line=283
evm_interpreter/src/opcodes.rs
pub const OPCODE_JUMPMAP
/* 0x5d */ Some("STORE"),
```

This discrepancy only affects debug output and logging --- the actual opcode execution is unaffected since the interpreter uses numeric constants rather than their text labels.

## Recommendation

Update the string mapping to correctly identify the opcode as `"TSTORE"` for accurate debug output.

# Code duplicates

## Auxiliary function `nb_rounds`

```rust filepath line=28
basic_system/src/system_functions/sha256.rs
fn nb_rounds(len: usize) -> u64 {
    let full_chunks = len / SHA256_CHUNK_SIZE;
    let tail = len % SHA256_CHUNK_SIZE;
    let num_rounds: u64 = full_chunks as u64;
    if tail <= 55 {
        num_rounds + 1
    } else {
        num_rounds + 2
    }
}
```

Implementations of both `sha256` and `ripemd160` system functions include such auxiliary function.

## Implementations of the `Drop` trait

The `ElementPool` type implements the `Drop` trait:

```rust filepath line=17
zk_ee/src/common_structs/history_map/element_pool.rs
impl<V, A: Allocator + Clone> Drop for ElementPool<V, A> {
    fn drop(&mut self) {
        if let Some(head) = self.head {
            let mut elem = unsafe { Box::from_raw_in(head.as_ptr(), self.alloc.clone()) };

            while let Some(n) = elem.previous.take() {
                let n = unsafe { Box::from_raw_in(n.as_ptr(), self.alloc.clone()) };

                elem = n;
            } // `n` is dropped here.
        } // Last elem is dropped here.
    }
}
```

The same code snippet is duplicated for the `Drop` trait implementation of the `ElementWithHistory` type, within lines 23-34 of the `zk_ee/src/common_structs/history_map/element_with_history.rs` file. This code can be extracted into an auxiliary function.

## Module `modexp` and the buffer-fill logic

The following code copies the `base` value:

```rust filepath line=131
basic_system/src/system_functions/modexp.rs
let mut input_it = input.iter();
let mut base = Vec::try_with_capacity_in(base_len, allocator.clone())
	.map_err(|_| SystemError::Internal(InternalError("alloc")))?;
base.resize(base_len, 0);
for (dst, src) in base.iter_mut().zip(&mut input_it) {
	*dst = *src;
}
```

Exactly same code blocks are used in the same file, for `exponent` and `modulus` values:

- lines 139-144
- lines 146-151

## Derivation of `is_static` flag

```rust filepath context line=43
system_hooks/src/l1_messenger.rs
pub fn l1_messenger_hook
let mut error = false;
// There is no "payable" methods
error |= nominal_token_value != U256::ZERO;
let mut is_static = false;
match modifier {
    CallModifier::Constructor => {
        return Err(
            internal_error!("L1 messenger hook called with constructor modifier").into(),
        )
    }
    CallModifier::Delegate
    | CallModifier::DelegateStatic
    | CallModifier::EVMCallcode
    | CallModifier::EVMCallcodeStatic => {
        error = true;
    }
    CallModifier::Static | CallModifier::ZKVMSystemStatic => {
        is_static = true;
    }
    _ => {}
}

if error {
    return Ok((make_error_return_state(available_resources), return_memory));
}
```

[page-break]

The same or very similar code snippets can also be observed in:

- `system_hooks/src/l2_base_token.rs` (lines 38-60)
- `system_hooks/src/contract_deployer.rs` (lines 38-61)

## Burning tokens from `L2_BASE_TOKEN_ADDRESS`

While there is no immediate impact, the following code snippets are important enough to reduce chance of a mistake during refactoring.

```rust filepath context line=146
system_hooks/src/l2_base_token.rs
fn l2_base_token_hook_inner
// Burn nominal_token_value
match system.io.update_account_nominal_token_balance(
    ExecutionEnvironmentType::parse_ee_version_byte(caller_ee)
        .map_err(SystemError::LeafDefect)?,
    resources,
    &L2_BASE_TOKEN_ADDRESS,
    &nominal_token_value,
    true,
) {
    Ok(_) => Ok(()),

    Err(UpdateQueryError::NumericBoundsError) => Err(SystemError::LeafDefect(
        internal_error!("L2 base token must have withdrawal amount"),
    )),
    Err(UpdateQueryError::System(e)) => Err(e),
}?;
```

The same code snippet can be also observed within lines 246-262 of the same file.

## Bytecode hash validation

The bytecode hash is recomputed and validated against the expected value using this code:

```rust filepath context line=105
basic_system/src/system_implementation/flat_storage_model/preimage_cache.rs
fn expose_preimage
use crypto::blake2s::Blake2s256;
use crypto::MiniDigest;
let digest = Blake2s256::digest(buffered.as_slice());
let mut result = Bytes32::uninit();
let recomputed_hash = unsafe {
    result
        .assume_init_mut()
        .as_u8_array_mut()
        .copy_from_slice(digest.as_slice());
    result.assume_init()
};

if recomputed_hash != *hash {
    return Err(internal_error!("Account hash mismatch").into());
}
```

This code is duplicated without any difference for `PreimageType::AccountData` and `PreimageType::Bytecode` branches of the `match` expression. The whole `match` expression is duplicated again in two branches of the `if PROOF_ENV` statement.

## Validation of zero bytes in Ethereum addresses

This code preforms the validation of the 12 highest bytes in Ethereum addresses:

```rust filepath context line=147
system_hooks/src/contract_deployer.rs
fn contract_deployer_hook_inner
// check that first 12 bytes in address encoding are zero
if calldata[0..12].iter().any(|byte| *byte != 0) {
    return Ok(Err(
        "Contract deployer failure: setBytecodeDetailsEVM called with invalid calldata",
    ));
}
```

Almost exact duplicate is located in `system_hooks/src/l2_base_token.rs:170-175`.

# Error handling issues

The codebase heavily relies on panics instead of returning error values.

While panics may be appropriate in certain scenarios, it is usually a trade-off:

- _Missed error handling_: panics are not subject to type-checking, so it is easier to overlook error-handling mechanisms.
- _Unrecoverable failures_: if a panic signals a recoverable error and is not properly handled, the program may terminate unnecessarily.
- _Reduced readability and debugging complexity_: codebases that rely on panics are harder to follow and debug, increasing maintenance overhead.
- _Performance overhead_: handling the panic requires unwinding the stack, which decreases program performance.

Consequently, relying on panics as the primary error-handling mechanism increases the difficulty of ensuring codebase safety and maintainability. It is better to approach error handling with meaningful error messages rather than using panics. This approach improves code maintainability, debugging efficiency, and overall developer experience.

## Examples in the EVM interpreter

The EVM interpreter contains multiple instances of `panic!` and `assert!` statements, instead of returning proper errors. Although these statements represent internal invariants that should not be violated in correct implementations, relying on panics reduces code robustness and renders the system vulnerable to future changes.

```rust filepath line=28
evm_interpreter/src/interpreter.rs
if let Some(call) = external_call {
    assert!(exit_code == ExitCode::ExternalCall);
```

```rust filepath line=119
evm_interpreter/src/ee_trait_impl.rs
if scratch_space_len != 0 || decommitted_bytecode.len() != bytecode_len as usize {
    panic!("invalid bytecode supplied, expected padding");
}
```

```rust filepath line=182
evm_interpreter/src/ee_trait_impl.rs
a => {
    panic!("modifier {:?} is not expected", a);
}
```

```rust filepath context line=28
evm_interpreter/src/u256.rs
pub(crate) fn log2floor
    assert!(value != &U256::ZERO);
```

These panic conditions complicate maintenance. In contrast, other errors in the same files are handled appropriately, returning `FatalError::Internal`.

[page-break]

## Examples in the rest of the system

```rust filepath context line=37
basic_bootloader/src/bootloader/runner.rs
pub fn run_till_completion
assert!(callstack.is_empty());
```

```rust filepath context line=477 highlight=[3]
basic_system/src/system_implementation/system/io_subsystem.rs
fn finish
// TODO (EVM-989): read only state commitment
let fsm_state =
    <BasicIOImplementerFSM::<FlatStorageCommitment<TREE_HEIGHT>> as UsizeDeserializable>::from_iter(&mut initialization_iterator).unwrap();
```

```rust filepath context line=180
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
pub fn set_next
assert!(self.next.is_none());
self.next = Some(value);
```

```rust filepath context line=97
system_hooks/src/lib.rs
pub fn add_hook
let existing = self.inner.insert(for_address_low, hook);
// TODO: internal error?
assert!(existing.is_none());
```

```rust filepath context line=33
system_hooks/src/precompiles.rs
fn new
let buffer = system.memory.empty_managed_region();
let buffer = system
    .memory
    .grow_heap(buffer, Self::INITIAL_LEN)
    .expect("must grow buffer for precompiles")
    .expect("must grow buffer for precompiles");
```

```rust filepath context line=56
system_hooks/src/precompiles.rs
fn extend
self.buffer = self
    .system
    .memory
    .grow_heap(buffer, new_len)
    .expect("must grow buffer for precompiles")
    .expect("must grow buffer for precompiles");
}
```

```rust filepath context line=117
system_hooks/src/precompiles.rs
pub fn pure_system_function_hook_impl
system
    .memory
    .copy_into_return_memory(&buffer[..offset])
    .expect("must copy into returndata")
    .take_slice(0..offset)
```

## Recommendation

Unless when it is supposed to be an unbreakable invariant, replace all `panic!` and `assert!` statements with proper error handling by returning typed error messages. This would prevent unnecessary node crashes.

# Magic numbers

## In system hooks implementation

Numerous numeric literals are used without proper documentation in files `contract_deployer.rs`, `l1_messenger.rs` and `l2_base_token.rs`. Some of them reflect fundamental properties of Solidity ABI, other are specific to calldata and messages layout and are re-used in similar parts of the codebase.

```rust filepath context line=140
system_hooks/src/contract_deployer.rs
fn contract_deployer_hook_inner
calldata = &calldata[4..];
if calldata.len() < 128 {
```

```rust line=148
if calldata[0..12].iter().any(|byte| *byte != 0) {
```

```rust line=153
let address =
    B160::try_from_be_slice(&calldata[12..32]).ok_or(SystemError::LeafDefect(
```

```rust line=159
let bytecode_hash =
    Bytes32::from_array(calldata[32..64].try_into().expect("Always valid"));
```

```rust line=169
let observable_bytecode_hash =
    Bytes32::from_array(calldata[96..128].try_into().expect("Always valid"));
```

```rust filepath context line=143
system_hooks/src/l2_base_token.rs
fn l2_base_token_hook_inner
// following solidity abi for withdraw(address)
if calldata.len() < 36 {
```

The number `36` means the sum of `32` and `4` but it should be better to adopt the "self-documented code" and explicitly compose the expression from named constants.

```rust line=169
message[0..4].copy_from_slice(FINALIZE_ETH_WITHDRAWAL_SELECTOR);
// check that first 12 bytes in address encoding are zero
if calldata[4..4 + 12].iter().any(|byte| *byte != 0) {
```

```rust line=195
// following solidity abi for withdrawWithMessage(address,bytes)
if calldata.len() < 68 {
```

And multiple other similar issues.

## Length of word

```rust filepath context line=161 highlight=[4]
system_hooks/src/precompiles.rs
fn execute
cycle_marker::wrap_with_resources!("id", resources, {
    let cost_ergs =
        ID_STATIC_COST_ERGS + ID_WORD_COST_ERGS.times((src.len() as u64).div_ceil(32));
```

While the length of word in EVM is always `32`, it is still worthwhile to declare an explicit constant for better maintainability. This is relevant to implementations of system functions `identity`, `keccak256`, `ripemd160` and `sha256`.

## Modexp system function

```rust filepath line=180 highlight=[1,3,6]
basic_system/src/system_functions/modexp.rs
let ic = if exp_size <= 32 && exp_highp.is_zero() {
    0
} else if exp_size <= 32 {
    exp_highp.bit_len() as u64 - 1
} else {
    8u64.checked_mul(exp_size - 32)
```

Similarly, in the implementation of the `modexp` system function, the number `32` is used on lines 112, 120, 121, 180, 182, 185.

```rust filepath context line=197 highlight=[1]
basic_system/src/system_functions/modexp.rs
pub fn ergs_cost
let gas = core::cmp::max(200, computed_gas);
let ergs = gas
    .checked_mul(ERGS_PER_GAS)
    .ok_or(SystemError::OutOfErgs)?;
Ok(Ergs(ergs))
```

Additionally, the number `200` which acts as a minimum base price to charge on EVM could be declared as dedicated constant, too.

## Access list gas costs

Gas costs `1900` for storage and `2400` for accounts should be declared as named constants:

```rust filepath context line=68 highlight=[2]
basic_system/src/system_implementation/system/mod.rs
fn charge_cold_storage_read_extra
if is_access_list {
    Ergs(1900 * ERGS_PER_GAS)
```

```rust filepath context line=129 highlight=[2]
basic_system/src/system_implementation/flat_storage_model/account_cache.rs
fn materialize_element
if is_access_list {
    resources.charge(&R::from_ergs(Ergs(2400 * ERGS_PER_GAS)))?
```

```rust line=202 highlight=[2]
if is_access_list {
    resources.charge(&R::from_ergs(Ergs(2400 * ERGS_PER_GAS)))?
```

## In pubdata calculation

The number `32` is used in the `calculate_pubdata_used_by_tx` function without proper documentation:

```rust filepath context line=628 highlight=[3]
basic_system/src/system_implementation/flat_storage_model/storage_cache.rs
pub fn calculate_pubdata_used_by_tx
if initial_value != current_value {
    // TODO(EVM-1074): use tree index instead of key for repeated writes
    pubdata_used += 32; // key
```

## Undocumented callstack limitation

The number `1024` is used twice in the deployment handler:

```rust filepath context line=473
basic_bootloader/src/bootloader/runner.rs
fn handle_requested_deployment
if self.callstack_height > 1024 {
    return Ok(Some(CallResult::Failed {
```

```rust line=660
if self.callstack_height > 1024 {
    resources_for_deployer.reclaim(launch_params.external_call.available_resources);
```

There is the corresponding constant declared:

```rust filepath line=25
zk_ee/src/system/mod.rs
pub const MAX_GLOBAL_CALLS_STACK_DEPTH: usize = 1024; // even though we do not have to formally limit it,
                                                      // for all practical purposes (63/64) ^ 1024 is 10^-7, and it's unlikely that one can create any new frame
                                                      // with such remaining resources
```

However, the constant `MAX_GLOBAL_CALLS_STACK_DEPTH` is not used.

## Undocumented length of blocks history

```rust filepath context line=102
zk_ee/src/system/mod.rs
pub fn get_blockhash
let current_block_number = self.metadata.block_level_metadata.block_number;
if block_number >= current_block_number
    || block_number < current_block_number.saturating_sub(256)
```

The number `256` reflects the maximum number of hashes to consider and is chosen arbitrary.

## Undocumented border of precompile addresses

```rust filepath context line=165
evm_interpreter/src/utils.rs
pub fn is_precompile
let highest_precompile_address = 10;
```

The number `10` means the border between precompile addresses and regular addresses. It should be extracted as a named constant.

## Keccak256 gas cost

```rust filepath context line=963
basic_bootloader/src/bootloader/transaction/mod.rs
fn eip712_tx_calculate_signed_hash
let domain_separator = Self::domain_hash_struct(chain_id, resources)?;
let hs = self.hash_struct(resources)?;
charge_keccak(2 + 2 * U256::BYTES, resources)?;
```

The number `2` should represent a computation price for Keccak256 in RISC-V cycles. However, this semantics should be properly documented by using named constants.

# Misleading docs and messages

## The `identity` precompile does not stay in the caller's frame

```rust filepath line=74 highlight=[4]
system_hooks/src/precompiles.rs
/// It parses call request, calls system function, and creates execution result.
///
/// NOTE: "pure" here means that we do not expect to trigger any state changes (and calling with static flag is ok),
/// so for all the purposes we remain in the callee frame in terms of memory for efficiency
```

```rust line=98 highlight=[1]
// NOTE: we did NOT start a frame here, so we are in the caller frame in terms of memory, and must be extra careful
// here on how we will make returndata
```

In the description of the `identity` system function, it is stated that the execution does not create additional frame and stays in the "caller's" frame. However, besides the typo on the line 77, this note also seems to be incorrect, because there is the frame creation further in the flow:

```rust line=106
// cheat
let snapshot = system.memory.start_memory_frame();
```

[page-break]

## Miscellaneous outdated comments

```rust filepath line=24 highlight=[3]
basic_system/src/system_functions/modexp.rs
/// Returns `InvalidInput` error if `base_len` > usize max value
/// or `mod_len` > usize max value
/// or (`exp_len` > usize max value and `base_len` != 0 and `mod_len` != 0).
```

If both `base_len != 0` and `mod_len != 0` are zero, `Ok(())` is returned, otherwise the execution flow proceeds to `exp_len` validation, i.e. if at least one is non-zero, not necessarily both.

```rust filepath line=18 highlight=[4]
basic_system/src/system_functions/bn254_ecadd.rs
/// If the input size is less than expected - it will be padded with zeroes.
/// If the input size is greater - redundant bytes will be ignored.
///
/// If output len less than needed(64) returns `InternalError`.
```

Technically, the output is always a vector of 64 bytes, although there can be zero bytes. The `InternalError` error is not utilized in this file.

```rust filepath line=13
basic_system/src/system_functions/p256_verify.rs
/// In case of invalid input `Ok(0)` will be returned and resources will be charged.
///
/// If dst len less than needed(1) returns `InternalError`.
```

While the docs mention the `Ok(0)` value being returned in case of an invalid input, in fact, the `Ok(())` value is returned. The same inaccuracy affects inline docs of the `ecrecover` system function. Additionally, the `dst` parameter is not validated to be non-empty.

```rust filepath line=13
basic_system/src/system_functions/sha256.rs
impl<R: Resources> SystemFunction<R> for Sha256Impl {
    /// If output len less than needed(32) returns `InternalError`.
```

The `InternalError` error is not utilized in this file, same as the length of the output buffer is not validated.

```rust filepath line=56
basic_bootloader/src/bootloader/process_transaction.rs
// Safe to unwrap here, as this should have been validated in the
// previous call.
let tx_type = transaction.tx_type.read();
```

This states "Safe to unwrap here" but no `unwrap` is actually used.

## EIP-158 is mentioned instead of EIP-170

```rust filepath context line=171
system_hooks/src/contract_deployer.rs
fn contract_deployer_hook_inner
// Although this can be called as a part of protocol upgrade,
// we are checking the next invariants, just in case
// EIP-158: reject code of length > 24576.
if bytecode_length as usize > MAX_CODE_SIZE {
```

The `MAX_CODE_SIZE` cap is defined by EIP-170.

## Duplicated error message in "L2 Base Token" system hook

```rust filepath context line=172
system_hooks/src/l2_base_token.rs
fn l2_base_token_hook_inner
return Ok(Err(
    "Contract deployer failure: withdraw called with invalid calldata",
));
```

The message is duplicate from the file `contract_deployer.rs` and contains incorrect prefix `Contract deployer failure`.

## Bytecodes for system upgrades are not passed via calldata

In [the documentation of system hooks](https://github.com/matter-labs/zksync-os/blob/main/docs/system_hooks.md?plain=1#L44), it is stated:

```
Bytecodes will be published separately with Ethereum calldata.
```

However, during system upgrades, only the value `bytecode_hash` is decoded from the `calldata`:

```rust filepath context line=158
system_hooks/src/contract_deployer.rs
fn contract_deployer_hook_inner
let bytecode_hash =
    Bytes32::from_array(calldata[32..64].try_into().expect("Always valid"))
```

The bytecode itself is retrieved from the preimage storage.

# Redundant code

## Unreachable code in EVM arithmetic operations

The EVM interpreter contains several unreachable code locations that can never happen to be executed due to early returns or exhaustive conditions.

### In `i256_div` function

In pattern matching expression, implemented in the function `i256_div`, several clauses are impossible because the function returns early when the divisor is zero:

```rust filepath context line=76 highlight=[2]
evm_interpreter/src/i256.rs
pub fn i256_div
let second_sign = i256_sign::<true>(&mut second);
if second_sign == Sign::Zero {
    return U256::ZERO;
}
```

```rust line=95 highlight=[3,4,9]
match (first_sign, second_sign) {
    (Sign::Zero, Sign::Plus)
    | (Sign::Plus, Sign::Zero)
    | (Sign::Zero, Sign::Zero)
    | (Sign::Plus, Sign::Plus)
    | (Sign::Minus, Sign::Minus) => d,
    (Sign::Zero, Sign::Minus)
    | (Sign::Plus, Sign::Minus)
    | (Sign::Minus, Sign::Zero)
    | (Sign::Minus, Sign::Plus) => two_compl(d),
```

Since the function returns early when `second_sign` is `Sign::Zero`, these clauses can be removed.

### In the `log2floor` function

Similarly, the return statement of the `log2floor` function is unreachable because all paths through the loop return before reaching it.

```rust filepath line=27 highlight=[2, 7,11,13]
evm_interpreter/src/u256.rs
pub(crate) fn log2floor(value: &U256) -> u64 {
    assert!(value != &U256::ZERO);
    let mut l: u64 = 256;
    for i in 0..4 {
        let i = 3 - i;
        if value.as_limbs()[i] == 0u64 {
            l -= 64;
        } else {
            l -= value.as_limbs()[i].leading_zeros() as u64;
            if l == 0 {
                return l;
            } else {
                return l - 1;
            }
        }
    }
    l
}
```

The line 33 may look like there can be a scenario, in which the loop does not return early. However, if all the 4 iteration of the loop reach the line 33, that means that the `value` is zero what contradicts to the assertion on line 28.

## Potential for simplification

```rust filepath line=106 highlight=[2]
system_hooks/src/precompiles.rs
// cheat
let snapshot = system.memory.start_memory_frame();
let mut buffer = QuasiVec::new(system);
let result = F::execute(&calldata, &mut buffer, &mut resources, allocator);
```

```rust filepath line=122 highlight=[4]
system_hooks/src/precompiles.rs
} else {
    system.memory.empty_immutable_slice()
};
system.memory.finish_memory_frame(Some(snapshot));
```

The value `snapshot` is not actually utilized, since no other frames are being started between the frame creation and release. The `snapshot` value is simply the number of frames on the stack at the moment of opening new fram. Passing `None` during the frame release would have the same effect, in the current scenario. It is possible to discard the `snapshot` variable completely and simplify the flow:

```rust
system.memory.start_memory_frame();
...
system.memory.finish_memory_frame(None);
```

[page-break]

## Unnecessary validations

```rust filepath line=18 highlight=[3]
basic_system/src/system_functions/mod.rs
#[inline(always)]
fn bytereverse(input: &mut [u8]) {
    assert!(input.len() % 2 == 0);
```

This validation does not prevent any issue. If the `input` slice contains odd number of elements, the central element remains on its place, as expected.

## Structure `WarmStorageValue` and its field `initial_value_used`

```rust filepath context line=571 highlight=[1,2,7]
basic_system/src/system_implementation/flat_storage_model/storage_cache.rs
pub fn iter_as_storage_types(
// Using the WarmStorageValue temporarily till it's outed from the codebase. We're
// not actually 'using' it.
WarmStorageValue {
    current_value: *current_record.value(),
    is_new_storage_slot: initial_record.appearance() == Appearance::Unset,
    initial_value: *initial_record.value(),
    initial_value_used: true,
    ..Default::default()
}
```

The `initial_value_used` field of the `WarmStorageValue` structure is never initialized with values other than `true`. Additionally, the comment, located in the only location where the `WarmStorageValue` structure is constructed, states that this structure should be removed.

```rust filepath line=242 highlight=[5]
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
for (key, value) in reads_iter {
    let flat_key = derive_flat_storage_key(&key.address, &key.key);
    // reads
    let expect_new = value.is_new_storage_slot;
    assert!(value.initial_value_used);
```

As a consequence, the validation of the `initial_value_used` field, located in the batch verification code, is redundant.

[page-break]

## Commented-out code

```rust filepath line=247 highlight=[2,3,4,5,6]
basic_system/src/system_implementation/flat_storage_model/simple_growable_storage.rs
if expect_new {
    // assert_eq!(
    //     value.initial_value,
    //     WarmStorageValue::TRIVIAL_VALUE,
    //     "initial value of empty slot must be trivial"
    // );
    num_nonexisting_reads += 1;
```

## Unused error messages

These error variants are not utilized in the codebase. This is not only a redundancy but, sometimes, can indicate a missed corner case:

```rust filepath context line=28
basic_bootloader/src/bootloader/errors.rs
pub enum InvalidTransaction
/// EIP-1559: `gas_price` is less than `basefee`.
GasPriceLessThanBasefee,
```

```rust line=32
/// Initial gas for a Call is bigger than `gas_limit`.
///
/// Initial gas for a Call contains:
/// - initial stipend gas
/// - gas for access list and input data
CallGasCostMoreThanGasLimit,
```

```rust line=45
/// Overflow payment in transaction.
OverflowPaymentInTransaction,
```

```rust line=64
/// Transaction chain id does not match the config chain id.
InvalidChainId,
```

```rust line=66
/// Access list is not supported for blocks before the Berlin hardfork.
AccessListNotSupported,
```

[page-break]

## Recommendation

Remove unreachable code branches to improve code clarity and maintainability. Consider using exhaustive pattern matching without impossible cases. Simplify complex data flows and remove unnecessary validations, unused error messages, commented-out code and outdated comments.

# Unimplemented functionality

## Block timestamp and state root hash validation

```rust filepath context line=486 highlight=[7]
basic_system/src/system_implementation/system/io_subsystem.rs
fn finish
// chain state before
let chain_state_commitment_before = ChainStateCommitment {
    state_root: state_commitment.root,
    next_free_slot: state_commitment.next_free_slot,
    block_number: block_metadata.block_number - 1,
    last_256_block_hashes_blake: blocks_hasher.finalize().into(),
    // TODO(EVM-1080): we should set and validate that current block timestamp >= previous
    last_block_timestamp: 0,
};
```

```rust line=526 highlight=[7]
// chain state after
let chain_state_commitment_after = ChainStateCommitment {
    state_root: state_commitment.root,
    next_free_slot: state_commitment.next_free_slot,
    block_number: block_metadata.block_number,
    last_256_block_hashes_blake: blocks_hasher.finalize().into(),
    // TODO(EVM-1080): we should set and validate that current block timestamp >= previous
    last_block_timestamp: 0,
};
```

```rust filepath context line=620
let mut full_root_hasher = crypto::sha3::Keccak256::new();
full_root_hasher.update(self.logs_storage.tree_root().as_u8_ref());
full_root_hasher.update([0u8; 32]); // aggregated root 0 for now
```

## Backward-compatibility events related to withdrawals

```rust filepath context line=186
system_hooks/src/l2_base_token.rs
// TODO: emit event for withdrawal for Era compatibility
Ok(Ok(&[]))
```

```rust filepath line=286
fn l2_base_token_hook_inner
// TODO: emit event for Era compatibility
Ok(Ok(&[]))
```

## Nonce holder

```rust filepath line=56
system_hooks/src/lib.rs
// Temporarily disabled, only used for AA.
// pub mod nonce_holder;
```

```rust line=212
// /// Adds nonce holder system hook.
// ///
// pub fn add_nonce_holder(&mut self) {
//     self.add_hook(NONCE_HOLDER_HOOK_ADDRESS_LOW, nonce_holder_hook)
// }
```

```rust filepath line=13
system_hooks/src/nonce_holder.rs
// it's actually stateless since we can immediately delegate a functionality to system itself
```

The contract is not actually utilized. During transaction processing, account nonces are retreived from the oracle via the `basic_system` layer. Constants `NONCE_HOLDER_HOOK_ADDRESS` and `ACCOUNT_PROPERTIES_STORAGE_ADDRESS` have the same value of `x8003`.

```rust filepath context line=61
system_hooks/src/nonce_holder.rs
pub fn nonce_holder_hook
// TODO: ensure onlySystemCall
```

## Factory deps during L1 transaction preparation

```rust filepath context line=147
basic_bootloader/src/bootloader/process_transaction.rs
fn process_l1_transaction
// TODO: l1 transaction preparation (marking factory deps)
let chain_id = system.get_chain_id();
```

```rust context line=771
fn transaction_execution
// TODO: factory deps? Probably fine to ignore for now
```

# Using raw `u8` type instead of `enum`

In `runner.rs:1001-1009`, raw `u8` values is used for `ExecutionEnvironmentType` instead of enum:

```rust filepath line=1001
basic_bootloader/src/bootloader/runner.rs
fn create_ee<S: EthereumLikeTypes>(
    ee_type: u8,
    system: &mut System<S>,
) -> Result<Box<SupportedEEVMState<'static, S>, S::Allocator>, InternalError> {
    Ok(Box::new_in(
        SupportedEEVMState::create_initial(ee_type, system)?,
        system.get_allocator(),
    ))
}
```

Consider using `ExecutionEnvironmentType` enum throughout for better type safety and code maintainability.
