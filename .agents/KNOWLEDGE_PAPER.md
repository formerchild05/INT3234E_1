# CAMEF paper knowledge

## Scope and sources

- Updated: 2026-09-12.
- Current paper: `pp/2502.04592v3.pdf`.
- Title: CAMEF: Causal-Augmented Multi-Modality Event-Driven Financial Forecasting by Integrating Time Series Patterns and Salient Macroeconomic Announcements.
- Authors: Yang Zhang, Wenbo Yang, Jun Wang, Qiang Ma, Jie Xiong. KDD 2025; arXiv v3 dated 2025-08-08. DOI: 10.1145/3711896.3736872.
- The user requested a beginner-friendly explanation of the topic, problem, and solution. No implementation or reproduction has been requested.
- Primary PDF sections inspected: abstract/introduction p. 1; problem p. 3; data and counterfactuals pp. 4-5; architecture and loss pp. 5-7; experiments and ablations pp. 7-8; conclusion p. 9; Appendix C.2 p. 12.
- HTML cross-check: `https://arxiv.org/html/2502.04592v3`.
- Official repository: `https://github.com/lakebodhi/CAMEF`. Its README was inspected. Code, datasets, and weights have not been downloaded or inspected.

## Verified findings

- Task: use an announcement available at release time and the preceding financial time series to forecast the subsequent series. The model does not predict unreleased announcement content. [PDF Section 3, p. 3]
- Motivation: announcement text and historical market patterns contain complementary information. [PDF abstract and Sections 1-2]
- Dataset: six announcement types - FOMC, Unemployment Insurance Claims, Employment Situation, GDP Advance, CPI, and PPI - paired with SPX, INDU, NDX, USGG1M, and USGG5YR at 5-minute OHLC frequency. These are US stock indices and Treasury series, not individual stocks. Exact date ranges differ between the abstract, Section 4, and Table 1. [PDF Section 4.1, p. 4]
- Architecture: RoBERTa encodes text; MOMENT encodes historical series; projected representations are fused and passed through a GPT-2 decoder and post-regressor to produce numerical forecasts. GPT-2 is a network component rather than a price-answering chatbot. [PDF Sections 5.1-5.4]
- Counterfactual generation: LLaMA-3 8B summarizes announcements, assigns sentiment from 1 to 10, and modifies relevant numbers or phrases while keeping neutral context. The default set has 10 same-type variants and 5 nearby different-type announcements. [PDF Sections 4.2 and 5.5.1]
- Training combines MSE, MAE, and triplet loss. Triplet loss makes the factual announcement embedding closer to its paired time-series embedding than alternative announcements. Counterfactual worlds have no observed market outcomes as labels. [PDF Sections 5.4-5.5]
- Experiments use a 6:2:2 train/validation/test ratio. Horizons 35, 70, and 140 correspond to 175, 350, and 700 minutes. Section 6.1 does not say whether the split is chronological or random. Authors report first place in 24 of 30 asset/horizon/metric combinations; this is not 80 percent prediction accuracy.
- Example: SPX at horizon 35 has reported MSE 0.00048860 for CAMEF and 0.00073333 for TEST. Preprocessing, scaling, and reproduction remain unverified.

## Assessment and next checks

- Assistant assessment: counterfactual contrastive text may improve representations and forecasts, but the objective and prediction errors alone do not establish real-world causal identification. Keep this assessment separate from the authors' causal claims.
- No implementation or reproduction has been performed.
- Open questions: chronological splitting, overlapping-window leakage, counterfactual quality, tensor dimensions, normalization, and trading profitability.
- For future explanations, introduce intuition before equations.
- For reproduction, inspect Appendices B-C and source code, prioritizing split strategy, normalization, and information timing.
