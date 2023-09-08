Radiant Fuchsia Snake

medium

# PoolOracle utilizes vulnerable OZ 4.3.1 UUPS implementation
## Summary

The current contracts utilizes version 4.3.1 of OZ's contracts-upgradeable (as specified [here](https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/package.json#L37)). This version contains the critical vulnerability that uninitialized implementations can be [hijacked](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/security/advisories/GHSA-q4h9-46xg-m3x9) and completely bricked. PoolOracle utilizes this vulnerable UUPS implementation.

## Vulnerability Detail

See summary

## Impact

PoolOracle can be completely broken, crippling every pool that utilizes it.

## Code Snippet

[PoolOracle.sol#L38-L40](https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/oracle/PoolOracle.sol#L38-L40)

## Tool used

Manual Review

## Recommendation

Disable initializers in the constructor