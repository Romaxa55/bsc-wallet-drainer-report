# INDICATORS — canonical list

**This is the single file to watch.** All new indicators (domains, collectors, cash-out hops,
campaign IDs) are added here. Last updated: **2026-08-10** (Route A endpoint identified as Symbiosis bridge).

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

## Phishing infrastructure

| | |
|---|---|
| Domain | `trusted-settings.com` — registered 2026-07-28, live as of 2026-08-09 |
| Registrar | Fewmoretaps OÜ d/b/a Trustname.com (Estonia) — abuse case `#ABS-48193` |
| DNS | Cloudflare |
| Meta campaign ID | `120253487369790536` |
| Campaign name | `KING_SUMMER_080 338267898899915` |
| Landing param | `setting=m100` (operator's own; useful pivot) |
| fbclid internals | `app_id = 6628568379`, markers `adid`, `fdid`, `aem`, `srtc` |

**Earlier domain — still unidentified.** First sweep was 2026-06-30, four weeks before
`trusted-settings.com` was registered. At least one earlier domain fed the same contract.
Best pivots: the `KING_SUMMER` naming scheme, `setting=m100`, and the landing template
(card-selection step → mobile wallet deeplink → decoy pages for scanners).

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

## Reported to

FBI IC3 `5036615c3cd54d23ba7bd1c089786d1d` and `7f90f61bf96c4dc585999781b2483aa7` ·
HashDit Security · BNB Chain Support `4bcf92ea-7a94-4711-a02e-f882757f0fcc` ·
Trustname `#ABS-48193` · Chainabuse · SEAL 911
