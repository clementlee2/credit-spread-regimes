# credit-spread-regimes
Detecting credit market regimes using Hidden Markov Models to analyze spread dynamics and generate trading signals.

📊 Project Overview

Credit spreads exhibit distinct regime behavior, shifting between stable, stress, and recovery environments. This project aims to:

Identify latent regimes in US corporate credit spreads
Model regime dynamics using probabilistic methods
Evaluate whether regime signals can predict future spread movements and market risk
📁 Data Source

The dataset consists of historical US corporate credit spreads (Investment Grade and/or High Yield), sourced from publicly available market data providers (e.g., FRED, Bloomberg, or ICE indices). Data was cleaned and structured to ensure consistency across time series.

🧠 Methodology
Exploratory Data Analysis (EDA):

Analyzed spread distributions, volatility clustering, and time-series behavior
Regime Detection Models:
Applied Hidden Markov Models (HMM) and/or Gaussian Mixture Models (GMM) to identify latent market states
Feature Engineering:
Constructed inputs such as spread levels, changes, and rolling volatility
Backtesting & Signal Evaluation:
Assessed whether identified regimes have predictive power for forward spread movements and equity volatility

🔧 Tools & Technologies

Language: Python
Libraries: pandas, numpy, matplotlib, scikit-learn, hmmlearn


Techniques:

Hidden Markov Models (HMM)
Gaussian Mixture Models (GMM)
Time series analysis
Regime classification
Signal backtesting

📈 Key Outcomes

Identified distinct credit regimes corresponding to risk-on, stress, and recovery environments
Observed clustering of high-volatility periods within specific regimes
Found evidence that stress regimes are associated with future spread widening and increased market volatility
Demonstrated potential for regime-based signals to inform credit positioning and risk management

💡 Practical Implication

Regime identification can be used to:

Adjust credit exposure dynamically
Anticipate periods of market stress
Inform hedging strategies using equity or volatility instruments
