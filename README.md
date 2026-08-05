# Ethereum: Strengthening ETH as Preferred Asset

Early draft proposals exploring ways to make native ETH the most natural and preferred asset inside the EVM ecosystem.

These are complementary ideas, although they would be most effective if both proposals were enacted. They are published here as working drafts for community feedback before any formal Ethereum Magicians discussion or EIP submission.

## The Drafts

| # | Title | Status | Link |
|---|-------|--------|------|
| 1 | Native ETH as ERC-20 Precompile | Draft | [01-native-eth-as-erc20-precompile.md](./01-native-eth-as-erc20-precompile.md) |
| 2 | Preferential Gas Costs for Native ETH Operations | Draft | [02-preferential-gas-costs-for-native-eth-operations.md](./02-preferential-gas-costs-for-native-eth-operations.md) |

## Motivation

In the spirit of "being the change I want to see" and channeling my ideas into actionable items instead of hoping for others to understand and take them up for me, I hereby put forward ideas of how to grow the pie for Ethereum instead of getting lost in discussions about redistributing the existing pie.

The long-term motivation is a successful, resilient Ethereum network with ETH as its money. The starting point is the question: "What was I hoping to get when buying ETH?". This includes making ETH the natural reserve currency of the digital economy by applying monetary properties and principles of TradFi reserve currencies to ETH.

Reserve currencies in traditional finance derive their dominant status from a set of reinforcing properties:

- Deep and highly liquid markets — especially for safe, trusted assets
- Strong network effects that make it the default unit of account, medium of exchange, and pricing standard — effects that, once established, tend to persist and deepen over time
- Institutional credibility, stability and rule of law that underpin long-term trust
- Safe-haven characteristics that attract capital in times of stress

The U.S. Dollar illustrates this clearly: it is not treated as one currency among many; it receives preferential treatment across global trade, FX reserves, commodity markets, and capital flows precisely because of these attributes. Transposed to Ethereum, the same logic requires that ETH be deliberately elevated beyond the status of “just another ERC-20.” At present, native ETH is in important respects less convenient than a typical ERC-20 token (it lacks a uniform interface, forces special-casing or wrapping, and therefore carries friction that pure tokens do not). For ETH to function as the reserve asset of the digital economy it needs protocol-level preferential treatment — seamless ERC-20 compatibility without sacrificing native advantages, and relatively lower costs for value-moving operations that use ETH — so that users, protocols, and markets naturally prefer it over any other token.

Advancing the structural resilience and decentralization of the Ethereum validator set remains desirable. Substantial deliberations on continuous incentive mechanisms that could reward diversity and resilience properties were undertaken. After examination of prior art, practicality, residual gameability, and the difficulty of producing a crisp, high-traction design, **no EIP is being proposed in that direction at this time**.

## Goals

- Reduce friction so that native ETH becomes the seamless default asset wherever an ERC-20 interface is expected.
- Amplify the existing cost advantage of operations involving native ETH balances relative to pure token logic.

## Design Principles

- Neutral and network-centric
- Minimal viable changes that improve the usability and preferential status of native ETH
- Awareness of prior art (ERC-7528, 2022 ETH-as-ERC20 discussions, etc.)

## Status & Feedback

These are early drafts. Feedback, critiques, alternative designs, and potential collaborators are very welcome.

Please open an issue in this repository or reach out via the usual Ethereum discussion channels.

---

*Last updated: August 2026*
