# solidity-audit-checklist

# By project type
## Lending markets (OTC)

### ERC-777 are backwards compatible with ERC-20 and can be used without implementing their interface
ERC-777 tokens contain the hook `_callTokensReceived` which transfers control to the receiver when tokens are transferred to them.

**Attack vectors:**
1. DoS by always reverting when receiving tokens (critical issue if function relies on an account receiving tokens (loan buyout, repayment, batch payment, etc.)).
2. Allows re-entry

## Yield aggregation

### Users can transfer accounting tokens between accounts to exploit yield claimed
If the `_beforeTokenTransfer()` is not correctly implemented (the protocol is using a custom staking module compared to Convex's), and the `_checkpoint` is not updated correctly, the protocol will not account for transfers between users.

Example: [Yield Protocol](https://github.com/code-423n4/2022-01-yield-findings/issues/86)

## NFT collections

## Stablecoins

## Misc

### Low level calls & msg.sender, context
`delegateCall` preserves msg.sender & context, while `call` does not.

### Does not check existence of target account for low-level call
According to the Solidity docs), “The low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed”.

### Transfer may be unusable by some smart contract users
It only forwards 2300 gas, and since gas prices can change in the future for ETH, for a smart contract that does not implement the fallback function or one that requires more than 2300 to receive the native token, it will always fail. Pay attention to use of this with `WETH`, namely `WETH.withdraw` beforehand, and when it is used in critical logic - withdrawing collateral, etc..

### Validating Chainlink oracle feeds
When using Chainlink Price feeds it is important to ensure the price feed data was updated recently. While getting started with chainlink requires just one line of code, it is best to add additional checks for in production environments.

# Assorted tips

Inconsistent use of `safeTransfer` and `transfer` for ERC721 may warrant Medium risk. [See here](https://github.com/code-423n4/2022-04-backed-findings/issues/81#issuecomment-1100560835)
