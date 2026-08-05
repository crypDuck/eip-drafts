---
title: Native ETH as ERC-20 Precompile
description: Introduce a system contract at address 0x20 that exposes native ETH balances through a standard ERC-20 (and ERC-2612) interface, with a preferential gas schedule that makes the native path cheaper than ordinary ERC-20 tokens or wrapped ETH.
author: crypDuck
discussions-to: [https://github.com/crypDuck/eip-drafts/issues](https://ethereum-magicians.org/t/native-eth-as-erc-20-system-contract-draft-for-feedback/29301)
status: Draft
type: Standards Track
category: Core
created: 2026-08-04
requires: 20, 2612, 7528
---

## Abstract

This EIP introduces a system contract at address `0x20` that implements the ERC-20 and ERC-2612 interfaces against native ETH balances. Transfers through the contract move real account balances, emit standard `Transfer` events, and never execute recipient code. The methods are given a preferential gas schedule so that using native ETH via the standard ERC-20 interface is cheaper than the same operations on ordinary ERC-20 tokens or on wrapped ETH. The change removes the need for most WETH wrapping while preserving native ETH semantics for gas payment and value transfers.

Although the title retains the familiar term “precompile” for continuity with prior discussion and existing implementations, the facility is implemented as a stateful system contract. State is required to store allowances and nonces.

## Motivation

Native ETH and ERC-20 tokens expose different interfaces. Contracts that want to treat ETH uniformly with other tokens must either special-case native transfers or route through a wrapped representation (most commonly WETH9). Wrapping creates ongoing gas overhead, extra transactions, and state growth; WETH has historically ranked among the highest gas consumers on mainnet.<sup>1</sup>

A protocol-level ERC-20 view of native balances would:

- Make ETH the seamless default asset wherever an ERC-20 interface is expected.
- Reduce unnecessary wrapping traffic and the associated state growth.
- Complement the existing address convention of ERC-7528 (`0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`).

Because the system contract updates native balances directly (no token-balance mapping SSTOREs), its methods can be priced substantially lower than ordinary ERC-20 operations while remaining faithful to real resource costs. This creates a structural preference for native ETH as the unit of account and medium of exchange.

## Relation to Prior Work

An “ETH as ERC-20 precompile” was discussed on Ethereum Magicians in late 2022 (https://ethereum-magicians.org/t/eip-eth-as-erc20-precompile/12095). It was never assigned an EIP number and received limited engagement (mainly on function-dispatch complexity and gas pricing). It stalled without strong conceptual objections. An earlier related idea appeared in 2019.

This proposal revives the core idea and updates it for current account-abstraction expectations (EIP-7702, paymasters), explicit ERC-2612 support, and the finalized ERC-7528 address convention. The opt-in requirement for pre-activation contracts is retained. Evmos previously shipped a similar native-token-as-ERC-20 precompile.

## Specification

### Address

A new system contract is introduced at the fixed address:

`0x0000000000000000000000000000000000000020`

The address alludes to ERC-20. Final confirmation is left to AllCoreDevs.

### Interface

The contract MUST implement the full ERC-20 interface plus ERC-2612 `permit`. In addition it exposes:

- `optIn()` – callable by contracts created before the activation block.
- `isOptedIn(address) → bool` – returns whether the given address participates.

### Semantics

- `balanceOf(a)` returns the native balance of `a`.
- `transfer` / `transferFrom` update native balances and MUST emit a standard `Transfer` event.
- Transfers never execute recipient code.
- Ordinary `CALL` value transfers continue to emit no ERC-20-style `Transfer` event. Uniform ETH-transfer logging is left to complementary proposals such as EIP-7708.
- The invariant `nativeBalance(a) == balanceOf(a)` is maintained for participating accounts.
- Value-bearing calls that target `0x20` itself MUST revert.
- `totalSupply()` returns the total native ETH supply (sum of all balances).
- `name()` returns `"Ether"`, `symbol()` returns `"ETH"`, `decimals()` returns `18`.
- ERC-2612 `DOMAIN_SEPARATOR` uses the standard EIP-712 domain (name `"Ether"`, version `"1"`, chainId, verifyingContract = `0x20`).
- Opt-in status is a property of the address and is unaffected by temporary code installed via EIP-7702.

Legacy contracts must call `optIn()`; contracts created after the fork are opted in by default.

### Gas Schedule

The methods of the system contract are priced preferentially so that the native-ETH path is cheaper than the same operation on an ordinary ERC-20 or on WETH. Concrete numbers (illustrative; final values to be confirmed by client teams and AllCoreDevs):

| Method                        | Target gas (warm accounts) | Principle |
|-------------------------------|----------------------------|---------|
| `balanceOf` / `totalSupply`   | Same as native `BALANCE` (currently 100 warm / 2 600 cold) | Direct native balance read |
| `transfer` / `transferFrom`   | 15 000 – 20 000            | Native value-write component + `Transfer` event (LOG3) + minimal dispatch. No token-balance SSTOREs. |
| `approve` / `permit`          | 20 000 – 30 000            | Reflects the storage write for allowance / nonce; competitive with well-optimised ERC-20s |
| `allowance`                   | Same as a warm SLOAD       | Simple storage read |

These targets are deliberately lower than typical ERC-20 costs (45 000–65 000 for a transfer) because the system contract updates native balances directly. The resulting preference is structural: using ETH via the standard ERC-20 interface is cheaper than using any other token or a wrapped representation for the same economic effect.

The schedule above is independent of, and complementary to, the Preferential Gas Costs for Native ETH Operations EIP, which applies factors to the general native opcodes (`CALL`/`CREATE` family value-transfer component and `BALANCE`).

## Rationale

A fixed-address system contract can enforce the balance invariant with low overhead while providing the storage needed for allowances and nonces. Classic pure precompiles cannot hold this state without non-standard client mechanisms; a system contract follows established L1 patterns (deposit contract, EIP-4788 beacon roots) and is simpler for multi-client implementation.

Recipient-code execution is intentionally omitted. Transfers never execute recipient code. This matches WETH behaviour, eliminates the classic callback reentrancy surface, and keeps the mental model simple. Any future desire for receive-hooks can be addressed by a separate extension EIP.

Transfers through the contract emit a standard `Transfer` event so tools treating it as an ERC-20 see the expected logs. Ordinary native value transfers remain silent, avoiding log-volume growth. The design is complementary to EIP-7708.

Returning the conventional `"Ether"` / `"ETH"` / `18` values and a proper `DOMAIN_SEPARATOR` maximises compatibility with existing tooling. `totalSupply` reports the total native ETH supply rather than an ambiguous circulating figure.

Value-bearing calls to `0x20` revert so the system contract cannot become an ownerless ETH sink. Opt-in is defined as a persistent property of the address itself; temporary code under EIP-7702 does not change it.

The preferential gas schedule is an intentional policy choice that reinforces ETH as the preferred medium of exchange and unit of account. Because the contract avoids token-balance storage writes, the lower numbers remain faithful to real resource consumption while still creating a clear cost advantage over ordinary ERC-20 tokens and WETH.

## Security Considerations

- Transfers never execute recipient code, eliminating the classic callback reentrancy vector. Contracts that previously special-cased native value receipt versus ERC-20 receipt should still be reviewed for remaining behavioural differences.
- Pre-fork contracts must explicitly call `optIn()`. Selector-collision risk on the `optIn` method for legacy contracts should be considered. Opt-in status is an address-level property and is unaffected by EIP-7702 temporary code.
- The balance invariant must hold at all times for participating accounts, including under temporary code, self-transfers, and edge cases involving the system contract itself.
- Value-bearing calls to `0x20` MUST revert.
- Standard ERC-20 allowance race conditions and permit signature considerations apply; high-value native balances make them more critical.
- Storage-writing operations (`approve`, `transferFrom`, `permit`) are priced to discourage griefing while remaining competitive with ordinary ERC-20s.

## Backward Compatibility

The change is consensus-breaking and requires a hard fork. Existing contracts that do not opt in continue to see only native balances and are unaffected. WETH remains fully functional. Protocols using the ERC-7528 sentinel continue to work; the system contract is a distinct address. Implementation requires coordinated changes in execution clients; consensus clients are largely unaffected.

## Open Questions / Remaining Decisions

- Final confirmation of the exact gas constants by client teams and AllCoreDevs (targets above are illustrative but intentional).
- Final confirmation of address `0x20` by AllCoreDevs.

## References

1. Ethereum Magicians, “EIP: ETH as ERC20 Precompile”, December 2022. https://ethereum-magicians.org/t/eip-eth-as-erc20-precompile/12095 (source of the historical WETH gas-guzzler observation).
2. ERC-7528: ETH (Native Asset) Address Convention (Final). https://eips.ethereum.org/EIPS/eip-7528
3. ERC-20: Token Standard. https://eips.ethereum.org/EIPS/eip-20
4. ERC-2612: Permit Extension for EIP-20 Signed Approvals. https://eips.ethereum.org/EIPS/eip-2612
5. WETH9 canonical contract. https://etherscan.io/address/0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
6. Evmos native-token-as-ERC20 precompile (precedent). https://github.com/evmos/evmos/blob/main/precompiles/werc20/werc20.go
7. EIP-7702: Set Code for EOAs. https://eips.ethereum.org/EIPS/eip-7702
8. Earlier related discussion (2019). https://ethereum-magicians.org/t/add-to-ether-the-erc20-token-logics-get-rid-of-weth/3659
9. EIP-7708: ETH transfers emit a log (Review). https://eips.ethereum.org/EIPS/eip-7708 — complementary proposal for system-level Transfer-style logs on all ETH value transfers.
10. Preferential Gas Costs for Native ETH Operations (companion proposal that applies preferential factors to the general native opcodes).

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
