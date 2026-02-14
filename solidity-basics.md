---
description: Entry-level knowledge to Solidity
icon: square-binary
---

# Solidity Basics

### What is Solidity?

Solidity, created by Gavin Wood, is a JavaScript-like, object-oriented, and imperative programming language similar to C++ or Java.&#x20;

However, imperative languages are prone to side effects because code and data are often mixed. This allows state to be modified at unintended moments, leading to invalid state updates and potential vulnerabilities.

This causes issues such as the **Reentrancy attack.**&#x20;

```solidity
function withdraw() public {
        uint amount = balances[msg.sender];
        
        // 1. CHECKS
        require(amount > 0, "No balance");

        // 2. INTERACTIONS (The Flaw: Sending ETH before updating balance)
        // This hands control to the attacker!
        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");

        // -----------------------------------------------------
        // <--- ATTACKER RE-ENTERS HERE (Balance is still high!)
        // -----------------------------------------------------

        // 3. EFFECTS (Too late!)
        balances[msg.sender] = 0;
    }
```

In this scenario, because instructions execute sequentially, an attacker can exploit the logic flow by "re-entering" (repeatedly calling) the smart contract to trigger Ether transfers before the internal balance is updated. This creates a window for malicious actors to drain funds until the transaction runs out of gas.

To prevent this, Solidity developers must strictly adhere to the **CEI (Checks, Effects, Interactions)** pattern. By updating the state _before_ performing any external interactions, developers eliminate the window for re-entry.

```solidity
function withdraw() public {
        uint amount = balances[msg.sender];

        // 1. CHECKS
        require(amount > 0, "No balance");

        // 2. EFFECTS (Update the state FIRST)
        // The attacker's balance is now 0.
        balances[msg.sender] = 0; 

        // 3. INTERACTIONS (Send ETH last)
        // If they try to re-enter now, the Check at Step 1 will fail (0 > 0 is false).
        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");
    }
```

***

### Variables

Variables in Solidity can be generalized into three types: state variables, local variables, and global variables.

<table data-header-hidden><thead><tr><th width="97.44451904296875"></th><th></th><th width="146.00006103515625"></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Type</strong></td><td><strong>Declaration Scope</strong></td><td><strong>Data Location</strong></td><td><strong>Lifetime</strong></td><td><strong>Gas Cost</strong></td></tr><tr><td>State</td><td>Inside Contract (outside functions)</td><td>Storage (Blockchain "Hard Drive")</td><td>Permanent (Persists between transactions)</td><td>High (Expensive to write via <code>SSTORE</code>)</td></tr><tr><td>Local</td><td>Inside Function</td><td>Stack, Memory, or Calldata (RAM/CPU)</td><td>Temporary (Wiped after function execution)</td><td>Low (Cheap to use via <code>MSTORE</code>/<code>PUSH</code>)</td></tr><tr><td>Global</td><td>Pre-defined (e.g., <code>msg.sender</code>)</td><td>EVM Context</td><td>Read-Only (Available anywhere)</td><td>Low (Cheap to read)</td></tr></tbody></table>

#### Constant and Immutables

Constants and immutable variables can reduce gas by making their variables read-only.

<table><thead><tr><th width="150.3333740234375">Feature</th><th width="274.22216796875">Constant</th><th>Immutable</th></tr></thead><tbody><tr><td><strong>Timing</strong></td><td>Compile-time — value must be known before deployment.</td><td>Deployment-time — value is set in the constructor.</td></tr><tr><td><strong>Storage</strong></td><td>None — inlined directly into the bytecode.</td><td>Embedded into the <strong>runtime bytecode</strong> during deployment.</td></tr><tr><td><strong>Mechanism</strong></td><td>Compiler replaces references with the literal value.</td><td>Value is assigned once during deployment and embedded into runtime code.</td></tr><tr><td><strong>Gas Savings</strong></td><td>Maximum — no storage access.</td><td>High — cheaper than storage since it avoids <code>SLOAD</code>.</td></tr></tbody></table>

***

### Function Modifiers

Function modifiers are rules and checkpoints of a Solidity function. It can be classified into **visibility modifiers**, **state modifiers**, and **custom&#x20;**~~**modifiers**~~.

#### Visibility Modifiers

Visibility modifiers determines the accessibility of a particular function.

<table data-header-hidden><thead><tr><th width="112.3333740234375"></th><th width="183.66668701171875"></th><th width="97.99981689453125"></th><th></th></tr></thead><tbody><tr><td><strong>Visibility Modifiers</strong></td><td><strong>Who can call?</strong></td><td><strong>Included in ABI?</strong></td><td><strong>Description</strong></td></tr><tr><td><code>public</code></td><td>Everyone (Internal &#x26; External)</td><td>Yes</td><td>Callable by users, other contracts, and internally.</td></tr><tr><td><code>external</code></td><td>Outsiders Only</td><td>Yes</td><td>Callable only from the outside (users/contracts). Cannot be called internally (unless using <code>this.func()</code>).</td></tr><tr><td><code>internal</code></td><td>Family Only (Contract + Derived)</td><td>No</td><td>Accessible only by the contract executing it and any child contracts that inherit from it.</td></tr><tr><td><code>private</code></td><td>Self Only (Contract)</td><td>No</td><td>Accessible <em>only</em> within the contract where it is defined.</td></tr></tbody></table>

***

#### State Modifiers

Meanwhile, state modifiers restrict its capability to read and write to state variables. Depending on the read and write permissions, **those with lesser permissions consume less gas**.

<table data-header-hidden><thead><tr><th width="160.33331298828125"></th><th width="127.22222900390625"></th><th width="127.55560302734375"></th><th></th></tr></thead><tbody><tr><td><strong>State Modifiers</strong></td><td><strong>Read State?</strong></td><td><strong>Write State?</strong></td><td><strong>Purpose</strong></td></tr><tr><td>(Default)</td><td>Yes</td><td>Yes</td><td>Full Access. Used for changing balances or updating data. </td></tr><tr><td><code>view</code></td><td>Yes</td><td>No</td><td>Read-Only. You can look at the data, but you can't touch it.</td></tr><tr><td><code>pure</code></td><td>No</td><td>No</td><td>Zero Contact. It doesn't even know the state exists; it just compute math.</td></tr><tr><td><code>payable</code></td><td>Yes</td><td>Yes</td><td>The Cashier. Same as default, but it’s the only way to accept ETH.</td></tr></tbody></table>

Fun fact: Non-payable functions are actually larger and more expensive than payable ones because the Solidity compiler automatically inserts a safety check at the beginning of the bytecode to reject any incoming ETH.

```solidity
// What the EVM actually sees in a standard function:
if (msg.value > 0) {
    revert("Not Payable"); // Takes up space & gas!
}
// ... then your function logic starts
```

Adding the payable keyword tells the compiler to allow incoming ETH, so it removes the safety check. Result: smaller bytecode and slightly lower gas cost since the EVM skips the check.

***

#### Custom Modifiers

Custom modifiers allow programmers to define rules that control a function’s behavior, scope, and security. They use the `modifier` keyword to add checks or conditions that must be met before the function executes.

```solidity
modifier onlyOwner {
    require(msg.sender == owner, "Not the owner!"); // The Check
    _; // The "Merge" Wildcard, executes function logic after the check
}
```

***

### Events & Error Handling

Now we have an EVM where performing state updates is very expensive, but we need a way to query those updates so the outside world knows what has changed—from simple questions like, did a transaction go through? Did my balance really increase as claimed? For this, we use events and error handling alongside state updates.

Every block produced has a state root and a receipts root. The receipts root can only be read off-chain, not by smart contracts. Hence, it is very cheap to write to and requires no gas to read externally.

Events can be written using the `event` keyword and using the `emit` keyword to trigger the event.

```solidity
// Declaration
event Deposit(address indexed user, uint256 amount);

function deposit() external payable {
    // Triggering the event
    emit Deposit(msg.sender, msg.value);
}
```

Errors can be defined using the `error` keyword. Using `revert` with a custom error is more gas efficient than `require` with a string message because the error is compiled into a 4-byte selector instead of storing the entire string in memory.

```solidity
// Custom Error Declaration (Cheaper)
error InsufficientFunds(uint256 available, uint256 required);

function withdraw(uint256 amount) external {
    // OLD WAY (Expensive String)
    // require(balance[msg.sender] >= amount, "Insufficient funds");

    // NEW WAY (Gas Efficient)
    if (balance[msg.sender] < amount) {
        revert InsufficientFunds(balance[msg.sender], amount);
    }
}
```

Fallback functions act as a safety net for a smart contract. They are automatically triggered when a call does not match any existing function signature. They also execute when ETH is sent with calldata and the contract does not define a `receive()` function.

A `fallback` function is expensive because it has to handle _everything_ the contract doesn't understand. If you use it to accept ETH, it requires the `payable` modifier.

{% hint style="info" %}
Crucial gas limit: If a contract sends ETH to your fallback using `transfer()` or `send()`, it only forwards 2300 gas, barely enough to emit a log.&#x20;

If your fallback attempts to write to storage (`SSTORE`), the transaction will run out of gas and fail. That is why using `call` is generally preferred, as it forwards configurable gas and reduces the risk of unintended failures.
{% endhint %}

```solidity
// NOT recommended
payable(recipient).transfer(amount); 
// or
bool success = payable(recipient).send(amount);

// Recommended
(bool success, ) = payable(recipient).call{value: amount}("");
require(success, "ETH transfer failed");
```

