---
description: >-
  A deep dive into proxy contracts, UUPS, Transparent Upgradeable Proxies, and
  Diamond Proxies.
icon: comment-arrow-up
---

# Upgradable Smart Contracts

Smart contracts are designed to be immutable on Ethereum to preserve the credibility of “_**code is law**_”, ensuring deployed logic cannot be easily altered. However, vulnerabilities may surface after deployment, and protocols often need to evolve with new features.

Proxy contracts enable upgradeability by separating storage from logic. The proxy serves as the primary interaction layer, holding all state while delegating execution to an implementation contract via `delegatecall`. Because the call executes in the proxy’s context, contract logic can be upgraded without migrating state or changing the address users interact with.

### Eternal Storage

A brute-force way of preserving state across upgrades is to store state variables in a separate storage contract, allowing the implementation contract to update values through **setter** functions and retrieve them via **getters**.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/*  
    Storage contract holds ALL state.     
    Logic contracts can be replaced without losing data.
*/
contract EternalStorage {
    mapping(bytes32 => uint256) private uintStorage;

    function getUint(bytes32 key) external view returns (uint256) {
        return uintStorage[key];
    }

    function setUint(bytes32 key, uint256 value) external {
        uintStorage[key] = value;
    }
}

/*
    Logic contract reads/writes from storage.
    If we deploy a new logic contract, we reuse the same storage.
*/
contract Ballot {
    EternalStorage public storageContract;
    bytes32 private constant VOTES = keccak256("votes");

    constructor(address _storage) {
        storageContract = EternalStorage(_storage);
    }

    function vote() external {
        uint256 votes = storageContract.getUint(VOTES);
        storageContract.setUint(VOTES, votes + 1);
    }

    function getVotes() external view returns (uint256) {
        return storageContract.getUint(VOTES);
    }
}

```

However, because the storage contract remains permanently deployed, strict access control is required to prevent unauthorized writes from outdated or malicious implementations. This approach also introduces additional external calls, increasing gas costs and operational complexity compared to proxy patterns that rely on `delegatecall` to share storage.

While Eternal Storage was an early upgradeability strategy, modern proxy architectures are generally preferred for production systems due to stronger safety guarantees and simpler developer workflows.

***

### First Proxy

In this early proxy design, users interact with a contract called the _**Dispatcher**_, which acts as the execution entry point. When a function is called, the Dispatcher forwards the call using `delegatecall` to an implementation contract containing the logic.

Unlike a regular external call, `delegatecall` executes the implementation’s code in the context of the Dispatcher. When a user calls a function that does not exist on the Dispatcher, the <mark style="color:$warning;">fallback function</mark> is triggered and forwards the calldata to the implementation via `delegatecall`.&#x20;

This effectively turns the Dispatcher into a universal execution router. Because execution occurs in the Dispatcher’s storage context, <mark style="color:$warning;">the implementation does not store or return state</mark> <mark style="color:$warning;">— it directly reads from and writes to the proxy’s storage</mark>. As a result, upgrading the implementation changes the contract’s behavior while preserving its state.

<div data-full-width="true"><figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure></div>

This pattern established an important constraint: <mark style="color:$primary;">implementation contracts should not use constructors</mark>. Instead, state must be initialized through an initializer function invoked by the dispatcher via `delegatecall`. Because the initializer writes to the dispatcher’s storage, it must be protected to ensure it can only be executed once.

***

### Storage Collisions

Storage slots within contracts must be handled carefully when designing for upgradeability. Since `delegatecall` executes the implementation’s logic using the proxy’s storage, both contracts need to maintain the same storage layout. If variables are reordered, removed, or new ones are introduced before existing slots, they may overwrite important data already stored in the proxy, corrupting the contract’s state.

For example:

**Implementation V1**

```solidity
contract V1 {
    address public owner;   // slot 0
    uint256 public balance; // slot 1
}
```

**Implementation V2 (Bad Upgrade)**

```solidity
contract V2 {
    uint256 public newVariable; // slot 0  ❗ overwrites owner
    address public owner;      // slot 1  ❗ now points to old balance
    uint256 public balance;    // slot 2
}
```

Although this looks harmless, the proxy still stores the original data based on the old layout. After upgrading, `newVariable` will read the value that previously belonged to `owner`, effectively corrupting the contract’s state. In this scenario, control of the contract may be lost, as the real owner address is no longer stored in the expected slot.

A common way to mitigate this risk is by inheriting from the previous contract instead of redeclaring its storage. This preserves the original slot ordering and ensures new variables are appended rather than overwriting existing data.

```solidity
contract V2 is V1 {
    uint256 public newVariable; // appended to slot 2
}
```

***

### EIP-897 (The First Real Proxy)

This proposal introduced an early attempt to standardize proxy architecture by <mark style="color:$warning;">defining a delegation interface that exposes the implementation address</mark>, which greatly improved transparency. Early designs often used shared storage contracts to align storage layouts, though later proxy patterns moved toward more flexible storage strategies.

Storage collisions can be avoided by enforcing a shared storage layout through inheritance. Here, `ProxyStorage` acts as the single source of truth for the `otherContractAddress` variable, ensuring both the proxy and implementation reference the same storage slot. `ProxyNoMoreClash` functions as the proxy by setting the active implementation and delegating execution to it.

Example from [Ethereum Blockchain Developer](https://www.ethereum-blockchain-developer.com/advanced-mini-courses/solidity-proxy-pattern-and-upgradable-smart-contracts/eip-897-proxy):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.1;
 
 
contract ProxyStorage {
    address public otherContractAddress;
    
    function setOtherAddressStorage(address _otherContract) internal {
        otherContractAddress = _otherContract;
    }
}
 
 // Avoid collision using inheritance
contract NotLostStorage is ProxyStorage {
    address public myAddress;
    uint public myUint;
 
    function setAddress(address _address) public {
        myAddress = _address;
    }
    
    function setMyUint(uint _uint) public {
        myUint = _uint;
    }
 
}
 
 
contract ProxyNoMoreClash is ProxyStorage {
    
    constructor(address _otherContract) {
        setOtherAddress(_otherContract);
    }
    
    function setOtherAddress(address _otherContract) public {
        super.setOtherAddressStorage(_otherContract);
    }
    
     /**
  * @dev Fallback function allowing to perform a delegatecall to the given implementation.
  * This function will return whatever the implementation call returns
  */
  fallback() payable external {
    address _impl = otherContractAddress;
 
    assembly {
      let ptr := mload(0x40)
      calldatacopy(ptr, 0, calldatasize())
      let result := delegatecall(gas(), _impl, ptr, calldatasize(), 0, 0)
      let size := returndatasize()
      returndatacopy(ptr, 0, size)
 
      switch result
      case 0 { revert(ptr, size) }
      default { return(ptr, size) }
    }
  }
}
```

However, this design is not ideal because requiring implementations to inherit the shared storage contract tightly couples storage and logic, making upgrades more rigid and error-prone.

***

### EIP-1822 (UUPS)

This evolution led to modern proxy designs such as the Universal Upgradeable Proxy Standard (UUPS).&#x20;

Instead of relying on shared storage through inheritance, UUPS stores the implementation address in a <mark style="color:$warning;">deterministic storage slot</mark> derived from a hash (for example `"PROXIABLE"`). By writing the logic address to a <mark style="color:$warning;">fixed, hard-to-collide location</mark>, the proxy avoids overwriting commonly used storage slots and significantly reduces the risk of collisions.

```solidity
keccak256("PROXIABLE") = "0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7"

// storing the contract address
sstore(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7, contractLogic)

// retrieving the contract address
let contractLogic := sload(0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7)
```

However, it is important to note that <mark style="color:$danger;">state variables still cannot be removed or reordered</mark>. While UUPS provides a safer location for storing the implementation address, the overall storage layout must still obey the rules of the EVM, as the proxy permanently holds the contract state.

Since developers could hash arbitrary variable names to store the implementation address, this approach still lacks standardization. Can we do better?

***

### EIP-1967 (Standard Proxy Storage Slots)

Standardizing proxy storage slots is crucial for transparency and accessibility, particularly for block explorers to reliably associate a proxy contract with its implementation.&#x20;

To support this, Etherscan adopted the slot defined in EIP-1967, where\
`0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc`\
(derived from `bytes32(uint256(keccak256("eip1967.proxy.implementation")) - 1)`) is reserved for storing the implementation address.

It also introduced the concept of the <mark style="color:$success;">Beacon contract</mark>, designed to improve upgrade efficiency when multiple proxies share the same implementation. Instead of upgrading each proxy individually, proxies reference a beacon that stores the implementation address. <mark style="color:$warning;">Updating the beacon automatically points all connected proxies to the new logic</mark>, significantly reducing operational overhead.

Hence, <mark style="color:$warning;">Beacon proxies always query the updated beacon contract</mark> for the current implementation address before delegating execution.

***

### EIP-1167 (Minimal Proxy)

While not part of the upgradeability evolution, ERC-1167 plays a significant role in reducing deployment costs. It defines a minimal proxy with extremely small bytecode that delegates execution to a shared implementation, allowing developers to efficiently <mark style="color:$warning;">replicate contract instances</mark>.

This pattern is commonly used in factory-style architectures, where many lightweight contracts share identical logic but maintain independent state. A common example is <mark style="color:$warning;">DeFi vault deployment</mark> — each vault follows the same operational logic while storing different parameters such as supported tokens, APR, or lending rates.

Unlike upgradeable proxies, minimal proxies are typically designed to remain <mark style="color:$warning;">immutable, prioritizing economic efficiency and predictability</mark> over upgrade flexibility.

***

### EIP-1538 (Transparent Contract)

While earlier proxy patterns focused on upgrading entire contracts, production-grade systems often demand greater modularity and granular control. As a result, a mechanism is needed to route and upgrade individual function selectors without redeploying the full contract.&#x20;

This approach is particularly useful for complex smart contracts that approach or exceed the <mark style="color:$danger;">24KB maximum contract size limit</mark>.

Although EIP-1538 introduced this concept, it was later withdrawn in favor of a more mature approach — the EIP-2535 Diamond Standard.

***

### EIP-2535 (Diamond Proxy)

The EIP-2535 Diamond Standard enables <mark style="color:$warning;">function-level delegation</mark> by routing individual calls to external implementation contracts known as _<mark style="color:$success;">facets</mark>_. Instead of upgrading an entire contract, functions can be added, replaced, or removed selectively.

The flow is simply:  `selector → facet → delegatecall`

```solidity
contract Diamond {

    // maps function selector to facet address
    mapping(bytes4 => address) public facets;

    fallback() external payable {
        address facet = facets[msg.sig];
        require(facet != address(0), "Function does not exist");

        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), facet, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())

            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}

```

To maintain shared state across facets, Diamond uses a storage pattern where structs are anchored to deterministic storage slots. While this resembles the shared-state philosophy of Eternal Storage, the data remains inside the proxy itself, allowing facets to access it directly <mark style="color:$warning;">without relying on external getter and setter contracts.</mark>

Review this code snippet from [Ethereum Blockchain Developer](https://www.ethereum-blockchain-developer.com/advanced-mini-courses/solidity-proxy-pattern-and-upgradable-smart-contracts/eip-2535-diamond-standard):

```solidity
// A contract that implements diamond storage.
library LibA {
 
  // This struct contains state variables we care about.
  struct DiamondStorage {
    address owner;
    bytes32 dataA;
  }
 
  // Returns the struct from a specified position in contract storage
  // ds is short for DiamondStorage
  function diamondStorage() internal pure returns(DiamondStorage storage ds) {
    // Specifies a random position from a hash of a string
    bytes32 storagePosition = keccak256("diamond.storage.LibA");
    // Set the position of our struct in contract storage
    assembly {ds.slot := storagePosition}
  }
}
 
// Our facet uses the diamond storage defined above.
contract FacetA {
 
  function setDataA(bytes32 _dataA) external {
    LibA.DiamondStorage storage ds = LibA.diamondStorage();
    require(ds.owner == msg.sender, "Must be owner.");
    ds.dataA = _dataA;
  }
 
  function getDataA() external view returns (bytes32) {
    return LibA.diamondStorage().dataA
  }
}
```

The process of adding, replacing, or removing functions is known as a _<mark style="color:$success;">diamond cut</mark>_<mark style="color:$success;">,</mark> visualizing how the contract can be carefully shaped by cutting functionality into smaller, modular pieces over time.&#x20;

```solidity
function diamondCut(bytes4 selector, address facet) external {
    facets[selector] = facet;
}
```

To complement this, _<mark style="color:$success;">loupe</mark>_ functions provide transparency by exposing which facets implement specific function selectors, returning their signatures and addresses.

```solidity
function facetAddress(bytes4 selector)
    external view
    returns (address)
{
    return facets[selector];
}

```

The Diamond proxy is highly scalable because it <mark style="color:$warning;">isolates storage for each facet using namespaced storage slots derived from hashes</mark>. Unlike traditional upgradeable patterns that rely on carefully preserving <mark style="color:red;">linear storage layouts</mark>, Diamonds avoid slot conflicts by anchoring state to deterministic locations, allowing new facets to introduce storage without disrupting existing state.

Diamond proxies previously raised several concerns, many of which have since been mitigated through improved tooling. One common risk is function selector collision, as selectors are only four bytes. Tools such as the `hardhat-diamond-abi` plugin help detect these conflicts during development, reducing the likelihood of accidental overlap.

Storage safety was another concern, particularly when libraries wrote to commonly used slots such as slot `0`, potentially corrupting state. Modern libraries like OpenZeppelin v5 have moved toward [namespaced storage](https://www.openzeppelin.com/news/introducing-openzeppelin-contracts-5.0#Namespaced) to avoid these collisions by anchoring data to deterministic slots.

Upgrade workflows have also improved with tools like `diamond-cli`, which assist developers in constructing precise diamond cuts and managing facet changes more safely.
