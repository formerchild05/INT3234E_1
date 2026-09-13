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

