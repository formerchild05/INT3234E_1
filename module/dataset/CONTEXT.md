# Dataset context

## Routing and scope

- Updated: 2026-09-12.
- Dataset directory: `module/dataset`.
- Read this file at the start of every prompt, immediately after `.agents/KNOWLEDGE.md`.
- This is a compact working context. Inspect source CSVs again when exact values, code behavior, or changed files matter.
- This historical market bundle is for learning and experimentation, not a current market feed.

## Files and verified shape

- `prices.csv`: 851,264 rows, 7 columns, 501 symbols. Daily OHLCV from 2010-01-04 through 2016-12-30. Columns: `date`, `symbol`, `open`, `close`, `low`, `high`, `volume`. Sampled dates include a midnight timestamp.
- `prices-split-adjusted.csv`: 851,264 rows, the same columns, symbols, and date range. Prefer it for returns, volatility, forecasting, and most ML exercises so stock-split jumps are not mistaken for market moves.
- `securities.csv`: 505 rows, 8 columns: `Ticker symbol`, `Security`, `SEC filings`, `GICS Sector`, `GICS Sub Industry`, `Address of Headquarters`, `Date first added`, `CIK`.
- `fundamentals.csv`: 1,781 rows, 79 columns, 448 symbols. It contains annual statements, ratios, EPS, fiscal year, and estimated shares. `Period Ending` ranges from 2003-06-30 through 2017-01-01. Drop the first unnamed exported-index column by default.
- `NY-stock-exchange.zip`: the 32,148,316-byte source archive. Use extracted CSVs; never count archive and extracted copies as separate data.

## Grain, keys, and joins

- Prices: one observed trading date per symbol. Candidate key: (`symbol`, `date`). Sort on both before lags, rolling features, or future labels.
- Securities: one security per `Ticker symbol`. Candidate key: `Ticker symbol`.
- Fundamentals: approximately one fiscal period per company. Candidate key to validate: (`Ticker Symbol`, `Period Ending`).
- Join names differ across files. Normalize `symbol`, `Ticker symbol`, and `Ticker Symbol` explicitly.
- Price-to-securities is normally many-to-one. Validate metadata-key uniqueness and unmatched tickers.
- Fundamentals-to-prices requires a temporal/as-of join. `Period Ending` is not the public filing/release date. Assuming the data was available then creates look-ahead leakage. Use a real availability date or a documented conservative lag; otherwise omit fundamentals from leakage-sensitive prediction.
- Securities resembles a constituent snapshot whereas prices cover years. Do not assume point-in-time index membership or complete histories; survivorship bias is possible.

## Semantics and quality

- OHLC means open, high, low, close; `volume` is traded share volume. Raw and split-adjusted price tables are alternate views of the same observations, not extra samples.
- GICS sector/sub-industry are nominal categories. Use one-hot or another nominal encoding. Load `CIK` as text to preserve leading zeros; tickers are identifiers.
- Fundamentals lack a formal unit/currency dictionary. Some ratios appear percentage-scaled: the first AAL row has `Current Ratio = 78`, while assets/liabilities is about 0.785. Confirm each ratio's scale before interpreting it.
- Negative accounting values can be valid, including losses and cash outflows. Zero is not automatically missing.
- A prior audit found no blank/common NA markers in either price CSV. This does not prove unique keys, full histories, or valid OHLC values.
- Fundamentals has 1,508 missing cells across 482 rows: `Cash Ratio` 299, `Current Ratio` 299, `Quick Ratio` 299, `Earnings Per Share` 219, `Estimated Shares Outstanding` 219, `For Year` 173.
- Securities has 198 missing `Date first added` values. This does not prove absence from the index.
- Validate duplicate candidate keys, parsing, positive prices, nonnegative volume, and `low <= open/close <= high`. Check gaps against trading days; weekends and holidays are not missing rows.
- Validate accounting identities approximately with tolerance because reporting, restatement, rounding, and sign conventions may differ.

## Default analysis choices

- Start price work from `prices-split-adjusted.csv`; parse dates, sort by (`symbol`, `date`), and calculate time features within symbol.
- Prefer returns/log returns over pooled price levels. Prefer relative/log volume or deviation from a per-symbol rolling baseline.
- Crashes, rallies, and volume spikes are not automatically errors. Detect per symbol, inspect, and correct only confirmed errors; otherwise retain/flag or apply a justified robust transform.
- Split chronologically before preprocessing. A useful teaching split is 2010-2014 train, 2015 validation, 2016 test.
- Fit imputation, scaling, feature selection, PCA, and resampling only on training data. Apply SMOTE/undersampling inside training folds only.
- For next-day prediction, features at date `t` use information available by `t`; derive the label from the next observed row for the same symbol and drop each symbol's final targetless row.
- One proposed rare-event label is next-observation adjusted return <= -5%. A prior scan found 850,763 returns and about 0.8774% positives. Prefer precision, recall, F1, PR-AUC, balanced accuracy, and confusion matrices over ordinary accuracy.
- Regression may target next-period return or future realized volatility. Always compare simple baselines and do not equate metrics with profitable trading after costs.

## Recheck when relevant

- Recompute duplicate counts, ticker overlap, per-symbol coverage, OHLC violations, archive contents/checksums, delisted-name handling, and exact adjustment conventions when a task depends on them.
- The bundle does not provide trustworthy fundamentals release timestamps, corporate actions, point-in-time membership, or transaction costs. State these limitations when they affect conclusions.


## Recommended teaching subset (verified 2026-09-13)

- For Lesson 2 EDA/preparation, use AAPL (Information Technology), AMZN (Consumer Discretionary), JPM (Financials), MRK (Health Care), XOM (Energy), KO (Consumer Staples), BA (Industrials), and NEE (Utilities).
- Each has 1,762 adjusted-price rows spanning 2010-01-04 through 2016-12-30 and four fundamentals rows. Missing cells across all fundamentals fields: AAPL 0, AMZN 3, JPM 12, MRK 0, XOM 0, KO 0, BA 3, NEE 0. JPM's missing ratios are useful for discussing structural missingness in financial-sector statements.
- A smaller five-symbol subset is AAPL, JPM, MRK, XOM, and KO. Use one or two symbols for detailed time-series plots and the full subset for sector comparisons.

## Assignment notebook review (verified 2026-09-13)

- `module/les2/asg/asignment2.ipynb` has 13 cells. It loads the three CSVs, selects the eight-symbol subset, computes per-symbol returns before filtering 2016, plots returns and a boxplot, and flags 2016 Z-score outliers.
- A read-only check found 14,096 selected price rows, zero duplicate (`symbol`,`date`) keys, zero invalid OHLCV rows, and eight expected return NaNs from the first row of each symbol. In 2016, IQR flags 9-27 rows per symbol while absolute Z-score above 3 flags 2-6, showing method sensitivity. Missing fundamentals cells in the subset are AAPL 0, AMZN 3, JPM 12, MRK 0, XOM 0, KO 0, BA 3, NEE 0.

- Normality clarification for the current notebook: 2016 per-symbol histograms look roughly bell-shaped in the center, but excess kurtosis ranges from 2.874 to 15.723 and observed absolute Z-score above 3 rates range from 0.794% to 2.381%, versus about 0.270% under a normal distribution. The returns are therefore heavy-tailed despite the visual center. IQR 1.5 bounds are also about +/-2.698 sigma under exact normality, so they are inherently tighter than a +/-3 sigma rule; this, plus robust quartile scaling and heavy tails, explains more IQR flags.

- Current notebook now has 16 cells and includes actual per-symbol IQR flags, Z-score flags, their OR union, and a many-to-one securities-sector merge. Remaining priorities: missing report and cause-specific handling; duplicate/OHLC validation cells; outlier overlap and time/volume context; feature transformations such as intraday range, log/relative volume and rolling volatility; categorical encoding/reduction if required by Lesson 2; optional imbalance distribution without model training; Markdown interpretation and a final before/after data-quality summary. Sector output represents one selected company per sector, not sector-wide inference.
