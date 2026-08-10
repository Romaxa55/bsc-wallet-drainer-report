# Full cluster analysis — drainer contract `0x3a85da7f43c7b9946a450b55019f3e05e637ab11`

Derived by decoding **every one of the 52 transaction receipts** on the contract.
Compiled 2026-08-10. Everything below is reproducible from public chain data.

All 52 inbound transactions were sent by a **single address** — the operator
`0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d`. 51 of them are `Pull` calls (selector `0xdc772e94`).
The contract exposes `setOperator(address,bool)` / `operators(address)`, but that capability
appears unused — suggesting rented or templated tooling rather than bespoke development.

---

## Two collectors

| Collector | USDT received | Victims | Status |
|---|---|---|---|
| `0x5655e7a5197cc1a1805387bc82dfffe901dfc552` | 17,369.26 | 16 | **still holding — never moved** (nonce 1) |
| `0x62f1a65cdf65ab42a6b520510d58f2f71750ea56` | 46,108.10 | 31 | **already cashed out** (nonce 20, 387.51 left) |
| **Total** | **63,477.36** | **45 unique** | 2 victims were hit twice |

The case was originally reported as 14 victims / 17,369 USDT. That was only the visible half —
collector #1. Actual scope: **45 victims, 63,477.36 USDT** in stablecoins alone.

## Timeline

| | |
|---|---|
| First sweep | **2026-06-30 09:37 UTC** |
| Last sweep | 2026-08-08 22:18 UTC |
| Duration | **39 days** |
| Phishing domain registered | 2026-07-28 |

The operation predates the registration of `trusted-settings.com` by **four weeks**, which
confirms at least one earlier domain fed the same contract. Identifying it would likely
expand the victim count further.

## Other tokens swept

Amounts are in token units; USD value not assessed (most are low-liquidity).

| Token | Amount | Contract |
|---|---|---|
| RACA (Radio Caca V2) | 784,469.27 | `0x12bb890508c125661e03b09ec06e404bc9289040` |
| XPIN | 138,511.55 | `0xd955c9ba56fb1ab30e34766e252a97ccce3d31a6` |
| UPY (Upstarty) | 30,484.33 | `0xb849c4d843769d42812fb600b1bcc8a6ba843466` |
| TWT (Trust Wallet) | 28.75 | `0x4b0f1812e5df2a09796481ff14017e6005508003` |
| MOONSTAR | 15.28 | `0xce5814efff15d53efd8025b9f2006d4d7d640b9b` |
| USDC | 10.87 | `0x8ac76a51cc950d9822d68b83fe1ad97b32cd580d` |
| DaddyDoge | 2.67 | `0x7cce94c0b2c8ae7661f02544e62178377fe8cf92` |
| BONFIRE | 1.60 | `0x5e90253fbae4dab78aa351f4e6fed08a64ab5590` |
| ETH | 0.09 | `0x2170ed0880ac9a755fd29b2688956bd959f933f8` |

## Notable individual losses

| Victim | Loss | Note |
|---|---|---|
| `0x6dad95a415e575544a12b7dc52d425597928e3ce` | **27,220.47 USDT** | hit twice — 15,380.98 (July, collector #2) then 11,839.49 (23 Jul, collector #1) |
| `0xfbcc89ee0bca19958d5be03a238dec96d3d9c741` | **25,000.00 USDT** | largest single sweep, 2026-07-04 |

The repeat victim is instructive: after the first theft the approval was evidently never
revoked, so the wallet was drained again weeks later.

## Cash-out route — identified

Collector #2 (`0x62f1a65cdf65ab42a6b520510d58f2f71750ea56`) moved the funds out in two transfers:

| Amount | When | Destination |
|---|---|---|
| 24,867.196677 USDT | ~2026-07-14 | `0x980447ddcef79a7499da4538da8fc59bacad6997` (contract, now empty) |
| 21,027.369380 USDT | ~2026-07-05 | `0xb685760ebd368a891f27ae547391f4e2a289895b` — **Bridgers (SWFT Blockchain)** cross-chain swap |

**The established laundering path is a cross-chain swap service, not a KYC exchange.**
This matters for two reasons: recovery through a swap service is far harder than through an
exchange with identity verification, and it is predictable — the 17,369 USDT still sitting on
collector #1 will most likely follow the same route. Pre-notifying that service is more useful
than reacting after the fact.

## Token labeling note

The stolen stablecoin contract `0x55d398326f99059ff775485246999027b3197955` self-identifies as
`symbol() = "USDT"`, `name() = "Tether USD"`, while explorers label it **Binance-Peg BSC-USD**.
Same asset, two names. It is the Binance-Peg wrapper, **not** Tether-issued — which is why
Tether declined the freeze request (ticket #459702) and why no blacklist function exists in it.

---

Per-sweep data: **[ALL-SWEEPS.md](ALL-SWEEPS.md)** · machine-readable: **[all-sweeps.csv](all-sweeps.csv)**