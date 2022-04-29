# solidity-audit-checklist

## ERC-777 are backwards compatible with ERC-20 and can be used without implementing their interface
ERC-777 tokens contain the hook `_callTokensReceived` which transfers control to the receiver when tokens are transferred to them.

**Attack vectors:**
1. DoS by always reverting when receiving tokens (critical issue if function relies on an account receiving tokens (loan buyout, repayment, batch payment, etc.)).
2. Allows re-entry

## Low level calls
`delegateCall` preserves msg.sender & context, while `call` does not.
