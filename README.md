## Project Overview

This project is a solution to the technical assignment for the JetBrains Internship Project: "Predictive alerting for cloud metrics". The goal of this project is to design and implement a predictive alerting system capable of forecasting incidents in cloud services. This system utilizes supervised Machine Learning to predict the probability of a future incident based on historical metric patterns.

The core objective is to detect potential failures before they occur (predictive maintenance) while balancing Recall (catching real incidents) and Precision (minimizing false alarms).

## Task Specification

The assignment requires the implementation of a model that predicts whether an incident will occur within the next _H_ time steps based on the previous _W_ steps of time-series metrics.

**Key Requirements:**

- **Problem Formulation:** Sliding-window approach for time-series forecasting/classification.
- **Model Capabilities:** Handling noisy, non-stationary data typical of cloud environments.
- **Evaluation:** Empirical evaluation focusing on recall (target ~80%), false-positive rates, and detection lead time.

## Proposed Solution & Architecture

### 1. Data Selection: Server Machine Dataset

**Goal:** Choose a dataset to work with.

**Result:**

For this project I chose the Server Machine Dataset (SMD) from the OmniAnomaly repository. This dataset contains multivariate time-series data collected from a large internet company. It contains 38 metrics (CPU, Load, Memory, Disk, Network, etc.) collected from 28 distinct machines, which makes it similar to cloud service metrics.

Although SMD is designed for unsupervised learning, the testing portion is fully labeled. To adapt this for supervised learning, I initially started with the labeled test set of a single machine instance (machine-1-1), splitting it chronologically into training and evaluation subsets.

During exploratory data analysis, I observed that anomalies for a this machine were highly localized in time. In a time-series setting with a strict chronological split, this reduces the diversity of incident patterns available for training and makes evaluation less representative.

To address this, I expanded the dataset to include all machines from SMD Group 1, which is 8 machines in total. Each machine is treated as an independent time-series entity, which increases the number and temporal diversity of incident intervals while keeping the evaluation time-consistent.

To prevent data leakage (look-ahead bias), the dataset was split chronologically. Evaluation is always performed on future time relative to the corresponding training period.

**Source:** [NetManAIOps/OmniAnomaly](https://github.com/NetManAIOps/OmniAnomaly/tree/master/ServerMachineDataset)

### 2. Exploratory Data Analysis

**Goal:** Understand the dataset of the server metrics to inform feature engineering and model expectations.

**Tasks:**

- **Data Cleaning:** Assess data quality by checking for missing values.
- **Univariate Analysis:** Analyze the distribution and scale of individual features.
- **Bivariate Analysis:** Investigate the correlation between individual features and anomaly occurrences.

**Results:**

- **Data Cleaning:** Verified absence of missing values, identified and dropped zero-variance features that offer no predictive power.
- **Univariate Analysis:** Analyzed distributions, revealing highly skewed, sparse event-driven metrics alongside continuous, stable metrics.
- **Bivariate Analysis:** Mapped features against anomaly occurrences. Identified clear leading/coincident indicators.
- **Dataset adjustment based on EDA:** Discovered that anomalies were temporally clustered, which complicates supervised training/evaluation with chronological splits. To improve incident coverage and dataset diversity, the project scope was expanded from a single machine to all machines in SMD Group 1.

These tasks are implemented in the _data_analysis_ notebook.

### 3. Problem Formulation

**Goal:** Translate a raw time-series problem into a supervised Machine Learning problem that predicts future risk.

**Tasks:**

- **Sliding window definition:** Determine the optimal window size _W_ to capture sufficient historical context.
- **Prediction horizon definition:** Define the prediction horizon _H_ to establish the detection lead time.
- **Address Non-Stationarity:** Extract rolling statistics (mean, volatility/std, deltas, z-scores) from the raw window to give the model context.

**Results:**

- **Sliding window definition:** Selected two window sizes for evaluation: \(W=12\) and \(W=25\). This allows for a comparative analysis between capturing immediate shocks versus longer-term trends.
- **Prediction horizon definition:** Fixed at \(H=5\) minutes. This provides a realistic detection lead time sufficient for fiixng within standard SLAs.
- **Address Non-Stationarity:** Implemented a feature engineering pipeline that transforms raw time-series metrics into stationarity-invariant indicators. For every window, Rolling Mean (trend), Standard Deviation (volatility), Delta (rate of change), and Local Z-Scores (relative magnitude) are calculated.

These tasks are implemented in the _model_training_ notebook.

### 4. Model Selection & Training

**Goal:** Choose an appropriate algorithm and train it without violating time-series principles.

**Tasks:**

- **Model Justification:** Select an apropriate model and justify the choice.
- **Model Training & parameter tuning**

**Results:**

Based on information observed during EDA as well as the nature of the problem, I have chosen **Random Forest Classifier** for this task. The main reasons are:

1. **Non-Linear features:** In the bivariate analysis, I observed that some features (like feat_8 or feat_29) remain at 0.0 for long periods and spike suddenly during an incident. Linear models like Logistic Regression struggle to model these sharp cliffs. Decision Trees naturally model these non-linear thresholds.
2. **Interaction Effects:** Some features (like feat_15 or feat_22) appeared stable but are likely informative when combined with other indicators. Random Forests automatically learn these interaction effects without requiring manual feature combination.
3. **Robustness to Noise:** The cloud metrics contain jitter and spikes that are not incidents. By averaging the predictions of multiple trees, the Random Forest reduces the variance and is less likely to overfit to individual noisy data points compared to a single Decision Tree.
4. **Efficiency for Retraining:** The project description requires a system capable of periodic retraining. Random Forests parallelize training easily and are significantly faster and cheaper to train than recurrent neural networks.

- **Input:** A sliding window of size $W$ containing the past values of the metrics.
- **Output:** A binary label ($0=$ Normal, $1=$ Incident Imminent) for the next $H$ steps.

These tasks are implemented in the _model_training_ notebook.

### 5. Domain-Specific Evaluation and Analysis

**Goal:** Document the architecture, justify decisions, and analyze failure cases.

**Tasks:**

- **Feature Importance Analysis:** Analyse which features drove the predictions.
- **Failure Analysis:** Analyse false negatives (incidents missed) and false positives (fake alerts).

**Results:**

These tasks are implemented in the _model_training_ notebook.
