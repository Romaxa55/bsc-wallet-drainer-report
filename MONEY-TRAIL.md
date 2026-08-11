# Money trail: BNB Chain → Symbiosis bridge → Ethereum → exchange

Last verified: 2026-08-11. Every figure below was read from transaction receipts on public
RPC nodes, not from a block-explorer summary page. Reproduction commands are at the bottom.

Findings are split into **verified on-chain** and **inference** so a reader can weigh them
separately. Nothing in the verified section depends on a third-party label.

---

## 1. Executive summary

The operation runs two collector addresses on BNB Smart Chain. They behave completely
differently, and that difference is the single most important fact in this document.

| | Collector #1 | Collector #2 |
|---|---|---|
| Address | `0x5655e7a5197cc1a1805387bc82dfffe901dfc552` | `0x62f1a65cdf65ab42a6b520510d58f2f71750ea56` |
| Balance 2026-08-11 | **17,369.26 USDT — still sitting there** | 387.51 USDT — drained |
| Outgoing transactions | **1** | 20 |
| Status | **Untouched. Holds the reporter's stolen funds.** | Fully laundered off-chain |

Collector #2 shows the route the money takes. Collector #1 has not taken it yet.
That gap is the opportunity: **the destination is known before the funds arrive.**

Collector #2 used **two** different bridges, nine days apart. They ran through entirely
separate sets of intermediate wallets — and both ended at the same Ethereum address,
`0xdd3d72c53ff982ff59853da71158bf1538b3ceee`, which has received **286,467 USDT** from
this operation across the two channels.

The laundering route terminates in **real Tether USDT on Ethereum**, which — unlike
Binance-Peg BSC-USD on BNB Chain — Tether Limited is technically able to freeze.

---

## 2. The reporter's own theft

Verified from the receipt of `0xd2ae7c66d356598bbd01d9ac5eef912f64dac1cb7879285478a774d6ba31eb64`:

```
victim  0x7e7d14423827b7a3bc1feab64bd05393c08bc1b4
   ->   0x5655e7a5197cc1a1805387bc82dfffe901dfc552     Collector #1
        2,023.404141 USDT      2026-08-08 22:18:40 UTC
```

Collector #1 has made exactly **one** outgoing transaction in its entire history and still
holds 17,369.26 USDT. The stolen funds have **not** been moved, laundered, or sold.

---

## 3. The route Collector #2 already took

### 3.1 BNB Chain → Ethereum, via Symbiosis Finance

```
Collector #2  0x62f1a65cdf65ab42a6b520510d58f2f71750ea56
      |
      |   Symbiosis Finance cross-chain bridge
      |   2026-07-13 10:12:47 UTC  (block timestamp)
      |   in:  24,867.1966 USDT (BNB Chain)
      |   out: 24,838.6027 USDT (Ethereum)
      v
Ethereum tx 0x8535c46759fbd33fd89399328f8ac4a38db41e73e3be474b7e2d0295290f1c0a
```

Resolved through the Symbiosis explorer, which maps a bridge request to both legs:
`explorer.symbiosis.finance/transactions/56/{bscTxHash}`

### 3.2 First Ethereum hop — the staging address

```
0x18676b8832530e5ce5fca085a81d8490418c8b1c
    externally owned account, nonce 3, balance now 0
```

Its gas was supplied by three addresses that Etherscan has independently labelled as
phishing infrastructure — these labels were applied by Etherscan, not by this report:

```
Fake_Phishing4613733
Fake_Phishing4943569
Fake_Phishing4943736
```

This is corroboration from an unrelated party that the address sits inside a phishing
cluster, established before and independently of this case.

It then forwarded everything onward in three transfers:

| Amount (USDT) | Destination |
|---|---|
| 40,000.000000 | `0x7f8ac505bf807d2b380202a6dbf180975ce33008` |
| 40,000.000000 | `0x7f8ac505bf807d2b380202a6dbf180975ce33008` |
| 35,765.614645 | `0x7f8ac505bf807d2b380202a6dbf180975ce33008` |
| **115,765.614645** | **total** |

Note this exceeds the 24,838.60 that arrived from our bridge. The staging address pools
several separate criminal streams; ours is one of them.

### 3.3 Second hop — the consolidation address

```
0x7f8ac505bf807d2b380202a6dbf180975ce33008
    externally owned account, balance now 0
```

Its gas came from two sources, again per Etherscan's own labels:

- **`ChangeNOW: Deposit Funder 9`** — repeatedly, on 2026-07-13 between 17:27 and 18:33 UTC
- `Fake_Phishing4944448`

Every transfer it made went to a single destination:

| Amount (USDT) | Transaction |
|---|---|
| 35,765.614645 | `0xe0b00b358010b8ba70b4932f717b354bd066042ebdc2400700d0bccbb3aa39dd` |
| 40,000.000000 | `0x2fba9a85e7b5f53c03a9699155c8ab18ec9d90d1204aecc6732361d7d1453bea` |
| 40,000.000000 | `0xaccb71d97f219cf962e7579a16f7140200b7b06eecc171908cff26d212adafe1` |
| 51,745.554525 | `0xec0eb90c638545ce20f7eaa2aa7a3ad734eb13cc0d5f3fd0b743a797bcc1d1de` |
| 46,000.000000 | `0x4ef204b7bd2e1c381af7164aabc001a3599a5b240e056c8be5b41c7cb7d6cf72` |
| 20,000.000000 | `0xfac91840bacb90bd1032ac7a0c49ad46bab42ce9e428cfed13446471ccee5da7` |
| **233,511.169170** | **all to `0xdd3d72c53ff982ff59853da71158bf1538b3ceee`, one day** |

### 3.4 Terminus — a service wallet, not a person

```
0xdd3d72c53ff982ff59853da71158bf1538b3ceee
```

| Property | Value |
|---|---|
| Type | externally owned account (no contract code) |
| Transactions sent | 1,662,599 |
| Token transfers | 3,228,565 |
| Distinct tokens held | 760 |
| USDT balance now | 0.00 |

Volume at that scale is not an individual. This is the treasury of a service that processes
a very large number of small transactions across a very wide range of assets.

---

## 3A. The second channel — and it lands in the same place

Collector #2 did not use one bridge. It used two, nine days apart. The second one turned out
to be far easier to trace than the first, because the contract writes its destination into the
event log in plain ASCII.

### 3A.1 The swap that documents itself

```
BSC tx 0x2d39ad6eb94034fd489a1818de55e285f8f7271ee837b6faef4342b879c92e9e
block 108035225      2026-07-04 14:17:35 UTC
from 0x62f1a65cdf65ab42a6b520510d58f2f71750ea56   (Collector #2)
to   0xb685760ebd368a891f27ae547391f4e2a289895b   (Bridgers / SWFT Blockchain)
```

Log entry `[2]`, topic0 `0x45f377f845e1cc76ae2c08f990e15d58bcb732db46f92a4852b956580c3a162f`,
decodes to two ASCII strings:

```
"USDT(ERC20)|o3lkby|0.01|bridgers"
"0x7320BAA355bC112059FD28281aCc3ECf76CaDa78"
```

That is the destination asset (**USDT ERC-20 on Ethereum**), an order reference, the slippage
setting, the service name — and the receiving address. No support request needed: the swap
service records its own instructions on-chain.

- Sent: 21,027.369380 BSC-USD
- Expected out: 20,963.038439 USDT ERC-20

**Anyone monitoring `0xb685760e…` can read the destination of any swap through it directly
from the logs.** This applies to the 17,369 USDT still sitting on Collector #1: if it leaves
through Bridgers, its Ethereum destination is knowable within one block.

### 3A.2 Onward from `0x7320BAA355bC112059FD28281aCc3ECf76CaDa78`

Six USDT transfers, all on 2026-07-05 between 12:37 and 13:01 UTC, totalling **122,956.00 USDT**
(again more than arrived from our leg — the address pools several streams):

| Amount (USDT) | Destination |
|---|---|
| 22,956.00 | `0x2750d6fc57a29fe85521fc76831987425d9affcb` |
| 20,000.00 | `0x2750d6fc57a29fe85521fc76831987425d9affcb` |
| 5,000.00 | `0x2750d6fc57a29fe85521fc76831987425d9affcb` |
| 5,000.00 | `0x2750d6fc57a29fe85521fc76831987425d9affcb` |
| 35,000.00 | `0xe95f524eb72c4ae0ab1616a4b069f4e02bc8b280` |
| 35,000.00 | `0xdb01577d7805a0dfc4640a591ab339a14fda09fb` |

### 3A.3 Both channels converge

| From | Amount (USDT) | To |
|---|---|---|
| `0x2750d6fc…` (4 transfers) | 52,956.00 | **`0xdd3d72c53ff982ff59853da71158bf1538b3ceee`** |
| `0xe95f524e…` | 35,000.00 | `0x7f6eb7e213dd16cab6986cfb1539bf0a95c5e494` |
| `0xdb01577d…` | 35,000.00 | `0x7f6eb7e213dd16cab6986cfb1539bf0a95c5e494` |

**`0xdd3d72c5…` is the same terminus reached by the Symbiosis channel nine days later.**

Two different bridges, two different dates, two disjoint sets of intermediate addresses, one
endpoint. Documented into that address from this operation so far:

```
channel 1  (Symbiosis, 2026-07-13)   233,511.17 USDT
channel 2  (Bridgers,  2026-07-05)    52,956.00 USDT
                                     ─────────────────
                                     286,467.17 USDT
```

### 3A.4 A structural fingerprint: vanity-prefix gas funders

Throughout both channels the intermediate wallets are gas-funded by addresses sharing a
four-character prefix, in pairs:

| Prefix | Funder addresses observed |
|---|---|
| `0x2750` | `0x27500cf5d5…`, `0x2750eaE60b…`, `0x275079A7ac…` — and the recipient `0x2750d6fc…` |
| `0x7320` | `0x7320acA66C03…` — and the recipient `0x7320BAA355…` |
| `0xd8cf` | `0xD8cF608a7839D5…`, `0xd8cF8a5df848F2…` |
| `0x458C` / `0x2887` | `0x458C2A06ba8d…`, `0x288726150729…` |

Recipients share prefixes with their own funders. Vanity addresses at this volume are
generated deliberately, which points at automated infrastructure rather than hand-run wallets.
Each wallet receives gas, holds funds for minutes, sweeps onward, and is never reused.

This is offered as a pattern observation, not an attribution.

---

## 3B. Counter-forensic technique: counterfeit USDT and lookalike addresses

Anyone tracing this cluster will hit a deliberate trap. It is worth describing precisely,
because it survived a first pass of our own analysis.

### 3B.1 Two fake tokens named USDT

Address `0xd8cf9c601b40c48b4678e9f2ccaf9a36c639bf23` carries transfers of **three** different
ERC-20 contracts, all reporting `symbol() = "USDT"`:

| Contract | Status | Volume seen |
|---|---|---|
| `0xdac17f958d2ee523a2206206994597c13d831ec7` | **genuine Tether USD** | 220,000.00 |
| `0x74a33e7876b10c3bc609a568bf323349aa79eb8d` | counterfeit | 230,000.00 |
| `0x1aa3fc6e47927c4a45a0b486874c7d1880f8b061` | counterfeit | 70,000.00 |

An analysis that filters on the token *symbol* rather than the token *contract address* will
silently mix real and fake movements. Our own first pass reported 410,000 USDT leaving this
address; filtered correctly against `0xdac17f95…`, the real figure is **110,000.00**.

### 3B.2 Lookalike recipients

The counterfeit transfers are addressed to wallets generated to be visually indistinguishable
from the real recipients — matching leading characters *and* trailing characters:

| Real recipient (genuine USDT) | Decoy (counterfeit USDT) |
|---|---|
| `0xc02297fB0220cdeb0f78a2d04405AE73C38Db44F` | `0xC0251c19fFf76868d58C2E63550F4FfA3e8Db44F` |
| `0x7f6eb7e213dd16cab6986cfb1539bf0a95c5e494` | `0x7f6e0885ed5d6b9a4907cd28543df6708b1fe494` |
| `0xe95f524eb72c4ae0ab1616a4b069f4e02bc8b280` | `0xe95fb44e4e91331cf5d2b9b68062cd1677eeb280` |
| `0x73ab8643a60c44a3a2769ec2cd51d40a82a2f3ac` | `0x73ab2ea6d96717b02b8e71b70b0455bb6fb9f3ac` |

Zero-value transfers are also used, which any address can emit on Tether's contract because a
`transferFrom` of 0 passes the allowance check trivially. The effect is a transaction history
salted with plausible-looking movements that lead nowhere.

### 3B.3 How to avoid the trap

- Filter every transfer on the **token contract address**, never on `symbol()`.
- Treat any transfer of value `0` as noise, not as a hop.
- Sanity-check each hop against the account nonce. The decoy
  `0xC0251c19…Db44F` appears to have "received 300,000 USDT" while having a nonce of **0** —
  an address that has never sent a transaction cannot have forwarded anything. A nonce of 0
  combined with a zero balance and a large apparent inflow means the inflow was counterfeit.

---

## 3C. Where the trail ends, and why that is not a failure

Following the genuine USDT downstream from the second channel reaches, within a few hops,
addresses of a completely different character:

| Address | Nature | Holdings when checked (2026-08-11) |
|---|---|---|
| `0x83C41363cBee0081dab75cB841FA24f3dB46627e` | EOA, 186,005 outgoing txs, 207 tokens | service hot wallet |
| `0x7491f26A0FCb459111b3a1db2fbFC4035D096933` | EOA, 998 outgoing txs, 129 tokens | 6,986,751.61 USDT |
| `0xb8e6D31e7B212b2b7250EE9c26C56cEBBFBe6B23` | `ERC1967Proxy`, implementation `Vault`, deployed 2022-07-25, 664 tokens | 21,693,978.74 USDT |

**These are custodial and exchange infrastructure serving large numbers of unrelated users,
and this report makes no allegation against them.** They are listed only to mark where
on-chain attribution stops being meaningful.

Once funds enter a pooled service wallet they are commingled, and no amount of chain analysis
can separate one depositor's dollars from another's — this is a property of omnibus custody,
not a limitation of the tooling. Every intermediate address in sections 3 and 3A already
pools several unrelated criminal streams; by the time the path reaches a vault holding
21 million USDT, the contribution traceable to this case is a rounding error.

Beyond this point the question is answerable only from service-side records: order IDs,
counterparty addresses, timestamps, client IP addresses and verification data. That is a
matter for legal process, not for a block explorer.

**Restated plainly, because it matters for what should be asked of whom:** the funds
documented in sections 3 and 3A belong to victims robbed on 2026-07-04 and 2026-07-13. The
reporter's own 2,023.404141 USDT was taken on 2026-08-08 and has never moved. It remains on
Collector #1 with the rest of the 17,369.26 USDT. Those funds do not need to be found — they
need to be met when they move.

---

## 3D. Timing map — how long the funds sit at each stage

Both channels have been reconstructed with block-level timestamps. This answers the only
question that matters operationally: **how much time is there between the funds leaving BNB
Chain and the point where they stop being recoverable?**

Every timestamp below is a block timestamp read from the chain, in UTC.

### Channel 2 — Bridgers / SWFT

| When (UTC) | Elapsed | Event |
|---|---|---|
| 2026-07-04 14:17:35 | — | Collector #2 sends 21,027.369380 BSC-USD into Bridgers |
| 2026-07-05 12:37:35 | **+22:20:00** | first onward move on Ethereum: 35,000 USDT |
| 2026-07-05 12:40:59 | +22:23:24 | 5,000 USDT |
| 2026-07-05 12:48:59 | +22:31:24 | 5,000 USDT |
| 2026-07-05 12:54:11 | +22:36:36 | 35,000 USDT |
| 2026-07-05 12:54:59 | +22:37:24 | 20,000 USDT |
| 2026-07-05 13:01:35 | +22:44:00 | 22,956 USDT |

The entire dispersal took **24 minutes**, but it began **more than 22 hours** after the funds
left BNB Chain.

### Channel 1 — Symbiosis

| When (UTC) | Elapsed | Event |
|---|---|---|
| 2026-07-13 10:12:47 | — | bridge output lands on Ethereum |
| 2026-07-13 17:28:35 | **+7:15:48** | first onward move: 20,000 USDT |
| 2026-07-13 17:38:47 | +7:26:00 | 46,000 USDT |
| 2026-07-13 17:44:47 | +7:32:00 | 51,745.55 USDT |
| 2026-07-13 18:20:35 | +8:07:48 | 40,000 USDT |
| 2026-07-13 18:28:11 | +8:15:24 | 40,000 USDT |
| 2026-07-13 18:33:35 | +8:20:48 | 35,765.61 USDT |

Dispersal took **65 minutes**, beginning **7 hours 16 minutes** after the bridge paid out.

> **Correction to an earlier figure.** This report previously gave the Symbiosis bridge time as
> 2026-07-13 13:11:48 UTC, taken from the Symbiosis explorer interface. The block timestamp of
> the Ethereum leg is **10:12:47 UTC**; the explorer was displaying local time (UTC+3). The
> block timestamp is authoritative.

### What this means in practice

| Stage | Observed window |
|---|---|
| Detection of movement off Collector #1 | ≤ 3 minutes (monitor polls every 180 s) |
| Funds in transit / dormant after bridging | **7 to 22 hours** |
| Dispersal once it starts | 24 to 65 minutes |

**The usable window is hours, not minutes.** In both independent observations the funds sat
still on the Ethereum side for the better part of a working day before anything happened. A
freeze request that is already on file and simply needs to be triggered has ample time to be
acted upon; a request written from scratch after the alert does not.

This is the entire argument for pre-arming: not speed of reaction, but elimination of the
cold start.

---

## 3E. The operation runs on five chains, and money is sitting on Ethereum right now

Found 2026-08-11. Everything above was written on the assumption that this is a BNB Chain
operation. It is not.

### 3E.1 The same contract, five chains, one owner

`eth_getCode` against the drainer address on public nodes for each chain:

| Chain | Bytecode | `owner()` |
|---|---|---|
| Ethereum | 9,242 bytes | `0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d` |
| BNB Chain | 9,242 bytes | `0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d` |
| Arbitrum | 9,242 bytes | `0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d` |
| Optimism | 9,242 bytes | `0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d` |
| Base | 9,242 bytes | `0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d` |

Contract address in all five cases: `0x3a85da7f43c7b9946a450b55019f3e05e637ab11`.
Polygon could not be checked — public RPCs did not respond. Avalanche: no code.

Operator activity by nonce: **Ethereum 133**, BNB Chain 94, Arbitrum 16, Base 12, Optimism 7.
The operator is *more* active on Ethereum than on the chain where this investigation started.

### 3E.2 Sixteen victims on Ethereum, funds never moved

Collector #1 — the same address that holds the BNB Chain funds — on Ethereum:

```
0x5655e7a5197cc1a1805387bc82dfffe901dfc552
    4,191.58 USDT   (0xdac17f958d2ee523a2206206994597c13d831ec7 — Tether-issued)
    1,920.03 USDC
    nonce 0 — has never originated a transaction on Ethereum
```

Because nothing has ever left, the sum of inbound transfers equals the balance exactly. That
arithmetic check confirms the list below is complete rather than a sample.

| Victim address | Amount | Token | When (UTC) |
|---|---|---|---|
| `0xEe09AEC213e31095e6F5fCdEA52615659C1137dD` | 21.0000 | USDT | 2026-08-10 15:03:11 |
| `0x804AC48EE7aaBAAEF6b1a5598F4BC051d10A82cD` | 383.8947 | USDC | 2026-08-09 11:06:47 |
| `0xD5b9152f2b18F10c4541AdF2CDbd85C547987FBD` | 30.7867 | USDC | 2026-08-09 09:43:11 |
| `0x437Cf8D43c94772b386709Dfc91128531885cbd1` | 1,533.9441 | USDT | 2026-08-08 20:18:35 |
| `0x9A463e863abca8D501F43C92E037841b268c8930` | 19.0000 | USDT | 2026-08-03 06:01:59 |
| `0xc814F10Fc2F70748Df5688859339A002fB438d86` | 25.0000 | USDT | 2026-08-01 12:58:47 |
| `0x4Dc742bE60e95EdcDB64Cd87346FFD8Be6E330fe` | 194.7776 | USDT | 2026-07-31 18:59:59 |
| `0x4B05f7887C12276a4aFBf6cC74D10E7478eabE8A` | 2,030.0601 | USDT | 2026-07-30 17:47:47 |
| `0x493cF917bD1042e8a154b1259990Dd9Cdf213a4a` | 30.3592 | USDT | 2026-07-29 08:06:11 |
| `0xD8b175E747A31f51b2E1bEF54e71ADD9Fe483b5f` | 11.3400 | USDC | 2026-07-29 03:33:47 |
| `0x37337B4F3e29Da147026ca9d4223edFBaF4A79F1` | 169.8416 | USDT | 2026-07-27 13:55:11 |
| `0xa1D9Beae574867Db969D81DD42989937642a9B91` | 50.0000 | USDT | 2026-07-27 04:52:35 |
| `0x72c2ec5764343253092cD01723Bf962814f81630` | 42.8000 | USDT | 2026-07-26 10:19:11 |
| `0x777c415Ff100576046d324113652842211EA3819` | 19.0000 | USDT | 2026-07-25 21:11:47 |
| `0x7b98f6BD34d799F26FaD8f579c4aaA7c30Ad5E3E` | 1,494.0065 | USDC | 2026-07-25 20:33:47 |
| `0x4B05f7887C12276a4aFBf6cC74D10E7478eabE8A` | 30.8000 | USDT | 2026-07-22 20:10:47 |
| `0x7cD5187A409E6fA3E9CCd340E68bD6Ea45b74F9d` | 25.0000 | USDT | 2026-07-21 20:04:23 |

`0x4B05f7887C12276a4aFBf6cC74D10E7478eabE8A` appears twice — robbed on 22 July and again on
30 July, the allowance never having been revoked after the first theft. The same pattern as
victim `0x6dad95a4…` on BNB Chain.

### 3E.3 Freezable balances across chains

Unlike Binance-Peg BSC-USD, these are issuer-controlled tokens:

| Chain | Address | Amount | Issuer |
|---|---|---|---|
| Ethereum | `0x5655e7a5…` collector #1 | 4,191.58 USDT | Tether |
| Ethereum | `0x5655e7a5…` collector #1 | 1,920.03 USDC | Circle |
| Ethereum | `0x62f1a65c…` collector #2 | 429.70 USDT | Tether |
| Optimism | `0x62f1a65c…` collector #2 | 168.57 USDT | Tether |
| Arbitrum | `0x5655e7a5…` collector #1 | 34.00 USDC | Circle |
| Arbitrum | `0x62f1a65c…` collector #2 | 37.28 USDC | Circle |
| | **total** | **≈ 6,781 USD** | |

Other tokens held at these addresses (WMC, DIA, OVR, ENA, WLFI, ELON, GTAVI and two
domain-name spam tokens) are unsolicited airdrops of no value and are excluded.

### 3E.4 Why these funds are still there

Collector #2 on BNB Chain shows the cash-out rule: it accumulated to roughly 21,000–25,000 USDT
and then emptied itself in a single bridging transaction — twice, on 2026-07-04 and 2026-07-13.
The Ethereum balance has not reached that threshold. That appears to be the only reason it
remains.

Two data points do not establish a rule, and this is offered as an observation, not a law.

---

## 4. Verified vs inferred

**Verified on-chain** — reproducible by anyone from public nodes:

- every amount, hash, address and direction in section 3
- the two collectors' balances and nonces
- that the terminus is an EOA with 1.66M transactions and 760 token types

**Inference, stated as inference:**

- `0x7f8ac505…` is most likely a **ChangeNOW deposit address**. Basis: instant exchanges
  fund their per-order deposit addresses with gas so the address can sweep itself, and
  Etherscan labels the funder `ChangeNOW: Deposit Funder 9`. The observed pattern —
  gas in, sweep out, repeat — matches that mechanism exactly.
- `0xdd3d72c53f…` is most likely the corresponding **ChangeNOW collection wallet**, on the
  scale-of-activity evidence above.

Neither address currently carries a public name tag on Etherscan, Blockscout or Ethplorer.
The attribution therefore needs confirmation from ChangeNOW itself — which only ChangeNOW
or a law-enforcement request can supply. It is offered here as a lead, not a conclusion.

---

## 5. Why this matters, precisely

**Do not overstate this.** The 233,511 USDT that moved on 2026-07-13 is *earlier victims'*
money. The reporter's own theft happened on 2026-08-08, almost a month later, and those
funds sit untouched in Collector #1. This trail does not directly recover them.

What it does establish:

1. **Scale.** A quarter of a million dollars through one consolidation address in one day,
   pooling multiple phishing streams — far beyond the 63,477 USDT documented for this
   single drainer contract.

2. **A subpoena target.** An instant exchange holds order records: source address, amount,
   asset swapped to, payout address, timestamps, client IP, and any verification performed.
   That is a documentary path to identity where the blockchain alone gives none.

3. **Freezability.** On BNB Chain the funds are Binance-Peg BSC-USD
   (`0x55d398326f99059ff775485246999027b3197955`), which has **no freeze function** —
   confirmed in writing by BNB Chain support. Once bridged, they become Tether's own
   ERC-20 USDT (`0xdac17f958d2ee523a2206206994597c13d831ec7`), which Tether Limited
   **can** blacklist. Crossing the bridge is the moment the funds become recoverable.

4. **Advance warning.** Collector #1 has not moved. Its route is now documented. The
   addresses, the bridge, and the exchange are known *before* the funds travel.

---

## 6. Reproduction

```bash
# the reporter's theft — read the transfer straight from the receipt
cast receipt 0xd2ae7c66d356598bbd01d9ac5eef912f64dac1cb7879285478a774d6ba31eb64 \
  --rpc-url https://bsc-dataseed.binance.org

# Collector #1 balance and nonce (BNB Chain)
cast call 0x55d398326f99059ff775485246999027b3197955 \
  "balanceOf(address)(uint256)" 0x5655e7a5197cc1a1805387bc82dfffe901dfc552 \
  --rpc-url https://bsc-dataseed.binance.org
cast nonce 0x5655e7a5197cc1a1805387bc82dfffe901dfc552 \
  --rpc-url https://bsc-dataseed.binance.org

# the six Ethereum transfers into the terminus
for tx in 0xe0b00b358010b8ba70b4932f717b354bd066042ebdc2400700d0bccbb3aa39dd \
          0x2fba9a85e7b5f53c03a9699155c8ab18ec9d90d1204aecc6732361d7d1453bea \
          0xaccb71d97f219cf962e7579a16f7140200b7b06eecc171908cff26d212adafe1 \
          0xec0eb90c638545ce20f7eaa2aa7a3ad734eb13cc0d5f3fd0b743a797bcc1d1de \
          0x4ef204b7bd2e1c381af7164aabc001a3599a5b240e056c8be5b41c7cb7d6cf72 \
          0xfac91840bacb90bd1032ac7a0c49ad46bab42ce9e428cfed13446471ccee5da7; do
  cast receipt $tx --rpc-url https://eth.llamarpc.com
done

# the bridge leg
open https://explorer.symbiosis.finance/transactions/56/<bscTxHash>
```

---

## 7. Address reference

| Role | Chain | Address |
|---|---|---|
| Victim (reporter) | BSC | `0x7e7d14423827b7a3bc1feab64bd05393c08bc1b4` |
| Drainer contract | BSC | `0x3a85da7f43c7b9946a450b55019f3e05e637ab11` |
| Operator / deployer | BSC | `0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d` |
| **Collector #1 — funds still here** | BSC | `0x5655e7a5197cc1a1805387bc82dfffe901dfc552` |
| Collector #2 — laundered | BSC | `0x62f1a65cdf65ab42a6b520510d58f2f71750ea56` |
| Symbiosis bridge | BSC | `0x5aa5f7f84ed0e5db0a4a85c3947ea16b53352fd4` |
| Bridgers / SWFT | BSC | `0xb685760ebd368a891f27ae547391f4e2a289895b` |
| Staging address — channel 1 | ETH | `0x18676b8832530e5ce5fca085a81d8490418c8b1c` |
| Consolidation — channel 1 | ETH | `0x7f8ac505bf807d2b380202a6dbf180975ce33008` |
| Bridge landing — channel 2 | ETH | `0x7320baa355bc112059fd28281acc3ecf76cada78` |
| Consolidation — channel 2 | ETH | `0x2750d6fc57a29fe85521fc76831987425d9affcb` |
| Consolidation — channel 2 | ETH | `0xe95f524eb72c4ae0ab1616a4b069f4e02bc8b280` |
| Consolidation — channel 2 | ETH | `0xdb01577d7805a0dfc4640a591ab339a14fda09fb` |
| Secondary terminus — channel 2 | ETH | `0x7f6eb7e213dd16cab6986cfb1539bf0a95c5e494` |
| **Shared terminus — both channels** | ETH | **`0xdd3d72c53ff982ff59853da71158bf1538b3ceee`** |
| BSC-USD — cannot be frozen | BSC | `0x55d398326f99059ff775485246999027b3197955` |
| Tether USDT — can be frozen | ETH | `0xdac17f958d2ee523a2206206994597c13d831ec7` |
