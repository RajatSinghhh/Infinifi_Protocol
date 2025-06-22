# Summary

The `mint` function involves transferring `USDC` tokens from a user to the protocol’s address before minting new tokens.
However, `USDC` tokens have an internal blacklist mechanism enforced by their smart contract (managed by Circle), which can prevent blacklisted addresses from sending or receiving tokens.
This creates a potential failure point where legitimate users may be unable to mint tokens if the protocol's address itself gets blacklisted or if transferring from a blacklisted user is attempted.

# Finding Description

The `mint` function implementation:

```solidity
function mint(
    address _to,
    uint256 _amount
) external whenNotPaused returns (uint256) {
    ERC20 usdc = ERC20(getAddress("USDC"));
    MintController mintController = MintController(
        getAddress("mintController")
    );

    usdc.safeTransferFrom(msg.sender, address(this), _amount);
    usdc.approve(address(mintController), _amount);

    return mintController.mint(_to, _amount);
}
```

**How it works:**

- The function first calls `safeTransferFrom` to pull `USDC` from the `msg.sender`.
- Then it calls `approve` to allow the `mintController` to use these `USDC` tokens.
- Finally, it mints new tokens to the specified `_to` address.

**Problem:**

- If `msg.sender` (user) is blacklisted by `USDC`’s contract, `safeTransferFrom` will fail, reverting the transaction.
- If the protocol’s address (the receiver) becomes blacklisted, `safeTransferFrom` will also fail even if the sender is clean.
- No specific handling of blacklisting is implemented, causing silent denial of service.

Thus, minting new tokens becomes impossible for blacklisted users or if the protocol is blacklisted, causing disruption in token distribution.

# Impact Explanation

The impact is considered `High`:

- **Denial of Service:** Users would be unable to mint tokens if they are blacklisted, even without prior notice.
- **Protocol Level Risk:** If the protocol’s `USDC` holding address is blacklisted by Circle, the entire minting process halts for all users.
- **User Frustration:** Failure without clear messaging could confuse users and degrade trust in the system.

# Likelihood Explanation

The likelihood is considered `Moderate`:

- Blacklisting is rare but not impossible. Circle has blacklisted addresses in the past (notably after regulatory demands).
- Legitimate users are unlikely to be blacklisted under normal operations.
- However, in cases of compromised wallets or regulatory enforcement, addresses may be blacklisted without the protocol's control.

# Proof of Concept

An attacker (or an unlucky user) submits a `mint` request:

- The user’s address is on `USDC`’s blacklist.
- When `usdc.safeTransferFrom(msg.sender, address(this), _amount)` is called, the transfer will revert.
- As a result, the `mint` function will fail and no new tokens will be minted.

Alternatively, if Circle blacklists the protocol’s receiving address:

- Even legitimate users who are clean cannot send `USDC` to the protocol.
- `safeTransferFrom` will fail at every `mint` attempt.
- Minting operations for the whole system become unusable.

# Recommendation

- **Whitelist Monitoring:** Regularly monitor the protocol’s operational addresses against `USDC` blacklists.
- **Error Handling Improvements:** Catch and log transfer failures separately to provide users with more informative error messages.
- **Alternative Payment Methods:** Allow minting via alternative stablecoins or assets not subject to third-party blacklisting.
- **Pre-Transfer Checks (if feasible):** Before calling `safeTransferFrom`, attempt a dry-run or off-chain check to identify blacklisted addresses early.
- **Documentation:** Inform users that transactions may fail if their wallet is blacklisted by `USDC`.
