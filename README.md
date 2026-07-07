# mlops-rolling-signal-task0
# MLOps Task 0 — Rolling Mean Signal Batch Job

A minimal, reproducible batch job that reads OHLCV data, computes a rolling
mean on `close`, generates a binary trading signal, and writes structured
metrics + logs. Built to run identically on a local machine or inside Docker.

## What it does

1. Loads and validates `config.yaml` (`seed`, `window`, `version`)
2. Loads and validates `data.csv` (must contain a `close` column)
3. Computes a rolling mean of `close` over `window` rows
4. Generates `signal = 1` if `close > rolling_mean`, else `0`
   - The first `window - 1` rows have no rolling mean yet; they are left
     as `NaN` and **excluded** from the `signal_rate` calculation, since
     there isn't enough history to define a signal for them.
5. Writes `metrics.json` (success or error schema) and `run.log`
6. Prints the final metrics JSON to stdout and exits `0` (success) or
   non-zero (failure)

## Files

| File | Purpose |
|---|---|
| `run.py` | Main pipeline script |
| `config.yaml` | Config (`seed: 42`, `window: 5`, `version: "v1"`) |
| `data.csv` | Sample 10,000-row synthetic OHLCV dataset |
| `requirements.txt` | Python dependencies |
| `Dockerfile` | Container build definition |
| `metrics.json` | Sample output from a successful run |
| `run.log` | Sample log from a successful run |

## Run locally

```bash
pip install -r requirements.txt

python run.py --input data.csv --config config.yaml --output metrics.json --log-file run.log
```

No paths are hard-coded — `--input`, `--config`, `--output`, and
`--log-file` are all CLI arguments.

## Run with Docker

```bash
docker build -t mlops-task .
docker run --rm mlops-task
```

The container bundles `data.csv` and `config.yaml`, runs the pipeline on
build's default CMD, prints the metrics JSON to stdout, and writes
`metrics.json` / `run.log` inside the container. Exit code is `0` on
success and non-zero on failure (missing file, bad config, etc.).

To copy the output files out of a run:

```bash
docker create --name tmp-mlops mlops-task
docker cp tmp-mlops:/app/metrics.json ./metrics.json
docker cp tmp-mlops:/app/run.log ./run.log
docker rm tmp-mlops
```

## Example `metrics.json` (success)
