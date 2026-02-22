# LookThroughProfits Supabase Data Pipeline (US Only)

Loads US daily prices and financial statements into a Supabase (Postgres) database using SimFin bulk datasets.

## Features

- US annual financial statements via SimFin bulk into normalized `financials`
- US daily prices via SimFin bulk into `stock_prices`
- Insert-if-newer policy for prices (no upserts)
- Batched inserts for performance (`execute_values`)
- `.env`-based configuration for DB and SimFin settings

## Repository structure

- `Orchestrator.py` — Runs the US pipeline end-to-end
- `ingestion/`
  - `ingest_simfin_financials_api_to_postgres_us.py` — US financials ingest
  - `ingest_simfin_prices_us.py` — US prices ingest
- `sql/`
  - `create_tables.sql` — `financials` schema
  - `create_tables_stockprice.sql` — `stock_prices` schema
  - `create_tables_ingest_logs.sql` — `ingest_logs` schema
- `utils/`
  - `logger.py` — Logging setup

## Setup

1. Create a virtual environment (PowerShell):

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

2. Install dependencies:

```powershell
python -m pip install -r requirements.txt
```

3. Create `.env` at repo root.

Required:

- `DB_HOST`
- `DB_PORT` (default `5432`)
- `DB_NAME`
- `DB_USER`
- `DB_PASSWORD`
- `SIMFIN_API_KEY`

Common optional:

- `SIMFIN_MARKET` (default `us`)
- `SIMFIN_PRICES_VARIANT` (`latest` or `daily`)
- `SIMFIN_INSERT_BATCH` (default `5000`)
- `PRICES_INSERT_BATCH` (default `1000`)
- `CLEAR_STOCK_PRICES` (`true`/`false`)
- `LOG_LEVEL`

## Usage

Run full US pipeline:

```powershell
python Orchestrator.py
```

Run individual steps:

```powershell
python ingestion/ingest_simfin_financials_api_to_postgres_us.py
python ingestion/ingest_simfin_prices_us.py
```

## Database schema

Apply SQL in `sql/` before first run:

- `create_tables.sql`
- `create_tables_stockprice.sql`
- `create_tables_ingest_logs.sql`

## Notes

- The orchestrator truncates `financials` and `stock_prices` at startup.
- SimFin steps are skipped if `SIMFIN_API_KEY` is missing.
