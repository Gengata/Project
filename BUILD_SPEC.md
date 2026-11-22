# SYNAPSE (working name) — Build Brief for Claude Code

> Paste this whole file into Claude Code (or save it as `BUILD_SPEC.md` in an empty repo and say: "Read BUILD_SPEC.md and start with Phase 0. Show me your plan before writing code.").
> Items marked **[DECISION]** are open: propose a default, tell the founder, and carry on. Items marked **[GATE]** must NOT be built until the founder explicitly says so.

---

## 0. How you (Claude Code) should work

1. **Plan first.** Before any code, post a short plan for the current phase (files, libs, risks) and wait for a go-ahead.
2. **Build phase by phase** (section 9). Finish a phase, run its acceptance checks, then stop and report. Do not jump ahead.
3. TypeScript everywhere. Monorepo (pnpm workspaces). Strict mode. Tests for anything involving money, credits, risk limits or the strategy schema.
4. Small commits, clear messages. No secrets in the repo, ever. Use `.env.example`.
5. Verify third-party APIs (Solana RPC providers, Pump.fun/PumpSwap data, Jupiter, Stripe, Anthropic) against their **current docs** before using them. Do not rely on memory for endpoints, limits or pricing.
6. If something in this brief conflicts with safety (section 7), safety wins.

---

## 1. Executive summary & hook

"SYNAPSE is where anyone can build a trading bot, or any AI agent, just by talking. Describe what you want in plain words. The AI designs the strategy, writes it, tests it, and then shows you the bot **thinking in real time** inside a living digital brain. You watch signals fire, safety checks block bad trades, and every decision explain itself."

It is "Lovable for trading bots and AI agents":
- Chat on the left, live preview sandbox on the right.
- Credits-based pricing with a free tier.
- Launch focus: **Solana meme-coin trading bots**. Then general-purpose AI agents on the same engine.

---

## 2. Core platform features

### A. Builder workspace (Lovable-style)
- **Left panel: AI chat** with three modes:
  - **Chat**: brainstorm, ask questions, nothing changes.
  - **Plan**: AI proposes a step-by-step plan and the exact strategy changes; the user approves before anything is built.
  - **Build**: AI applies the approved plan (creates or edits the bot).
- **Right panel: Preview Sandbox** with tabs: **Brain** (default), **Strategy** (readable spec), **Code**, **Backtest**, **Logs**.
- Top bar: project name, mode badge (**DEMO / BACKTEST / PAPER / LIVE**), credits left, Run/Pause, Kill switch, Share.
- Dark theme, fast, mobile-friendly (chat and sandbox become two tabs on a phone).
- Version history: every Build creates a bot version; one-click restore.

### B. Auth
- Email magic link + Google sign-in. Age gate: **18+ only** (checkbox plus date of birth on signup). Region block list is configurable **[DECISION: with lawyer]**.

### C. Credits (see section 5)
- Free: **5 credits after login, refilled every 24 hours.**
- Paid: monthly subscription with a monthly credit pool.

### D. Bot lifecycle
`Describe → Plan → Build → Backtest → Paper trade → (paid plans) Live`
- Free users can Build, Backtest and Paper trade.
- Live trading is **paid plans only** and **[GATE]** (section 9, Phase 6).

### E. Export & ownership
- Download the strategy spec (JSON) and generated code. GitHub sync is a later nice-to-have.

---

## 3. How the AI works

The product AI uses the Anthropic API (Claude). Model IDs come from env vars, not hard-coded: `MODEL_BUILD` (default `claude-sonnet-5-5`), `MODEL_CHEAP` (default `claude-haiku-5-5`) for chat and summaries, `MODEL_HARD` (default `claude-opus-5-5`) for complex strategy design. Verify IDs in the current docs.

### Key design rule: the AI writes a **Strategy Spec**, not free-form code
- The model's main output is a **validated JSON Strategy Spec** (zod schema, section 6) that composes **vetted, pre-built agent modules** (scanner, rug gate, sizer, executor...) with parameters.
- Custom logic (rare) is allowed later as sandboxed code (Phase 7) with no network, no keys, and only a narrow read-only data API.
- Why: safer, testable, easy to render in the Brain, easy to backtest, cheap to generate.

### AI behaviors
- **Understands intent** ("snipe new Pump.fun tokens with locked liquidity and take profit at 2x") and asks at most 1-2 clarifying questions.
- **Plan mode** returns a human-readable plan plus a diff of the spec.
- **Self-check before presenting**: schema-validate the spec, run a quick backtest smoke test, run the risk-limit linter, and fix its own errors (max 2 retries) before showing the result.
- **Explains everything**: every trade decision carries a human-readable `reason`.
- **Project memory**: remembers the user's risk preferences, wallet rules, past decisions.
- **Honest by default**: never promises profits; warns when a strategy has no stop-loss, huge position size, or no rug filter; shows backtest results with fees, slippage and failed-trade assumptions.
- Uploads: users can attach screenshots (e.g., a dashboard they like) as design/strategy inspiration.

### Prompt-injection safety
Token names, descriptions, social posts and any on-chain text are **untrusted data**. They must never be placed in prompts as instructions. If an agent LLM-reads such text, it is wrapped, labeled as data, and its output is schema-constrained. It can never change risk limits or trigger trades directly.

---

## 4. The Brain Sandbox (the signature feature — put real craft here)

### 4.1 Concept
The preview is a **living, moving human-brain visualization** of the user's bot. Every agent is a **region of the brain**. Real events from the bot become **signals traveling along synapses**. The user can literally watch the bot "think": perceive, get suspicious, decide, size, act, remember.

Reference screenshots (uploaded by the founder, now in `/docs/reference/`): three Solana bot dashboards: a "Quantum Orb" with ~1,600 launch-states and a decision stack, a pipeline dashboard (Scanner → Rug Check → Sizer → Executor → Logger, "every trade gets a reason"), and a 12-agent "command center" ringed around a hub with candlestick chart, whale signals and fills. **Make ours cooler, better organized, and more readable. Do not copy their branding or exact layout.**

### 4.2 Visual design
- **Center: 3D brain** (two hemispheres with a midline fissure and gyri-like folds), built from ~2,000-4,000 glowing nodes and thousands of thin synapse curves. Procedurally generated (no heavy asset needed); optionally a low-poly mesh as a guide shape.
- **Tech**: three.js via react-three-fiber (WebGL), instanced meshes for nodes, shader-based glow/bloom, GPU particles for pulses. Fallback to Canvas2D on weak devices. Target **60 fps desktop, 30+ fps mobile**; adaptive node count; respect `prefers-reduced-motion` (slow, minimal motion mode).
- **Regions = agent groups** (each with its own color and label):
  | Region | Brain analogy | Agents |
  |---|---|---|
  | **Perception** | sensory cortex | Pump Scout, Volume Spike, Whale Watch, KOL Watch, Holder Flow |
  | **Safety** | amygdala | Rug Gate (honeypot sell-simulation, LP lock, dev holding), Bundle Hunter (sniper/bundle clusters), Creator Risk |
  | **Decision** | prefrontal cortex | Strategy Brain (scores EV after costs), Decision Stack |
  | **Risk** | brainstem | Sizer (% of equity, Kelly-capped), Stop Guard, Take-Profit Ladder, Daily Loss Limit |
  | **Action** | motor cortex | Executor (quotes, slippage cap, fills) |
  | **Memory** | hippocampus | Scribe/Logger (every decision + reason), Equity tracker, Learning (post-trade labeling) |
- **Idle life**: subtle ambient firing so the brain always feels alive (clearly "ambient", not data).
- **Camera**: slow auto-orbit; drag to rotate; scroll to zoom; double-click a region to focus.
- **Style**: dark space-like background, neon but tasteful palette, crisp typography (monospace for data), no clutter. Accessible contrast; colorblind-safe palette option.

### 4.3 What the animation means (all driven by real events)
- **New token scanned** → small spark at Perception.
- **Signal pulse** travels Perception → Safety (along curved synapses).
- **Rejected by safety** → pulse turns red at Safety, node flashes, reason text appears in the Decision Stack ("honeypot: sell sim failed").
- **Passed** → pulse turns green into Decision, then Risk (sizing), then Action.
- **Trade filled** → gold burst at Action; the Memory region "writes" it (pulse back into hippocampus).
- **Take-profit / stop-loss** → distinct burst colors; equity curve ticks.
- **Kill switch** → whole brain dims, pulses stop, big status banner.
- **Click any region** → Inspector drawer: that agent's config, last 20 decisions with reasons, hit-rate, latency.
- Node/edge brightness encodes recent activity; thickness encodes volume.

### 4.4 Panels around the brain (organized, not cluttered)
- **Top strip**: mode badge, balance, P&L (day / total), open positions, win/loss, slippage cap, latency, network health.
- **Pipeline strip**: `Scan → Check → Size → Execute → Log` with live counts (scanned / filtered / traded).
- **Left column**: Agent list with status (online / idle / blocked) and a one-line "what I'm doing now".
- **Right column**: **Decision Stack**: newest on top; each card shows token, action (TRADE / SKIP / BLOCK), score, **reason**, and expected value after costs.
- **Bottom row** (tabbed on mobile): **Trade Tape**, **Agent Log** (`tail -f` style), **Equity Curve**, **Candlestick chart** with buy/sell/TP/SL markers for the focused token.
- **Extras (Phase 2b, after the core works)**: Monte-Carlo **Exit Simulator** (fan of simulated price paths with TP/SL barriers), **Wallet Cluster map** (connected wallets / bundled buys), **Slot River** (per-slot activity bars), **Risk gauges** (exposure, daily loss, kill switch distance).
- **Layout**: CSS grid with fixed zones; panels collapsible; "Focus brain" button hides panels for a full-screen view; "Explain" button narrates what the brain is doing in plain language.

### 4.5 Data contract (events drive everything)
Single typed event bus; the UI never invents data.

```ts
type BrainEvent = {
  id: string; ts: number; runId: string;
  mode: 'demo' | 'backtest' | 'paper' | 'live';
  agent: AgentId;                       // e.g. 'rug_gate'
  region: 'perception'|'safety'|'decision'|'risk'|'action'|'memory';
  type: 'scan'|'signal'|'block'|'pass'|'skip'|'size'|'order'|'fill'|'exit'|'log'|'risk_alert'|'kill';
  token?: { mint: string; symbol: string };
  payload?: Record<string, unknown>;    // numbers, scores, tx sig
  reason?: string;                      // REQUIRED for block/skip/exit/order/fill
  severity?: 'info'|'warn'|'critical';
};
```
- Transport: WebSocket (or SSE) from the runtime to the browser; events also stored in Postgres for replay.
- **Replay**: any backtest or past run can be replayed in the Brain with a timeline scrubber and speed control (1x/10x/100x).
- **DEMO mode**: a built-in simulator emits synthetic events so the Brain looks alive while the user is still building. It is **always** labeled **DEMO / SIMULATED**; demo data must never be shown as real results.

---

## 5. Credits & pricing

### Free tier
- **5 credits after first login.**
- **Daily refill:** when `now - last_refill_at >= 24h`, set `balance = max(balance, 5)` and `last_refill_at = now`. Refill tops up to 5; it does **not** accumulate **[DECISION: confirm no-rollover]**.
- Free users: Build + Backtest + Paper trade, limited concurrent paper bots (e.g., 1), no Live.

### Paid plans (monthly, Stripe subscriptions)
Seed values below are placeholders **[DECISION: founder sets prices and limits]**:
| Plan | Credits / month | Notes |
|---|---|---|
| Starter | 100 | more paper bots, longer backtests |
| Pro | 300 | Live trading unlocked, priority models |
| Scale | 1000 | multiple live bots, API access |
- Optional one-off **credit top-up packs**.
- Monthly credits reset at the billing date (no rollover by default). Top-up credits never expire.

### What costs credits (configurable table in DB, defaults)
| Action | Credits |
|---|---|
| Chat message | 0.25 |
| Plan | 0.5 |
| Build / edit | 1 |
| Complex build (uses `MODEL_HARD`) | 2 |
| Backtest, paper run, replay | 0 (but rate- and size-limited per plan) |

### Implementation requirements
- **Ledger-based**: `credit_ledger` (append-only), balance derived; atomic debit with an **idempotency key**; **auto-refund** if a generation fails.
- Check balance **before** the model call; show a clear "out of credits: refills in HH:MM:SS or upgrade" state with a countdown.
- Stripe: Checkout + Customer Portal + webhooks (signature verified, idempotent handlers). Handle upgrade, downgrade, cancel, failed payment.
- Abuse protection: rate limits, one free account per verified email + device/IP heuristics, disposable-email block.

---

## 6. Strategy Spec (the heart of the system)

A versioned zod schema in `packages/spec`. Example shape (finalize in Phase 1):

```jsonc
{
  "version": 1,
  "name": "Pump.fun momentum sniper",
  "venue": { "chain": "solana", "sources": ["pumpfun", "pumpswap"] },
  "agents": {
    "scanner":  { "newPairs": true, "minVolumeSpikeX": 5, "maxAgeMin": 10 },
    "whale_watch": { "enabled": true, "minWallets": 3, "minUsd": 5000 },
    "rug_gate": { "honeypotSellSim": true, "minLpLockedPct": 90, "maxDevHoldPct": 5, "maxTop10HolderPct": 40 },
    "bundle_hunter": { "maxBundledSupplyPct": 20 },
    "sizer":    { "method": "pct_equity", "pct": 2, "maxPositionUsd": 50 },
    "executor": { "maxSlippagePct": 2, "priorityFeeSol": 0.0005 },
    "exits":    { "takeProfit": [{ "atX": 2, "sellPct": 50 }, { "atX": 3, "sellPct": 50 }],
                  "stopLossPct": 12, "trailingPct": 25, "timeStopMin": 60 }
  },
  "risk": { "maxOpenPositions": 6, "maxDailyLossPct": 5, "maxExposurePct": 15, "killSwitchOnLossPct": 10 }
}
```
- Schema enforces **hard ceilings** (e.g., `maxPositionUsd`, `maxExposurePct`) that the AI cannot exceed even if the user asks. Ceilings are plan-dependent.
- The same spec drives: backtester, paper runtime, live runtime and the Brain layout (which agents exist → which regions light up).

---

## 7. Safety, risk & legal (non-negotiable)

1. **Risk Engine is deterministic code, outside the AI and outside any generated code.** Enforces: max position, max exposure, max open positions, max slippage, daily loss limit, kill switch, per-minute order rate limit. AI output can only *tighten* limits, never loosen past plan ceilings.
2. **Paper-first.** A bot must complete a minimum paper-trading run (e.g., 24h or N trades) before Live can be enabled **[DECISION: thresholds]**.
3. **Token safety pipeline** runs before any buy: honeypot sell simulation, LP lock / mint & freeze authority checks, dev and top-holder concentration, bundle/sniper clusters, creator history. Log the reason for every block.
4. **Kill switch** always visible; stops all bots instantly; halts on RPC/data anomalies.
5. **Wallet/custody [GATE].** Preferred: non-custodial, a dedicated bot wallet per user with strict funding caps and withdraw-only-to-owner. Private keys, if ever stored, are envelope-encrypted with a cloud KMS, never logged, never sent to the browser, never visible to the AI or generated code. Because this can count as custody, **do not build Live until the founder confirms legal review.** The founder is in the EU (MiCA and financial-promotion rules may apply).
6. **Honest marketing/UI.** Persistent risk disclaimer ("most bots lose money; meme coins can go to zero; not financial advice"). No profit claims, no fake leaderboards. **The reference screenshots show unverified profit numbers; never copy their style of claims. Demo data is always labeled.**
7. **Generated-code sandbox** (Phase 7): isolated container or V8 isolate, no network (except an allowlisted internal data API), CPU/memory/time limits, no filesystem, no env, no keys.
8. **Security basics**: auth on every endpoint, row-level security per user, input validation, CSRF protection, audit log for credit, billing and trading actions, dependency scanning, secrets in a manager.
9. Content/IP: don't ship others' logos or branding in the UI.

---

## 8. Architecture

**Monorepo (pnpm)**
```
apps/web          Next.js (App Router) + Tailwind + shadcn/ui: builder, sandbox, billing, auth
apps/runtime      Node/TS worker: backtest / paper / live runners, event bus, WebSocket server
packages/spec     zod Strategy Spec + migrations + linter
packages/agents   vetted agent modules (scanner, rug_gate, sizer, executor, exits...)
packages/risk     deterministic risk engine (100% test coverage target)
packages/backtest event-driven backtester (fees, slippage, failed tx, latency model)
packages/brain    brain visualization (react-three-fiber), event reducer, demo simulator
packages/ai       prompts, Anthropic client, plan/build pipelines, evals
packages/db       Postgres schema + migrations (Drizzle or Prisma)
docs/             reference screenshots, ADRs
```
- **DB/Auth**: Postgres (Supabase is fine) with row-level security. **Queue/cache**: Redis. **Billing**: Stripe. **Hosting**: web on Vercel-like; runtime on containers (Fly.io/Railway/AWS) **[DECISION]**.
- **Solana**: RPC/websocket provider (e.g., Helius, Triton; verify), Pump.fun / PumpSwap event feeds, Jupiter for quotes and swaps, price/OHLCV from a provider (e.g., Birdeye/DexScreener/own recorder; verify ToS and limits).
- **Historical data**: a recorder service stores launch events and trades from day one (backtests depend on it).
- **Observability**: structured logs, error tracking, metrics on credit usage, model cost per user, runtime latency.

**Core tables**: `users`, `profiles`, `credit_ledger`, `credit_config`, `subscriptions`, `projects`, `messages`, `bot_versions`, `strategy_specs`, `backtests`, `runs` (paper/live), `events`, `trades`, `wallets` (gated), `audit_log`.

**Key endpoints**: `POST /api/chat` (mode: chat|plan|build, debits credits), `GET /api/credits`, `POST /api/stripe/webhook`, `POST /api/bots/:id/backtest`, `POST /api/bots/:id/run` (paper|live), `POST /api/bots/:id/kill`, `GET /ws/runs/:id` (events).

---

## 9. Build phases (for Claude Code)

Stop at the end of each phase and report. Each phase lists **acceptance checks**.

### Phase 0: Foundation
Monorepo, Next.js app, Tailwind/shadcn, auth (magic link + Google, 18+ gate), Postgres + RLS, credit ledger + **daily refill job**, Stripe subscriptions (test mode), `.env.example`, CI.
- ✅ New user gets 5 credits; after 24h refill tops up to 5 (unit-tested with fake clock); debits are atomic and idempotent; Stripe test checkout grants monthly credits.

### Phase 1: Builder chat + Strategy Spec
Lovable-style two-pane layout; Chat/Plan/Build modes; Claude pipeline that outputs a validated Strategy Spec; spec viewer; version history; credit debit + refund-on-failure.
- ✅ "Make a bot that buys new Pump.fun tokens with locked LP and sells at 2x" yields a valid spec with a stop-loss and risk limits; invalid specs are auto-repaired or rejected; out-of-credits UI with countdown.

### Phase 2: Brain Sandbox (biggest craft phase)
`packages/brain`: procedural brain, regions, synapses, pulses, bloom, camera, inspector, all panels from 4.4, event reducer, **DEMO simulator**, mobile layout, perf budget.
- ✅ 60 fps on a mid laptop at default quality; every event type in 4.3 has a distinct animation; click region → inspector; reduced-motion mode works; DEMO badge always visible; Storybook/test page with an event-replay fixture.
- 2b: Monte-Carlo exit simulator, wallet cluster map, slot river, risk gauges.

### Phase 3: Backtester + recorder
Event recorder, historical store, event-driven backtester with fees/slippage/failed-tx/latency model, report tab, **replay in the Brain**.
- ✅ Same spec → deterministic results; report shows drawdown, win rate, per-reason block counts; replay scrubber works.

### Phase 4: Paper trading with live market data
Runtime service, real data feeds, simulated fills with realistic slippage, WebSocket events to the Brain.
- ✅ Brain is driven by real events in PAPER mode; pause/resume/kill work; reconnect without losing state.

### Phase 5: Hardening
Risk-engine fuzz/property tests, prompt-injection tests on token text, load test, security review, cost dashboards, abuse limits.
- ✅ No path lets generated output exceed plan ceilings; red-team test suite passes.

### Phase 6: Live trading **[GATE: needs explicit founder go-ahead + legal review]**
Bot wallet flow, KMS-encrypted keys, funding caps, Jupiter execution, owner-only withdrawals, paid-plan check, paper-first requirement, audit log, incident runbook.

### Phase 7: General AI agents ("agents for everything")
Generalize the engine: Agent = graph of nodes (tools, triggers, schedules, memory) using the same spec → runtime → Brain pipeline. Tool registry with per-tool permissions; templates (research agent, social-monitor agent, portfolio-alert agent, etc.); sandboxed custom code; the Brain regions are configurable per template.
- ✅ A non-trading agent can be built from chat, run on a schedule, and appear in the Brain with its own regions.

### Phase 8 (later)
Publish/share bots, templates marketplace, GitHub export, team workspaces, EVM chains and CEX adapters.

---

## 10. 60-second pitch (for the landing page)

"SYNAPSE turns trading-bot and AI-agent building into a conversation. Tell it what you want. It plans, builds and backtests, and then you watch your bot think in a living brain: scanning, doubting, deciding, acting, and explaining every move. Start free with five credits a day. Upgrade when you're ready for more power. Bots are not magic and markets are risky, so we built the safeguards in: paper trading first, hard risk limits, and a kill switch that is always one click away."

---

## 11. Open decisions for the founder

1. Product name and brand.
2. Exact prices, plan limits and credit costs (section 5).
3. Confirm: daily refill tops up to 5 with no rollover.
4. Confirm: Live trading only on paid plans, after a minimum paper run.
5. Launch chain: Solana (default) vs also EVM/CEX.
6. Data providers and hosting.
7. Legal: region blocks, custody model, disclaimers, MiCA, **before Phase 6**.
