# Ethereum: Strengthening ETH as Preferred Asset & Validator-Set Resilience

Early draft proposals exploring two complementary directions for Ethereum:

1. Making native ETH the most natural and preferred asset inside the EVM ecosystem.
2. Creating continuous, incentive-compatible mechanisms that reward structural diversity and resilience in the validator set.

These are independent ideas. They are published here as working drafts for community feedback before any formal Ethereum Magicians discussion or EIP submission.

## The Drafts

| # | Title | Status | Link |
|---|-------|--------|------|
| 1 | Native ETH as ERC-20 Precompile | Draft | [01-native-eth-as-erc20-precompile.md](./01-native-eth-as-erc20-precompile.md) |
| 2 | Preferential Gas Costs for Native Value Transfers | Draft | [02-preferential-gas-costs-for-native-value-transfers.md](./02-preferential-gas-costs-for-native-value-transfers.md) |
| 3 | Continuous Diversity Scoring and Incentives for Validator Resilience | Draft | [03-continuous-diversity-scoring-and-incentives.md](./03-continuous-diversity-scoring-and-incentives.md) |

## Motivation

The trigger for writing these proposals was the appearance of EIP-8361 "Tapered Issuance Burn". While agreeing with the need for curbing staking intermediaries, I do not think the EIP can achieve those goals.
Therefore, in the spirit of "being the change I want to see" and channeling my objection into something constructive, I hereby put forward my own ideas of how to grow the pie for Ethereum instead of getting
lost in discussions about redistributing the existing pie.
As a home and solo staker, as well as node operator for Rocket Pool and Nodeset, my long-term motivation is the creation of a successful, resilient network Ethereum with ETH as its money. For me, this means

- Making ETH the natural reserve currency of the digital economy by applying monetary properties and principles of TradFi reserve currencies to ETH
- Promoting incentive mechanisms to further the decentralization of the Ethereum validator set in "peacetime" to prepare the network for adverse scenarios

Reserve currencies in traditional finance derive their dominant status from a set of reinforcing properties:

- Deep and highly liquid markets — especially for safe, trusted assets
- Strong network effects that make it the default unit of account, medium of exchange, and pricing standard - effects that, once established, tend to persist and deepen over time
- Institutional credibility, stability and rule of law that underpin long-term trust
- Safe-haven characteristics that attract capital in times of stress

The U.S. Dollar illustrates this clearly: it is not treated as one currency among many; it receives preferential treatment across global trade, FX reserves, commodity markets, and capital flows precisely because of these attributes.
Transposed to Ethereum, the same logic requires that ETH be deliberately elevated beyond the status of “just another ERC-20.” At present, native ETH is in important respects less convenient than a typical ERC-20 token
(it lacks a uniform interface, forces special-casing or wrapping, and therefore carries friction that pure tokens do not). For ETH to function as the reserve asset of the digital economy it needs protocol-level preferential treatment —
seamless ERC-20 compatibility without sacrificing native advantages, relatively lower costs for value-moving operations that use ETH, and structural incentives that keep the underlying network maximally credible and resilient —
so that users, protocols, and markets naturally prefer it over any other token.

Safe-haven characteristics that attract capital in times of stress can only exist if the underlying network remains operational and credible precisely when stress materializes. For ETH to inherit this property, the validator set must
be structurally resilient to extreme events — continental-scale internet partitions, large natural disasters, infrastructure failures, or coordinated attacks. Such resilience cannot be improvised at the moment of crisis; it must be
cultivated in peacetime through continuous incentives that favor geographic, client, hardware, and operational diversity. A validator set that optimizes purely for efficiency and cost during calm periods will tend toward concentration
and correlation, leaving the network fragile exactly when capital most needs a reliable digital safe haven. By deliberately rewarding the structures that can survive extreme conditions while times are still smooth, Ethereum can embed
genuine safe-haven credibility into ETH itself, reinforcing its potential role as the preferred reserve asset of the digital economy.

## Goals

- Reduce friction so that native ETH becomes the seamless default asset wherever an ERC-20 interface is expected.
- Amplify the existing cost advantage of moving native value relative to pure token logic.
- Introduce a continuous (non-binary) diversity score for validators based on a broad mix of observable signals, with modest reward redistribution that slightly reduces overall issuance while encouraging resilience properties.

## Design Principles

- Neutral and network-centric
- Continuous signals and incentives rather than hard binary classifications
- Minimal viable incentive design (pay only enough to encourage the desired behavior)
- Awareness of prior art (ERC-7528, 2022 ETH-as-ERC20 discussions, EIP-7716 and related anti-correlation research, geographic diversity work, etc.)

## Status & Feedback

These are early drafts. Feedback, critiques, alternative designs, and potential collaborators are very welcome.

Please open an issue in this repository or reach out via the usual Ethereum discussion channels.

---

*Last updated: August 2026*
