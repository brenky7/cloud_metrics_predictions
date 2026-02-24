## Project Overview
This project is a solution to the technical assignment for the JetBrains Internship Project: "Predictive alerting for cloud metrics". The goal of this project is to design and implement a predictive alerting system capable of forecasting incidents in cloud services. This system utilizes supervised Machine Learning to predict the probability of a future incident based on historical metric patterns.

The core objective is to detect potential failures before they occur (predictive maintenance) while balancing Recall (catching real incidents) and Precision (minimizing false alarms).

## Task Specification
The assignment requires the implementation of a model that predicts whether an incident will occur within the next *H* time steps based on the previous *W* steps of time-series metrics.

**Key Requirements:**
- **Problem Formulation:** Sliding-window approach for time-series forecasting/classification.
- **Model Capabilities:** Handling noisy, non-stationary data typical of cloud environments.
- **Evaluation:** Empirical evaluation focusing on recall (target ~80%), false-positive rates, and detection lead time.

## Proposed Solution & Architecture

### 1. Data Selection: Server Machine Dataset (SMD)
This project utilizes the *Server Machine Dataset (SMD)* from the *OmniAnomaly* repository.
-   **Source:** [NetManAIOps/OmniAnomaly](https://github.com/NetManAIOps/OmniAnomaly/tree/master/ServerMachineDataset)
-   **Characteristics:** Real-world, multivariate data collected from a large internet company. It contains 38 metrics (CPU, Load, Memory, Disk, Network, etc.)

### 2. Modeling Approach: Random Forest Classifier
The problem is framed as a binary classification task. I chose Random Forest Classifier, because it models the non-linear thresholds in predictors well and is robust against noisy data. It is also more lightweight and faster to train then neural networks.
-   **Input:** A sliding window of size $W$ containing the past values of the 38 metrics.
-   **Output:** A binary label ($0=$ Normal, $1=$ Incident Imminent) for the next $H$ steps.

### 3. Feature Engineering Strategy
To combat the **non-stationary** nature of cloud data, the raw metrics are transformed:
-   **Rolling Statistics:** Mean and Standard Deviation over the window $W$.
-   **Deltas:** Step-to-step changes to detect sudden shocks.
