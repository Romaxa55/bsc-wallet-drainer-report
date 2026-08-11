# Active multi-chain wallet drainer — incident report

**Status: funds still unmoved.** As of 2026-08-11 the attacker has not laundered the current
balances — they sit on collector wallets, untouched, on two chains.

> **Update 2026-08-11 — this is not a BNB Chain operation.** The same drainer contract address
> `0x3a85da7f43c7b9946a450b55019f3e05e637ab11` carries **identical bytecode (9,242 bytes) on
> five chains** — Ethereum, BNB Chain, Arbitrum, Optimism and Base — and `owner()` returns the
> same address on every one. Sixteen further victims were identified on Ethereum, where
> **4,191.58 USDT and 1,920.03 USDC are still sitting on a collector that has never sent a
> single transaction** (nonce 0). See [MONEY-TRAIL.md](MONEY-TRAIL.md).

| | |
|---|---|
| **Victims** | **61** — 45 on BNB Chain, 16 more on Ethereum |
| **Total stolen** | **63,477.36 USD** on BNB Chain, plus 6,111.61 USD on Ethereum |
| **Chains** | **5** — Ethereum, BNB Chain, Arbitrum, Optimism, Base (same contract, same owner) |
| **Collectors** | **2 addresses**, each holding balances on multiple chains |
| **Period** | 2026-06-30 → 2026-08-10 |
| **Still recoverable** | 17,369.26 USD (BNB Chain, no freeze function) + **6,781 USD in freezable stablecoins** on Ethereum / Optimism / Arbitrum |
| **Operation status** | five phishing domains restricted by Cloudflare 2026-08-10; last victim deposit 2026-08-10 15:03 UTC |

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
- **[MONEY-TRAIL.md](MONEY-TRAIL.md)** — where the laundered funds actually went: BNB Chain → Symbiosis bridge → Ethereum → a service wallet with 1.66M transactions. 233,511 USDT through one consolidation address in a single day. **Start here if you work at an exchange or at Tether** — the funds become real ERC-20 USDT, which *can* be frozen.
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

Therefore the funds cannot be frozen at token level by anyone. The only realistic interception
point is the moment the attacker deposits them into a centralized exchange, where KYC and
account freezes apply. The collector wallet is under continuous monitoring for exactly that
event.

---

## Reported to

| Date (2026) | Channel | Reference / status |
|---|---|---|
| 08-08 | Tether compliance | #459702 — declined (correctly: token is Binance-issued) |
| 08-09 | **FBI IC3** | Submission `5036615c3cd54d23ba7bd1c089786d1d` |
| 08-09 | **FBI IC3 — update** | Submission `7f90f61bf96c4dc585999781b2483aa7` (full victim list) |
| 08-09 | Registrar abuse (Trustname) | Case **#ABS-48193** — under investigation, RAA §3.18.1 |
| 08-09 | Chainabuse | Filed — all three addresses tagged |
| 08-09 | SEAL 911 / Security Alliance | Contacted for tracing assistance |
| 08-09 | BscScan | Address watchlist + phishing report |

---

## If you are one of the 14 victims

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
