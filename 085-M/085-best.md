Quick Chocolate Swift

medium

# Pool can become inaccessible rendering users unable to access and retrieve their funds
## Summary
Pool can become inaccessible rendering users unable to access and retrieve their funds

## Vulnerability Detail 
Malicious or compromised owner(from README he is restricted) can upgrade PoolOracle.sol to a new version, for example, one that doesn't have a write function. This change would cause _tweakPosition() to revert, and if _tweakPosition reverts, the burn and mint functions will also revert
```solidity
146:    poolOracle.write(_blockTimestamp(), currentTick, baseL);
```

## Impact
Pool will become inaccessible, leaving users unable to retrieve their funds

## Code Snippet
https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/Pool.sol#L146

## Tool used

Manual Review

## Recommendation
I don't think that PoolOracle.sol needs to be upgradeable, but if you want to keep this contract upgradeable, you should implement a multisig