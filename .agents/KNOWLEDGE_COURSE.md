# Course and stock-dataset knowledge

## Course and objective

- Updated: 2026-09-12.
- Syllabus: Lesson 2 EDA and preparation; Lesson 3 PCA/RFECV; Lesson 4 linear, polynomial, and logistic regression; Lesson 5 ARIMA, MA, and exponential smoothing; Lesson 6 Bayes, trees, neural networks, SVM, and ensembles.
- The user wants to learn financial-data concepts gradually with the stock bundle in `module/dataset`.
- Current focus: apply Lesson 2 data exploration, missing-value handling, outlier handling, and class-imbalance techniques. No implementation has yet been requested.
- The user previously preferred data from about 2018 to present but accepted this older dataset for learning.

## Verified dataset

- `securities.csv`: 505 rows and 8 columns, containing ticker, company, GICS sector/sub-industry, headquarters, date first added, and identifiers.
- `prices.csv`: 851,264 rows and 501 symbols, daily OHLCV from 2010-01-04 through 2016-12-30.
- `prices-split-adjusted.csv`: same row, symbol, and date counts, with split-adjusted prices. Prefer this file for return analysis.
- `fundamentals.csv`: 1,781 rows, 448 symbols, and 79 columns including an unnamed exported index. It contains annual income-statement, balance-sheet, cash-flow, ratio, fiscal-year, EPS, and estimated-share fields. Period Ending ranges from 2003-06-30 through 2017-01-01.
- Period Ending is not the public release date. Fundamentals can cause look-ahead leakage if merged into predictive data as though they were known on that date.
- Financial values need sign and unit checks. Cash outflows may be negative. Ratio fields appear percentage-scaled in some rows: the first AAL Current Ratio is 78 while Total Current Assets divided by Total Current Liabilities is about 0.785.

## Missing-value audit

- The audit counted blank cells and common NA/NaN/null markers; zero was not treated as missing.
- `fundamentals.csv`: 482 of 1,781 rows contain missing cells, with 1,508 missing cells total. Counts are Cash Ratio 299, Current Ratio 299, Quick Ratio 299, Earnings Per Share 219, Estimated Shares Outstanding 219, and For Year 173.
- `securities.csv`: 198 of 505 rows lack Date first added.
- Both price files have no missing cells. This does not prove that every expected trading observation exists or that there are no duplicates or invalid values.
- Do not treat weekends and market holidays as missing trading days.
- Possible treatments: derive For Year from Period Ending; recompute ratios when denominators are valid; add missingness indicators; compare sector/year median, KNN, and MICE imputation. Do not casually interpolate annual financial statements. Fit imputers on training data only.

## Lesson 2 mapping and proposed exercise

- `module/les2/Lesson-2-DATA EXPLORATION & DATA PREPARATION.pdf` has 31 pages.
- Covered topics: data types and distributions; cleaning, integration, transformation, reduction, discretization, and splitting; categorical encoding; useless/constant fields; Z-score, IQR, visualization, and LightGBM outlier replacement; SMOTE and undersampling; MCAR/MAR/MNAR; deletion, constant/mean/median/mode, ffill/bfill/interpolation, KNN, and MICE imputation.
- Recommended exercise: predict whether a stock falls at least 5 percent on the next trading observation. The source files have no class label, so imbalance handling becomes meaningful only after defining a classification target.
- A local scan of adjusted closes sorted by symbol/date found 850,763 valid daily returns. Rates were 3.6354 percent for returns <= -3 percent, 0.8774 percent for <= -5 percent, and 0.0834 percent for <= -10 percent. The -5 percent threshold gives a usable rare class.
- Use adjusted prices for market features, securities for sector encoding, and fundamentals for the missing-data exercise. Do not merge raw and adjusted price files as separate observations.
- Candidate features: lagged returns, rolling volatility, intraday range, relative or log volume, and one-hot GICS sector. Use label encoding only for ordinal categories; use one-hot encoding for sector and sub-industry.
- Detect outliers per symbol using return and relative volume, not pooled raw prices. Start with IQR or Z-score and plot flagged points. Real crashes, rallies, and volume spikes are not automatically errors. Correct confirmed errors; retain and flag real events; compare robust scaling or winsorization if needed.
- Validate `low <= open, close <= high`, positive prices, nonnegative volume, duplicate `(symbol,date)` rows, and approximate accounting identities.
- Drop the unnamed fundamentals index and only genuinely useless or constant fields. Start with 10-15 interpretable features. Weekly/monthly aggregation and return or volatility buckets can demonstrate aggregation and discretization.
- Split chronologically before preprocessing, for example 2010-2014 train, 2015 validation, and 2016 test. Features use information at time t; the label uses t+1.
- Fit imputation, scaling, feature selection, and resampling on training data only. Apply SMOTE or undersampling only within training folds. Compare them with no resampling and class weights.
- Evaluate the rare class using precision, recall, F1, PR-AUC, balanced accuracy, and a confusion matrix. Ordinary accuracy is misleading because predicting the majority class can exceed 99 percent.

## Learning direction

- Recommended order: identifiers and row granularity; OHLCV and splits; returns and volatility; missing/duplicate/consistency checks; outliers; financial statements and ratios; safe joins; transformation and feature engineering; chronological splitting; then PCA, regression, ARIMA, and other models.
- Regression can target next-period return; classification can target direction or rare sharp decline. Exclude future-derived features and compare with simple baselines. Correlation does not establish causality, and model accuracy does not guarantee trading profit.

## Current outlier-context EDA (2026-09-13)

- For the eight selected symbols in 2016, using per-symbol IQR OR absolute Z-score above 3 produced 125 return-outlier days. Of these, 38 (30.40%) had relative volume at least 2, compared with 31 of 1,891 (1.64%) non-outlier days. Mean relative volume was 1.851 on outlier days and 0.958 otherwise; Spearman correlation between absolute return and relative volume was about 0.399. This supports association, not causation.
- The relative-volume scatter plot is for contextual diagnosis: x is current volume divided by the prior-20-session mean, y is absolute daily return, color is the return-outlier flag, and x = 2 marks volume spikes. Since the flag is derived from return and y is absolute return, the plot does not independently validate the detector.

## Imbalance preparation check (2026-09-13)

- For the eight-symbol subset, a next-observation return <= -5% target has 61 positives among 14,088 valid rows (0.433%, about 1:230). Chronological partitions have 44 positives in 2010-2014, 6 in 2015, and 11 in 2016; the 2016 rate is 11/2,008 = 0.548%. Current EDA scope should create and report the target but leave validation/test untouched; SMOTE or undersampling belongs only to later numeric training matrices or walk-forward training folds. Continuous price/return regression has no class imbalance and should not use SMOTE.

- For the same eight-symbol subset, a <= -3% next-return label gives 317/14,088 positives (2.250%), which is more stable for a classroom resampling demonstration than the <= -5% label, though the threshold must be justified by the task rather than chosen only to improve balance.

## Next-day direction balance check (2026-09-13)

- For the eight-symbol subset, defining 1 as next return > 0 and 0 as next return <= 0 yields 7,253 up rows (51.484%) and 6,835 non-up rows (48.516%) among 14,088 valid targets. There are 83 exactly-zero next returns inside class 0. For 2016 the split is 1,070 up (53.287%) versus 938 non-up (46.713%). This target is sufficiently balanced, so SMOTE/undersampling is not justified; use the rare sharp-drop target if the assignment needs a meaningful imbalance-handling demonstration.

## Lesson 3 cleaning notebook (2026-09-14)

- `module/les3/asg/asignment3.ipynb` is now scoped to AMZN only and contains a cleanly structured, cleaning-only pipeline for the full 2010-2016 history. It does not filter 2016 or create returns, rolling indicators, targets, scaling, RFECV, or PCA.
- It loads adjusted prices, fundamentals, and securities; validates price missingness, duplicate keys, positive prices/nonnegative volume, and OHLC ordering; prepares AMZN metadata; and handles AMZN's three missing fundamentals with audit flags.
- A full sequential run passed: `prices_clean` is 1,762 x 7, `metadata_clean` is 1 x 4, and `fund_clean` is 4 x 81. All three outputs have zero duplicate keys and zero remaining missing cells. AMZN's missing For Year, estimated shares, and EPS in the latest row were filled by the documented rules. A scan found no stale ticker names or mojibake in the notebook.

## AMZN-only Lesson 3 scope (2026-09-14)

- The user reset the main study scope to AMZN only. The recommended task remains next-observation direction classification after market close: target 1 when next adjusted close is above the current adjusted close, else 0.
- AMZN has 1,762 daily rows from 2010-01-04 through 2016-12-30 and 1,761 valid next-observation targets: 913 up (51.846%) and 848 non-up. Chronological counts are 645/613 up/non-up for 2010-2014, 130/122 for 2015, and 138/114 for 2016. This target needs no imbalance treatment.
- For the AMZN-only predictive matrix, use normalized lagged price/return, momentum, rolling-volatility, intraday, and relative-volume features. Exclude date, the constant symbol and sector, raw identifier fields, the target, and annual fundamentals. Fundamentals have only four AMZN rows and no trusted public-availability dates, so they remain a cleaning demonstration rather than model input.

## Recommended AMZN feature design (2026-09-14)

- Use a compact 21-feature numeric matrix at end of day t: returns over 1/5/20/60 sessions; close-to-MA ratios over 5/20/60; MA5-to-MA20 and MA20-to-MA60 ratios; return volatility over 5/20/60; relative volume versus prior-only 5/20/60-session means; one-day volume change and log volume; intraday range, candle body, overnight gap, and close position.
- The horizons encode current shock (1), trading week (5), trading month (20), and trading quarter/regime (60). They provide correlated multi-scale variables for PCA/RFECV while the maximum warm-up removes only about 3.4% of AMZN's 1,762 rows. Treat these windows as hypotheses and assess alternatives only with training/validation data, never the 2016 test set.
- Exclude raw date, constant symbol/sector, annual fundamentals, raw OHLC price levels, future next-close fields, and the target from X. Build deterministic lag/rolling features on the full ordered AMZN history, then split chronologically; fit scaling and dimensionality reduction on training data only.

## AMZN feature implementation (2026-09-14)

- module/les3/asg/asignment3.ipynb now implements all 25 agreed AMZN features after the cleaning summary: 1/5/10/20/60-session returns; four close-to-MA ratios and two MA-ratio features; 5/10/20/60-session return volatility; log volume, one-session log-volume change, four prior-only relative-volume ratios; and four intraday/gap features. No target, chronological split, scaling, RFE/RFECV, or PCA has been added yet.
- The notebook explicitly audits natural warm-up missingness and creates eatures_ready by dropping rows missing any selected feature. A sequential run and direct formula checks passed: 25 unique feature names, 1,702 ready rows from 1,762 inputs, 60 warm-up rows removed, first ready date 2010-03-31, last date 2016-12-30, and zero infinite values.
- Moving averages and volatility include day t because prediction is assumed after the close. Relative-volume baselines exclude day t with shift(1). The old feature-name placeholder and an empty cell were removed, and the duplicate AMZN scope constant was cleaned up.

## AMZN target balance table (2026-09-14)

- module/les3/asg/asignment3.ipynb now creates an overall label_balance table after model_data, with label meaning, count, percentage, total observations, and majority/minority ratio. The executed result on the 1,701 feature-complete labeled rows is class 0: 818 (48.09%), class 1: 883 (51.91%), majority/minority ratio 1.079. The table sums to 100% and its counts equal len(model_data), so the target is acceptably balanced overall. Balance should be checked again per chronological split after train/validation/test are created.
