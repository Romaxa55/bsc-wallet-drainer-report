# Active wallet drainer on BNB Smart Chain — incident report

**Status: funds still unmoved.** As of 2026-08-10 the attacker has not laundered anything — all
stolen funds sit on a single collector wallet, untouched.

| | |
|---|---|
| **Victims** | 14 |
| **Total stolen** | **17,369.26 USD** (Binance-Peg BSC-USD) |
| **Period** | 2026-07-22 → 2026-08-08 |
| **Funds moved out** | **None** (collector nonce = 1) |
| **Operation status** | Phishing domain still live as of 2026-08-09 |

Everything below is verifiable by anyone on [bscscan.com](https://bscscan.com) — no trust required.

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
3. The site asks the victim to connect a wallet and press an "issue card" button.
   What is actually signed is an **unlimited ERC-20 approval** to the drainer contract.
4. Later — hours or days — the operator calls function `0xdc772e94` (labelled **"Pull"**)
   on the contract, which executes `transferFrom` against the victim and sweeps the balance
   into the collector wallet.

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

Full table with amounts, timestamps and transaction hashes: **[VICTIMS.md](VICTIMS.md)**

Summary: 14 victims, 17,369.26 USD. Largest single loss **11,839.49 USD**; three losses above
2,000 USD; the remainder are small wallets drained of everything they held (down to 11 USD).

The sum of all 14 incoming transfers matches the collector's current balance **exactly** —
confirming nothing has been withdrawn.

---

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
