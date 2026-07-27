# Dynamic HMM Regime Switching & Portfolio Optimization

## Project Overview
This project implements an institutional-grade, out-of-sample quantitative trading pipeline in a single, top-to-bottom Jupyter Notebook. It uses an unsupervised Gaussian Hidden Markov Model (HMM) to detect underlying market regimes from historical NSE (Nifty 50) and macro data. These regime classifications are then fed dynamically into a convex optimization engine (CVXPY) to adjust portfolio weights in real-time across a 3-asset universe (Equities, Gold, Cash).

## Key Architecture Decisions

**Why 3 Regimes?**
Financial markets historically exhibit three structurally distinct states: steady, low-volatility uptrends (**Bull**); high-volatility, sideways or downward distribution (**Bear**); and violent liquidity/panic events (**Crisis**). Using a 3-state HMM perfectly captures these macroeconomic realities without overfitting to market noise, which often happens when using 4 or more states.

**Why These Features?**
The model is fed rolling momentum (21-day and 63-day) and rolling volatility. 
*   **Momentum** captures the directional trend and rate of change. 
*   **Volatility** acts as the universal mathematical signature of market stress. 
By using these specific features, the unsupervised clustering algorithm naturally separates the data in a way that maps perfectly to known market environments, eliminating the need for manual data labeling.

**Optimization Logic & Strict Validation**
To prevent data leakage, the pipeline utilizes a strict walk-forward validation harness. The HMM is re-fit *only* on past data at every step. Based on the predicted out-of-sample regime, the convex solver targets distinct objectives:
*   **Bull:** Maximizes risk-adjusted returns (Sharpe).
*   **Bear:** Minimizes global portfolio variance (GMV).
*   **Crisis:** Minimizes variance with a hard constraint forcing at least 50% of capital into the Cash/Liquid asset for strict capital preservation. The covariance lookback window is tightened to 10 days to allow hyper-fast reactions to sudden volatility spikes.

## Notebook Structure & Deliverables
The submitted Jupyter Notebook (`.ipynb`) is completely self-contained and sequentially generates all required outputs:
1.  **Data & Features:** Automatically fetches historical data and engineers features.
2.  **Regime Detection:** Trains the HMM, outputs the Transition Probability Matrix, and plots the inferred regime labels directly on top of the historical price data.
3.  **Walk-Forward Validation:** Executes the expanding-window HMM prediction loop.
4.  **Backtesting & Convex Optimization:** Runs the real-time simulation enforcing transaction costs (10 bps) and dynamic weight shifting.
5.  **Results & Equity Curves:** Plots the final portfolio growth and generates the performance matrix.

## Performance Summary (Out-of-Sample)
By successfully utilizing actual negative correlations (Equities vs. Gold/Cash) during detected market panics, the Dynamic HMM drastically reduced portfolio drawdowns while capturing upside yield. 

*Performance of the HMM Strategy (including 10 bps transaction cost friction) vs Benchmarks:*
*   **Annualized Return:** 15.86% *(vs 60/40: 14.19%)*
*   **Sharpe Ratio:** 1.70 *(vs 60/40: 1.00)*
*   **Sortino Ratio:** 2.20 *(vs 60/40: 1.21)*
*   **Max Drawdown:** -8.40% *(vs 60/40: -22.33%)*
*   **Calmar Ratio:** 1.89 *(vs 60/40: 0.64)*
*   **Annualized Turnover:** Captured and reported in the final metrics table. Both 0-bps and 10-bps friction models are executed to demonstrate the real-world viability of the strategy after exchange fees.

## How to Run and Reproduce
Since the entire pipeline is contained within a single file, reproducing the results is straightforward:

1.  **Prerequisites:** Ensure your Python environment has the following packages installed:
    ```bash
    pip install pandas numpy yfinance hmmlearn cvxpy matplotlib scikit-learn
    ```
2.  **Execution:** Open the `Final_Project.ipynb` file in Jupyter Notebook, JupyterLab, or VS Code.
3.  **Run:** Select **"Restart Kernel and Run All Cells"**. 
4.  **Output:** The notebook will automatically download the required ticker data, process the walk-forward loops, and print the final transition matrices, regime charts, equity curves, and metric summaries at the bottom of the file.
