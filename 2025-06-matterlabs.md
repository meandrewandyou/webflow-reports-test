---
title: ZKsync OS
client: Matter Labs
created_at: '2025-08-04'
updated_at: '2025-10-15'
draft: false
---

<script>
  import Levels from '$lib/components/levels.svelte';
  import LevelText from '$lib/components/colored-level.svelte';
  import StatusText from '$lib/components/colored-status.svelte';
</script>

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

- **Operation layer** is a Rust-based program that can be upgraded independently of the proving circuits, enabling seamless upgrades and customization for app-specific chains.

- **Proving layer** can be fully optimized or replaced without affecting the operation layer, ensuring long-term flexibility and innovation.

## Operation Layer

The operation layer serves as the system-level implementation of the chain’s state transition function. It manages memory and I/O operations to maximize performance and efficiency. Additionally, it supports multiple virtual machines, each tailored to different use cases:
- **EVM (Ethereum Virtual Machine):** Ensures full compatibility with Ethereum.
- **EraVM:** Maintains backward compatibility with existing applications.
- **WASM (WebAssembly):** Expands possibilities by allowing smart contracts in various programming languages.
- **Native RISC-V Contracts:** Delivers maximum performance for high-efficiency use cases.

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

# Test heading
