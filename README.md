# Active multi-chain wallet drainer — incident report

**Status: intake fully down, proceeds still unmoved.** As of 2026-08-12 nine domains are
restricted by Cloudflare, six no longer resolve, and the origin server is offline. A tenth
domain — `trust-premium.pro`, registered 2025-12-26 and missed by every earlier report —
was found dormant-but-unrestricted on 2026-08-12 and reported; see
[LIVE-INFRASTRUCTURE.md](LIVE-INFRASTRUCTURE.md). As of 2026-08-11 16:25 UTC every
hostname in this cluster returns "Suspected Phishing | Cloudflare" — the operation has no live
intake anywhere. The stolen balances sit on collector wallets, untouched, on two chains.

> **Update 2026-08-11 — this is not a BNB Chain operation.** The same drainer contract address
> `0x3a85da7f43c7b9946a450b55019f3e05e637ab11` carries **identical bytecode (9,242 bytes) on
> five chains** — Ethereum, BNB Chain, Arbitrum, Optimism and Base — and `owner()` returns the
> same address on every one. Sixteen further victims were identified on Ethereum, where
> **4,191.58 USDT and 1,920.03 USDC are still sitting on a collector that has never sent a
> single transaction** (nonce 0). See [MONEY-TRAIL.md](MONEY-TRAIL.md).

| | |
|---|---|
| **Victims** | **60 unique** — 45 on BNB Chain, 42 on Ethereum, overlap counted once |
| **Total stolen** | **405,338 USD** — 63,477 on BNB Chain, 341,861 on Ethereum. Includes a single 184.77 stETH theft worth 330,364 USD, found 2026-08-12 |
| **Chains** | **5** — Ethereum, BNB Chain, Arbitrum, Optimism, Base (same contract, same owner) |
| **Collectors** | **2 addresses**, each holding balances on multiple chains |
| **Period** | 2026-06-30 → 2026-08-10 |
| **Still on the collectors** | 24,487 USD — 17,369 on BNB Chain (no freeze function) + **~6,790 in freezable stablecoins** on Ethereum / Arbitrum / Optimism. The 330,364 USD stETH theft was liquidated on 2026-07-12 and is gone |
| **Registrars** | **4** — Fewmoretaps/Trustname (8 domains), NiceNIC (3), PDR/PublicDomainRegistry (3, bought Dec 2025–Feb 2026), plus registry data still being collected |
| **Operation status** | **no live intake** — nine domains restricted by Cloudflare, six no longer resolve, one dormant zone reported 2026-08-12, origin server offline. Last victim deposit 2026-08-10 15:03 UTC |

Everything below is verifiable by anyone against public nodes — no trust required.

---

## The three addresses

| Role | Address |
|---|---|
| **Drainer contract** | [`0x3a85da7f43c7b9946a450b55019f3e05e637ab11`](https://bscscan.com/address/0x3a85da7f43c7b9946a450b55019f3e05e637ab11) |
| **Collector wallet** (holds the stolen funds) | [`0x5655e7a5197cc1a1805387bc82dfffe901dfc552`](https://bscscan.com/address/0x5655e7a5197cc1a1805387bc82dfffe901dfc552) |
| **Operator / contract owner** | [`0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d`](https://bscscan.com/address/0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d) |

Token stolen: Binance-Peg BSC-USD (USDT) `0x55d398326f99059ff775485246999027b3197955`

---

## How the attack works

1. **Paid Facebook/Reels advertising** promotes a "issue a virtual card" offer.
   Meta campaign ID `120253487369790536`, campaign name `KING_SUMMER_080`.
2. The ad links to the phishing site **`trusted-settings.com`**.
3. The site presents a working-looking product flow — see below — ending in a wallet
   connection. What is actually signed is an **unlimited ERC-20 approval** to the drainer
   contract.
4. The operator calls function `0xdc772e94` (labelled **"Pull"**) on the contract, which
   executes `transferFrom` against the victim and sweeps the balance into the collector wallet.

### The victim-side flow (mobile)

Reported by a victim who went through it on iOS:

1. Tap the ad in Facebook Reels → land directly on `trusted-settings.com` (no intermediate
   redirect page)
2. Landing page offers to issue a virtual card → tap **"issue card"**
3. **A choice of card types is presented** — several variants to pick from. This step exists
   purely to make the flow feel like a genuine product
4. After choosing: *"connect wallet to issue the card"*
5. Tapping connect fires a **deeplink that opens the wallet app directly** (Trust Wallet in
   this case) — mobile-first, no desktop WalletConnect QR
6. Inside the wallet, the prompt is framed as card issuance. It is the unlimited approval
7. Sweep lands 6 seconds later

Note that the decoy pages described below were **never shown during the real attack** — the
victim saw the full product interface. The decoys are what the site serves to scanners and
reviewers with no wallet present.

The card-selection step and the mobile deeplink flow are useful fingerprints for identifying
the same template on the earlier, still-unidentified domains.

### The sweep is automated — six seconds

Timing measured on one victim (this report's author):

| Time (UTC) | Event |
|---|---|
| `22:18:34` | victim signs the approval — `100,000,000,000` USDT to `0x3a85da7f…ab11` |
| `22:18:40` | **sweep executed — 6 seconds later**, 2,023.40 USDT taken |
| `22:39:18` | victim revokes the approval — 21 minutes too late |

Approval tx: [`0xeff085534c4e643c…`](https://bscscan.com/tx/0xeff085534c4e643cbe927bceae06d3b5c2097a6c09080a5a12dd352d7b13f24d)
Sweep tx: [`0xd2ae7c66d3565…`](https://bscscan.com/tx/0xd2ae7c66d356598bbd01d9ac5eef912f64dac1cb7879285478a774d6ba31eb64)

The operator address monitors the chain and sweeps in the next block. **There is no practical
window in which a victim can notice and revoke.** Any advice along the lines of "revoke quickly
if you think you signed something bad" does not apply to this operation.

The victim never signs the theft transaction itself: **it is sent and gas-paid by the attacker.**
No seed phrase or private key is ever involved — the unlimited approval is enough.

### The drainer contract is purpose-built

Functions exposed by the bytecode of `0x3a85da7f…ab11`:

```
owner()                      → 0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d
transferOwnership(address)
setOperator(address,bool)
operators(address)
0xdc772e94                   → sweep(token, victim, recipient, amount) → transferFrom
```

An owner-plus-operators access model on a contract whose only purpose is moving third-party
funds. This is not a misconfigured dApp; it is tooling built for draining wallets at scale.

Each sweep emits an event, which makes full victim enumeration reproducible:

```
topic0 = 0x6b293682fe640fa0011198470a59ab942eba2642e807496cfb96d32861107ce6
topics = [operator, token, victim]
data   = (recipient, amount)
```

---

## Victims

> **For security teams:** [INDICATORS.md](INDICATORS.md) is the canonical, always-current
> list of every address, domain and cash-out hop in this cluster. Watch that file — new
> indicators go there first.

**Complete data:**
- **[LIVE-INFRASTRUCTURE.md](LIVE-INFRASTRUCTURE.md)** — the domain cluster, the exposed origin server, deployment timestamps, cloaking mechanics and every report filed. **Start here if you work at a registrar, CDN or hosting provider.**
- **[CASE-MAP.md](CASE-MAP.md)** — **start here.** The whole operation on one page: flow diagram, what is verified vs inferred, the eight open questions and who can answer each, and the predicted path the remaining funds will take.
- **[MONEY-TRAIL.md](MONEY-TRAIL.md)** — where the laundered funds actually went, traced transaction by transaction. Includes section 3F: the **184.77 stETH theft worth 330,364 USD**, liquidated the same day via CoW Protocol and 1inch, and followed through five previously undocumented addresses to the same terminus. **Start here if you work at an exchange or at Tether** — the proceeds become real ERC-20 USDT, which *can* be frozen.
- **[ALL-SWEEPS.md](ALL-SWEEPS.md)** — all 51 sweep events: victim → collector → token → amount → timestamp
- **[all-sweeps.csv](all-sweeps.csv)** — same data, machine-readable
- **[GAS-FUNDING.md](GAS-FUNDING.md)** / [gas-funding.csv](gas-funding.csv) — every gas-funding transfer the operator sent to his targets, decoded tx-level
- **[FULL-CLUSTER-ANALYSIS.md](FULL-CLUSTER-ANALYSIS.md)** — two collectors, cash-out route, totals
- [VICTIMS.md](VICTIMS.md) — the original 14 (collector #1 only, kept for reference)

Summary: 14 victims, 17,369.26 USD. Largest single loss **11,839.49 USD**; three losses above
2,000 USD; the remainder are small wallets drained of everything they held (down to 11 USD).

The sum of all 14 incoming transfers matches the collector's current balance **exactly** —
confirming nothing has been withdrawn.

---

### Ad campaign parameters (for threat hunting)

The full landing URL as delivered by the ad, with parameters intact:

```
https://trusted-settings.com/?utm_source=fb&utm_medium=cpc
  &utm_campaign=KING_SUMMER_080+338267898899915
  &utm_content=KING_SUMMER_080+338267898899915
  &utm_term=New+Sales+ad
  &setting=m100
  &utm_id=120253487369790536
  &fbclid=…
```

| Parameter | Value |
|---|---|
| `utm_id` (Meta campaign ID) | `120253487369790536` |
| `utm_campaign` / `utm_content` | `KING_SUMMER_080 338267898899915` |
| `utm_term` | `New Sales ad` |
| `utm_medium` | `cpc` — confirms paid placement |
| `setting` | `m100` — operator's own parameter, likely a landing variant ID |

Decoding the base64 segments inside the `fbclid` yields `app_id = 6628568379` plus the field
markers `adid`, `fdid`, `aem`, `srtc`.

The naming convention `KING_SUMMER_080` and the `setting=m100` parameter are the most useful
pivots for identifying the earlier domain(s) — the same operator most likely reused both the
naming scheme and the landing template.

## The phishing site

| | |
|---|---|
| Domain | `trusted-settings.com` |
| Registered | **2026-07-28** — 11 days before the last theft, minimum 1-year term |
| Registrar | Fewmoretaps OÜ d/b/a Trustname.com (Estonia) |
| DNS | Cloudflare (`asa.ns.cloudflare.com`, `clay.ns.cloudflare.com`) |
| Status | **Live as of 2026-08-09** |

### It uses cloaking

The raw HTML **always** contains the drainer payload — `Crypto Cards`, `connect wallet`,
`WalletConnect`, `approve`, `Ethereum`. But client-side JavaScript decides what to render:

- **No wallet detected / automation suspected** → a harmless placeholder is shown.
  Observed variants: *"Your request is in progress / Access Queue / Automatic check"* and
  *"secure-gateway / Preparing access / route verification"*.
- **Wallet present and anti-bot checks passed** → the "issue card" flow with the approval prompt.

This is why a reviewer visiting the URL sees nothing suspicious, and it is how the advertising
passed Meta's ad review. Independent evidence:

- [urlscan.io scan 019fbeba](https://urlscan.io/screenshots/019fbeba-ac8c-702a-a250-a4d6991b8861.png) (2026-08-01)
- [urlscan.io scan 019fbeb8](https://urlscan.io/screenshots/019fbeb8-4b34-72f7-8578-e10f7fbcb214.png) (2026-08-01)
- `evidence/cloak-secure-gateway.png` — captured 2026-08-09, second decoy variant

### There was an earlier domain

The **first theft occurred 2026-07-22**, but `trusted-settings.com` was only registered
**2026-07-28**. The same contract and the same collector were therefore fed by at least one
**earlier domain**. This is a rotating-domain operation, and the real victim count is very
likely higher than the 14 documented here.

---

## Why the funds cannot simply be frozen

The stolen token is **Binance-Peg BSC-USD**, issued by Binance — not Tether. Tether's freeze
mechanism (thousands of addresses blacklisted) **does not apply here**.

Bytecode analysis of `0x55d398326f99059ff775485246999027b3197955` shows:

- present: `owner()`, `getOwner()`, `transferOwnership()`, `mint()`, `burn()`, standard transfers
- **absent: `blacklist`, `addBlackList`, `freeze`, `pause`, `destroyBlackFunds`**
- contract is **not upgradeable** — such functions cannot be added later

Therefore the funds cannot be frozen at token level by anyone **while they remain on BNB Chain**.

That is not the end of it, though. Tracing both of the operator's earlier cash-outs showed that
he does not cash out on BNB Chain at all — he bridges to Ethereum first, and on the other side
the value arrives as **genuine ERC-20 Tether** `0xdac17f958d2ee523a2206206994597c13d831ec7`,
Tether's own contract, with a working blacklist function. The same applies to the 1,920.03 USDC
already sitting on the Ethereum collector, which Circle can freeze.

So the interception point is not the exchange deposit at the end of the chain — by then the
funds are commingled with thousands of unrelated users' money and cannot be separated. **It is
the bridge crossing**, and both previous crossings were followed by 7–22 hours of inactivity
before distribution began. See [MONEY-TRAIL.md](MONEY-TRAIL.md) for the full route with
transaction hashes.

---

## Reported to

| Date (2026) | Channel | Reference / status |
|---|---|---|
| 08-08 | Tether compliance | **#459702** — declined, correctly: the stolen token is Binance-issued. Reopened 08-11 on new grounds, see below |
| 08-09 | **FBI IC3** | `5036615c3cd54d23ba7bd1c089786d1d` — main complaint |
| 08-09 | **FBI IC3** — update 1 | `7f90f61bf96c4dc585999781b2483aa7` — full victim list |
| 08-09 | Registrar abuse (Trustname) | **#ABS-48193**, later **#ABS-48857** — RAA §3.18.1 |
| 08-09 | Chainabuse | `dd005758-ff9e-4aca-a747-45fe48b467fc` — all addresses tagged |
| 08-09 | SEAL 911 / Security Alliance | Contacted for tracing assistance |
| 08-09 | BscScan | Address watchlist + phishing report |
| 08-10 | **FBI IC3** — updates 2–5 | `37289b4b2028461289a70e32c4345128` (45 victims, bridges), `f020e29e05a446618f2911d68b1b51a0` (Binance withdrawal), `012e182ac6f049e18e95f3c067748f46` (domain cluster), `074a52029d8c4936aefaf50a9aa56b2f` (hosting, deploy timings) |
| 08-10 | **BNB Chain** official support | `4bcf92ea-7a94-4711-a02e-f882757f0fcc` — **answered**: confirmed in writing that Binance-Peg BSC-USD has no freeze/blacklist function and is not upgradeable |
| 08-10 | **FBI IC3** — update 6 | `805f8b82ecd54261aa1979c288fac363` — five chains, 16 further victims, freezable balances |
| 08-10/11 | Cloudflare abuse | Six reports — **all nine domains restricted**, hosting provider disclosed |
| 08-10/11 | HashDit (BNB Chain security partner) | Addresses **flagged with partner services**, phishing domains flagged |
| 08-11 | Tether — reopened | **#459702** — new grounds: both laundering channels convert the proceeds into genuine ERC-20 Tether on Ethereum, which *is* freezable |
| 08-11 | **ChangeNOW** compliance | **Answered**: *"Upon request from authorities we will be able to block all the relevant addresses and provide all the data available"* |
| 08-11 | **SWFT / Bridgers** | Notified, incl. their own order ID `o3lkby` recovered from the contract log |
| 08-11 | **Symbiosis** | Notified — records preservation request for the 2026-07-13 bridge transaction |
| 08-11 | **Binance** support | Case **#167289747** — records preservation request, escalated to the Security team |

---

## For investigators — where legal process should go

Compiled the hard way, by asking each service directly. Every item below is either a written
answer received or a published policy, not a guess.

| Target | Channel | What they can actually do |
|---|---|---|
| **Binance** | Law Enforcement Portal at `binance.com/en/support/law-enforcement` — mark the legal process type as **`Exigent`** for urgent requests. Russian/Belarusian agencies: `case@binanceholdings.ru` | Identify the account behind withdrawal `0xe26636d7…`. Confirmed to cooperate **only** with law enforcement — victims cannot obtain account data or freezes directly |
| **Tether** | `support@tether.to`, existing ticket **#459702** | Freeze genuine ERC-20 USDT on Ethereum. Cannot touch Binance-Peg BSC-USD on BNB Chain — different issuer, no freeze function |
| **Circle** | `support@circle.com` | Freeze USDC — 1,920.03 USDC sits on the Ethereum collector |
| **ChangeNOW** | `compliance@changenow.io`, guidelines at `changenow.io/en/law-enforcement-request-guidelines` | Stated they will block relevant addresses and provide data on an authorities' request. Wants addresses in XLSX/CSV. Has an expedited track — mark `URGENT` with justification |
| **SWFT / Bridgers** | `contact@swft.pro`, `bd@swft.pro` | **Cannot freeze anything** — published policy states they are "unable to restrict, transfer, or otherwise perform sensitive operations on users' crypto assets". Entity is Explorerx Digital Limited, **Seychelles**; foreign agencies need authorisation signed by the Seychelles Attorney General. Retention **12 months** — the window on the 2026-07-04 transaction closes around July 2027 |
| **Symbiosis** | `legal@symbiosis.finance` | Bridge request records, relayer records, originating IP, integrator attribution for the 2026-07-13 swap |
| **BNB Chain** | `@bnbchain_official_bot` on Telegram | Confirmed no technical recovery path exists at token level. Routes address flagging via HashDit |

**The single interception point.** On BNB Chain the stolen token cannot be frozen by anyone —
this is settled, in writing, by BNB Chain themselves. After a bridge crossing the same value
becomes genuine ERC-20 Tether on Ethereum, which Tether *can* freeze. **That crossing is the
only moment the money is recoverable**, and both previous cash-outs took 7–22 hours between
leaving BNB Chain and being distributed. The route is known in advance; the funds have not
moved yet.

---

## If you are one of the 60 victims

**Not sure if you were hit?** → **[CHECK-IF-AFFECTED.md](CHECK-IF-AFFECTED.md)** — a
2-minute check using only a block explorer, no wallet connection required.

Check [VICTIMS.md](VICTIMS.md) — if your address is listed, this happened to you too.

**What is worth doing:**

1. **Revoke your approvals** at [revoke.cash](https://revoke.cash) (BNB Chain network) or via
   BscScan's Token Approval Checker. The approval survives until revoked and will drain any
   new funds arriving in the wallet.
2. **File your own IC3 complaint** at [ic3.gov](https://www.ic3.gov). Reference the addresses
   above — multiple independent complaints about one operation carry far more weight than one.
3. Report to your local law enforcement as well.

**A warning, from experience.** The moment you mention publicly that you were drained, people
will contact you offering "recovery". They will ask you to "synchronize" or "validate" your
wallet on some site, and that site will ask for your seed phrase, keystore file or private key.
**Every single one of them is a thief.** No legitimate service — not BNB Chain, not Trust
Wallet, not any exchange, not law enforcement — will ever ask for your seed phrase or charge
you to recover stolen funds. Losing the wallet itself is far worse than losing what was
already taken.

---

## Contact

Compiled by one of the victims (loss: 2,023.40 USD, 2026-08-08).
Full evidence file — including all transaction hashes — available on request:
**romaxa552015@gmail.com**

If you work in security at BNB Chain, an exchange, or a wallet provider: flagging these three
addresses in your tooling would warn users before they sign the next approval. The operation
is still funded and, as of this writing, still live.
