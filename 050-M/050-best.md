Brief Amethyst Frog

medium

# Admin can selectively prevent people from withdrawing funds
## Summary

The contest rules state the protocol owner and all admins are supposed to be restricted. Despite that, they can selectively disable withdrawals because Pool.sol reverts whenever poolOracle.write reverts. Specifically, they can upgrade the PoolOracle so that poolOracle.write always reverts, or selectively reverts for certain values of tx.origin, or selectively reverts if the price would enter a certain tick.

## Vulnerability Detail

Specifically, _tweakPosition calls swap when changing liquidity for a range that includes the current price, and swap calls poolOracle.write on every tick change. These operations will revert if poolOracle.write does. The admin can upgrade PoolOracle to revert under select circumstances.

As mentioned in the summary,  they can use this either to brick the whole contract, or selectively prevent certain EOAs from withdrawing if their position includes the current tick. For someone who is providing liquidity on a pool trading between two stablecoins and has a position covering the price range 0.95-1.05, this means they would likely never be able to withdraw their money. Further,  because poolOracle.write takes the tick as a parameter, the admin also has the power to make it so that poolOracle.write reverts if the price ever leaves a specified range, meaning no swap  that would move the price outside of a select range can work.

The README.md states: "Owner shouldn’t be able to steal funds, but is trusted with setting fee recipient address and fee collection (up to 20%)." If the owner shouldn't be able to steal funds,  presumably they shouldn't be able to permanently lock funds either.

## Impact

Admin can lock people out of their funds

## Code Snippet

PoolOracle is upgradable: https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/oracle/PoolOracle.sol#L42

Calls to poolOracle.write:

https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/Pool.sol#L146
https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/Pool.sol#L476

## Tool used

Manual Review

## Recommendation

Change all calls to poolOracle.write to not require the call succeed.