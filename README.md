# Day-Ahead Photovoltaic (PV) Power Forecasting

A machine learning project focused on predicting 24-hour day-ahead solar power generation for the **"Am Aawasser"** local energy community in Buochs, Switzerland. This project was developed for the *Energy Data Analytics & Forecasting* module at the Lucerne University of Applied Sciences and Arts (HSLU).

---

## Authors & Context
- **Authors:** Flora Gashi, Teckla Achieng Onyango, Sevan Sherbetjian
- **Institution:** Lucerne University of Applied Sciences and Arts (HSLU)
- **Deliverables:** PDF Report & 20-minute Presentation

---

## Project Overview
Photovoltaic power generation is highly intermittent as it directly depends on weather dynamics. The goal of this project is to model past operational and meteorological context to deliver an accurate 24-hour day-ahead forecast of PV output in kilowatts ($\text{kW}$).

- **Input Horizon:** Past 168 hours (7 days) of historical PV power, hourly weather measurements, and cyclical time features.
- **Output Horizon:** 24 consecutive hourly predictions ($\hat{y} \in \mathbb{R}^{24}$) for the next day.

---

## Datasets & Feature Selection

The project evaluates data collected across 2022 from Buochs and Horw, Switzerland:
1. **PV Data:** 15-minute resolution power measurements ($\text{kW}$) covering a full year. Aggregated to hourly averages to align with weather data.
2. **Weather Data (MeteoBlue Horw):** Hourly measurements including temperature, relative humidity, cloud cover total, snowfall amount, sunshine duration, and shortwave radiation components.
3. **Household Load Data:** Provided as context but excluded from training, as PV generation is physically independent of local household consumption.

### Key Data Preprocessing Steps:
- **Resampling & Alignment:** Aggregated 15-minute PV readings to 1-hour intervals.
- **Cyclical Encoding:** Encoded hour-of-day (24h) and day-of-week (7d) using $\sin$ and $\cos$ transformations to represent continuous daily cycles.
- **Feature Selection:** Removed highly collinear features (e.g., diffuse shortwave radiation).
- **Sequence Generation:** Used a sliding window approach with a 6-hour stride to create non-overlapping split sequences for training, validation, and testing to eliminate data leakage.

---

## Model Architecture & Training

### Model: Gated Recurrent Unit (GRU)
- **Encoder:** 1-layer GRU with 64 hidden units processing the sequence $X \in \mathbb{R}^{168 \times F}$.
- **Predictor:** Linear dense layer mapping the final hidden state $h_T \in \mathbb{R}^{64}$ to a 24-dimensional output vector ($\hat{y} \in \mathbb{R}^{24}$).

### Training Setup:
- **Loss Function:** Huber Loss ($\text{Smooth L1}$, $\delta \in \{0.5, 1.0\}$) for robustness against extreme values.
- **Optimizer:** Adam with $L_2$ Weight Decay and `ReduceLROnPlateau` learning rate scheduling.
- **Grid Search:** Systematic hyperparameter search across 144 configurations (tuning learning rate, hidden size, weight decay, and loss delta).
- **Baseline:** Persistence Model (predicting that tomorrow's generation profile equals today's profile).

---

## Results & Evaluation

Forecasting performance evaluated on the unseen test set:

| Model | MAE (kW) | RMSE (kW) |
| :--- | :---: | :---: |
| **Persistence Baseline** | **5.27** | 13.31 |
| **GRU Neural Network** | 6.87 | **11.80** |

### Key Insights & Error Analysis:
- **Peak Underestimation:** The GRU successfully captured daily diurnal cycles but exhibited regression-to-the-mean behavior, underestimating noon peak power during high-solar days.
- **RMSE vs. MAE:** The GRU achieved a lower RMSE than the baseline (11.80 vs 13.31 kW), indicating it makes fewer extreme peak errors, though the baseline achieved a better overall MAE.
- **Future Improvements:** Incorporating day-ahead weather forecasts (forward-looking exogenous features) instead of relying purely on historical context would significantly reduce day-ahead uncertainty.
