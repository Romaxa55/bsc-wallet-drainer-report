# Live phishing infrastructure — five sibling domains, still serving

> **Status as of 2026-08-10 11:35 UTC: all five domains return HTTP 200 and serve the
> live drainer funnel.** This is not a post-mortem. The operation is running right now.

The domain that took my funds, `trusted-settings.com`, is not a single site. It is one of
**five domains registered inside a 77-second window from a single registrar account**, all
serving byte-for-byte equivalent builds of the same application.

## The registration burst

Every domain below was created on **2026-07-28** through **Fewmoretaps OU d/b/a Trustname.com**
(Estonia), abuse contact `abuse@trustname.com`, with nameservers delegated to Cloudflare.

| Created (UTC) | Domain | Registry | Live check 2026-08-10 |
|---|---|---|---|
| 17:32:11 | `promo-settings.com` | .com | HTTP 200, 33 490 b |
| 17:32:31 | **`trusted-settings.com`** | .com | HTTP 200, 33 505 b |
| 17:33:07 | `wallet-settings.org` | .org | HTTP 200, 33 482 b |
| 17:33:18 | `promo-settings.org` | .org | HTTP 200, 33 339 b |
| 17:33:28 | `trusted-settings.org` | .org | HTTP 200, 32 836 b |

**Elapsed time from first to last: 77 seconds.** Registrations spaced 11–36 seconds apart
are a single interactive checkout session in one registrar account. This is the strongest
non-blockchain pivot in the case: **one account at one registrar holds the billing details,
registration IP, and contact email behind all five domains.**

`trusted-settings.com` was used in the paid Facebook Reels campaign that drained
2 023.404141 USDT from my wallet on 2026-08-08. The other four are its siblings.

## Same codebase, five deployments

All five serve a Next.js application with identical chunk structure and near-identical
bundle sizes (508 967 – 511 488 bytes of combined HTML + JS). Page routes are randomised
per deployment — `/geccxceumlf`, `/oduvakjlgx`, `/pgzdtwaimmxxtvx`, `/qteqayqcwgev`,
`/yfntgwwiuko` — a standard anti-fingerprinting measure, but the underlying build is one
product deployed five times.

### Cloudflare Web Analytics enabled on all five zones

The beacon is not in the origin HTML — Cloudflare injects it at the edge, and only for real
browser requests, which is why it is absent from non-browser fetches. Captured from a live
browser session on 2026-08-10 13:20–13:22 UTC:

| Domain | RUM site token |
|---|---|
| `trusted-settings.com` | `096e19198fbc489e9f32df6b14b9cedb` |
| `trusted-settings.org` | `c7195f8643184c73a48013ea5f28aaaa` |
| `promo-settings.com` | `b1af2c3f4e9b4448a818b78d424ae24a` |
| `promo-settings.org` | `1cbb8d8748b041b1b82c274ac7d899e9` |
| `wallet-settings.org` | `5db917b1452b4ec6ae7dcaa63b251bc0` |

All five report `version 2024.11.0` and load the identical beacon build
`beacon.min.js/v4513226cdae34746b4dedf0b4dfa099e1781791509496`. RUM is confirmed active by
`POST /cdn-cgi/rum` returning 204.

These are per-zone site tags, not per-account, so they do not by themselves prove common
ownership. What they establish is that Web Analytics was **deliberately switched on for all
five zones** — a manual dashboard action, performed five times on five domains registered 77
seconds apart. For Cloudflare, each token resolves directly to the owning account.

### Live chat: Smartsupp

The served markup carries the Smartsupp `noscript` fallback ("Powered by Smartsupp"), so the
widget is genuinely deployed. The widget key is fetched dynamically and does not initialise
while the page is cloaked, so it has not been recovered and is not asserted here. Smartsupp
s.r.o. is a Czech company, placing that account in EU jurisdiction with billing data attached.

### Correction: Google Tag Manager is not a lead

An earlier version of this document listed Google Tag Manager as an account-based pivot. That
was wrong. The tag loads as `googletagmanager.com/gtag/js?id=test` — the container ID is the
literal placeholder string `test`, left in the template. It maps to no Google account. The
error came from noting the request without reading its `id` parameter.

## Cloaking confirmed by differential response

The same URL returns different content depending on the requesting client:

| Client | Response |
|---|---|
| Desktop browser / automation | `<title>Loading…</title>` — decoy shell, no offer |
| Mobile Safari user agent (iOS) | `<title>Crypto Cards</title>` — full drainer funnel |

This is why the campaign survived Meta's ad review: reviewers and scanners see a blank
loading page. The funnel only renders for the mobile traffic the ads deliver.

## The lure, verbatim from the bundle

The brand impersonated is **"TrustCard" — "A crypto card from Trust"**, trading on Trust
Wallet's name. Products offered: *Mastercard Pro* and *Visa E-commerce*, each priced at
**1.00 USDT** — a deliberately trivial amount that makes the transaction feel safe.

The approval is disguised as a connection step:

> **"Connect your wallet"** — *"Open the app and **approve the Trust connection**. Payments
> will be charged directly from your wallet."*

The victim believes they are authorising a $1 card issuance fee. They are signing an
unlimited ERC-20 allowance.

The funnel then escalates:

> `step3Text`: "Confirm the transaction in your wallet"
> `step4Text`: "Please confirm the transaction in your wallet **again**"
> `step6Text`: "Card issuance costs $1. **Connect another wallet**"

Step 4 requests a *second* signature. Step 6 fires when the wallet is empty — the victim is
asked to connect a different wallet, so the operator harvests approvals across every wallet
a victim controls.

Targeting is visible in the copy: the cards are pitched for paying *"Google services, Open AI,
ChatGPT, Microsoft, Meta, Shopify, AWS, Telegram, TikTok"* — aimed squarely at users cut off
from conventional payment rails who need a workaround card.

## Already blocklisted, still online

All five domains have been in the **PhishDestroy** blocklist since **2026-08-01** — seven days
*before* the campaign drained my wallet, and ten days before this writing. urlscan.io scans
from 2026-08-01 19:03–19:08 UTC carry the tags `phishdestroy` and `@phish_report`.

Blocklisting did not stop them. The domains resolve, Cloudflare proxies them, the registrar
has not suspended them, and Meta accepted payment to advertise one of them a week after it
was publicly flagged as phishing.

## What is being requested

**Cloudflare** — all five domains are behind Cloudflare DNS and proxy, under what appears to
be one authenticated account (RUM enabled). Suspension of the account, and preservation of
account registration data, payment method, and access logs.

**Fewmoretaps OU / Trustname.com** — seven domains tied to this campaign, five of them from one
account in 77 seconds. Suspension, and preservation of the account's registration data, payment
instrument, and IP logs. Existing abuse ticket: `#ABS-48193`, extended twice on 2026-08-10.

**OVH Hosting** — dedicated server `ns560354` / `54.39.106.37` hosted the earlier wave.
Preservation of customer identity, payment instrument, billing history, control-panel login IPs,
and the IP assignment history from 2026-07-01 onward. Reported 2026-08-10.

**Meta Platforms** — paid campaign `120253487369790536`, object `338267898899915`, campaign
name `KING_SUMMER_080`, served 2026-08-08 to Tanzania. Advertiser identity, payment
instrument, and login IPs. Facebook advertising cannot be purchased with cryptocurrency;
a bank card or payment account is required, and it is attached to a real person or company.

**Smartsupp** (Czech Republic, EU jurisdiction) — the live-chat widget is tied to a
registered account with billing data. Operators answer that chat personally. The widget key
has not been recovered; see the correction above regarding Google Tag Manager, which is a
placeholder and not a lead.

## An earlier wave ran without a CDN, and exposed the origin

Searching public urlscan.io records for the page title invariant — `page.title:"Crypto Cards"`
— surfaced an earlier wave of the same kit that was **not** behind Cloudflare.

```
IP           54.39.106.37
Provider     OVH Hosting, Inc. (OrgId HO-2)
Reverse DNS  ns560354.ip-54-39-106.net   — OVH dedicated-server naming
Status       not responding as of 2026-08-10 13:35 UTC
```

Five domains were observed serving from that address, all under the same page title:

| First seen | Domain | Title | Registrar |
|---|---|---|---|
| 2026-07-23 | `transfer-tws.ink` | Crypto Cards | NameCheap |
| 2026-07-25 | `trusttws.net` | Crypto Cards | NameCheap |
| 2026-07-26 | `trusttws.com` | Crypto Cards | — |
| 2026-07-28 | `trustaws.com` | Crypto Cards | **Fewmoretaps OU / Trustname** |
| 2026-08-03 | `trustwailet.net` | 403 Forbidden | **Fewmoretaps OU / Trustname** |

The naming impersonates Trust Wallet throughout — `tws`, `trustaws`, and `trustwailet`, the
last a homoglyph misspelling of *trustwallet*.

### The registrar count is eight, not five

Domains sharing the registrar of the 77-second burst raise the total registered through
Fewmoretaps OU for this campaign to eight:

```
2026-07-24 15:52:57 UTC   promo-premium.pro       LIVE (found 2026-08-10, same kit)
2026-07-26 07:43:21 UTC   trustaws.com
2026-07-28 17:32:11 UTC   promo-settings.com
2026-07-28 17:32:31 UTC   trusted-settings.com
2026-07-28 17:33:07 UTC   wallet-settings.org
2026-07-28 17:33:18 UTC   promo-settings.org
2026-07-28 17:33:28 UTC   trusted-settings.org
2026-07-30 02:07:07 UTC   trustwailet.net
```

`promo-premium.pro` was confirmed live on 2026-08-10 (~16:05 UTC), serving the same kit
(page title "Crypto Cards", brand "TrustCard", USDT, Next.js bundle, Smartsupp fallback,
approval funnel). It sits in the same `promo-` family as `promo-settings.*`.

`trustwailet.net` used the registrar's own nameservers (`ARES`/`ZEUS.TRUSTNAME.COM`) rather
than a CDN. `trustaws.com` was scanned on 2026-07-28 at 18:07–18:13 UTC, roughly 34 minutes
after the five-domain burst was registered.

### What is proven and what is inferred

**Proven:** `54.39.106.37` served the same kit under the same page title; two domains it hosted
share a registrar with the five live ones; the naming families overlap.

**Not proven:** that the five current domains resolve to that same origin. Host-header and SNI
overrides against `54.39.106.37` for three of them returned nothing, because the host is down.
The origin link for the current cluster rests on kit, registrar and timeline — it is a strong
lead, not an established fact.

The operational reading is that they ran on a bare dedicated server until the end of July and
moved behind Cloudflare between 2026-07-30 and 2026-08-03. A dedicated server requires a
billing relationship, and the reverse-DNS handle `ns560354` identifies the exact machine in
OVH's inventory.

## Response headers identify the origin stack and timestamp every deployment

DNS, certificate transparency and subdomain enumeration all dead-end — the operator enabled
Cloudflare at registration, configured no mail, and uses a wildcard record so no forgotten
host leaks. HTTP response headers, however, are informative.

### The origin runs nginx, not a serverless platform

Static assets return a native nginx ETag — hex modification time, hyphen, hex content length:

```
etag: "6a698bb7-f45"
0x6a698bb7 = 1785388983 = Wed, 29 Jul 2026 05:12:23 GMT   (matches last-modified exactly)
0xf45      = 3909                                          (matches content-length exactly)
```

Vercel, Netlify and Cloudflare Pages do not emit this format. The origin is a conventional
server serving files from disk — the same class of machine as the OVH dedicated server that
hosted the earlier wave.

Dynamic responses additionally expose `x-powered-by: Next.js` and, on any non-existent path,
`x-middleware-rewrite: /pgzdtwaimmxxtvx` — Next.js middleware rewrites **every** request to a
randomised internal route, which is the mechanism behind the cloak. Every response also carries
`x-robots-tag: noindex, nofollow, noarchive, nosnippet`: the operation deliberately hides from
search engines so victims cannot find warnings. Security headers appear **duplicated**
(`x-frame-options`, `x-content-type-options`, `referrer-policy` each sent twice), indicating a
second proxy layer between nginx and Cloudflare.

### Deployment timestamps, to the second

Because nginx exposes `last-modified` on static chunks, each deployment is precisely dated:

| Deployed (UTC) | Domain | ETag | Chunk size |
|---|---|---|---|
| 2026-07-26 00:25:34 | `promo-premium.pro` | `6a6553fe-f44` | 3 908 |
| 2026-07-29 05:08:25 | `promo-settings.com` | `6a698ac9-f45` | 3 909 |
| 2026-07-29 05:12:23 | **`trusted-settings.com`** | `6a698bb7-f45` | 3 909 |
| 2026-07-30 10:39:09 | `wallet-settings.org` | `6a6b29cd-f45` | 3 909 |
| 2026-07-31 14:40:10 | `promo-settings.org` | `6a6cb3ca-f45` | 3 909 |
| 2026-07-31 14:43:43 | `trusted-settings.org` | `6a6cb49f-f47` | 3 911 |

**The deployments come in pairs minutes apart** — 05:08:25 and 05:12:23 on 29 July, then
14:40:10 and 14:43:43 on 31 July. Four-minute and three-minute gaps are one person deploying
sites consecutively in a single working session, not scheduled automation and not independent
owners. Chunk sizes within three bytes of each other confirm one product rebuilt per domain.

**Investigative value:** these are the moments the operator was connected to the server pushing
files. For a hosting provider holding access logs, six second-accurate timestamps are a direct
filter — whoever authenticated at 05:08 on 29 July, at 10:39 on 30 July and at 14:40 on 31 July
is the operator, identified without touching the blockchain.

## The funnel, recovered in full — including an instruction to hold more money

The complete funnel copy ships to every visitor inside the page payload and can be read without
passing the gate. Recovered 2026-08-10:

**Products.** Three cards: two at *"Price: 1.00 USDT"*, one premium at *"Price: 50.00 USDT"*.
Advertised limits: 10,000 USD, 20,000 USD and *"Unlimited"* per transaction.

**Trust-building claims, all fabricated.** *"Trust ranks first in the world by number of users,
with more than 300,000 cards already issued"*; *"Join 500,000+ users who trust Trust Card"*;
*"Get approved in seconds, not days. No credit checks, no paperwork. Just connect your wallet."*

**A referral scheme.** *"Get 5 USDT for every friend"* — *"Enjoying the card? Recommend TrustCard
to your friends and get paid for it."* Victims are recruited to bring further victims.

**The reassurance, offered by the people about to take everything:**

> *"Your funds cannot be frozen, and the card cannot be blocked."*

**The aggravating instruction.** Two separate tiers push the visitor to keep a large balance in
the very wallet that will be emptied:

> *"**Maintain a balance of $20,000 or more in your connected wallet.** Unlock Priority Pass,
> higher limits, and elevated rewards."*
>
> *"**Hold $20,000 or more in your portfolio** and receive the physical metal card."*

This is not passive deception. The operation deliberately instructs victims to increase the
amount at risk before the approval is signed, which bears directly on the assessment of intent
and of loss.

**The signing ladder**, verbatim:

```
step1  Connect your wallet
       "Open the app and approve the Trust connection. Payments will be charged
        directly from your wallet."
step2  We are preparing your transaction
step3  Confirm the transaction in your wallet
step4  Please confirm the transaction in your wallet again
step5  You closed WalletConnect
step6  Card issuance costs $1. Connect another wallet
```

Error strings confirm a WalletConnect integration and repeated retry prompts
(*"The user rejected the transaction signature"*, *"WalletConnect session expired. Please
reconnect your wallet"*, *"You canceled the transaction. If needed, you can try again"*).

### Why the gate could not be triggered from a workstation

For completeness: the funnel could not be made to render outside a genuine device session.
Everything forgeable in a request was attempted and changed nothing — Facebook in-app user
agents for both iOS (`FBAN/FBIOS`) and Android (`FB_IAB/FB4A`), the campaign's own advertising
parameters, `Referer`, `Accept-Language`, cookies, `X-Forwarded-For`, a full browser header set,
a simulated web3 provider injected before page scripts, and DOM removal of the decoy overlay.
The rewrite target and response were identical in every case.

What remains is not forgeable from a workstation: the bot score Cloudflare passes to the origin,
and the TLS fingerprint of the connection. That is consistent with a middleware decision keyed
on client reputation rather than on request contents — and it is why the decoy is what scanners,
researchers and ad reviewers always see.

The page routes reveal the decoy is a purpose-built product in itself. Its stylesheet defines ten
variants (`v1`–`v10`) and named layouts `cardScanner`, `cardQueue`, `cardDevice`, `cardTunnel`,
`cardTimeline`, `cardTerminal`, `cardMinimal` — seven of which were observed live.

## A third wave was registered the same day — through a second registrar

On 2026-08-10 at 14:14 UTC, three more domains of the same campaign were registered within
**47 seconds of each other** — the same burst signature as the 77-second run on 2026-07-28,
but this time through a **different registrar, NICENIC INTERNATIONAL GROUP** (Hong Kong),
again on Cloudflare nameservers:

```
14:14:00 UTC   gettrustcard.pro       (magali/odin.ns.cloudflare.com)
14:14:17 UTC   trust-credit-card.pro  (gail/sean.ns.cloudflare.com)
14:14:47 UTC   trust-credit.pro       (laura/alberto.ns.cloudflare.com)
```

All three already return Cloudflare's "Suspected Phishing" interstitial — detection caught
them within roughly two hours. The operator is moving registrars (Fewmoretaps → NiceNIC)
while running the same kit, which is why preservation requests to the registrars matter now
rather than later.

## Reports filed on this cluster

| Recipient | Reference | Scope |
|---|---|---|
| FBI IC3 | `5036615c3cd54d23ba7bd1c089786d1d` | original complaint |
| FBI IC3 | `7f90f61bf96c4dc585999781b2483aa7` | 14 victims, token analysis |
| FBI IC3 | `37289b4b2028461289a70e32c4345128` | 45 victims / 63,477 USDT, bridges |
| FBI IC3 | `f020e29e05a446618f2911d68b1b51a0` | Binance withdrawal → operator deanon |
| FBI IC3 | `012e182ac6f049e18e95f3c067748f46` | five-domain cluster, legal-process asks |
| Trustname registrar | `#ABS-48857` | all domains, under RAA 3.18.1 review |
| Cloudflare | `32255066b4e04a36` + 4 more | one per live hostname |
| OVH | `QPRHLVDXHC` | origin server ns560354 / 54.39.106.37 |
| BNB Chain support | `4bcf92ea-7a94-4711-a02e-f882757f0fcc` | freeze/blacklist confirmed impossible |
| Chainabuse | `264747c7-8915-471b-a362-d63f6364db9d` | public address flag |
| HashDit | (Telegram case) | indicators, RUM tokens, decoy analysis |

## Evidence preserved

Full HTML of all five sites, complete mirrors including every JavaScript chunk with per-file
SHA-256 manifests, and WHOIS snapshots were captured on 2026-08-10 11:35–11:47 UTC and are held
in the private case archive (`cluster-live-*.tar.gz`, SHA-256 recorded). Blockchain indicators
are in [INDICATORS.md](INDICATORS.md); the drainer contract, operator, and collector addresses
are unchanged from the original filing.
