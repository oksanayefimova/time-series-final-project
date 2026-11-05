# Project report: 48-hour electricity generation forecast

## 1. The task

Predict hourly electricity generation for 27 October 2026 01:00 to 29 October 2026 00:00, with a 95% interval. The score combines three numbers: RMSE (average error size), MAPE (average error in percent) and the Winkler score, which rewards narrow intervals that still contain the real value.

The baseline ("benchmark") is the seasonal naive forecast: copy the same hour from the last day of data.

## 2. What I tried

I compared several model types on past periods. Each test used 48-hour windows that the model had not seen.

- Models without weather: different SARIMA training lengths, Ridge regression, LightGBM.
- Models with weather: the weather forecast for day 1, and one of three ways to handle day 2.

One lesson shaped everything: if a model uses the **actual** weather in a test, it looks better than it really is, because in the real test we only have a forecast. So every comparison in this report uses forecast weather, or a simulation of forecast errors.

## 3. The final model

- **Model:** SARIMA(3,0,0)(0,1,3) with a daily season of 24 hours, with cloud cover, temperature and wind speed as extra inputs.
- **Training:** the last 7 days of data.
- **Weather:** the forecast for day 1; the last known weather for day 2.
- **Interval:** based on past errors, computed separately for day 1 and day 2 and never below zero.

## 4. Validation results

| Model | Test windows | RMSE | Winkler | Coverage (target 0.95) |
|---|---|---|---|---|
| Benchmark (seasonal naive) | 6 | 267.5 | 1564 | 0.944 |
| Final model | 6 | 225.0 | 990 | 0.955 |
| Final model, 5 most recent windows | 5 | 214.6 | 970 | 0.962 |
| Final model, 5 most recent, with stronger weather error | 5 | 217.7 | 972 | 0.962 |
| Benchmark, 5 most recent windows | 5 | 243.2 | 1181 | 0.979 |

The final model beats the benchmark on all of these. I kept the version that stayed stable when the weather forecast was made less accurate. A version that forecast day 2 weather directly was slightly better when the forecast was perfect, but got worse than the benchmark once I added realistic errors, so I rejected it.

The number of test windows is small (6), so some of these differences could be chance.

## 5. Known weaknesses

- **Midday on day 2.** The model predicts too low. Its average error there is about 278 MW, against 195 MW for the benchmark. This is the main place the model can lose.
- **Night on day 2.** Generation rose sharply at night, and neither model predicted it. The weather does not explain that rise.
- **The most recent window.** The benchmark was better there (120 MW error against 139 MW). The test week looks similar to that window, so this risk is real.

## 6. Leaderboard results

| Period | RMSE | MAPE | Winkler | Coverage | Score |
|---|---|---|---|---|---|
| My model, first 24 hours (day 1) | 83.1 | 53.2% | 529 | 100% | 11.8 |
| Benchmark, same period | 115.7 | 84.3% | 521 | 100% | 14.6 |

On day 1 my model beats the benchmark on RMSE, MAPE and the overall score (7th place out of 31 at the time). It is slightly worse on Winkler. Coverage of 100% means the intervals are wider than needed for this week.

Day 2 results: [to be added after the update on 6 November]

## 7. Conclusions

The model is better than the benchmark on day 1 and on most of the test windows. Day 2, especially midday, is its weak point. The 100% coverage on this week suggests the intervals could be narrower, but one week is not enough evidence to tune them to it.

## 8. What I would do next

1. Narrow the intervals using only the 5 most recent windows, as long as coverage on them stays near 95%.
2. Blend day 2 with the benchmark (about half and half) if the tests show lower midday error.
3. Test on more windows to make the choice between models more reliable.
