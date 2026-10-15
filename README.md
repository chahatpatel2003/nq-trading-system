# NQ Research Platform (`nqlab`): Survival-First Strategy Research for Prop-Firm Futures

A quantitative research laboratory for **NQ / MNQ futures**. It discovers, validates, ranks and stress-tests intraday strategies against a realistic **Topstep prop-firm account model**. The objective is survival and payouts, not raw backtest return. This is a research platform, not a trading bot: it has no live execution.

The repository contains the platform and **59 documented research phases**. Most phases ended in a rejection, and each one says why.

| | |
|---|---|
| **Instrument** | NQ E-mini (MNQ sizing via point value); GC/MGC comparison in phases 57–59 |
| **Data** | Databento NQ trades and MBP-1, 2021 → 2026 (licensed; not included) |
| **Stack** | Python 3.12, uv, Polars, PyArrow, DuckDB, Pydantic, Typer, Hypothesis (property tests), pytest, ruff, mypy `--strict` |
| **Scale** | ~37k lines of Python (233 modules) · 59 phase reports · 129 trade-replay and finalist charts |
| **Tests** | 308 pytest tests (incl. Hypothesis property tests), all passing |

---

## What the platform does

- **Bars engine.** Trades become session-labelled OHLCV bars. Continuous contracts are volume-rolled, and every derived layer is point-in-time safe.
- **Feature store.** Point-in-time-safe features, versioned and reproducible.
- **Strategy contract.** Each strategy is a declarative, versioned object, so a backtest cannot see future data.
- **Event-driven backtester.**
  - Stops, targets, time stops and end-of-day flattening.
  - NQ → MNQ sizing.
  - An explicit cost model: commissions, spread and slippage, with a harsh-fill stress mode.
- **Topstep account simulator.**
  - 50K / 100K / 150K combines and funded accounts.
  - Trailing maximum loss limit and daily loss limits.
  - Consistency rules, payout paths and fee accounting.
  - Monte-Carlo lifecycle simulation: probability of passing, probability of payout, blow-up rate, and expected value per month.
- **Validation gauntlet.**
  - Pre-registered lockboxes, walk-forward and out-of-sample splits.
  - Blind tournaments.
  - Statistical audits that try to kill earlier results (phases 27, 34A and 58).
- **CLI.** `nqlab data | bars | features | backtest | research | topstep …` runs the whole loop. See [TECHNICAL_README.md](TECHNICAL_README.md).

The full blueprint is in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md): architecture, invariants, dependency graph, storage and validation design. The data ground truth is in [docs/DATA_PROFILE.md](docs/DATA_PROFILE.md).

## Research findings (phases 7–59)

Every phase is a written report in [`docs/`](docs/), each ending in an explicit verdict. Highlights:

| Phases | Question | Verdict |
|---|---|---|
| 7–11 | Survival-optimise the positive out-of-sample setup families; confirm them in a lockbox | **No.** 0 of 165 candidates accepted (P7); 0 of 1,161 feasible across account × timeframe (P8); the lockbox confirmed nothing (P10, P11) |
| 12–21 | Frequency expansion, structure/liquidity/displacement, payout-first and high-win-rate searches, classic indicators, MBP-1 order flow | **All rejected.** 105 classic-indicator candidates and 24 order-flow candidates failed validation |
| 22–33 | Exact cost audit, higher-timeframe POI / lower-timeframe reclaim models, daily bias, high-profit-factor search, integrity audit, volume profile, micro-scalping | **Rejected.** P27 confirmed a process flaw did not change any conclusion. P33: micro-scalping has no edge and costs too much |
| 34A | Independent stats audit of the phase 30–33 numbers | **STATS_VERIFIED** |
| 37–40 | A simple daily workhorse that trades most sessions | **P40: PASS for forward-paper, with conditions.** Out-of-sample PF 1.17–1.20, Monte-Carlo payout/blow ratio 1.6–3.0. Flags: a losing 2023 and ~50 % of profit from one sleeve |
| 42–55 | Turn the frozen strategy into a Topstep pass-and-payout plan | **Conditional yes on 150K.** Blind anti-bias tournaments (P49–P50) kept the benchmark. Fast or aggressive extraction plans were abandoned (P47, P55) |
| 56 | Can any strategy earn ≥ $1,000 per month per account? | **No.** The nominal winners were a lottery: median payout $0, 73 % blow-ups, and expected value halved out of sample. The robust ceiling is **~$400 per month per account** |
| 57–59 | Gold (GC/MGC) instead of NQ; engine bug audit; overnight gold | **GC fails.** P58 found the engine sound and confirmed NQ's best strategy (PF 1.22). Overnight gold is long beta, not an edge |

The most useful outcome is the honest ceiling. A survivable, out-of-sample-valid strategy exists, but the prop-firm wrapper economics cap it far below the income target. Every configuration claiming more depended on sizing whose survival rate was 3–30 %.

## Repository layout

```
├── TECHNICAL_README.md     original project README: CLI, setup, development commands
├── Makefile                install / test / lint / type / check
├── config/                 backtest, forward-paper, instruments, profiles, risk, roll, sessions, validation
├── src/nqlab/              the platform: data, domain, features, backtest, research, discovery, validation, paper, experiment, cli
├── scripts/                phase drivers (phaseNN*.py) and utilities
├── tests/unit/             unit tests
└── docs/                   ARCHITECTURE, DATA_PROFILE, ROADMAP, PHASE7…PHASE56 reports
    └── phaseNN/            per-phase tables, finalist and trade-replay charts, frozen configurations
```

## Running it

```bash
git clone https://github.com/chahatpatel2003/nq-trading-system.git
cd nq-trading-system
make install      # uv venv (Python 3.12) + editable install
make check        # ruff + mypy --strict + pytest
```

`uv.lock` pins every dependency. **Data is not included:** raw Databento files are licensed. The platform reads them from a configurable read-only `raw/` root and writes derived layers to a separate `curated_root`. Large intermediate artefacts (bars, features, full trade logs) remain private.

## Third-party material

- **Market data:** Databento NQ trades and MBP-1, and GC bars (licensed; not included).
- **Topstep:** account rules are summarised and modelled from Topstep's public documentation. Topstep and TopstepX are trademarks of their owner; this project is not affiliated with Topstep.
- **Trading concepts:** several phases test publicly taught ideas (ICT-style higher-timeframe points of interest, fair-value gaps, market-structure shifts, volume profile). The concepts belong to their originators and are implemented here independently.
- **Open-source libraries** are installed as dependencies under their own licences; none are copied into this repository.

## Notes

- The folder name on the original machine was `TRADING SYSYEM NQ PART 1`. This repository was assembled from it on 2026-10-14. File contents are byte-identical to that snapshot (SHA-256 verified), except 8 files where a hardcoded local path was made portable without changing any logic: 2 documents, and 6 phase 57–58 scripts that now read GC data from `~/Desktop/gc` and keep their scratch files in `$NQLAB_SCRATCH` (default: the system temp folder).
- Research software only. Nothing here is investment advice.

© Chahat Patel. All rights reserved. This notice covers the original code, documentation and research results in this repository. It does not extend to the third-party material listed above, which belongs to its respective owners. No license to reuse this code is granted; it is published for review.
