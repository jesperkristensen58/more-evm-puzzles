# My Solutions

- [x] Puzzle 1: Here I need to compute (CALLVALUE) ** (CALLDATASIZE). So I can just pick CALLDATASIZE=1 to make it simpler. This means CALLVALUE simply dictates where JUMP goes (the hex of the destination). I see that I can JUMP all the way to the end to hex 47. So I actually just need to provide CALLVALUE=0x47=71 and CALLDATA=0x01 (e.g., just something with size 1).

- [x] Puzzle 2: In this puzzle I need to deploy code which, when CREATE is run against it, creates a contract that returns data of size 10. We need to first construct the init code (what returns the runtime code). The initi code and runtime code are structured like this in the call to CREATE: <init code><runtime code>. The init code should simply return the runtime code to CREATE. So the simplest form is: 0x<6013600C60003960136000F3><69ffffffffffffffffffff600052600a6016f3>. So the first part is the init code and it does this: Pushes 0x13 (the size of the runtime code) and the offset (0xc) into this entire set of opcodes where the runtime code starts -- both onto the stack. Then, it pushes 0 (where we want in memory the CODECOPY to go) and then we call CODECOPY. This command takes the <runtime code> part and puts into memory. And then it simply returns with where in memory the runtime code is (which is simply from 0 through 0x13 as we put in the RETURN). Okay, so far so good. Now, what are we deploying to that address? What is the runtime code? Well it needs to be something that just returns data of size 10 (0xa) when called with no arguments or value etc. We can create this function by doing this: When called, push 10 bytes (I just picked FF's) to memory starting at location 0. That's what the "69 <FF's> 600052" does. Then, simply push the size of that (60 0a) and the starting location in memory (60 16) -- note the offset is 0x16 because the FFs are pushed to the LSB and so we need to return starting from location 32-10=22 (32 is the entire word size, and 10 is the code size). And then F3 simply returns that to the caller. So now RETURNDATASIZE is 0xa as we want it to be.

- [x] Puzzle 3: Here, we need the base init code again (used in puzzle 2) but now the runtime code simply needs to modify the storage location 5 to contain "aa". The runtime code that does this against DELEGATECALL (b/c that will change the storage location in the calling contract when running *our runtime code*) is simply: 60 aa 60 05 55, so we push aa (the value) then 05 (the key) and then SSTORE it with 55. This will modify the caller's storage location (since we use DELEGATECALL). The size of this runtime code is 05. So modify the init code to that reality and we get: 0x<6005600C60003960056000F3><60aa600555> as the answer (I highlighted the init code and runtime code again, note how we use 60 05 in the beginning to signal the code size of 5 bytes).

- [x] Puzzle 4: In this example we need to send in init code via CREATE which, when done, holds half the VALUE we originally sent in. We can do this by having the init code send back half the incoming value to ORIGIN (tx.origin). Note, we don't need any runtime code for this one. So just create init code that does this and then send in some value that is divisible by 2. So we can use VALUE=4, e.g. What data to send in? The init code that transfers half the incoming Wei to tx.origin is: 0x600080808060023404325AF15060006000F3. Here is why: We simply push data to make a CALL: 6000808080(0,0,0,0 on the stack) 60023404 (<- divide(04) incoming callvalue(34) by 2(6002)) 325AF1(<-then specifiy the recipient (tx.origin=32), then put gas for the call(5A), and finally make the CALL(F1) forwarding half the incoming wei). That's it, the init code will return to CREATE and the created address will now have 2 Wei in it. This will mean that 4/2=2 and we compare with 2 in the puzzle code and jump if 2==2. This solves it.

- [x] Puzzle 5: We need to send in CALLDATA that is at least 0x20 in size. Later on, we also need <CALLDATASIZE> - <MSIZE> = 0x03 = 3. So MSIZE is 0x40 thus we need to send in CALLDATA of size 0x40-0x03=61. We can do this in Python with `"0x" + "FF" * 61` = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF. That's the answer.

- [x] Puzzle 6: We simply need to send in a value that when added to 0xfffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff0 gives 1. We can do this by overflowing it. So if we send in 17 this would overflow by 1 because: if we send 15=0xf this simply is 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff, if we send in 16=0x10 we get an overflow to exactly 0: 0x0 so we need 17 to get 0x1.

- [x] Puzzle 7: We need a value in Wei which allows us to run around the loop as many times as we need in order for the gas to be consumed so much that the difference between the gas after this loop and the initial gas is 0xa6=166. The loop is basically over i where i ranges from 0 to the CALLVALUE. So the question is, how many times should we iterate in the loop until we hit a GAS difference of 166? The answer is 4 as can be found from trial and error (alternatively the exact gas usage can be computed from the opcodes within the loop).

- [x] Puzzle 8: In this one, we don't need to send in any CALLVALUE, because the JUMPI treats the NOT(ISZERO(CALLVALUE)) as true even if its value is fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe. So the JUMP happens to JUMPDEST 0x7 which is good. Next a contract is created with the CALLDATA. But we can just send in no CALLDATA and that is valid init code. We need the SELFBALANCE to be 0, but that's fine we just don't send any CALLVALUE in anyway in the beginning so that is satisfied. We then make a call to the contract we just deployed. But the init code was empty (including the runtime code). So nothing is done or returned, but this still means that "1" is the result of the CALL - because technically the call was successful (it did not revert). This "1" from the CALL is all we need to JUMP to the next destination which is 0x28. At that point we simply need to ensure that SELFBALANCE is 0 (which hasn't changed from 0 since last we checked) and we are done. So the realization here is that JUMPI can base used with something other than "0x1" as it's truthy value. We can do anything that is not 0x0. In this case we just used the "ffff..." from the beginning of this paragraph. So the answer is: simply don't send in any CALLVALUE and don't send in any CALLDATA!

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
