# Primary stack

- C++20 (engine hot path)

- CMake (build)

- Catch2 or GoogleTest (unit tests)

- CLI11 (CLI args) or cxxopts

- fmt + spdlog (logging; keep logging off hot-path)

- nlohmann/json (configs + small datasets) or yaml-cpp (YAML configs)

- Python 3.10+ (analysis scripts + plotting + run orchestration)

- pandas, matplotlib (plots), optional numpy

# Data/storage

- Local CSV/Parquet outputs initially

- Postgres/TimescaleDB for runs/metrics (integrating with ZetaPulse)

# Deploy

Docker 

GitHub Actions

# Zetaforge — Deterministic Market Replay & Execution Lab

Zetaforge is a production-style **trading-systems portfolio project** focused on what quant dev / low-latency engineering interviews actually test:

- **Market data ingestion + normalization**
- **Deterministic replay + exchange simulation**
- **Order lifecycle + risk gates (OMS/order gateway state machine)**
- **Stage-level latency measurement (p50 / p99 / p99.9)**
- **Research → production bridge (Python → C++ config / plugins)**

> **Scope note:** Zetaforge is a *research + simulation* environment. It is designed to be correct, measurable, and reproducible. It is not presented as a live trading system managing real capital.

---

## Architecture (high level)

```
[Live / Recorded Market Data]
          |
          v
   Ingest + Normalize  --->  Persistent event log (optional)
          |
          v
   Deterministic Replay Engine
          |
          v
  Exchange Simulator (matching, cancels, partial fills, latency injection)
          |
          v
 Strategy Plugin / Zeta Module (C++ runtime, Python research configs)
          |
          v
 Risk Gate (limits, price bands, throttle, kill-switch)
          |
          v
 Order Gateway (state machine)  --->  Metrics/Tracing/Benchmarks
```

### Design goals
- **Determinism:** same inputs → same outputs (seeded RNG for stochastic models)
- **Hot-path discipline:** predictable control flow; avoid heap allocations on critical path where practical
- **Reproducible performance:** pinned cores, warmups, stable configs, recorded environments
- **Testability:** golden scenarios for book state, fills, order transitions

---

## Features

### Market data
- Live ingest (start with crypto WebSocket feeds) and/or file replay
- Normalized internal event format (ticks/trades/book deltas)
- In-memory limit order book with validation hooks

### Replay + Simulation
- Deterministic event replay (timestamped events, controlled clock)
- Exchange simulator:
  - price-time priority matching (baseline)
  - partial fills + cancels + replace
  - configurable latency injection (network + venue delay model)
  - optional queue-position approximation for maker strategies

### Execution framework
- Order gateway with explicit state machine:
  - NEW → ACK → PARTIAL → FILLED / CANCELED / REJECTED
- Risk gate:
  - max order rate (token bucket)
  - max notional / position limits
  - price bands + fat-finger checks
  - kill switch

### Measurement & observability
- Stage-level timing (ingest → normalize → book → signal → risk → send)
- Latency histograms (p50 / p95 / p99 / p99.9)
- CSV/JSON export for dashboards
- Optional tracing spans (OpenTelemetry-style)

---

## Quickstart

### Requirements
- Linux recommended (macOS works for most components)
- C++20 compiler (clang/gcc)
- Python 3.10+
- Optional: Docker + docker-compose for the full stack

### Build (C++)
```bash
mkdir -p build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . -j
```

### Run a deterministic replay
```bash
./Zetaforge_replay --input data/sample_day.jsonl --seed 42 --speed 10x
```

### Run simulator + execution loop
```bash
./Zetaforge_trade --config configs/demo_strategy.yaml --seed 42
```

### Export benchmark results
```bash
./Zetaforge_bench --config configs/bench.yaml --out out/bench.csv
```

---

## Benchmarks (publish these on your website)

Fill this table after you run on your machine:

| Scenario | Throughput (msgs/sec) | p50 | p99 | p99.9 | Notes |
|---|---:|---:|---:|---:|---|
| Replay → Book update | TBD | TBD | TBD | TBD | pinned core, warmup 10s |
| Sim → OMS roundtrip | TBD | TBD | TBD | TBD | includes risk + state machine |
| End-to-end (replay→fill) | TBD | TBD | TBD | TBD | includes simulator latency model |

**Methodology checklist**
- CPU pinning / affinity
- warm-up period
- fixed input dataset + seed
- release build flags
- record machine + kernel version

---

## Suggested repo layout

```
Zetaforge/
  cpp/
    ingest/   book/   sim/   oms/   risk/   bench/
  python/
    research/ backtest/ features/
  configs/
  data/               # small sample datasets only
  docs/
    design.md
    benchmarks.md
    architecture.png
  out/
  docker/
  README.md
```

---

## Roadmap (credible next steps)
- Add an ITCH-style equities replay (or futures incremental feed) after crypto baseline
- Improve queue-position model for maker strategies
- Add kernel-bypass ingest experiment (AF_XDP) behind a feature flag
- Continuous benchmark regression via CI
- Drift monitoring + PnL attribution panels in MarketOps dashboard

---

## License
MIT (or your preferred license).
