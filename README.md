# Financial Data Pipeline (US Only)

Simple US pipeline using SimFin + Postgres.

## What it does

- Loads US financial statements into `financials`
- Loads US prices into `stock_prices`

## Run it

```powershell
python Orchestrator.py
```

## What happens when `Orchestrator.py` runs

1. Connects to Postgres using `.env`
2. Clears `financials` and `stock_prices`
3. Runs `ingestion/ingest_simfin_financials_api_to_postgres_us.py`
4. Runs `ingestion/ingest_simfin_prices_us.py`
5. Stops on critical failures and logs results

## Required `.env` values

- `DB_HOST`
- `DB_PORT`
- `DB_NAME`
- `DB_USER`
- `DB_PASSWORD`
- `SIMFIN_API_KEY`

## Optional `.env` values

- `SIMFIN_MARKET` (default `us`)
- `SIMFIN_PRICES_VARIANT` (`latest` or `daily`)
- `SIMFIN_INSERT_BATCH`
- `PRICES_INSERT_BATCH`
- `ORCH_MAX_ERRORS` (default `3`)
- `LOG_LEVEL`

## Install

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```
