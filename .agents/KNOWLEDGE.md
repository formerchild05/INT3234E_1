# Knowledge index

- Updated: 2026-09-15.
- Answer the user in Vietnamese. Store context in English ASCII using UTF-8.
- For every prompt, read `module/dataset/CONTEXT.md` immediately after this file to load the shared dataset context.
- For CAMEF paper work, read and update `.agents/KNOWLEDGE_PAPER.md`.
- For course lessons and the current stock dataset, read and update `.agents/KNOWLEDGE_COURSE.md`.
- Read both only when a request explicitly connects the paper and course work.
- Scientific-paper work uses `.agents/skills/paper-study/SKILL.md`.
- Context summaries do not replace primary evidence or newer user instructions.
- Do not commit or push only to save context.


## Current course focus (2026-09-14)

- The active Lesson 3 study scope is now AMZN only. Use next-observation direction as the balanced target and exclude annual fundamentals from the daily prediction matrix; details are in .agents/KNOWLEDGE_COURSE.md.

- Current module/les3/asg/asignment3.ipynb split cells still need correction before modeling: create eature_data['target_date'] = feature_data['date'].shift(-1) before constructing model_data; then drop missing feature/target/target-date rows. Define boolean Series masks from model_data['target_date'], not filtered DataFrames. Recommended baselines are a DummyClassifier(strategy='prior') and a standardized Logistic Regression using all 25 features. Tune RFECV/PCA on time-ordered train/validation data and evaluate the chosen pipeline once on the untouched 2016 test set.

- Use the verified diverse stock subset recorded in `module/dataset/CONTEXT.md`. Current scope is Lesson 2 exploration and preparation without model training.

- For later time-series forecasting, outlier detection separates data errors, corporate-action artifacts, and genuine market shocks. Correct confirmed errors; retain and optionally flag genuine shocks. Detect per symbol on returns/relative volume with past-only rolling thresholds. Do not delete extreme targets that the future model is expected to forecast.

- Current artifact under review: `module/les2/asg/asignment2.ipynb`. Review only was requested; do not edit until asked.

- Current notebook normality conclusion: describe each 2016 return histogram as roughly bell-shaped near zero, not fully normal. Support this with Q-Q plots, skewness, excess kurtosis, and tail frequency. Earlier shorthand attributing IQR sensitivity only to non-normality was corrected: the IQR 1.5 rule is itself tighter than absolute Z-score > 3 under normality.

- Return-outlier interpretation: a flagged return can be statistically rare yet domain-valid. Candidate causes include company news, macro/sector shocks, overnight gaps, volatility-regime changes, liquidity effects, corporate actions, longer calendar gaps between trading observations, or actual data errors. Diagnose via OHLC validity, raw-versus-adjusted comparison, date gaps, relative volume, neighboring days, cross-stock same-date movement, and rolling volatility. Without event/news data, report a candidate explanation rather than asserting a cause. Distinguish point, contextual, and collective anomalies.

- Latest notebook review (16 cells): missing handling should be cause-specific. Selected fundamentals have 18 missing cells: AMZN and BA each lack 2016 For Year, EPS, and Estimated Shares Outstanding; JPM lacks Cash Ratio, Current Ratio, and Quick Ratio for all four years. Derive For Year from Period Ending. For AMZN/BA, either retain EPS/shares as missing with flags or, for an explicitly documented teaching imputation, forward-fill prior shares within ticker and derive EPS from common-shareholder net income divided by imputed shares. Treat JPM liquidity ratios as structural/not applicable because current assets and current liabilities are recorded as zero; do not fabricate them with global median/KNN. Selected securities lacks Date first added for BA, KO, XOM, and MRK; omit this unused field or retain NA. The eight return NaNs are expected first-observation artifacts and should be dropped only from return-dependent analysis, not filled with zero.

- JPM liquidity-ratio calculation check (2026-09-13): `Total Current Assets` and `Total Current Liabilities` are both zero in all four JPM fundamentals rows, so standard Current, Quick, and Cash ratios cannot be reconstructed from this dataset; they are undefined, not zero. Do not impute them from non-bank companies. Available transparent proxies are Cash and Cash Equivalents / Total Assets (20.00%, 25.00%, 28.31%, 24.37% for 2012-2015), Cash / Total Liabilities (21.89%, 27.40%, 31.11%, 27.24%), and Equity / Total Assets (8.65%, 8.74%, 9.01%, 10.53%). Label these as proxies rather than standard liquidity ratios. Short-Term Investments is zero and Investments has negative values under this dataset's sign convention, so it should not be used naively as a liquid-assets numerator.

- Outlier-context feature decision: compute intraday range, log volume, prior-20-day mean volume, relative volume, and prior-20-day return volatility on the full sorted `COMP` before filtering 2016. Use `shift(1).rolling(20)` for volume and volatility baselines so the current outlier does not contaminate its own reference window. Recreate `COMP_2016` afterward and rerun downstream cells. Treat the first rolling NaNs as expected history-window gaps, not values to impute. For EDA, define volume spike as relative volume >= 2 and high prior volatility using a per-symbol descriptive threshold; summarize outliers with these context flags rather than removing them.

- Notebook cell 14 diagnosis (2026-09-13): `KeyError: Label(s) ['high_prior_volatility'] do not exist` occurs because `volatility_20` is created in cell 7 but the Boolean `high_prior_volatility` column is never created before aggregation. Fix by computing each symbol's 2016 Q3 of `volatility_20` with groupby-transform, then setting `high_prior_volatility = volatility_20 > Q3` before filtering `outlier_rows`. The corrected pipeline was run read-only and completed successfully.
