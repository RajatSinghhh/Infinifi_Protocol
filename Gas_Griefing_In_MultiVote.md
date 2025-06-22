# Summary

The `multiVote` function in the protocol allows users to supply dynamic arrays and iteratively calls an external contract without enforcing strict limits.
This introduces a risk of Gas Griefing Attacks, where malicious users can intentionally supply oversized or complex inputs to cause transaction failures or significantly increase gas costs.

# Finding Description

The `multiVote` function accepts dynamic user input (`_assets`, `_unwindingEpochs`, `_liquidVotes`, `_illiquidVotes`) and loops over these arrays to call the `vote` function of an external contract (`allocationVoting`).

```solidity
function multiVote(
    address[] calldata _assets,
    uint32[] calldata _unwindingEpochs,
    AllocationVoting.AllocationVote[][] calldata _liquidVotes,
    AllocationVoting.AllocationVote[][] calldata _illiquidVotes
) external whenNotPaused {
    AllocationVoting allocationVoting = AllocationVoting(
        getAddress("allocationVoting")
    );

    for (uint256 i = 0; i < _assets.length; i++) {
        allocationVoting.vote(
            msg.sender,
            _assets[i],
            _unwindingEpochs[i],
            _liquidVotes[i],
            _illiquidVotes[i]
        );
    }
}
```

Since users fully control the input size and content, a malicious actor could submit:

- Extremely large arrays.
- Complex nested `AllocationVote` structures.
- Data that significantly inflates gas consumption during execution.

If the gas used exceeds the block gas limit, the transaction will fail entirely, preventing legitimate voting activity and degrading protocol usability.

⚠️ **Warning:** No restrictions on input sizes or complexity currently exist.

# Impact Explanation

The impact is considered `Medium`:

- **Denial of Service:** Legitimate users may be unable to complete their `multiVote` transactions if maliciously crafted transactions consume too much gas.
- **Increased Gas Costs:** Even successful transactions could become prohibitively expensive for regular users.
- **No Direct Loss of Funds**, but continuous griefing can severely degrade protocol performance and trust.

# Likelihood Explanation

The likelihood is considered `Moderate to High`:

- Any user can invoke `multiVote` with arbitrary data.
- No input validation or gas-limiting checks are in place.
- Therefore, attackers can easily exploit the function without requiring any special permissions.

# Proof of Concept

A malicious user can craft a transaction as follows:

- Supply `_assets` with `1000+` elements.
- Provide deeply nested and complex `_liquidVotes` and `_illiquidVotes` arrays.
- Each call to `allocationVoting.vote()` inside the loop consumes excessive gas.

**Result:**

- The transaction may exceed the block gas limit and revert.
- Other users attempting reasonable votes may experience high gas fees or failed transactions.
- System throughput and reliability are negatively impacted.

# Recommendation

To mitigate gas griefing risks:

### Input Size Limitations

```solidity
require(_assets.length <= MAX_ALLOWED_ASSETS, "Too many assets");
require(_unwindingEpochs.length == _assets.length, "Mismatched array lengths");
require(_liquidVotes.length == _assets.length, "Mismatched votes length");
require(_illiquidVotes.length == _assets.length, "Mismatched votes length");
```

### Bound Nested Arrays

- Restrict the number of `AllocationVote` entries allowed per asset.

### Batch Large Operations

- Advise users to split large voting actions into multiple smaller `multiVote` transactions.

### Gas Estimation Warnings

- Warn users through the frontend if their intended transaction is expected to consume excessive gas.

> **Note:** These mitigations will not only improve security but also enhance user experience and protocol scalability.
