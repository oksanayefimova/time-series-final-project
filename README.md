# MATH840 Final Project: 48-hour electricity generation forecast

This project predicts hourly electricity generation for the next two days (27–28 October 2026, 48 hours), with a 95% prediction interval: a range the real value should fall inside 95% of the time.

## What is in this folder

| File | What it is |
|---|---|
| `train.csv` | Hourly generation (`y`) and observed weather, from 1 Jan 2023 to 27 Oct 2026 00:00 |
| `weather_forecast.csv` | Weather forecast for the 48 hours to predict (no generation values) |
| `final_notebook.ipynb` | The notebook that produces the submitted file |
| `YEFIMOVA_project.csv` | The submitted forecast: time, forecast, lower and upper bound |
| `scripts/` | Extra validation scripts I used while exploring. The final file does not come from them |
| `REPORT.md` | Project report with the results |

## How to run it

1. Install the packages: `pip install numpy pandas scikit-learn statsforecast lightgbm matplotlib`.
2. Put `train.csv` and `weather_forecast.csv` in the same folder as the notebook.
3. Run the notebook from top to bottom. It writes `YEFIMOVA_project.csv`.

**Do not run the very last cell** of the notebook. It was written for an earlier version and would overwrite the forecast with the benchmark.

## The model in plain words

- **Forecast:** a SARIMA model with weather as extra inputs (cloud cover, temperature, wind speed). It learns from the last 7 days of data.
- **Weather for the next two days:**
  - Day 1 uses the weather forecast.
  - Day 2 repeats the last known weather, because the forecast for day 2 is less reliable.
- **Interval:** I measured how wrong the model was on recent past periods, separately for day 1 and day 2, and took the range that covered 95% of those errors. The lower bound is never below zero.

## Use of AI tools

I used Claude (Anthropic) to write and debug code, to explain validation output, and to edit the text. I decided on the validation method, ran the experiments and chose the final model myself. I checked the numbers in the report against the notebook output.
