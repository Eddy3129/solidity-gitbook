---
description: A sneak peek into the depths of Solidity
icon: robot-astromech
---

# Advanced Solidity

The internals of the EVM can be highly complex. However, understanding its storage model is crucial for achieving meaningful gas optimizations.

One of the most effective storage-level optimization techniques is **variable packing**, where multiple smaller variables are stored within a single 32-byte storage slot. Since each storage slot in the EVM is exactly **256 bits (32 bytes)**, declaring several smaller types instead of multiple `uint256` variables allows the compiler to pack them together automatically when possible.

For example, instead of writing:

{% code fullWidth="true" %}
```solidity
uint256 a;
uint256 b;
uint256 c;
```
{% endcode %}

We can have all packed in a single 32-byte slot:

```solidity
uint128 a;
uint64 b;
uint32 c;
uint32 d;
```

If their combined size does not exceed 256 bits and they are declared contiguously, Solidity packs them into a single slot. This significantly reduces expensive `SSTORE` operations.

To update a value stored within a packed storage slot, a **bitmasking operation** must be performed. Because multiple variables may occupy the same 256-bit slot, directly overwriting the slot would corrupt adjacent data. Instead, the update must target only the bits allocated to the variable.

The process follows four deterministic steps:

1. **Locate the target bits** by computing the variable’s bit offset within the slot.
2. **Clear the existing value** using a negated mask to zero only the relevant bits.
3. **Shift the new value** left by the offset to align it with the cleared region.
4. **Merge the value** back into the slot using a bitwise OR operation.

```solidity
uint256 offset = 8;
// mask to isolate the bit range
uint256 mask = uint256(type(uint32).max) << offset;

// overwrite only the uint32 region
slot = (slot & ~mask) | (uint256(newValue) << offset);
```
