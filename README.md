# My Solutions

- [x] Puzzle 1: Here I need to compute (CALLVALUE) ** (CALLDATASIZE). So I can just pick CALLDATASIZE=1 to make it simpler. This means CALLVALUE simply dictates where JUMP goes (the hex of the destination). I see that I can JUMP all the way to the end to hex 47. So I actually just need to provide CALLVALUE=0x47=71 and CALLDATA=0x01 (e.g., just something with size 1).

- [x] Puzzle 2: In this puzzle I need to deploy code which, when CREATE is run against it, creates a contract that returns data of size 10. We need to first construct the init code (what returns the runtime code). The initi code and runtime code are structured like this in the call to CREATE: <init code><runtime code>. The init code should simply return the runtime code to CREATE. So the simplest form is: 0x<6013600C60003960136000F3><69ffffffffffffffffffff600052600a6016f3>. So the first part is the init code and it does this: Pushes 0x13 (the size of the runtime code) and the offset (0xc) into this entire set of opcodes where the runtime code starts -- both onto the stack. Then, it pushes 0 (where we want in memory the CODECOPY to go) and then we call CODECOPY. This command takes the <runtime code> part and puts into memory. And then it simply returns with where in memory the runtime code is (which is simply from 0 through 0x13 as we put in the RETURN). Okay, so far so good. Now, what are we deploying to that address? What is the runtime code? Well it needs to be something that just returns data of size 10 (0xa) when called with no arguments or value etc. We can create this function by doing this: When called, push 10 bytes (I just picked FF's) to memory starting at location 0. That's what the "69 <FF's> 600052" does. Then, simply push the size of that (60 0a) and the starting location in memory (60 16) -- note the offset is 0x16 because the FFs are pushed to the LSB and so we need to return starting from location 32-10=22 (32 is the entire word size, and 10 is the code size). And then F3 simply returns that to the caller. So now RETURNDATASIZE is 0xa as we want it to be.

- [x] Puzzle 3: Here, we need the base init code again (used in puzzle 2) but now the runtime code simply needs to modify the storage location 5 to contain "aa". The runtime code that does this against DELEGATECALL (b/c that will change the storage location in the calling contract when running *our runtime code*) is simply: 60 aa 60 05 55, so we push aa (the value) then 05 (the key) and then SSTORE it with 55. This will modify the caller's storage location (since we use DELEGATECALL). The size of this runtime code is 05. So modify the init code to that reality and we get: 0x<6005600C60003960056000F3><60aa600555> as the answer (I highlighted the init code and runtime code again, note how we use 60 05 in the beginning to signal the code size of 5 bytes).

- [ ] Puzzle 4:
- [ ] Puzzle 5:
- [ ] Puzzle 6:
- [ ] Puzzle 7:
- [ ] Puzzle 8:
- [ ] Puzzle 9:
- [ ] Puzzle 10:

# 10 more EVM puzzles

Here are 10 more puzzles, inspired by the 10 EVM puzzles created by [@fvictorio](https://github.com/fvictorio/evm-puzzles). These ones are harder and more focused on the CREATE and CALL opcodes. Have fun!

Each puzzle consists of sending a successful transaction to a contract. The bytecode of the contract is provided, and you need to fill the transaction CALLDATA and CALLVALUE that won't revert the execution.

## How to play

Clone this repository and install its dependencies (`npm install` or `yarn`). Then run:

```
npx hardhat play
```

And the game will start.

In some puzzles you only need to provide the value that will be sent to the contract, in others the calldata, and in others both values.

You can use [`evm.codes`](https://www.evm.codes/)'s reference and playground to work through this.
