---
description: Basic introduction on Solidity Smart Contracts
icon: user-robot-xmarks
---

# Smart Contracts & Solidity

### What is a Smart Contract?

Smart contracts are computer programs stored on the blockchain. They produce a deterministic output for every input and are considered immutable (unless implementing UUPS or Proxy patterns for upgradability).

The properties of a smart contract includes:

* **Immutability**: Code cannot be changed once deployed, which necessitates deploying a new instance.
* **Deterministic:** For every same input, every node running it will produce the same output.&#x20;
* **Atomic:** Either transaction completes or not executed at all when an error occurs. If an error occurs, the EVM halts execution of that specific transaction and reverts all state changes to the pre-transaction state, so the network can continue to operate without interruption.
* **Single-threaded:** Every transaction is handled sequentially, and operate in a single-threaded manner.

***

### What is the Ethereum Virtual Machine (EVM)?

A virtual machine is software that acts as an abstraction layer between code and the machine. It allows for portable execution, meaning it produces the same output across different physical machines, just like the Java Virtual Machine (JVM).

The EVM is the runtime environment for executing smart contracts. It executes programs that have been compiled into EVM Opcodes, which behave similarly to assembly instructions. For every function call, the opcodes are processed on the EVM stack, while accessing data from storage (persistent on-chain state variables) or memory (temporary data during function calls).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

_The EVM architecture, from_ [_https://www.saxenism.com/blog/evm-deep-dive_](https://www.saxenism.com/blog/evm-deep-dive)

This architecture is crucial for enabling nodes running on entirely different operating systems to maintain the same functionalities and execution consistency across the entire Ethereum network.

When you call a function, you are technically sending a message (transaction) to the Ethereum network. The EVM uses specific fields in this message to route your request.

<table data-header-hidden><thead><tr><th width="103.55555725097656">Field</th><th>Description</th></tr></thead><tbody><tr><td><strong>Field</strong></td><td><strong>Description</strong></td></tr><tr><td><code>to</code></td><td>The Destination. The address of the smart contract you want to interact with.</td></tr><tr><td><code>value</code></td><td>The Payment. The amount of ETH (in wei) to send along with the call. (Optional, 0 by default).</td></tr><tr><td><code>data</code></td><td>The Instructions (Calldata). A hexadecimal string containing the Function Selector (first 4 bytes) and Arguments (encoded parameters).</td></tr></tbody></table>

Once the transaction reaches the contract, the EVM follows these steps:

1. **Dispatch**: The EVM reads the first 4 bytes of the `data` (the Function Selector). It compares this against the contract's functions.
   * _Match Found:_ Jumps to the specific code for that function.
   * _No Match:_ Triggers the `fallback()` function (if it exists) or reverts the transaction immediately.
2. **Execution**: The EVM spins up a fresh Stack and Memory for this specific call.
   * Stack/Memory: Temporary workspaces for calculations. Removed after execution.
   * Storage: The persistent database of the contract where state vars are stored.
3. **Outcome**:
   * Success: The transaction finishes, update blockchain state, and any return values are sent back.
   * Revert: If an error occurs (e.g., `require` fails), the EVM halts. All changes to Storage are undone. Gas used up to that point is not refunded.

***

### EVM Opcodes, Storage, and Memory

Opcodes are the lowest level of instructions the EVM executes. Each opcode is represented as a single byte like 0x01. Each opcode has a specific gas cost based on its complexity.

<table data-header-hidden><thead><tr><th width="97.66667175292969"></th><th width="159.88888549804688"></th><th width="296.5555419921875"></th><th width="150.4322509765625"></th></tr></thead><tbody><tr><td><strong>Opcode</strong></td><td><strong>Mnemonic</strong></td><td><strong>Description</strong></td><td><strong>Gas Cost</strong></td></tr><tr><td><code>0x01</code></td><td>ADD</td><td>Adds the top two items on the stack.</td><td>3</td></tr><tr><td><code>0x35</code></td><td>CALLDATALOAD</td><td>Reads input data from the transaction.</td><td>3</td></tr><tr><td><code>0x51</code></td><td>MLOAD</td><td>Reads a word from volatile Memory.</td><td>3+</td></tr><tr><td><code>0x54</code></td><td>SLOAD</td><td>Reads from the global database.</td><td>100 (Warm) - 2100 (Cold)</td></tr><tr><td><code>0x55</code></td><td>SSTORE</td><td>Writes a word to persistent Storage. </td><td>2900 - 20000+</td></tr></tbody></table>

When a function executes an operation like `1 + 1`, the EVM performs a specific sequence of stack operations. It first uses `PUSH1` to place each value onto the Stack, which is the only place where computation occurs. The `ADD` opcode then "pops" (removes) the top two items, calculates the sum, and pushes the result back onto the stack. This result remains temporary until it is explicitly moved elsewhere—either saved to volatile Memory using `MSTORE` (costing only \~3 gas) or written to persistent Storage using `SSTORE`.

The distinction in cost between Memory and Storage is driven by persistence and global consensus. Memory acts as temporary RAM, reading and writing here is cheap (\~3 gas) because it requires no disk access. Storage, however, is the permanent "hard drive" of the blockchain. Writing to it (`SSTORE`) is gas expensive because every node in the network must update the global state trie. This is why developers often "cache" storage variables in memory for calculations before writing them back.
