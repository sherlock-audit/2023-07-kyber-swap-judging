# Issue M-1: UUPSUpgradeable vulnerability in OpenZeppelin Contracts 

Source: https://github.com/sherlock-audit/2023-07-kyber-swap-judging/issues/25 

## Found by 
0x52, MohammedRizwan
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



## Discussion

**sherlock-admin**

1 comment(s) were left on this issue during the judging contest.

**Trumpero** commented:
> valid medium about OZ upgradable 4.3.1



**manhlx3006**

**Sponsor Confirmed**

- The finding is valid and a known issue (we have initialized all Pool Oracle for deployed contracts and transferred ownership to a multisig). The UUPSUpgradable has a vulnerability with the version we used. If the implementation is not initialized, someone can initialize and try to destroy the implementation.
- For deployed contracts: We have initialized all implementation of deployed contracts and currently our multisig is the owner of all, thus, for the current deployed contracts, the issue won’t happen.
- For the future deployment: we will move to the new OZ version 4.3.2.


