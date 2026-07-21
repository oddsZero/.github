<!-- ============================================================= -->
<!--                          O D D S Z E R O                      -->
<!-- ============================================================= -->

<div align="center">

<a href="https://sui.io/">
  <img src="https://github.com/oddsZero/OddsZero/blob/main/sui-predict/frontend/public/images/OddsZeroCover.PNG" alt="OddsZero — On-chain prediction markets on Sui" width="100%" />
</a>

<br/>
<br/>

<h1>OddsZero</h1>

<p><strong>Fully on-chain, multi-outcome prediction markets on Sui.</strong></p>

<p>
  Trade outcome shares, provide liquidity, and settle against collateral —<br/>
  settled entirely in Move. Non-custodial. Fully collateralized. Permissionless.
</p>

<br/>

<!-- ── Badges ─────────────────────────────────────────────── -->


<!-- ── Badges ─────────────────────────────────────────────── -->

<p>
  <img alt="Move 2024" src="https://img.shields.io/badge/Move-edition%202024-4B8BF5?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0iI2ZmZiI+PHBhdGggZD0iTTEyIDJMMiA3djEwbDEwIDUgMTAtNVY3TDEyIDJ6Ii8+PC9zdmc+" />
  <img alt="Sui" src="https://img.shields.io/badge/Sui-Testnet-6FBCF0?style=for-the-badge&logo=sui&logoColor=white" />
  <img alt="Next.js 14" src="https://img.shields.io/badge/Next.js-14-000000?style=for-the-badge&logo=nextdotjs&logoColor=white" />
  <img alt="Pyth" src="https://img.shields.io/badge/Oracle-Pyth-8247E5?style=for-the-badge" />
  <img alt="License MIT" src="https://img.shields.io/badge/License-MIT-22C55E?style=for-the-badge" />
</p>

<p>
  <img alt="Non-custodial" src="https://img.shields.io/badge/non--custodial-100%25-16A34A?style=flat-square" />
  <img alt="Collateralized" src="https://img.shields.io/badge/backing-1:1_fully_collateralized-16A34A?style=flat-square" />
  <img alt="Modules" src="https://img.shields.io/badge/Move_modules-14-4B8BF5?style=flat-square" />
  <img alt="Outcomes" src="https://img.shields.io/badge/outcomes-2%E2%80%9364_per_market-4B8BF5?style=flat-square" />
</p>

<br/>

<!-- ── Nav ────────────────────────────────────────────────── -->

<p>
  <a href="#-quick-start"><b>Quick Start</b></a> &nbsp;·&nbsp;
  <a href="#-architecture"><b>Architecture</b></a> &nbsp;·&nbsp;
  <a href="#-the-protocol"><b>Protocol</b></a> &nbsp;·&nbsp;
  <a href="#-development"><b>Development</b></a> &nbsp;·&nbsp;
  <a href="#-repository-layout"><b>Layout</b></a> &nbsp;·&nbsp;
  <a href="#-security"><b>Security</b></a> &nbsp;·&nbsp;
  <a href="#-documentation"><b>Docs</b></a>
</p>

</div>

<br/>

<!-- ============================================================= -->

## ✦ What is OddsZero?

> **OddsZero is a fully on-chain prediction-market protocol on the [Sui](https://sui.io/) blockchain.**
> Anyone can trade on the outcome of future events — elections, sports, crypto prices,
> science, economics, and more — using shares whose prices reflect the market's
> collective probability estimate.

Unlike most prediction markets, which are custodial or partially off-chain, OddsZero
keeps **every market, every trade, and every resolution** in auditable Move smart
contracts. There is no backend that can freeze your funds.

<table>
<tr>
<td width="50%" valign="top">

#### 🔗 Fully on-chain
Markets, AMM, resolution, and disputes live in a **single Move package** on Sui.

#### 🔒 Non-custodial
Shares and collateral live in contracts you interact with directly. Only winning
shares are ever redeemable — **parimutuel** against the finalize-time collateral snapshot.

#### 🛡️ Fully collateralized
The complete-set invariant guarantees that the total winning shares at resolution equals
the trader-staked collateral, so parimutuel winner payouts can **never** be underfunded.

</td>
<td width="50%" valign="top">

#### 🎯 Multi-outcome
Two or more outcomes per market — binary *Yes/No* or many candidates (**2–64**).

#### 🌐 Open & permissionless
Anyone can create a market, trade, provide liquidity, or raise a dispute.

#### ⚡ Automated price markets
Short-expiry binary markets *("Will BTC be UP in 15m?")* resolve automatically
against a real **Pyth** reading — no human oracle required.

</td>
</tr>
</table>

<details>
<summary><b>📖 The one-paragraph version</b></summary>

<br/>

A market holds a pool of USDC (or any Sui coin) collateral, seeded with a **fixed 10,000-unit** creator deposit ring-fenced in a `seed_vault`. When you buy shares of an outcome, collateral enters the vault and a **complete set** of shares is minted (one of every outcome); the constant-product AMM prices the shares so the pool always implies a probability for each outcome. When the event ends, a registered oracle proposes the winning outcome and a dispute window opens. If no successful dispute occurs, the outcome is finalized, the creator's seed is **auto-refunded in full** from the ring-fenced vault, and holders of winning shares settle **parimutuel** — they split the remaining trader-staked collateral pro-rata by winning-share weight. A fixed protocol fee and creator fee are taken on each trade and routed to the protocol treasury and the market creator; fees never enter the vault.

</details>

<br/>

<!-- ============================================================= -->

## 🏗 Architecture

A monorepo built in three cooperating layers — the **chain** is the single source of
truth; the indexer and dApp are read-optimized views on top of it.

```mermaid
flowchart TB
    subgraph U["👤 Users"]
        W["Wallet · @mysten/dapp-kit"]
    end

    subgraph FE["🖥  Frontend — Next.js 14"]
        APP["App Router · Trade · Create · Portfolio · Admin"]
    end

    subgraph BE["⚙  Backend — Indexer & API"]
        IDX["Indexer  ·  events subscriber"]
        API["GraphQL /graphql  +  REST /api"]
        KEEP["Keeper  ·  auto-resolve & finalize"]
        DB[("SQLite · Prisma")]
    end

    subgraph CHAIN["⛓  Sui — Move package: oddszero"]
        MKT["prediction_market"]
        AMM["amm_pool"]
        ORA["oracle_integration"]
        GOV["governance"]
    end

    PYTH["🔮 Pyth Network"]

    W --> APP
    APP -->|reads| API
    APP -->|txns| CHAIN
    IDX -->|subscribe events| CHAIN
    IDX --> DB
    API --> DB
    KEEP -->|resolve / finalize| CHAIN
    KEEP -->|price| PYTH
    ORA -.->|price feed| PYTH
```

<br/>

<!-- ============================================================= -->

## 🧱 Technology Stack

<table>
<thead>
<tr><th align="left">Layer</th><th align="left">Technology</th></tr>
</thead>
<tbody>
<tr><td>🔷 <b>Smart contracts</b></td><td><a href="https://move-book.com/">Move</a> (edition 2024) on Sui</td></tr>
<tr><td>🔮 <b>Oracle</b> (price markets)</td><td><a href="https://pyth.network/">Pyth Network</a> via Wormhole</td></tr>
<tr><td>⚙️ <b>Indexer API</b></td><td>Node.js · Express · GraphQL (<code>graphql-http</code>) · Prisma</td></tr>
<tr><td>🗄️ <b>Indexer store</b></td><td>SQLite (zero-config) via Prisma</td></tr>
<tr><td>🖥️ <b>Frontend</b></td><td>Next.js 14 (App Router) · React 18 · Tailwind CSS</td></tr>
<tr><td>🔌 <b>Sui integration</b></td><td><code>@mysten/sui</code> · <code>@mysten/dapp-kit</code> · <code>@tanstack/react-query</code></td></tr>
<tr><td>📚 <b>Docs</b></td><td>VitePress</td></tr>
<tr><td>🧰 <b>Toolchain</b></td><td>Sui CLI ≥ 1.30 · Node.js ≥ 20</td></tr>
</tbody>
</table>

<br/>

<!-- ============================================================= -->


<!-- ============================================================= -->

## 🧠 The Protocol

### The complete-set invariant

While a market is trading, every unit of collateral that enters the vault mints
**one share of every outcome** (a *"complete set"*). This keeps a core invariant true
at all times:

```
Σ_outcome minted[outcome]  ==  num_outcomes × collateral_in_vault
```

When the market resolves, exactly one outcome wins. Each complete set contributes
exactly one winning share, so the global complete-set invariant implies:

```
winning_shares_in_circulation  ==  collateral_in_vault
```

Fees are taken *out* of the trader's payment and **never** enter the vault, so share
backing can never be diluted. At settlement, winners are paid **parimutuel**: they split
the trader-staked collateral snapshot pro-rata by winning-share weight, so total payouts
are always bounded by the collateral that traders actually deposited and the ring-fenced
creator seed is never drawn on.

<br/>

### Market lifecycle

```mermaid
stateDiagram-v2
    direction LR
    [*] --> TRADING
    TRADING --> RESOLVING : oracle proposes
    RESOLVING --> DISPUTED : raise_dispute
    DISPUTED --> RESOLVING : window re-extends
    RESOLVING --> RESOLVED : finalize_resolution
    DISPUTED --> RESOLVED : finalize (any caller; outcome bound by dispute consensus)
    RESOLVED --> [*] : redeem_shares (parimutuel winner settlement)
    TRADING --> ABANDONED : reclaim_abandoned_seed (30d past ends_at, never resolved)
```

| Status | Code | Meaning |
| --- | :---: | --- |
| `STATUS_OPEN` | `0` | *(reserved)* |
| `STATUS_TRADING` | `1` | Open for trading, liquidity, and auto-resolution. |
| `STATUS_RESOLVING` | `2` | Oracle proposed an outcome; dispute window open. |
| `STATUS_RESOLVED` | `3` | Final outcome locked; shares redeemable. |
| `STATUS_DISPUTED` | `4` | At least one dispute raised; window re-extended. |
| `STATUS_ABANDONED` | `5` | Never resolved past grace window; creator seed reclaimable by anyone. |

> A **closing-only window** opens at `closing_only_at = ends_at − closing_only_window_ms`.
> After that, opening new positions is disallowed; only position-reducing close-out
> trades are permitted, at the `maker_rebate_bps`-discounted protocol fee (rebate comes
> from governance, not a fixed value).

<br/>

### Two market types

<table>
<tr>
<td width="50%" valign="top">

**1 · Normal / oracle-resolved**

2–64 free-form outcomes, resolved by a registered oracle after
`ends_at + min_resolution_delay_ms`.

</td>
<td width="50%" valign="top">

**2 · Price-backed (Up/Down/Push)**

Outcomes `["Up","Down","Push"]` with a `PriceFeedConfig`. Auto-resolves against a real
asset price at `ends_at`. A tie within tolerance resolves as **Push** (market void);
Up and Down holders are refunded **pro-rata** from the collateral snapshot taken at
finalize — 1:1 refund of both legs is not solvent with a single complete-set backing.

</td>
</tr>
</table>

<br/>

### Two trading venues

- **AMM (primary)** — all `buy_shares` / `sell_shares` flow through `amm_pool`, integrated
  with resolution & redemption. This is what the frontend uses.
- **CLOB order book (optional, parallel)** — `order_book.move` is an independent
  price-time limit-order book whose positions are *not* reconciled with the AMM and do
  not affect the 1:1 backing invariant. Resolution/redemption integration is a future
  extension.

<br/>

### Market creation

Every market is created with a **fixed 10,000-unit** creator seed (e.g. 10,000 USDC),
ring-fenced in a `seed_vault` that is never touched by traders. The seed is
**auto-refunded in full** to the creator at finalize time, before any winner payouts,
so the creator's principal is always safe regardless of outcome. Markets must satisfy:

- **2–64 outcomes** (up to 64 for oracle-resolved markets; 3 fixed outcomes for price markets).
- **Duration**: minimum 1 hour, maximum ~1 year from creation.
- **Category and question** length caps (`MAX_CATEGORY_LEN = 64`, `MAX_QUESTION_LEN = 512`).
- **Outcome label** length cap (`MAX_OUTCOME_LABEL_LEN = 128`).

The `creator_fee_bps` argument is accepted but **ignored** — the creator fee is locked
at 0.25% (25 bps) for every market.

<br/>

### Fees

All fees are in basis points (`10000 bps = 100%`). Fees are **never** added to the
collateral vault, so backing is never diluted.

| Fee | Default | Bound | Paid to |
| --- | :---: | :---: | --- |
| `protocol_fee_bps` | 75 (0.75%) | ≤ 1000 | Treasury |
| `creator_fee_bps` | 25 (0.25%) | FIXED | Market creator |
| `dispute_bond_bps` | 100 (1.00%) | ≤ 1000 | Bond (returned/forfeited) |
| `maker_rebate_bps` | governance | ≤ 10000 | Close-out discount |

> **Note:** `creator_fee_bps` is **locked at 0.25% (25 bps)** for every market; the `creator_fee_bps` argument on market creation is accepted but always ignored. Referral fees are **not implemented** — the `referrer` field is accepted but never recorded and no referral slice is taken.

> See the [Fees reference](./OddsZero_Docs/protocol/fees.md) for the exact buy/sell split formulas.

<br/>

### Resolution, disputes & redemption

1. **Propose** — a registered oracle calls `resolve_market` (or `resolve_market_price` for
   price markets), which opens the dispute window.
2. **Dispute** — anyone may `raise_dispute` by posting a bond. Disputes from the oracle or
   creator are recorded but ignored for consensus. The window re-extends on each dispute
   (up to `MAX_DISPUTES = 10`).
3. **Finalize** — after the window, `finalize_resolution` (or `finalize_price_resolution`
   for disputed price markets) locks the outcome. Any caller may finalize; when disputes
   exist and agree on a claimed outcome, that outcome is binding for all callers including
   the admin.
4. **Redeem** — holders call `redeem_shares` to claim their parimutuel share of the
   finalize-time collateral snapshot, pro-rata by winning-share weight. For Push (tie)
   voids, Up and Down holders split the snapshot pro-rata.

<br/>

### Governance

A single shared `Governance` object holds global parameters. Markets **snapshot** relevant
values at creation, so global changes affect only new markets. The admin may update parameters
directly; DAO voting using transferable `GovernanceShare` objects is also available — a
proposal needs quorum + majority before `execute_proposal` applies it.

<br/>

<!-- ============================================================= -->

## 📦 Contract Modules

All modules live under the `oddszero::` namespace.

| Module | Responsibility |
| --- | --- |
| `prediction_market` | `Market<T>` object, `MarketRegistry`, trading & resolution orchestration. |
| `amm_pool` | Constant-product N-outcome AMM pricing, reserves, LP math. |
| `share_token` | CTF complete-set share ledger & mint/burn accounting. |
| `coin_wrapper` | Typed `Collateral<T>` vault & coin helpers (USDC integration point). |
| `oracle_integration` | Oracle propose / dispute / finalize lifecycle & bonds. |
| `price_oracle` | Automated real-price resolution (Up/Down/Push). |
| `order_book` | Optional CLOB limit-order book venue. |
| `treasury` | Protocol fee collection & distribution. |
| `governance` | Global parameters & DAO voting. |
| `admin` | Capability-based access control (`AdminCap`, `AdminRegistry`). |
| `lp_incentives` | LP reward emission stream & `IncentiveVault`. |
| `events` | Canonical event structs & `emit_*` wrappers. |
| `errors` | Centralized, stable error-code catalog (codes 1–29). |
| `utils` | Overflow-safe math & string helpers. |

> See `sui-predict/contracts/README.md` for the full per-function guide and calling examples.

<br/>

<!-- ============================================================= -->

## 🗂 Repository Layout

<details>
<summary><b>Expand the full monorepo tree</b></summary>

```
OddsZero/
├── sui-predict/               # Application (contracts + indexer + frontend)
│   ├── contracts/             # Move smart contracts (package: oddszero)
│   │   ├── sources/           # 14 Move modules — the source of truth
│   │   ├── tests/             # Move unit & e2e tests
│   │   ├── examples/          # Sample market seeds (test_markets.json)
│   │   ├── Move.toml          # Package manifest (edition 2024)
│   │   └── README.md          # Per-package contract documentation
│   ├── backend/               # Off-chain indexer, pricing engine & API
│   │   ├── src/               # Express + GraphQL server, Pyth, keeper
│   │   ├── prisma/            # SQLite schema mirroring on-chain events
│   │   └── package.json
│   ├── frontend/              # Next.js 14 dApp (app router, Tailwind)
│   │   ├── app/               # Routes: markets, create, portfolio, admin…
│   │   ├── lib/               # Sui client, chain calls, AMM math, formatting
│   │   ├── hooks/             # Wallet & chain polling hooks
│   │   └── package.json
│   ├── scripts/               # Dev seed/deploy helpers (dev.mjs, seed-contracts.mjs)
│   ├── dev-all.mjs            # Single-command dev orchestrator (backend + frontend)
│   └── package.json
├── OddsZero_Docs/             # VitePress documentation site
│   ├── .vitepress/            # VitePress config & theme
│   ├── concepts/              # Core concepts & architecture
│   ├── protocol/              # Lifecycle, AMM math, fees, resolution, disputes…
│   ├── guides/                # User, developer, and deployment guides
│   ├── security/              # Security model & audit guide
│   ├── reference/             # Events, error codes, glossary
│   ├── introduction.md        # Docs landing content
│   └── index.md               # Docs home / hero
├── LICENSE                    # MIT
└── .gitignore
```

</details>

<br/>

<!-- ============================================================= -->

## ⚙ Backend — Indexer & API

The backend is an off-chain service that **mirrors** on-chain events into a SQLite store
and exposes read-optimized views. It is **not** a source of truth — the contracts are.

- 🔎 **GraphQL** (`/graphql`) & **REST** (`/api`) — markets, trades, positions, categories, global stats.
- 📡 **Indexer** — subscribes to `MoveEventModule: { module: "events" }` and keeps AMM reserves, volumes, liquidity, and positions in sync.
- 📈 **Pricing engine** (`amm.ts`) — derives probabilities, prices, and history from reserves.
- 🔮 **Pyth integration** (`pyth.ts`) — price-market resolution.
- 🤖 **Keeper** (`keeper.ts`) — resolves ended price markets against Pyth and finalizes markets whose dispute window has elapsed.

```bash
cd sui-predict/backend
npm run dev              # tsx watch server
npm run keeper           # automated resolution worker
npm run db:push          # sync Prisma schema to SQLite
npm run db:generate      # generate Prisma client
npm run typecheck
npm run lint
```

> ⚙️ Configure `DATABASE_URL` and contract object IDs (`PACKAGE_ID`, `REGISTRY_ID`,
> `GOVERNANCE_ID`, `TREASURY_ID`, `USDC_TYPE`, `KEEPER_*`) via the backend `.env`.

<br/>

<!-- ============================================================= -->

## 🖥 Frontend — dApp

A Next.js 14 application to browse markets, trade shares, create markets, manage
portfolios, view leaderboards, and (for admins) govern the protocol.

| Route | Purpose |
| --- | --- |
| `/` | Market list & global stats |
| `/markets/[id]` | Market detail · trade panel · order book · history |
| `/create` | Create a new market |
| `/portfolio` | A wallet's positions & redemptions |
| `/leaderboard`, `/traders` | Analytics |
| `/admin` | Admin console (gated by `ADMIN_HOST`) |

```bash
cd sui-predict/frontend
npm run dev              # next dev on :3000
npm run build && npm run start
npm run typecheck
npm run lint
```

> ⚙️ Copy `frontend/.env.example` → `frontend/.env.local` and fill in the API URL and
> deployed contract IDs (`NEXT_PUBLIC_PACKAGE_ID`, `NEXT_PUBLIC_REGISTRY_ID`,
> `NEXT_PUBLIC_TREASURY_ID`, etc.).

<br/>

<!-- ============================================================= -->

## 🛠 Development

### Useful scripts (root)

```bash
cd sui-predict
npm run dev              # backend + frontend together (dev-all.mjs)
npm run dev:frontend
npm run dev:backend
npm run build            # build frontend
npm run typecheck        # frontend typecheck
npm run seed:contracts   # seed sample markets (frontend/seedMarkets.mjs)
```

### Local env files

- `sui-predict/backend/.env` — DB URL, contract IDs, keeper credentials.
- `sui-predict/frontend/.env.local` — API URL, Sui GraphQL URL, contract IDs.

> 🔐 Both have `.example` / documented counterparts. **Never commit secrets** (see `.gitignore`).

<br/>

<!-- ============================================================= -->

## 🔐 Security

OddsZero is designed around a small set of auditable invariants:

- ✅ The **complete-set** invariant guarantees winner payouts are always fully backed by the
  trader-staked collateral snapshot taken at finalize.
- ✅ **Ring-fenced `seed_vault`**: the creator's seed is escrowed separately and auto-refunded
  in full at finalize, before any winner payouts, so the seed is never at risk.
- ✅ **Parimutuel winner settlement**: winners split the finalize-time collateral snapshot
  pro-rata by winning-share weight, so total payouts can never exceed the collateral
  traders deposited.
- ✅ **Fees never enter the vault**, so backing can never be diluted.
- ✅ **Capability-based access control** (`AdminCap` — non-copyable / non-drop / non-transferable) gates privileged calls; oracles are registered in `AdminRegistry`.
- ✅ **Dispute bonds** and dispute-consensus binding prevent oracle self-dealing and ensure
  the final outcome reflects the economic signal of independent disputants.

For the full threat model, invariant catalog, and audit checklist, see:

- [Security Model](./OddsZero_Docs/security/model.md)
- [Auditing the Contracts](./OddsZero_Docs/security/audit.md)
- Contract `errors.move` for the complete error-code catalog.

> [!WARNING]
> OddsZero is research/educational software. Contracts have **not** been formally audited
> at the time of writing. Use on **testnet only** — do not risk funds you cannot afford to lose.

<br/>

<!-- ============================================================= -->

## 🌍 Networks & Contract Addresses

| Item | Value |
| --- | --- |
| Package name | `oddszero` |
| Language | Move (edition 2024) |
| Collateral | Native USDC (protocol whole-unit convention: 1 unit = 1 USDC; 6-decimal base units at the coin boundary are handled by the wrapper) — or any Sui coin type `T` |
| Testnet package id | [`0x37573a1060e150e2cbc48ea310e1a05b859dd18541344ffe1c2e304fee702916`](https://suiscan.xyz/testnet/object/0x37573a1060e150e2cbc48ea310e1a05b859dd18541344ffe1c2e304fee702916) |
| Toolchain | Sui CLI `1.74.1` |
| Explorer | [suiscan.xyz/testnet](https://suiscan.xyz/testnet) |

> 🚀 Mainnet addresses will be published here before launch.

<br/>

<!-- ============================================================= -->

## 📚 Documentation

The full documentation site lives in [`OddsZero_Docs/`](./OddsZero_Docs) and is built with VitePress.

```bash
cd OddsZero_Docs
npm install
npm run dev        # vitepress dev
npm run build      # vitepress build
```

<table>
<tr>
<td valign="top" width="33%">

**Concepts**
- [Introduction](./OddsZero_Docs/introduction.md)
- [Core Concepts](./OddsZero_Docs/concepts/)
- [Architecture](./OddsZero_Docs/concepts/architecture.md)

</td>
<td valign="top" width="33%">

**Protocol**
- [Market Lifecycle](./OddsZero_Docs/protocol/lifecycle.md)
- [AMM & Pricing Math](./OddsZero_Docs/protocol/amm.md)
- [Resolution & Oracles](./OddsZero_Docs/protocol/resolution.md)
- [Disputes](./OddsZero_Docs/protocol/disputes.md)
- [Governance](./OddsZero_Docs/protocol/governance.md)
- [LP Incentives](./OddsZero_Docs/protocol/incentives.md)
- [Price-Backed Markets](./OddsZero_Docs/protocol/price-markets.md)

</td>
<td valign="top" width="33%">

**Guides & Reference**
- [User Guide](./OddsZero_Docs/guides/user-guide.md)
- [Developer Guide](./OddsZero_Docs/guides/developers.md)
- [Deployment Guide](./OddsZero_Docs/guides/deployment.md)
- [Events Reference](./OddsZero_Docs/reference/events.md)
- [Error Codes](./OddsZero_Docs/reference/errors.md)
- [FAQ](./OddsZero_Docs/faq.md)

</td>
</tr>
</table>

<br/>

<!-- ============================================================= -->

## 🤝 Contributing

Contributions are welcome. Please open an issue or pull request on the repository.
For security-sensitive findings, follow **responsible disclosure** rather than opening a
public issue.

<br/>

## 📄 License

Released under the [MIT License](./LICENSE).

<br/>

<div align="center">

<sub>Built with ⚡ on <a href="https://sui.io/">Sui</a> · Powered by <a href="https://move-book.com/">Move</a> & <a href="https://pyth.network/">Pyth</a></sub>

<br/><br/>

<sub>Copyright © 2026 <b>OddsZero</b></sub>

</div>
