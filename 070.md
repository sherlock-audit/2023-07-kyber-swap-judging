Eager Hazel Mule

medium

# Loss of rTokens owed if a user is owed more rTokens than the Position Manager's balance
## Summary
If a user calls `BasePositionManager::burnRTokens` and they're owed more than the Position Manager's current balance they only receive up to the balance of the Pool and have their remaining balance of rTokenOwed wiped out.

## Vulnerability Detail
In `BasePositionManager::burnRTokens` when the `rTokenQty > rTokenBalance` the following call to `Pool::burnRTokens` burns the entire balance of the BasePositionManager: 

```solidity
(amount0, amount1) = pool.burnRTokens(
      rTokenQty > rTokenBalance ? rTokenBalance : rTokenQty,
      false
);
```

this results in the LP that called the function only receiving `rTokenBalance - rTokenQty` when the burn is complete.

PoC:

1. Bob has `rTokenQty = 100`
2. The `BasePositionManager` only has `rTokenBalance = 90` because the other 10 were earned from fees gained through rewards earned on the reinvested `rTokens` and so are only minted when `_syncFeeGrowth` is called
3.  If bob calls `BasePositionManager::burnRTokens` , it sets `pos.rTokenOwed = 0` and calls `pool.burnRTokens(90)`
4. `Pool::burnRTokens` then makes a call to `_syncFeeGrowth` which mints the 10 remaining `rTokens` but the function was already called with `_qty = 90`, so only those are burned for Bob and the remaining 10 stay in BasePositionManager 
5. If Bob tries to call `BasePositionManager::burnRTokens` again he won't receive any reward because he's had `rTokenOwed` for his position reset to 0

## Impact
LPs lose `rTokenQty - rTokenBalance` whenever they try to burn `rTokenQty > rTokenBalance`. They are unable to recover these lost `rTokens` and they remain stuck in `BasePositionManager`.

## Code Snippet
[BaseTokenManager::burnRTokens](https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/periphery/BasePositionManager.sol#L254-L265)

```solidity
    rTokenQty = pos.rTokenOwed;
    require(rTokenQty > 0, 'No rToken to burn');


    PoolInfo memory poolInfo = _poolInfoById[pos.poolId];
    IPool pool = _getPool(poolInfo.token0, poolInfo.token1, poolInfo.fee);


    pos.rTokenOwed = 0;
    uint256 rTokenBalance = IERC20(address(pool)).balanceOf(address(this));
    (amount0, amount1) = pool.burnRTokens(
      rTokenQty > rTokenBalance ? rTokenBalance : rTokenQty, // @audit when the rTokenQty is greater this only burns rTokenQty - rTokenBalance for the balance of BasePositionManager before reinvestment fees are minted
      false
    );
```

[Pool::burnRTokens](https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/Pool.sol#L260-L287)

```solidity
  function burnRTokens(uint256 _qty, bool isLogicalBurn)
    external
    override
    lock
    returns (uint256 qty0, uint256 qty1)
  {
    if (isLogicalBurn) {
      _burn(msg.sender, _qty);


      emit BurnRTokens(msg.sender, _qty, 0, 0);
      return (0, 0);
    }
    // SLOADs for gas optimizations
    uint128 baseL = poolData.baseL;
    uint128 reinvestL = poolData.reinvestL;
    uint160 sqrtP = poolData.sqrtP;
    _syncFeeGrowth(baseL, reinvestL, poolData.feeGrowthGlobal, false); // @audit feeGrowth is synced here to account for change in reinvested liquidity


    // totalSupply() is the reinvestment token supply after syncing, but before burning
    uint256 deltaL = FullMath.mulDivFloor(_qty, reinvestL, totalSupply());
    reinvestL = reinvestL - deltaL.toUint128();
    poolData.reinvestL = reinvestL;
    poolData.reinvestLLast = reinvestL;
    // finally, calculate and send token quantities to user
    qty0 = QtyDeltaMath.getQty0FromBurnRTokens(sqrtP, deltaL);
    qty1 = QtyDeltaMath.getQty1FromBurnRTokens(sqrtP, deltaL);

    _burn(msg.sender, _qty); // @audit when _qty passed in is the BasePositionManager balance before syncing only that much is burned
```

[Pool::_syncFeeGrowth](https://github.com/sherlock-audit/2023-07-kyber-swap/blob/main/ks-elastic-sc/contracts/Pool.sol#L620-L647)
```solidity
  /// @dev sync the value of feeGrowthGlobal and the value of each reinvestment token.
  /// @dev update reinvestLLast to latest value if necessary
  /// @return the lastest value of _feeGrowthGlobal
  function _syncFeeGrowth(
    uint128 baseL,
    uint128 reinvestL,
    uint256 _feeGrowthGlobal,
    bool updateReinvestLLast
  ) internal returns (uint256) {
    uint256 rMintQty = ReinvestmentMath.calcrMintQty(
      uint256(reinvestL),
      uint256(poolData.reinvestLLast),
      baseL,
      totalSupply()
    );
    if (rMintQty != 0) {
      rMintQty = _deductGovermentFee(rMintQty);
      _mint(address(this), rMintQty);
      // baseL != 0 because baseL = 0 => rMintQty = 0
      unchecked {
        _feeGrowthGlobal += FullMath.mulDivFloor(rMintQty, C.TWO_POW_96, baseL);
      }
      poolData.feeGrowthGlobal = _feeGrowthGlobal;
    }
    // update poolData.reinvestLLast if required
    if (updateReinvestLLast) poolData.reinvestLLast = reinvestL;
    return _feeGrowthGlobal;
  }
```

## Tool used
Manual Review

## Recommendation
Add the following conditional statement to ensure pos.rTokenOwed isn't set to 0 in the case where rTokenQty > rTokenBalance: 

```solidity
 if(rTokenQty > rTokenBalance) {
      pos.rTokenOwed = rTokenBalance - rTokenQty;
    } else {
      pos.rTokenOwed = 0;
    }
```

this will allow the LP to call `BasePositionManager::burnRTokens` again to burn their remaining `rTokens` once the fees for reinvested liquidity growth are synced. 
