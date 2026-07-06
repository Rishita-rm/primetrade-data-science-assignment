# Data Science / Analytics Intern Assignment
## Trader Performance vs Market Sentiment on Hyperliquid

This repository contains my submission for the Round-0 Analytics Assignment at Primetrade.ai. The project explores how macro-level Bitcoin market sentiment (Fear vs. Greed) interacts with high-frequency execution metrics and behavioral patterns among traders on the Hyperliquid protocol.

---

## Repository Contents

* `trader_performance_vs_sentiment.ipynb` - The full Google Colab data pipeline, containing data ingestion, cleaning, feature engineering, statistical tests, and predictive modeling.
* `sentiment_behavior_charts.png` - Publication-grade exploratory visualization showing how win rates and position sizes adjust across different market regimes.
* `requirements.txt` - Python dependencies needed to reproduce the environment.

---

## Setup & Local Reproduction Instructions

Since this project was built entirely using standard data science libraries, you can run it easily in Google Colab or your local machine.

### Option 1: Run via Google Colab (Easiest)
1. Go to [Google Colab](https://colab.research.google.com/).
2. Upload the `trader_performance_vs_sentiment.ipynb` notebook.
3. Upload `Bitcoin_Market_Sentiment.csv` and `Historical_Trader_Data.csv` into the temporary storage side-panel.
4. Go to **Runtime > Run all**.

### Option 2: Local Installation
If running locally, ensure you have Python 3.10+ installed, then follow these terminal steps:

```bash
# Clone the repository
git clone [https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git](https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME.git)
cd YOUR_REPO_NAME

# Install the required dependencies
pip install -r requirements.txt

# Run your jupyter server
jupyter notebook
```
## Core Methodology & Cleaning Steps

*   **Data Purging:** Removed direct duplicate row entries and handled missing contextual elements using targeted `.dropna()` filters across core transaction indicators.
*   **Regime Harmonization:** Grouped volatile sentiment descriptions like "Extreme Fear" and "Fear" into a single consolidated `fear` state (and did the same for `greed`) to ensure cleaner statistical comparative testing.
*   **Vectorized Metrics Generation:** Aggregated high-frequency execution logs up to a standardized **Account-Day** granularity. Engineered core parameters including:
    *   **Daily PnL:** Summed `Closed PnL` per account per day.
    *   **Win Rate:** Ratio of profitable trade outputs relative to total active executions.
    *   **Average Trade Size:** Total daily transaction dollar volume divided by trade count.
    *   **Long/Short Ratio:** Buy vs Sell counts smoothed via Laplace adjustments to prevent zero-division errors.

---

## Analytical Insights & Evidence (Part B)

Instead of just checking basic averages, I used a **Mann-Whitney U test** to confirm whether shifts in trader choices between Fear and Greed days were statistically real or just random noise.

### 1. Performance Under Pressure (PnL & Win Rates)
The data flags a definitive drop in trader performance during market contractions. The test showed a **Significant Shift** ($p < 0.05$) in win rates between regimes. 

On Greed days, win rates lean consistently higher across almost all participants. On Fear days, distributions flatten out aggressively. Furthermore, using an account's worst single-day negative PnL as a proxy for tail-risk, the absolute worst-case losses occur almost exclusively during Fear regimes.

### 2. Behavioral Shifts (Frequency vs Sizing)
*   **The Overtrading Paradox:** Surprisingly, the total number of trades per day showed **No Significant Difference** across regimes. Traders do not scale down their speed when scared—they keep clicking buttons through the volatility.
*   **Sizing Corrections:** While trade frequency stays fixed, trade size drops. The test flagged a **Significant Shift** ($p < 0.01$) in average trade size in USD. Traders scale back their dollar risk per entry during Fear, but they overtrade the noise, meaning transaction fees heavily eat into their capital.

### 3. Account Clustering Profiles
Running an unsupervised K-Means algorithm segmented the trader user base into 3 distinct behavioral archetypes:
*   **Segment 1 (High-Volume Scalpers):** Small trade sizes but high execution rates. They try to scalping-trade through Fear states but end up overtrading and bleeding platform fees.
*   **Segment 2 (Low-Frequency Whales):** Massive position sizes but low trade counts. Their win rates stay highly resilient regardless of whether it is a Fear or Greed day.
*   **Segment 3 (Systemic Dip-Buyers):** Moderate frequency with a structural long bias. They run fine in bull states but get wiped out trying to catch falling knives on Fear days.

---

## Actionable "Rules of Thumb" (Part C)

### Rule 1: The "Scalper Fee Cap" for Segment 1 (Fear States)
*   **The Rule:** If market sentiment hits Fear or Extreme Fear, automatically enforce a 30% reduction cap on maximum daily trade frequency for accounts identifying with Segment 1 characteristics.
*   **Why it works:** The data proves these traders do not slow down when losing; they overtrade the chaos. Capping their daily executions protects them from revenge-trading and fee-bleeding.

### Rule 2: The "Whale-Copy Momentum" Signal (Greed States)
*   **The Rule:** During Greed states, programmatically monitor the aggregate directional choices of Segment 2 (Whales) over rolling 4-hour windows. If their Long/Short ratio breaks past 1.5, trigger a trend-following buy signal.
*   **Why it works:** Segment 2 consists of consistent winners who effectively scale position values during structural uptrends. Mimicking their core direction captures high-probability moves while weeding out retail noise.

---

## Predictive Model Validation (Bonus Task)

To test the predictive value of these metrics, I built a Random Forest Classifier to forecast next-day account profitability based on a blend of market sentiment and lagged behavioral features (`total_trades`, `long_short_ratio`, `avg_trade_size_usd`, `is_greed_regime`).

*   **Overall Model Accuracy:** 77%
*   **ROC-AUC Score:** 0.7429
*   **Class 1 (Profitable Day) Recall:** 92%
*   **Class 0 (Risk/Unprofitable) Precision:** 81%

### Strategic Takeaway:
With an 81% precision score for unprofitable days (Class 0), this system serves as an elite risk filter. If the pipeline flags an upcoming high probability of an unprofitable day for a specific cohort, the platform can programmatically recommend safety strategies or temporarily reduce maximum default position limits to protect retail capital.
