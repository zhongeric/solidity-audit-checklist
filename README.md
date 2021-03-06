# solidity-audit-checklist

# By project type

[OTC lending markets](#otc-lending-markets)

[Yield aggregators](#yield-aggregators)

[NFT collections](#nft-collections)

[Stablecoins](#stablecoins)

[Misc](#misc)

## OTC Lending markets

### ERC-777 are backwards compatible with ERC-20 and can be used without implementing their interface
ERC-777 tokens contain the hook `_callTokensReceived` which transfers control to the receiver when tokens are transferred to them.

**Attack vectors:**
1. DoS by always reverting when receiving tokens (critical issue if function relies on an account receiving tokens (loan buyout, repayment, batch payment, etc.)).
2. Allows re-entry

## Yield aggregators

### Users can transfer accounting tokens between accounts to exploit yield claimed
If the `_beforeTokenTransfer()` is not correctly implemented (the protocol is using a custom staking module compared to Convex's), and the `_checkpoint` is not updated correctly, the protocol will not account for transfers between users.

Example: [Yield Protocol](https://github.com/code-423n4/2022-01-yield-findings/issues/86)

## NFT collections
`safeMint` opens up re-entrancy where attackers can mint more than allowed

## Stablecoins

## Misc

### Low level calls & msg.sender, context
`delegateCall` preserves msg.sender & context, while `call` does not.

### Not checking existence of target account for low-level call will lead to silent failure
According to the Solidity docs), “The low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed”.

### Transfer may be unusable by some smart contract users
It only forwards 2300 gas, and since gas prices can change in the future for ETH, for a smart contract that does not implement the fallback function or one that requires more than 2300 to receive the native token, it will always fail.

WETH can always be withdrawn from a function using `WETH.withdrawUnderlying`

### Validating Chainlink oracle feeds
When using Chainlink Price feeds it is important to ensure the price feed data was updated recently. While getting started with chainlink requires just one line of code, it is best to add additional checks for in production environments. [Reference example](https://github.com/code-423n4/2022-01-yield-findings/issues/136)

Example validation:
```solidity
        (uint80 roundID, int256 usdcPrice, , uint256 timestamp, uint80 answeredInRound) = ChainlinkFeed(0x986b5E1e1755e3C2440e960477f25201B0a8bbD4).latestRoundData();
        require(usdcPrice > 0, "ChainLink: price <= 0");
        require(answeredInRound >= roundID, "ChainLink: Stale price");
        require(timestamp != 0, "ChainLink: Round not complete");
```

### UniswapV3 TWAP logic

### ERC20 token approvals
Not setting the approval to 0 before approving an amount for an ERC20 may cause it to fail for non standard tokens like USDT (Tether). Consider using OZ's SafeERC20's `safeTransfer()` instead. [Reference issue](https://github.com/code-423n4/2022-04-dualityfocus-findings/issues/39)

### erecover may fail silently, returning 0 if the signature is invalid
Reference [issue](https://github.com/code-423n4/2021-04-meebits-findings/issues/77)

# Assorted tips

Inconsistent use of `safeTransfer` and `transfer` for ERC721 may warrant Medium risk. [See here](https://github.com/code-423n4/2022-04-backed-findings/issues/81#issuecomment-1100560835) . [Also here, Consensys audit - Med sev](https://consensys.net/diligence/audits/2021/01/fei-protocol/#unchecked-return-value-for-iweth-transfer-call)
