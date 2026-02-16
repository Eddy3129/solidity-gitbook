---
description: A brief history of Ethereum
icon: circle-chevron-up
---

# Ethereum Upgrades

This section aims to provide a brief history of past Ethereum upgrades, hard forks, and the crucial Ethereum Improvement Proposals (EIPs) introduced in each upgrade that have shaped Ethereum into what it is today. Much of the information in this section is sourced from the [official Ethereum documentation on network upgrades](https://ethereum.org/ethereum-forks).

### The Inception (2013 - 2014)

Vitalik Buterin first published the introductory [Ethereum whitepaper](https://ethereum.org/whitepaper/) in 2013, proposing the initial vision of Ethereum as a “world computer” capable of executing decentralized applications beyond the scope of Bitcoin. The paper outlined a blockchain platform designed to support programmable smart contracts and general-purpose computation.

In 2014, Gavin Wood introduced the [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf), which formally defined the Ethereum protocol. It provided a rigorous technical specification of the Ethereum Virtual Machine (EVM), opcode structure, computational model, and overall system architecture, establishing the foundation for client implementations and network consensus.

In August 2014, [Ether was officially offered for sale](https://blog.ethereum.org/2014/07/22/launching-the-ether-sale) at an initial rate of 2,000 ETH per BTC as part of Ethereum’s public crowdsale. The capital raised enabled the project’s development and launch, laying the foundation for what has since become one of the most important public blockchain platforms today.

***

### Frontier (2015)

Frontier marked the official launch of the Ethereum mainnet on July 30, 2015, operating under a Proof-of-Work consensus mechanism. Shortly after, the protocol also incorporated the difficulty bomb, which progressively increased mining difficulty to incentivize the network’s eventual migration to Proof-of-Stake.

***

### Homestead (2016)

The Homestead upgrade improved the security and reliability of the Ethereum network. It optimized transaction and contract creation behavior through EIP-2, introduced the `DELEGATECALL` opcode to allow contracts to execute code from external contracts while preserving the caller’s state and context, and enhanced peer-to-peer protocol compatibility via EIP-8 to support future network upgrades.

***

### DAO Fork (2016)

Perhaps one of the most impactful early events in Ethereum’s history, the DAO fork was initiated following the exploitation of _The DAO_, a decentralized venture fund implemented as a smart contract. The attacker drained approximately 3.6 million ETH by leveraging a reentrancy vulnerability in the contract’s code. In response, the community became divided: one group supported a hard fork to restore the stolen funds by rewriting the blockchain’s state, while another opposed the intervention on the grounds of immutability.

This disagreement resulted in Ethereum’s first major chain split in 2016, creating two separate networks: Ethereum, which adopted the state change, and Ethereum Classic, which continued on the original chain and still operates under Proof-of-Work today.

***

### Tangerine Whistle & Spurious Dragon (2016)

Following a series of denial-of-service (DoS) attacks that exploited underpriced opcodes, Ethereum introduced several protocol changes to improve network resilience. [EIP-150](https://eips.ethereum.org/EIPS/eip-150) repriced state-intensive opcodes to better align gas costs with computational resources, significantly raising the cost of spam attacks. [EIP-161](https://eips.ethereum.org/EIPS/eip-161) implemented state-clearing mechanisms to remove empty accounts and limit state bloat. Additionally, [EIP-155](https://eips.ethereum.org/EIPS/eip-155) introduced chain IDs to transactions, preventing replay attacks across different chains after the network split.

***

### Byzantium (2017)

An important upgrade that strengthened Ethereum’s security, cryptographic capabilities, and smart contract reliability. This marked a crucial step in Ethereum’s evolution from a platform primarily focused on executing smart contracts to a more secure network capable of supporting advanced cryptography (SNARKs) and privacy-preserving applications.

<table data-header-hidden><thead><tr><th width="109.44439697265625"></th><th width="220.33331298828125"></th><th></th></tr></thead><tbody><tr><td><strong>EIP</strong></td><td><strong>Feature Introduced</strong></td><td><strong>Significance</strong></td></tr><tr><td><strong>EIP-649</strong></td><td>Difficulty bomb delay &#x26; block reward reduction</td><td>Delayed the difficulty bomb and reduced block rewards from 5 ETH to 3 ETH</td></tr><tr><td><strong>EIP-140</strong></td><td><code>REVERT</code> opcode</td><td>Enabled state rollback on failure.</td></tr><tr><td><strong>EIP-198</strong></td><td>Modular exponentiation precompile</td><td>Enabled efficient modular exponentiation, supporting practical on-chain RSA verification</td></tr><tr><td><strong>EIP-214</strong></td><td><code>STATICCALL</code> opcode</td><td>Allowed read-only external contract calls.</td></tr><tr><td><strong>EIP-658</strong></td><td>Transaction status codes</td><td>Distinguishes successful and failed transactions.</td></tr></tbody></table>

***

### Constantinople (2019)

A refinement-focused upgrade that reduced block rewards from 3 ETH to 2 ETH while delaying the difficulty bomb. Key improvements included EIP-1052 (`EXTCODEHASH`) for more gas-efficient contract verification and [EIP-1014](https://eips.ethereum.org/EIPS/eip-1014) (`CREATE2`), which enabled deterministic contract addresses before deployment, allowing funds and transactions to be prepared in advance.

***

### Istanbul (2019)

An upgrade focused on gas repricing to improve efficiency and support Layer-2 scaling. [EIP-2028](https://eips.ethereum.org/EIPS/eip-2028) reduced calldata costs, allowing more data per transaction and lowering the cost of <mark style="color:$warning;">rollup-based solutions</mark>. EIP-1344 introduced the `CHAINID` opcode, enabling smart contracts to verify the active chain and improving safety for cross-chain and Layer-2 applications. Additionally, EIP-152 introduced a BLAKE2b precompile to support efficient cross-chain cryptography, including cheaper verification of data from networks such as Zcash.

***

### Muir Glacier (2020)

A delay of the difficulty bomb and increasing mining dificulty.

***

### Beacon Chain & Staking (2020)

The official staking of Ethereum was introduced with the launch of the <mark style="color:$success;">Beacon Chain</mark>, a parallel Proof-of-Stake consensus network requiring a minimum deposit of 32 ETH to operate a validator node. During this period, the execution layer continued to operate under Proof-of-Work, while the Beacon Chain established the Proof-of-Stake consensus layer to prepare for The Merge.

***

### Berlin (2021)

Berlin introduced gas optimizations to improve transaction efficiency. [EIP-2718 ](https://eips.ethereum.org/EIPS/eip-2930)implemented a typed transaction envelope, establishing a flexible framework for future transaction formats. Building on this, [EIP-2930](https://eips.ethereum.org/EIPS/eip-2930) introduced Type 1 transactions with access lists, allowing transactions to declare in advance which storage slots and addresses they will access, making gas more predictable. The upgrade also increased gas costs for state access to further mitigate DoS attacks.

***

### London (2021)

One of Ethereum’s most significant upgrades, [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) introduced Type 2 transactions and transformed the fee market by implementing a dynamically adjusting base fee based on block demand, alongside a priority fee to incentivize miners (prior to The Merge) and validators thereafter. The base fee is burned, reducing ETH supply, and transaction fees are calculated as <mark style="color:$warning;">**gas used × (base fee + priority fee)**</mark>. [EIP-3198](https://eips.ethereum.org/EIPS/eip-3198) further enabled contracts to access the block’s base fee at the opcode level.

Another key change, [EIP-3529](https://eips.ethereum.org/EIPS/eip-3529), removed gas refunds for `SELFDESTRUCT` and reduced refunds for `SSTORE`. While refunds were originally intended to encourage clearing unused storage, they were exploited through mechanisms such as GasToken to store gas when fees were low and redeem it during high-fee periods. Removing these refunds helped limit state growth and stabilize block gas usage.

***

### Altair (2021)

This marked the first major upgrade to the Beacon Chain. It introduced <mark style="color:$warning;">sync committees</mark>, where randomly selected validator groups (512 validators) provide <mark style="color:$warning;">aggregated signatures</mark> that enable <mark style="color:$warning;">light clients</mark> to verify the chain securely with minimal data. The upgrade also increased inactivity and slashing penalties to improve validator accountability and finalized key economic parameters for staking participation.

***

### Arrow Glacier (2021) & Gray Glacier (2022)

Hmmm, another delay of the difficulty bomb...

***

### Bellatrix (2022)

Final preparation for proof-of-work to proof-of-stake. It updated the fork choice rule to guide validators toward the canonical chain, increased penalties to strengthen validator reliability, and introduced Merge logic that allowed the Beacon Chain to take over block production once the Terminal Total Difficulty (TTD) was reached.

***

### Paris - The Merge (2022)

The highly anticipated transition to Proof-of-Stake ([EIP-3675](https://eips.ethereum.org/EIPS/eip-3675))! The execution layer transitioned from PoW to PoS once the TTD was reached, officially ending PoW on Ethereum. This upgrade connected execution clients with consensus clients via the [Engine API](https://hackmd.io/@danielrachi/engine_api), enabling coordinated block production under PoS. It also introduced a verifiable randomness source through the `PREVRANDAO` opcode ([EIP-4399](https://eips.ethereum.org/EIPS/eip-4399)), supporting validator selection and other protocol functions.

***

### Shanghai-Capella ("Shapella") (2023)

This upgrade enabled <mark style="color:$warning;">validator withdrawals</mark> through [EIP-4895](https://eips.ethereum.org/EIPS/eip-4895), allowing staked ETH to be transferred from the consensus layer to the execution layer and completing Ethereum’s staking lifecycle. The upgrade also introduced gas optimizations, including EIP-3651, which warmed the block proposer address to reduce gas costs for validator payments and improve transaction efficiency, and deprecated the `SELFDESTRUCT` opcode (EIP-6049), shift away from destructive contract patterns.

***

### Dencun (2024)

One of the most significant upgrades toward Layer-2 scaling, this upgrade introduced [EIP-4844](https://www.eip4844.com/) (proto-danksharding), which adds a “blob” data structure for storing Layer-2 transaction data and significantly reduces data availability costs. Blobs operate under a separate fee market from traditional Type 2 transactions, priced dynamically through a blob base fee, while EIP-7516 introduced the `BLOBBASEFEE` opcode to allow contracts to read this value.

Blobs use a separate fee market priced through a blob base fee, while [EIP-7516](https://eips.ethereum.org/EIPS/eip-7516) added the `BLOBBASEFEE` opcode so contracts can read this value. Blob contents are consensus-layer data that are not accessible to the EVM (aside from their cryptographic commitments). Because they are not executed, do not modify state, and are prunable, blobs are significantly cheaper than calldata for data availability.

Today, validators download blob sidecars to verify availability before attesting to a block, introducing blob-carrying transactions with dynamically targeted data bandwidth. This is far more scalable than reducing calldata costs, which would permanently increase block and state size.

Blobs are intentionally ephemeral: nodes must serve the data for a limited retention window (\~18 days) after which it can be pruned. Long-term storage is expected to be handled by rollups, archival providers, or decentralized systems like the [Portal Network](https://ethportal.net/), while the consensus layer ensures the data was published and available long enough for replication.

Each blob is represented by a KZG commitment, enabling efficient verification via point-evaluation proofs (P(x)=y), a primitive particularly useful for ZK rollups. EIP-4844 serves as a step toward full danksharding, where data availability sampling will eliminate the need for validators to download entire blobs.

The upgrade also included several other important EIPs:

<table data-header-hidden><thead><tr><th width="102.33331298828125"></th><th width="215.7777099609375"></th><th></th></tr></thead><tbody><tr><td><strong>EIP</strong></td><td><strong>Change Introduced</strong></td><td><strong>Significance</strong></td></tr><tr><td><strong>EIP-1153</strong></td><td>Transient storage opcodes</td><td>Enables temporary storage cleared after each transaction, supporting more gas-efficient patterns such as reentrancy guards and multicall designs.</td></tr><tr><td><strong>EIP-4788</strong></td><td>Beacon block roots accessible to EVM</td><td>Allows trust-minimized access to consensus data, improving designs for staking protocols, bridges, and oracle systems.</td></tr><tr><td><strong>EIP-5656</strong></td><td><code>MCOPY</code> opcode</td><td>Provides more efficient memory copying than <code>MLOAD</code>/<code>MSTORE</code></td></tr><tr><td><strong>EIP-6780</strong></td><td>Redefined <code>SELFDESTRUCT</code> behavior</td><td>Redefines <code>SELFDESTRUCT</code> so contracts are no longer deleted except when invoked in the same creation transaction</td></tr></tbody></table>

***

### Pectra (2025)

The Pectra upgrade focuses on improving validator scalability and expanding account capabilities.

A major change comes from [EIP-7251,](https://eips.ethereum.org/EIPS/eip-7251) which raises the maximum effective balance per validator to 2,048 ETH. This allows large operators to consolidate stake across fewer validators, reducing validator set growth and improving consensus efficiency while simplifying operations.

Meanwhile, [EIP-7002](https://eips.ethereum.org/EIPS/eip-7002) introduces execution-layer triggered exits and withdrawals. By allowing withdrawal credentials to be controlled programmatically from the execution layer, it enables more flexible and trust-minimized staking designs such as automated exits and contract-managed validators.

On the account side, [EIP-7702](https://eip7702.io/) enhances wallet UX by allowing externally owned accounts to <mark style="color:$warning;">temporarily delegate execution to smart contract code within a transaction</mark>. This enables features like transaction batching, gas sponsorship, and programmable spending, representing a pragmatic step toward broader account abstraction.

***

### FuSaka (2025)

The main purpose of the Fusaka upgrade is to extend the Layer-2 scaling capabilities introduced in the Dencun upgrade. [EIP-7594](https://eips.ethereum.org/EIPS/eip-7594) (Peer Data Availability Sampling, or PeerDAS) removes the need for validators to download full blobs during verification. Instead, blob data is erasure-coded into shares and distributed across the peer-to-peer network, allowing nodes to verify availability by sampling small portions of the data rather than replicating it entirely. I have built a simple [demo](https://eip-7594-peer-das-explainer.vercel.app/) to visualize this.

With erasure coding, the full dataset can be reconstructed from roughly 50% of the shares, providing strong guarantees that the data was available at publication time while significantly reducing bandwidth requirements per node.

Another economically important EIP is [EIP-7918](https://eips.ethereum.org/EIPS/eip-7918). It introduces a reserve price for blob fees, ensuring Layer-2 protocols pay a baseline cost for data availability even during periods of low demand. As network activity increases, blob fees scale accordingly, aligning Layer 2 demand with validator incentives while avoiding underpriced blockspace.

The upgrade also proposes [EIP-7825](https://eips.ethereum.org/EIPS/eip-7825), which sets a cap on the maximum gas a single transaction can consume. This prevents any transaction from monopolizing block gas, reducing DoS risk, limiting excessive state growth, and lowering validation overhead.

Some other important EIPs includes:

<table data-header-hidden><thead><tr><th width="108.5555419921875">EIP</th><th width="201.66668701171875">Title</th><th>Purpose</th></tr></thead><tbody><tr><td><strong>EIP</strong></td><td><strong>Title</strong></td><td><strong>Purpose</strong></td></tr><tr><td>EIP-7951</td><td>secp256r1 Precompile (Passkey Support)</td><td>Introduces support for hardware-backed signatures, enabling passkeys and device-native authentication. </td></tr><tr><td>EIP-7935</td><td>Block Gas Limit Increase</td><td>Expands execution capacity by allowing more transactions per block from 36M to 60M.</td></tr><tr><td>EIP-7892</td><td>Blob-Parameter-Only Forks</td><td>Enables blob capacity to be adjusted without major hard forks.</td></tr><tr><td>EIP-7934</td><td>RLP Execution Block Size Limit</td><td>Caps execution payload size to 10 MiB and limits the RLP-encoded block to 8 MiB, improving propagation predictability and reducing oversized-block risks.</td></tr></tbody></table>
