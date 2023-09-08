Odd Chartreuse Shetland

high

# UUPSUpgradeable vulnerability in OpenZeppelin Contracts
## Summary
UUPSUpgradeable vulnerability in OpenZeppelin Contracts

## Vulnerability Detail
Openzeppelin has found the critical severity bug in UUPSUpgradeable. The kyber-swap contracts has used both openzeppelin contracts as well as  openzeppelin upgrabable  contracts with version v4.3.1. This is confirmed from package.json.

```Solidity
File: ks-elastic-sc/package.json

    "@openzeppelin/contracts": "4.3.1",
    "@openzeppelin/test-helpers": "0.5.6",
    "@openzeppelin/contracts-upgradeable": "4.3.1",
```

The `UUPSUpgradeable` vulnerability has been found in openzeppelin version as follows,

>>  @openzeppelin/contracts : Affected versions >= 4.1.0 < 4.3.2
>>  @openzeppelin/contracts-upgradeable : >= 4.1.0 < 4.3.2

However, openzeppelin has fixed this issue in versions 4.3.2

Openzeppelin bug acceptance and fix: [check here](https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories/GHSA-5vp3-v4hc-gx76)

The following contracts has been affected due to this vulnerability

1) PoolOracle.sol
2) TokenPositionDescriptor.sol

Both of these contracts are UUPSUpgradeable and the issue must be fixed.

## Impact
Upgradeable contracts using UUPSUpgradeable may be vulnerable to an attack affecting uninitialized implementation contracts.

## Code Snippet
https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/package.json#L35-L37

https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/oracle/PoolOracle.sol#L19

https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/periphery/TokenPositionDescriptor.sol#L13

## Tool used
Manual Review

## Recommendation
1) Update the openzeppelin library to latest version.
2) Check [this](https://forum.openzeppelin.com/t/security-advisory-initialize-uups-implementation-contracts/15301) openzeppelin security advisory to initialize the UUPS implementation contracts.
3) Check [this](https://docs.openzeppelin.com/contracts/4.x/api/proxy) openzeppelin UUPS documentation.