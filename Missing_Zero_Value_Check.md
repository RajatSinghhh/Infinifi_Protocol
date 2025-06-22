# Summary

A `0` check for `assetPrice` should be implemented to ensure valid data is returned by the oracle.

# Finding Description

In the `Accounting.sol` contract, both the functions `totalAssetsValue()` and `totalAssetsValueOf()` include a variable `uint256 assetPrice`, which stores the price returned by the oracle for a given asset address. There is a potential risk that the oracle may return a value of `0`.

To mitigate this risk, a `require` statement should be added after calculating `assetPrice` to ensure that the function reverts if the oracle returns a value of `0`.

# Impact Explanation

The impact is considered `medium`. While it is unlikely that the oracle will return a `0` price (since the address is submitted by the governor), the worst-case scenario must be accounted for. If the oracle does return `0`, this would negatively affect the calculation of the total value of assets held in protocol contracts listed in the farm registry. This issue could also impact the calculation of the total value of liquid assets across the protocol.

# Likelihood Explanation

The likelihood of this issue occurring is considered `medium-high`, as this bug could affect the protocol whenever calculating the total supply of liquid assets held within the farm registry.

# Proof of Concept

As illustrated in the screenshots provided below, the oracle call for `assetPrice` may return `0`, which could lead to incorrect calculations.

**AssetPrice 1.png**
**AssetPrice 2.png**

# Recommendation

To address this issue, we recommend adding the following `require` statement after the assignment of `assetPrice` within the `for` loop in both functions. This ensures that the function will revert if the oracle returns a `0` value for the asset price:

```solidity
uint256 assetPrice = IOracle(oracle[assets[i]]).price();
require(assetPrice > 0, "Invalid price fetch");
```
