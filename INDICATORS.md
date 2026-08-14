# INDICATORS — canonical list

**This is the single file to watch.** All new indicators (domains, collectors, cash-out hops,
campaign IDs) are added here. Last updated: **2026-08-14** — intake confirmed live again; a victim was swept 2026-08-13 23:36 UTC through a domain not in this report.

**How this repo is organised.** This file is the *operational* view: what to act on, who is at
risk right now, what to capture when funds move. The full decoded datasets live alongside it —
[ALL-SWEEPS.md](ALL-SWEEPS.md) (every theft), [GAS-FUNDING.md](GAS-FUNDING.md) (every gas
pre-fund), each with a CSV. Indicators appear here first; the datasets are the reference
behind them.

Everything is on BNB Smart Chain (chainId 56) unless stated otherwise.

---

## Core cluster

| Role | Address | Status |
|---|---|---|
| Drainer contract | `0x3a85da7f43c7b9946a450b55019f3e05e637ab11` | active, 51 sweeps |
| Operator / contract owner | `0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d` | sole caller of all 52 txs |
| Collector #1 | `0x5655e7a5197cc1a1805387bc82dfffe901dfc552` | **holding 17,369.26 USDT, never moved** |
| Collector #2 | `0x62f1a65cdf65ab42a6b520510d58f2f71750ea56` | cashed out, 387.51 USDT left |

Contract interface: `owner()`, `transferOwnership(address)`, `setOperator(address,bool)`,
`operators(address)`, sweep `0xdc772e94(token, victim, recipient, amount)`.
Sweep event topic0: `0x6b293682fe640fa0011198470a59ab942eba2642e807496cfb96d32861107ce6`

## Cash-out path — traced

Collector #2 emptied itself through **two different routes**:

### Route A — DEX swap, 2026-07-13 (traceable, did not use a bridge)

`0x7f036a6002a7ecc1883e14c62012c4df57d009e083d22dfea264af8e8a2e3f8d`

```
collector #2  ──24,867.196678 USDT──►  0x980447ddcef79a7499da4538da8fc59bacad6997
                                       (router/intermediary, now empty, nonce 1)
              ──► pools: 0xa58bdd0ab5ebbb8dc425090fea8fd0ba969c1668
                         0x28e2ea0908...  0x16b59c9056...  0x52aa899454...
              ──► 1inch v5 router 0x1111111254eeb25477b68fb85ed929f73a960582
              ──24,845.976714 USDC──►  0x5aa5f7f84ed0e5db0a4a85c3947ea16b53352fd4
```

**Final destination `0x5aa5f7f84ed0e5db0a4a85c3947ea16b53352fd4` — IDENTIFIED:
Symbiosis Finance cross-chain bridge.**

- Contract is **verified**, and is a **proxy** (implementation `0x80347BfC…BC9B51630`)
- Contract creator: **`Symbiosis: Deployer`**, deployed ~3 years 269 days ago
- All inbound calls use the **`Synthesize`** method — Symbiosis' cross-chain entry function
- 3,640 transactions; >$1,019,084 in >110 tokens; $1,054,491 multichain portfolio

**Correction:** the ~506k USDC balance is the bridge's own liquidity, **not** attacker funds.
An earlier version of this file suggested it might be an aggregation wallet — that was wrong.

So Route A is also a bridge, not an off-ramp: USDT was swapped to USDC via 1inch and then
pushed cross-chain through Symbiosis. **Both cash-out routes leave BSC through bridges**
(Symbiosis and Bridgers/SWFT), which means the funds' destination chain is where any
interception would have to happen.

### Route B — cross-chain swap, 2026-07-04

```
0x2d39ad6eb94034fd489a1818de55e285f8f7271ee837b6faef4342b879c92e9e  21,027.36938 USDT
0x52c5fad0bd671593efda739a354f0fad1d9b356b2bfdf33307d697b768da6128   9,000.00000 USDT
                                    ──►  0xb685760ebd368a891f27ae547391f4e2a289895b
                                         labelled "Bridgers" (SWFT Blockchain)
                                         currently holds 94.24 BNB + 7,611 USDT
```

Method selector used on the bridge: `0x9ddf93bb`. 30,027 USDT total left the chain here.

**Prediction:** the 17,369.26 USDT still sitting on collector #1 will most likely follow one
of these two routes. Pre-notifying both endpoints beats reacting afterwards.

---

## ⚠ When funds move — what to capture

**Read this before doing anything else if collector #1 moves.**

Both observed cash-out routes exit BSC through bridges, and the bridge request ID is the
only field that lets a bridge team map the destination-chain payout. Miss it, and the trail
stops at the BSC boundary. Capture it *from the transaction receipt*, not from the explorer
summary view.

### 1. The bridge transaction hash

The tx where funds leave `0x5655e7a5197cc1a1805387bc82dfffe901dfc552`.

### 2. The bridge request ID — the critical pivot

**If the destination is Symbiosis** (`0x5aa5f7f84ed0e5db0a4a85c3947ea16b53352fd4`):

Look in the receipt logs for an event on the Symbiosis contract with

```
topic0 = 0x31325fe0a1a2e6a5b1e41572156ba5b4e94f0fae7e7f63ec21e9b5ce1e4b3eab
```

From that event, record:

| Field | Where | Example from Route A |
|---|---|---|
| **Request ID** | `data[0]` | `4b6b193fff82da0cb254601eac1b9fbb6258c810dda98073d15f8852d2f8047e` |
| Target chain internal ID | `topic2` | `0xd38bb4` = 13863348 |
| Sender | `topic1` | the collector |
| Amount / token | `data[2]`, `data[3]` | 24,845.976714 USDC |

A second event, `topic0 = 0x5a297b2c9a9f94a0f4e5a796c74ad38e219d1185fccf5f79c18726a830c2b6f5`,
carries the client identifier in `topic1` as ASCII — Route A showed `symbiosis-app-tw`.

**If the destination is Bridgers / SWFT** (`0xb685760ebd368a891f27ae547391f4e2a289895b`):

Capture any order ID present in the calldata or logs. Route B used method selector
`0x9ddf93bb`; the observed transactions carried only the Transfer event, so the order
reference may need to be requested from SWFT directly using the tx hash.

### 3. Do NOT chase these

- The address in `topic3` of the Symbiosis event is a **relayer/executor** of the bridge, not
  attacker-controlled. Route A showed `0xd99ac0681b904991169a4f398b9043781adbe0c3` — nonce
  253k on Arbitrum, 267k on BSC. It appears in every Symbiosis transaction.
- The bridge contract's own token balances are **protocol liquidity**, not stolen funds.

### 4. Quick way to pull the fields

```bash
# replace TX with the bridge transaction hash
curl -s -X POST https://bsc-dataseed.binance.org/ \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_getTransactionReceipt","params":["TX"]}' \
  | python3 -m json.tool | grep -A6 '31325fe0a1a2e6a5b1e41572156ba5b4e94f0fae7e7f63ec21e9b5ce1e4b3eab'
```

### 5. Where to send it

HashDit Security (Telegram `@hashdit_security_bot`) — case already open, reference this repo.
Include: bridge tx hash, request ID, target chain ID, amount, and which bridge.

---

---

---

## 🎯 DEANONYMIZATION LEAD — operator withdrew from a KYC exchange

**Found 2026-08-10. This is the most significant lead in the case.**

The operator's wallet received exactly one inbound USDT transfer in its entire history — and
it came **directly from Binance**.

```
tx    0xe26636d7d2d8c8f7186e89c5be5a66fdca8fd3af70f826f612f42e459267c843
from  0xeb2d2f1b8c558a40207669291fda468e50c8a0bb   ← "Binance: Hot Wallet 10"
to    0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d   ← the operator
value 14.99 USDT
date  2026-07-11 18:12:35 UTC   (block 109409511, status success)
```

Re-verified 2026-08-11 directly from the transaction receipt. An earlier revision of this
file gave the date as "~2026-07-12"; the block timestamp is 2026-07-11 18:12:35 UTC.

**The 14.99 USDT has never been spent.** The operator wallet still holds exactly that amount
today. The withdrawal has the shape of an address test — a small sum moved off the exchange to
confirm the destination works — carried out two days before the Symbiosis bridging of
2026-07-13.

### Why this matters

Withdrawing from Binance requires a **verified account**. Binance holds, for that account:
identity documents, selfie verification, address, phone, email, login IP history, device
fingerprints and the full transaction record — including the withdrawal to this exact
address.

The operator moved funds from a KYC'd exchange account **straight onto the wallet he uses to
deploy and operate the drainer contract**. Every subsequent action — deploying
`0x3a85da7f43c7b9946a450b55019f3e05e637ab11`, all 51 sweeps, funding victims' gas — traces
back to a wallet that Binance can attribute to a real, verified person.

### Also confirmed

The same operator address **deployed** the drainer contract:

```
contract creation tx 0x6adeed7d793e23c75cd1862680933a33ed814171f94d0f9ea0d4672d2ebb4245
deployer 0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d
```

So deployer, operator and Binance-withdrawal recipient are **one and the same address** —
this is not rented drainer-as-a-service with separated roles.

### What would resolve it

A law-enforcement request to Binance for the account that made withdrawal
`0xe26636d7…c843` on ~2026-07-12 to address `0x7c51bc88…4f6d`. That single query maps the
entire 45-victim operation to an identified individual.

Binance operates a law-enforcement request portal (Kodex). This is not something a victim can
submit — it requires an agency. The FBI IC3 complaints referenced below are the vehicle.


## 🚨 INTAKE IS LIVE AGAIN — 2026-08-13

A victim was drained last night. The sequence, from the chain:

| Time (UTC) | Event | Tx |
|---|---|---|
| 2026-08-13 23:35:47 | Operator sends the victim gas — 0.000163769161353252 BNB | [`0xda5c6130…ac6a`](https://bscscan.com/tx/0xda5c6130c6a080868506964d26d239b4dc0701dc2876092c0b1d8b7651bbac6a) |
| 2026-08-13 23:36:19 | **Victim signs the approval** (+32 s) | [`0x9fed5965…7ae4`](https://bscscan.com/tx/0x9fed596564a80cf988fb13e0f696114893cf45ef4304454cfe28231ccd397ae4) |
| 2026-08-13 23:36:26 | Swept — **7,731.173607880465111116 BSC-USD** (+7 s) | [`0xd838f41c…f61c`](https://bscscan.com/tx/0xd838f41c6e5f64958cbf8474b8321e4da4844d8077b936972ce76b3a6ccff61c) |

Victim: [`0xa5187f47d789e7333bf4803199407c7882958da8`](https://bscscan.com/address/0xa5187f47d789e7333bf4803199407c7882958da8) —
balance now 0, allowance still live.

Drainer, operator and collector are all unchanged.

### Why this matters more than the amount

**The approval is 7 seconds old at the moment of the sweep.** This is not an old allowance
being exercised — it is a fresh signature from a working funnel.

**The gas arrived 32 seconds *before* the victim signed.** The operator's backend saw a wallet
connect with no gas and topped it up so the victim could confirm. The site was serving at that
moment.

**Yet no domain in this report was reachable.** All 17 were checked the same morning: NXDOMAIN
or the Cloudflare interstitial, and the origin 54.39.106.37 is offline.

The conclusion is unavoidable: **at least one live hostname exists that this report does not
have.** If you can identify a BNB Chain phishing page active around 2026-08-13 23:30–23:40 UTC
requesting an approval to `0x3a85da7f43c7b9946a450b55019f3e05e637ab11`, please open an issue.

### Two approval formats — detection note

Not every approval in this campaign is max-uint:

```
victim 2026-08-02   100000000000000000000000000000
victim 2026-08-08   115792089237316195423570985008687907853269984665640564039457584007913129639935
victim 2026-08-13   100000000000000000000000000000
```

100 billion exceeds Tether's entire supply, so the practical effect is identical to an
unlimited approval — but wallet UIs raise the red "Unlimited approval" warning only for
`2^256-1`. A specific number passes without it.

We cannot date when the alternation began or explain it; both formats appear across the
campaign. Noted because **detection rules keyed on max-uint will miss half of these**.

### Collector balance

`0x5655e7a5197cc1a1805387bc82dfffe901dfc552` now holds **25,100.430942 BSC-USD**, up from
17,369.26. The increase is exactly last night's theft. The address has made **one** outgoing
transaction in its entire history — nonce is still 1. Nothing has ever been withdrawn.

---

## 🔴 OPERATION IS STILL LIVE — victims currently in the pipeline

Discovered 2026-08-10 by decoding the operator's own transaction history
(`0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d`, 95 txs: 51 `Pull` + 43 `Transfer`).

### The operator funds his victims' gas

43 of his transactions are small BNB transfers **to the victims themselves**. A victim with
no BNB cannot sign the approval, so the operator pays for it. This is a distinctive,
searchable pattern: *small BNB inbound from the operator, followed days later by an approval
and a sweep*.

It also means victims can be identified **before** they are drained.

### Confirmed at risk — funded AND approved

| Address | Evidence |
|---|---|
| **`0xf08ddde735643ccdf922dd7f8a47b350fe56c743`** | Gas from operator 2026-08-02 02:20:15, approval to the drainer 02:21:21 — **66 seconds later**. Allowance of 100,000,000,000 USDT is **still live**. Balance 0.53 USDT, so nothing has been swept yet; any deposit will be taken automatically. |

Approval tx: [`0xd9f290af…d202`](https://bscscan.com/tx/0xd9f290af73c14fafe4a168edd0ea4b67f33aa2f25abb687dec1246463e8ad202) ·
Gas funding: [`0xbf88d616…5e07`](https://bscscan.com/tx/0xbf88d6166c72c6a3983b8cb775e7f0878e05cdabd56a9c050790ef31bf0b5e07)

> Full decoded dataset of every gas-funding transfer: **[GAS-FUNDING.md](GAS-FUNDING.md)** ·
> [gas-funding.csv](gas-funding.csv). The list below is the short operational subset — the
> wallets worth acting on right now.

### Funded by the operator — status unknown

These received the operator's gas but have **no approval to the drainer**. Whether they are
prospective victims who never signed, the operator's own wallets, or test addresses **cannot
be determined from on-chain data**. Listed for pattern correlation only — no accusation is
implied.

| Address | Funded (UTC) | Amount | Funding tx | State |
|---|---|---|---|---|
| `0x32c24318be3863467fec944ca0058dfe97563bb9` | 2026-08-07 02:58:09 | 0.00016863 BNB | [`0x71e8db67…814e`](https://bscscan.com/tx/0x71e8db67f7097554d0629e793ca5d00b9234930ef42fe6d147fbbe70edbf814e) | nonce 0 — never transacted; holds 30.05 USDT |
| `0xb9c6a49e782d7a742263333a4164d577e75ba107` | 2026-08-08 12:55:50 | 0.00016799 BNB | [`0xd836cf6f…de30`](https://bscscan.com/tx/0xd836cf6f0f39437024105a8ca43bc1f63877849899f54194d272d969c8aade30) | nonce 0 — never transacted; empty |
| `0x531bdf6c39b3b85c40b371c19eac99fe49b0af67` | see dataset | in band | [dataset](GAS-FUNDING.md) | nonce 0 — never transacted; **holds 40.00 USDT** |
| `0xecc6ef652667496aec74d0646ad22609b018db3d` | 2026-08-05 17:27:49 | 0.00016703 BNB | [`0xd77ad15a…2ae2`](https://bscscan.com/tx/0xd77ad15a38ed9195472636ce96d0d7223510047d468f8a611deadb82e4592ae2) | nonce 5; empty; no allowance |

### The funding amounts are computed, not arbitrary

| Date (UTC) | Amount BNB | Recipient |
|---|---|---|
| 2026-08-02 02:20:15 | 0.00017258 | `0xf08ddde7…` (later approved) |
| 2026-08-05 17:27:49 | 0.00016703 | `0xecc6ef65…` |
| 2026-08-07 02:58:09 | 0.00016863 | `0x32c24318…` |
| 2026-08-08 12:55:50 | 0.00016799 | `0xb9c6a49e…` |

Spread across all four is **under 3%**, every one around $0.10. These aren't round numbers or
a fixed constant — they track the prevailing gas price, i.e. the sender is **calculating the
cost of exactly one `approve` transaction at send time** and transferring precisely that.

That makes a tight detection signature: *an inbound native transfer of ~0.00016–0.00018 BNB,
sized to a single approval, from an address that has previously interacted with a known
drainer contract.* An alert fired on that pattern would reach the victim **before** they sign,
which is the only point at which this is preventable.

None of the three overlap with the 45 confirmed victims, and none overlap with the six
wallets that fund the collector — so they are neither confirmed victims nor known
infrastructure.

### Operator activity timeline (most recent first)

```
34 hrs ago   Pull       ← the 2026-08-08 theft
44 hrs ago   Transfer   → 0xb9c6a49e…  (new target funded)
3 days ago   Transfer   → 0x32c24318…
4 days ago   Transfer   → 0xecc6ef65…
6 days ago   Pull
8 days ago   Transfer   → 0xf08ddde7…
11 days ago  Pull
```

The operation did not stop after the last recorded theft — the operator kept funding new
targets as recently as 44 hours ago.

### At-risk wallet — full transaction chain

`0xf08ddde735643ccdf922dd7f8a47b350fe56c743`, documented end to end. Note how fast it runs:

| Time (UTC) | Event | Tx |
|---|---|---|
| 2026-08-02 02:20:15 | operator sends metered gas, 0.00017258 BNB | `0xbf88d6166c72c6a3983b8cb775e7f0878e05cdabd56a9c050790ef31bf0b5e07` |
| 2026-08-02 **02:21:21** | **victim approves 100,000,000,000 USDT to the drainer** — 66 seconds later | `0xd9f290af73c14fafe4a168edd0ea4b67f33aa2f25abb687dec1246463e8ad202` |
| 2026-08-02 02:26:58 | victim approves Rango Exchange (legitimate aggregator) | `0x1ca8eda942fa902c0e5910ef70fd38dfbd6d3820d72ff2e286f5b23f14d86548` |
| 2026-08-02 02:27:02 | victim swaps/bridges 1,000 USDT via Rango — **normal user activity, not theft** | `0x2f236fd37c5e8de56fc5dd09f4acad4240a13169f69c87f7939a9caa44bfb880` |

The approval to `0x3a85da7f…ab11` is **still live** as of 2026-08-10. That wallet is armed.

## Phishing infrastructure

Domains are split into two waves deliberately. The operator moved from a bare dedicated
server to a CDN between 2026-07-30 and 2026-08-03, and the two waves have different
evidentiary weight — merging them into one list would blur what is proven.

### WAVE 2 — current, Cloudflare-fronted (all LIVE)

Six domains, all serving the same kit, all behind Cloudflare with Web Analytics manually
enabled per zone. Verified live 2026-08-10.

| Created (UTC) | Domain | RUM site token |
|---|---|---|
| 2026-07-24 15:52:57 | `promo-premium.pro` | — |
| 2026-07-28 17:32:11 | `promo-settings.com` | `b1af2c3f4e9b4448a818b78d424ae24a` |
| 2026-07-28 17:32:31 | **`trusted-settings.com`** ← used in the theft | `096e19198fbc489e9f32df6b14b9cedb` |
| 2026-07-28 17:33:07 | `wallet-settings.org` | `5db917b1452b4ec6ae7dcaa63b251bc0` |
| 2026-07-28 17:33:18 | `promo-settings.org` | `1cbb8d8748b041b1b82c274ac7d899e9` |
| 2026-07-28 17:33:28 | `trusted-settings.org` | `c7195f8643184c73a48013ea5f28aaaa` |

Five of the six were created inside a **77-second window** — one interactive checkout session
in one registrar account. Beacon build, identical across all:
`static.cloudflareinsights.com/beacon.min.js/v4513226cdae34746b4dedf0b4dfa099e1781791509496`,
version `2024.11.0`; `POST /cdn-cgi/rum` returns 204.

### WAVE 1 — earlier, non-Cloudflare, origin exposed (all dead)

Served directly from an OVH dedicated server. **Correlation, not proven common origin with
Wave 2** — see the caveat below.

```
IP           54.39.106.37
Provider     OVH Hosting, Inc. (OrgId HO-2)
Reverse DNS  ns560354.ip-54-39-106.net   — OVH dedicated-server naming
Status       not responding since ~2026-08-06
Abuse ticket QPRHLVDXHC
```

| First seen | Domain | Registrar |
|---|---|---|
| 2026-07-23 | `transfer-tws.ink` | NameCheap |
| 2026-07-25 | `trusttws.net` | NameCheap |
| 2026-07-26 | `trusttws.com` | — |
| 2026-07-26 07:43:21 | `trustaws.com` | **Fewmoretaps OÜ / Trustname** |
| 2026-07-30 02:07:07 | `trustwailet.net` | **Fewmoretaps OÜ / Trustname** (NS: `ARES`/`ZEUS.TRUSTNAME.COM`) |

**What links the waves:** identical kit and page title; `trustaws.com` and `trustwailet.net`
share the registrar of Wave 2; overlapping timeline — `trustwailet.net` still resolved to the
OVH IP on 2026-08-03, five days after the Wave 2 burst was registered.

**What is NOT established:** that the Wave 2 domains resolve to that same origin. Host-header
and SNI overrides against `54.39.106.37` for three Wave 2 domains returned nothing, because the
host is down. Treat the OVH origin as a strong earlier-wave lead, not a confirmed origin for
the live cluster.

### WAVE 3 — registered 2026-08-10, second registrar

Three domains created within **47 seconds** through **NICENIC INTERNATIONAL GROUP** (Hong Kong),
on Cloudflare nameservers. All three already return Cloudflare's "Suspected Phishing"
interstitial. The operator rotates registrars mid-campaign.

```
14:14:00 UTC   gettrustcard.pro
14:14:17 UTC   trust-credit-card.pro
14:14:47 UTC   trust-credit.pro
```

### WAVE 0 — bought six months early, through a third registrar

*Added 2026-08-12.*

Three domains predate the entire campaign. The first victim was drained 2026-06-30; these
were registered in December 2025 and February 2026, through a registrar that appears
nowhere in Waves 1–3:

```
2025-12-26   trust-premium.pro    PDR Ltd. d/b/a PublicDomainRegistry.com
2026-02-08   trustcard.pro        PDR Ltd. d/b/a PublicDomainRegistry.com
2026-02-26   trust-card.pro       PDR Ltd. d/b/a PublicDomainRegistry.com
```

First certificates were issued 2026-06-07, 06-21 and 06-24 — the domains sat unused for
months, then were prepared as the campaign was assembled.

**Hunting implication.** Waves 1–3 are reactive: domains registered in 77- and 47-second
bursts as earlier ones were taken down. Wave 0 is inventory. If this actor holds further
unused domains at the same registrar, **they cannot be found from outside** — a domain that
has never resolved leaves no certificate transparency record, no passive DNS entry, no
scanner history. Only the registrar can enumerate them.

If you are tracking this cluster: registrar account enumeration is the only avenue that
surfaces the next wave before it goes live.

### Registrar totals

**Fewmoretaps OÜ d/b/a Trustname.com (Estonia)** — EIGHT domains across Waves 1 and 2.
Abuse case `#ABS-48857`, under RAA 3.18.1 review (supersedes `#ABS-48193`).
**NICENIC INTERNATIONAL GROUP (Hong Kong)** — three domains, Wave 3.
**PDR Ltd. d/b/a PublicDomainRegistry.com (India)** — three domains, Wave 0.
Abuse contact `abuse@publicdomainregistry.com`; report filed 2026-08-12 requesting
account-wide review.

Four registrars, three of which were used concurrently. Registrar rotation is part of the
operating pattern, not a one-off reaction.

### Advertising and campaign identifiers

| | |
|---|---|
| Meta campaign ID | `120253487369790536` |
| Campaign name | `KING_SUMMER_080 338267898899915` |
| Landing param | `setting=m100` (operator's own; useful pivot) |
| fbclid internals | `app_id = 6628568379`, markers `adid`, `fdid`, `aem`, `srtc` |

### Detection invariants — use these, not the decoy headings

The decoy shell **rotates**: the stylesheet defines ten variants (`v1`–`v10`) with named layouts
`cardScanner`, `cardQueue`, `cardDevice`, `cardTunnel`, `cardTimeline`, `cardTerminal`,
`cardMinimal`; seven were observed live ("Access Queue", "Route Scanner", "Verification
Timeline", "Gateway Card", "Trust Gateway", "Device Trust", terminal "secure-gateway"). Any rule
keyed to one phrase will miss most hits. These hold across every variant:

1. **Title mismatch** — `<title>Crypto Cards</title>` while the rendered DOM contains only a
   loading/verification shell. Strongest single signal.
2. **Cookie banner string, byte-identical** across every variant and domain:
   `"We use cookies to keep your session secure, remember access settings, and improve page delivery."`
   with buttons `Decline` / `Confirm`. Best pivot for hunting other deployments of this kit.
3. **Status triad** — the tokens `DONE` / `LIVE` / `NEXT` used as step states.
4. **Fake-diagnostics lexicon** — route, gateway, session, access, verification, delivery,
   latency, region, tunnel, TLS. No such checks are actually performed.
5. **Randomised route path** — `/_next/static/chunks/app/<random-lowercase>/page-<hash>.js`;
   observed: `/geccxceumlf` `/oduvakjlgx` `/pgzdtwaimmxxtvx` `/qteqayqcwgev` `/yfntgwwiuko`.
   `x-middleware-rewrite` rewrites **every** path, including non-existent ones, to that route.
6. **`x-robots-tag: noindex, nofollow, noarchive, nosnippet`** on every response — deliberate
   concealment from search engines.
7. **The asymmetry** — the document carries ~30 KB of funnel data in script tags against ~2.8 KB
   of rendered markup. The lure ships to every client; only its display is withheld. One command
   confirms it:
   `curl -s https://trusted-settings.com/ | grep -o 'approve the Trust connection[^"\\]*'`
8. **nginx origin** — static assets return native nginx ETags (hex mtime + hex length), e.g.
   `"6a698bb7-f45"` = 2026-07-29 05:12:23 UTC / 3909 bytes. Not a serverless platform.

**Earlier domain, still unidentified.** First sweep was 2026-06-30, three weeks before Wave 1
appeared. At least one earlier domain fed the same contract. Best pivots: the `KING_SUMMER`
naming scheme, `setting=m100`, and the cookie-banner string above.

## Funding wallets around collector #1

Rotating senders that top up the collector with small BNB amounts, each left near-empty.
None of them called the drainer contract directly.

```
0xb7255863c4d8f2fad033b0850bc3414cfeca2930   2026-08-10, 0.02539662 BNB
0xc7e74fca26c8c6213540afc514d1ec983a3b69ec   ~2 BNB in two payments
0x0de4439dc6c626b808823576bb73ffe28e70e740
0xe3d954c3e09227def1c1cb2a5c291851d629b932
0x4df7412c39b8163ab766020a86a8e1909783c614
0x243d0b88b4c0e8528c522a411ad203bdaa6d1764   first funding, 2026-07-20
```

## Scope

| | |
|---|---|
| Victims | **45 unique addresses** |
| Stablecoins stolen | **63,477.36 USDT** (17,369.26 + 46,108.10) |
| Other tokens | RACA, XPIN, UPY, TWT, MOONSTAR, DaddyDoge, BONFIRE, USDC, ETH, BTCB |
| Active period | 2026-06-30 → 2026-08-08 (39 days) |
| Largest single loss | 25,000.00 USDT (`0xfbcc89ee0bca19958d5be03a238dec96d3d9c741`) |
| Largest total loss | 27,220.47 USDT (`0x6dad95a415e575544a12b7dc52d425597928e3ce`, hit twice) |

Per-sweep data: [ALL-SWEEPS.md](ALL-SWEEPS.md) · [all-sweeps.csv](all-sweeps.csv)

## Ruled out

Checked and found **not** to be part of the cluster — listed so nobody rediscovers them:

- `0x3d90f66b534dd8482b181e24655a9e8265316be9` — no drainer functions; several funding
  wallets interact with it, appears to be an unrelated common service
- `0x3978d16ce654ebddce8be840a7a6fbcb2e1f4444`, `0xa8905e2c81297214952b9c10cec7eee41baa4444` —
  vanity-suffix tokens, identical 3823-byte bytecode, only `owner()`/`transferOwnership()`
- `0xc92e8bdf79f0507f65a392b0ab4667716bfe0110` — unrelated approval revoked by a victim
- `0x71c7656ec7ab88b098defb751b7401b5f6d8976f` — BscScan UI placeholder, not a counterparty
- `0x69460570c93f9de5e2edbc3052bf10125f0ca22d` — **Rango V2 (Rango Diamond)**, a legitimate
  cross-chain aggregator with 2,014,804 transactions. An earlier revision of this file
  wrongly flagged it as a second scam contract after seeing a victim send 1,000 USDT into
  it; that was normal swap/bridge activity, the 7 USDT was standard fee, and there is zero
  victim overlap with this cluster. **Corrected 2026-08-10.**
- `0x4be54063df659898625fc48c8161432cdd793e9b` — owner of the Rango contract, likewise
  unrelated

## Reported to

FBI IC3 `5036615c3cd54d23ba7bd1c089786d1d`, `7f90f61bf96c4dc585999781b2483aa7`, `37289b4b2028461289a70e32c4345128`, `f020e29e05a446618f2911d68b1b51a0` ·
HashDit Security · BNB Chain Support `4bcf92ea-7a94-4711-a02e-f882757f0fcc` ·
Trustname `#ABS-48193` · Chainabuse · SEAL 911
