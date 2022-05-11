# EVM puzzles

A collection of EVM puzzles. Each puzzle consists on sending a successful transaction to a contract. The bytecode of the contract is provided, and you need to fill the transaction data that won't revert the execution.

## How to play

Clone this repository and install its dependencies (`npm install` or `yarn`). Then run:

```
npx hardhat play
```

And the game will start.

In some puzzles you only need to provide the value that will be sent to the contract, in others the calldata, and in others both values.

You can use [`evm.codes`](https://www.evm.codes/)'s reference and playground to work through this.

## Solutions
1. You must send a value to the contract such that the first byte of the SHA3 hash of the value is 0xa8
2. The first jump is to the line CALLVALUE ** CALLDATASIZE. You want this to be 64. The next jump is PC + CALLDATASIZE = 65 + CALLDATASIZE. You want this to equal 71. CALLVALUE = 2 and CALLDATASIZE = 6 will solve this
3. Execution succeeds if the RETURNDATASIZE of the newly created contract is 10 bytes. Thus you will pass this level if you pass in creation bytecode that deploys a contract that returns 10 bytes. Example: `0x6005600c60003960056000F3600a6000F3`.
4. This puzzle delegatecalls the deployed contract and then checks its storage. The only way to pass this level is if the deployed contract sets the value in storage slot
5 to 0xaa. Here is an example solution contract: `0x6006600c60003960066000F360aa60055500`
5. The puzzles executes a CREATE instruction using the calldata, sending along it's entire balance with it. It then expects the balance of the called contract to have
half the balance that was sent. To solve this puzzle we can create a very basic contract in Solidity, compile it, and then copy over the bytecode
```
contract Puzzle5 {
    constructor() payable {
        payable(address(0)).transfer(msg.value/2)
    }
}
```
We'll also need to send at least 2 wei to the puzzle contract.
6. TODO - The contract copies the calldata in memory. We need to supply a minimum of TODO bytes of calldata and also have MSIZE - CALLDATA = 3. How is this achievable? Well, memory expands in 32 byte words