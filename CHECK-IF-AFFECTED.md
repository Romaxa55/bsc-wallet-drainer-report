# Were you hit by this drainer? — a 2-minute check

No tools, no downloads, no connecting your wallet anywhere. Everything here is done by reading
a public block explorer.

---

## Step 1 — look up your wallet

Open:

```
https://bscscan.com/address/YOUR_WALLET_ADDRESS
```

Replace `YOUR_WALLET_ADDRESS` with your own address (the one you connected to the "virtual
card" site).

## Step 2 — look for these three fingerprints

You were hit by **this specific drainer** if you find any of the following.

**A. An approval to this spender**

Open the **Token Approvals** tab (or [BscScan Token Approval Checker](https://bscscan.com/tokenapprovalchecker))
and look for a spender of:

```
0x3a85da7f43c7b9946a450b55019f3e05e637ab11
```

That is the drainer contract. If it appears — with any allowance, including a huge number
like `100,000,000,000` — you signed the malicious approval.

**B. A transfer out to this collector**

In the **BEP-20 Token Txns** tab, look for an outgoing transfer to:

```
0x5655e7a5197cc1a1805387bc82dfffe901dfc552
```

That is where all 14 victims' funds went.

**C. The signature pattern**

The theft looks like this in your history:

| | |
|---|---|
| Method label | `Approve` — signed by you |
| then, within seconds | `Transfer` out of your wallet that **you did not send** |
| The theft tx `From` field | **not your address** — it is `0x7c51bc888362a93dab88cdbb2d6b6baed2d74f6d` |

That last point is the giveaway: **you never sent the theft transaction, and you paid no gas
for it.** The attacker sent it and paid the gas. Many victims look at their history, don't see
a transfer they signed, and wrongly conclude their seed phrase was stolen. It wasn't — the
approval alone was enough.

Timing note: on the documented case the sweep happened **6 seconds** after the approval. Do
not expect a gap of hours. If you signed and the money vanished almost instantly, that matches.

---

## Step 3 — if any of that matches

**1. Revoke your approvals — all of them, not just this one.**
[revoke.cash](https://revoke.cash) (select BNB Chain) or BscScan's Token Approval Checker.
An approval stays alive until revoked and will drain any new funds that arrive in the wallet.
Revoking costs a small amount of gas and requires a transaction — that is normal.

**2. Treat the wallet as burned.**
Even after revoking, move on: use a fresh wallet for anything new, and keep the old one empty.
It is now on lists that automated sweepers watch.

**3. If you ever typed your seed phrase into any website or app — anywhere, at any time —
rotate everything.** In that case the wallet is fully compromised, revoking does not help,
and every wallet derived from that seed must be replaced.

**4. File your own report.**
[ic3.gov](https://www.ic3.gov) (FBI — accepts complaints from victims worldwide), plus your
local police. Reference the three addresses above. Multiple independent complaints about the
same operation carry far more weight than one; the case is already on file with IC3, the
domain registrar, Chainabuse, SEAL 911 and HashDit.

---

## A warning that matters more than the money

The moment you say publicly that you were drained, people will contact you offering to
"recover" your funds. They will send you to a site that asks you to "synchronize", "validate"
or "activate" your wallet, and that site will ask for your **seed phrase, keystore file or
private key**.

**Every single one of them is a thief.** This is not a suspicion — one such site sent to a
victim of this exact drainer was inspected, and its HTML contained three input fields:
`phrase`, `Keystore JSON`, and `privateKey`, posting straight to a form-collection service.

There is no "wallet synchronization" procedure. It does not exist. No legitimate party — not
BNB Chain, not Trust Wallet, not any exchange, not law enforcement — will ever ask for your
seed phrase or charge you a fee to recover stolen funds.

Losing the wallet itself is far worse than losing what was already taken.

---

*Compiled by a victim of this drainer. Full technical evidence: [README.md](README.md) ·
Victim list: [VICTIMS.md](VICTIMS.md)*
