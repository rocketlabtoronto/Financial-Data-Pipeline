## Purpose
Short, focused guide for AI coding agents to be immediately productive in this repository.

## Big picture (read these files first)
- `Orchestrator.py` — the US-only end-to-end runner (US financials → US prices).
- `ingestion/` — ingest scripts (SimFin bulk for US):
  - `ingest_simfin_financials_api_to_postgres_us.py` (SimFin bulk → normalized `financials`)
  - `ingest_simfin_prices_us.py` (SimFin shareprices → `stock_prices`)
- `utils/` — helpers: `logger.py` (logging setup).
- `sql/` — authoritative DB schema; ingestion code relies on specific column names (see `create_tables*.sql`).

## Developer workflows and commands
- Create venv and install deps (PowerShell):
  `python -m venv venv; .\venv\Scripts\Activate.ps1; python -m pip install -r requirements.txt`
- One-command end-to-end: `python Orchestrator.py`
- Run ingesters individually (advanced):
  - `python ingestion/ingest_simfin_financials_api_to_postgres_us.py`
  - `python ingestion/ingest_simfin_prices_us.py`
- Environment: use a `.env` at repo root. Required DB vars: `DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD`.
  SimFin: `SIMFIN_API_KEY` (ingesters skip SimFin steps when missing). Common optional vars: `SIMFIN_MARKET, SIMFIN_PRICES_VARIANT, PRICES_INSERT_BATCH, SIMFIN_INSERT_BATCH, CLEAR_STOCK_PRICES, LOG_LEVEL`.

## Project-specific conventions & patterns (important to preserve)
- SimFin bulk API: ingestion scripts expect a ZIP containing a semicolon-delimited CSV. See `fetch_bulk_dataset()` in SimFin ingesters.
- Insert-only policy for prices: scripts do NOT perform upserts; they insert only when incoming row is newer than stored data. The gating uses `as_of` (timestamp) if present; otherwise `latest_day` (date).
- Batch inserts: ingestion uses `psycopg2.extras.execute_values` for performance. Batch sizes controlled by env `SIMFIN_INSERT_BATCH` and `PRICES_INSERT_BATCH`.
- Tag maps: the financials ingester uses tag → candidate column lists (INCOME_TAGS, BALANCE_TAGS, CASHFLOW_TAGS). Changes to these maps directly change what gets persisted.
- Logging and telemetry: ingesters write an `ingest_logs` entry at end/start/failure. Use these rows to summarize runs.

## Integration points & external dependencies
- SimFin (bulk ZIP API) — used for US financials and prices. Requires `SIMFIN_API_KEY`.
- Postgres / Supabase — authoritative sink. The SQL in `sql/` defines table names/columns the code expects (`financials`, `stock_prices`, `ingest_logs`).

## Debugging and modification guidance
- Use `LOG_LEVEL` to surface more logs; `utils.get_logger()` centralizes logging config.
- When changing DB-related code, first inspect `sql/create_tables*.sql` to ensure column names (especially `as_of`, `latest_day`, `previous_close`) are preserved.
- SimFin failures: scripts log errors and exit non-zero. Orchestrator skips SimFin steps if `SIMFIN_API_KEY` is not set.
- To test incremental price gating, check `table_has_column(conn, 'stock_prices', 'as_of')` logic in `ingest_simfin_prices_us.py` — this controls timestamp vs date gating.

## When you touch ingestion code — checklist
1. Keep `SIMFIN_*` env semantics intact (don't rename without updating `Orchestrator.py` and `README.md`).
2. Preserve tag maps shape (`dict[tag] -> list[candidate_columns]`) or update documentation & tests.
3. Maintain insert-only semantics for `stock_prices` unless you update SQL and tests.
4. Update `ingest_logs` entries so operational dashboards remain meaningful.

## Files to inspect for examples
- `Orchestrator.py` (process orchestration, error handling, PYTHONPATH injection)
- `ingestion/ingest_simfin_financials_api_to_postgres_us.py` (tag mapping, batch insert pattern)
- `ingestion/ingest_simfin_prices_us.py` (bulk CSV -> filter -> batch insert; gating logic)
- `utils/logger.py`
- `sql/*.sql` (DB schema expectations)

If anything above is unclear or you'd like more examples (small unit tests, a smoke test runner, or detailed DB column references), ask and I can expand.
