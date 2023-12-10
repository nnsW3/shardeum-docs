---
title: EVM Storage
sidebar_position: 1
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## How smart contracts store data

Smart contracts have storage slots to store state data.
Each storage slot is 32 bytes.

Here is an example smart contract where a user can change the state for a uint variable.
Note: a uint variable is 256 bits by default in Solidity, equivalent to 32 bytes:

<Tabs>
  <TabItem value="solidity" label="Solidity" default>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.15;

contract SimpleStorage {
    uint public storedData; //Do not set 0 manually it wastes gas!

    event setEvent();

    function set(uint x) public {
        storedData = x;
        emit setEvent();
    }

}
```

  </TabItem>
</Tabs>

## EVM storage opcodes

Storage (stored, expensive):

-SSTORE: write data to storage slot (20000 gas)

-SLOAD:  read data from a storage slot (800 gas)

Memory (run time, cheap):

-MSTORE (3 gas)

-MLOAD (3 gas)

Here is a smart contract that uses Assembly (Yul in Solidity) to access these opcodes:

<Tabs>
  <TabItem value="solidity" label="Solidity" default>

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.15;

contract AssemblyStorage {

    uint public storageSlot0; //Do not set 0 manually it wastes gas!
    uint public storageSlot1; //Do not set 0 manually it wastes gas!

    event setEvent();

    function assemblyStorage(uint x) public {
        assembly {
            sstore(0,x)         //Record memory value in storage slot 0.  
            sstore(1,sload(0))  //Record storage slot 0 to storage slot 1 (costs more gas reading data from storage than data).
        }
        emit setEvent();
    }

}
```

  </TabItem>
</Tabs>


##  How to compress data in storage slots

-If you are using a variable that never changes, make the variable immutable so that it doesn't use a storage slot.

-Try to fit data in less storage slots when possible. For example,
if you need a uint, an address and an uint with only 96 bits:

Correct:

```solidity
uint     [Slot 0, 32/32 Bytes]
address  [Slot 1, 20/32 Bytes]
uint96   [Slot 1, 32/32 Bytes]
```

Note: Pack the compressed data together (above),
or you will end up using another storage slot anyway (below)

Wrong:

```solidity
uint96  [Slot 0, 12/32 Bytes]
uint    [Slot 1, 32/32 Bytes]
address [Slot 2, 20/32 Bytes]
```


## Why is storing large data not recommended on chain?

The more storage slots you want to write to, the more gas you need to pay.
Storing large files like high resolution images is very expensive on chain.
Decentralized storage alternatives like IPFS and Filecoin solve this problem.
