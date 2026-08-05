---
title: Preferential Gas Costs for Native ETH Operations
description: Apply a preferential gas factor to the value-transfer component of CALL/CREATE-family opcodes and to BALANCE access costs, reinforcing native ETH as the preferred medium of exchange and unit of account.
author: [Author]
discussions-to: [to be filled]
status: Draft
type: Standards Track
category: Core
created: 2026-08-04
requires: 2929, 3529
related: Native ETH as ERC-20 Precompile
---

## Abstract

This EIP applies a multiplicative preferential factor to the value-transfer / account-write gas component of `CALL`, `CALLCODE`, `CREATE` and `CREATE2` when `value > 0`, and to both the cold and warm access costs of `BALANCE`. The change makes operations that move or query native ETH balances cheaper relative to the same operations on other tokens, reinforcing ETH as the preferred medium of exchange, unit of account, and digital reserve asset.

## Motivation

A plain native ETH transfer costs 21 000 gas. A typical ERC-20 transfer costs 45 000–65 000 gas. Balance queries via `BALANCE` are also cheaper than an ERC-20 `balanceOf`. These structural differences already favour native ETH for simple top-level transfers.

Inside contracts, however, a value-bearing `CALL` or `CREATE` currently pays an additional ~9 000 gas surcharge (the value-transfer / account-write component). This EIP reduces that surcharge by a factor of 0.75 and applies a matching reduction to the access cost of `BALANCE`. As a result, the same economic operation performed with native ETH becomes cheaper than the identical operation performed with other tokens or with wrapped ETH.

The factor is large enough to be economically consequential for high-frequency internal transfers and settlement flows, while remaining small enough that pure substitution constructions stay uneconomic.

## Specification

### Parameters

- `ETH_VALUE_TRANSFER_FACTOR` = 0.75  
  Multiplicative factor applied exclusively to the value-transfer / account-write gas component of eligible opcodes when `value > 0`.
- `ETH_BALANCE_ACCESS_FACTOR` = 0.75  
  Multiplicative factor applied to both the cold and warm account-access gas costs of `BALANCE`.

### Rules

1. For `CALL`, `CALLCODE`, `CREATE` and `CREATE2`: when `value > 0`, multiply the value-transfer / account-write gas component by `ETH_VALUE_TRANSFER_FACTOR`. All other costs remain unchanged. `DELEGATECALL` and `STATICCALL` never transfer value and are unaffected.
2. For `BALANCE`: multiply both the cold-account-access cost and the warm-account-access cost by `ETH_BALANCE_ACCESS_FACTOR`.
3. The factor is applied when the relevant component is charged. Normal EIP-3529 refund rules then apply to the resulting gas used.

Zero-value calls receive no discount under this EIP.

### Rationale for the 0.75 factor

A 25 % reduction on the value-transfer component (~2 250 gas absolute saving on a ~9 000 gas component) is large enough to be economically consequential for frequent internal transfers and settlement, yet small enough that two native transfers remain more expensive than one typical ERC-20 transfer. The factor is deliberately round and easy for clients to implement.

## Rationale

The existing 21 000 gas baseline already privileges top-level native transfers. The additional ~9 000 gas surcharge on internal value-bearing calls currently limits that advantage. A 0.75 factor on the surcharge, plus a matching reduction for `BALANCE` (both cold and warm), extends the privilege cleanly to all native-ETH balance operations.

Local rules based solely on opcode identity and the presence of non-zero value are preferred because they are unambiguous, cheap to evaluate, and free of consensus risk. A runtime analysis of call-frame composition would be ambiguous, expensive and itself gameable.

This EIP is independent of the Native ETH as ERC-20 Precompile EIP. The latter supplies a preferential schedule for the `0x20` system-contract methods; the present EIP supplies a general preference for the native opcodes themselves. Together they create a consistent tilt toward native ETH across both the low-level and the ERC-20 interface paths.

### On resource-cost pricing

Recent gas-schedule work prices operations closer to their actual computational and state-access cost in order to incentivize efficient use of network resources. This EIP intentionally under-prices a narrow class of native-ETH operations relative to that baseline.

Resource pricing is an incentive mechanism. The same mechanism is applied here: a controlled preference for native ETH reduces wrapping, intermediate token usage, and the associated state growth. Per-operation client work is identical; the lower metered cost allows more preferred operations into a block, increasing total work on ETH-heavy blocks. Strengthening ETH as the native medium of exchange and unit of account is treated as a goal that justifies this limited exception to pure resource pricing.

## Security Considerations

Under a 0.75 factor applied only to the value-transfer component, pure gaming constructions remain uneconomic: the work still costs 75 % of its real resource consumption and cannot undercut legitimate alternatives. The only theoretical boundary would be a discount large enough that two native transfers became cheaper than one equivalent ERC-20 transfer; that boundary lies outside the parameters of this proposal.

The computational and state-access work of each individual discounted operation remains unchanged. Because the metered cost is lower, more preferred operations can fit inside the fixed block gas limit, producing an intentional increase in total client work on an ETH-heavy block. This capacity effect is bounded by the 0.75 factor and is subject to the normal EIP-1559 base-fee adjustment.

Refunds follow ordinary EIP-3529 rules after the factor has been applied. The value-transfer and balance-query components themselves do not generate refunds; refunds arise almost exclusively from storage-clearing `SSTORE` operations. The normal refund cap (gas used // 5) continues to apply.

## Backward Compatibility

Consensus-breaking; requires a hard fork. Existing gas schedules for pure computation and ordinary ERC-20 operations are unchanged.

The opcode-level factors defined here are self-contained and apply independently of the Native ETH as ERC-20 Precompile EIP.

Clients MUST update gas-cost tables and metering logic for the affected opcodes. Gas estimators, wallets and explorers require corresponding updates. The actual computational and state-access work performed for any individual operation remains identical; only the metered gas numbers change.

## Open Questions

- Final confirmation of the 0.75 factor by client teams and AllCoreDevs after benchmarking against concurrent state-access changes (e.g. EIP-8038).
- Residual value handling in `SELFDESTRUCT` (if still relevant): optional application of the same factor for consistency; otherwise leave unchanged.

## References

1. EIP-2929: Gas cost increases for state access opcodes. https://eips.ethereum.org/EIPS/eip-2929  
2. EIP-3529: Reduction in refunds. https://eips.ethereum.org/EIPS/eip-3529  
3. EIP-2780: Reduce intrinsic transaction gas. https://eips.ethereum.org/EIPS/eip-2780  
4. EIP-8038: State-access gas cost update. https://eips.ethereum.org/EIPS/eip-8038  
5. Native ETH as ERC-20 Precompile (related proposal that defines a preferential schedule for the system contract at address 0x20).  
6. ERC-20. https://eips.ethereum.org/EIPS/eip-20  
7. EIP-1559. https://eips.ethereum.org/EIPS/eip-1559  
