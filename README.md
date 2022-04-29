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

## NFT collections

## Stablecoins

## Misc

### Low level calls
`delegateCall` preserves msg.sender & context, while `call` does not.

# Assorted tips

Inconsistent use of `safeTransfer` and `transfer` for ERC721 may warrant Medium risk. [See here](https://github.com/code-423n4/2022-04-backed-findings/issues/81#issuecomment-1100560835)
