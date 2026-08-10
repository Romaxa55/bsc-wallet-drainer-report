# INDICATORS — canonical list

**This is the single file to watch.** All new indicators (domains, collectors, cash-out hops,
campaign IDs) are added here. Last updated: **2026-08-10** — **operation confirmed still live**; victims currently in the pipeline listed below.

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

## 🔴 OPERATION IS STILL LIVE — victims currently in the pipeline

Discovered 2026-08-10 by decoding the operator's own transaction history
(`0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d`, 95 txs: 51 `Pull` + 43 `Transfer`).

### The operator funds his victims' gas

43 of his transactions are small BNB transfers **to the victims themselves**. A victim with
no BNB cannot sign the approval, so the operator pays for it. This is a distinctive,
searchable pattern: *small BNB inbound from the operator, followed days later by an approval
and a sweep*.

It also means victims can be identified **before** they are drained.

### Addresses funded but not yet swept — as of 2026-08-10

| Address | Funded | State |
|---|---|---|
| `0xf08ddde735643ccdf922dd7f8a47b350fe56c743` | 8 days ago | **⚠ ACTIVE APPROVAL of 100,000,000,000 USDT to the drainer.** Wallet nearly empty (0.53 USDT) — the sweep will trigger the moment funds arrive |
| `0x32c24318be3863467fec944ca0058dfe97563bb9` | 3 days ago | nonce 0, holds 30.05 USDT — funded, has not signed yet |
| `0xb9c6a49e782d7a742263333a4164d577e75ba107` | **44 hours ago** | nonce 0 — most recent target |
| `0xecc6ef652667496aec74d0646ad22609b018db3d` | 4 days ago | nonce 5, no approval currently |

`0xf08ddde7…` is the urgent one: the approval is live and unlimited. Any deposit into that
wallet is lost automatically.

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

FBI IC3 `5036615c3cd54d23ba7bd1c089786d1d`, `7f90f61bf96c4dc585999781b2483aa7`, `37289b4b2028461289a70e32c4345128` ·
HashDit Security · BNB Chain Support `4bcf92ea-7a94-4711-a02e-f882757f0fcc` ·
Trustname `#ABS-48193` · Chainabuse · SEAL 911
