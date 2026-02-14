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