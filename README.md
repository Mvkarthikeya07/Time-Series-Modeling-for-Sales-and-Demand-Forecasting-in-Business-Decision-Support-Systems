<div align="center">

<img src="https://img.shields.io/badge/Version-1.0.0-2ea44f?style=for-the-badge" alt="Version">
<img src="https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
<img src="https://img.shields.io/badge/Flask-3.x-000000?style=for-the-badge&logo=flask&logoColor=white" alt="Flask">
<img src="https://img.shields.io/badge/ARIMA-Statsmodels-blueviolet?style=for-the-badge" alt="ARIMA">
<img src="https://img.shields.io/badge/Chart.js-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white" alt="Chart.js">
<img src="https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge" alt="License">

<br/><br/>

# 📈 Time-Series Sales & Demand Forecasting System

### A Business Decision Support System powered by ARIMA, Flask, and Chart.js

> *Transforming 438,000+ rows of raw transactional data into actionable 6-month demand forecasts — with enterprise-ready accuracy metrics.*

<br/>

[Overview](#-overview) · [Features](#-features) · [Architecture](#-system-architecture) · [Tech Stack](#-tech-stack) · [Dataset](#-dataset) · [Model](#-forecasting-model) · [Getting Started](#-getting-started) · [Project Structure](#-project-structure) · [Screenshots](#-screenshots) · [Roadmap](#-roadmap) · [License](#-license)

<br/>

</div>

---

## 📌 Overview

The **Sales & Demand Forecasting System** is an end-to-end machine learning application that ingests multi-store, multi-product historical sales data, trains a time-series forecasting model, and delivers an interactive, browser-based decision support dashboard for business stakeholders.

The system bridges the gap between raw data and real decisions — giving operations, inventory, and finance teams a reliable, interpretable 6-month demand outlook.

| Problem | This System's Answer |
|---|---|
| Intuition-based inventory planning | ARIMA-driven demand forecast |
| No visibility into forecast accuracy | Live RMSE + MAPE KPI cards |
| Raw CSV data with no business context | Interactive chart + insights page |
| Overstocking / understocking cycles | 15% inventory adjustment guidance |

---

## ✨ Features

### 🏠 Landing Page
- Clean entry point with a single CTA directing users to the forecast dashboard
- Minimal, distraction-free design

### 📊 Forecast Dashboard (`/dashboard`)
- **KPI Cards** — Live RMSE, MAPE percentage, and 6-month forecast horizon
- **Dual-series Chart.js chart** — Historical sales (solid line) vs. forecasted sales (dashed line) on a single unified time axis
- Chart data injected server-side via Jinja2 `tojson` — zero client-side data fetching
- Navigation to Business Insights and Home

### 📌 Business Insights (`/insights`)
- Human-readable interpretation of model output
- Inventory planning guidance (~15% increase recommended)
- Revenue growth outlook and overstocking risk assessment
- Designed for non-technical business stakeholders

### ⚙️ ML Pipeline (Backend)
- Automated data loading, deduplication, and monthly aggregation
- Missing value interpolation (linear, business-safe)
- ARIMA(1,1,1) model training and 6-step ahead forecasting
- RMSE and MAPE computation on fitted values
- Forecast index auto-generated as monthly `DatetimeIndex`

---

## 🏗 System Architecture

```
Raw CSV (438,400+ rows)
         │
         ▼
┌─────────────────────────┐
│   data_processing.py    │  • parse dates • aggregate by month
│   load_and_prepare_data │  • set MS frequency • interpolate nulls
└─────────────────────────┘
         │
         ▼
┌─────────────────────────┐
│      forecast.py        │  • ARIMA(1,1,1) via statsmodels
│   train_and_forecast    │  • 6-step forecast • RMSE + MAPE
└─────────────────────────┘
         │
         ▼
┌─────────────────────────┐
│        app.py           │  • Flask routes • Jinja2 rendering
│   /  /dashboard         │  • chart arrays built server-side
│   /insights             │
└─────────────────────────┘
         │
         ▼
┌─────────────────────────┐
│   dashboard.html        │  • Chart.js dual-series line chart
│   + KPI cards           │  • RMSE / MAPE / horizon display
└─────────────────────────┘
```

---

## 🛠 Tech Stack

| Layer | Technology | Role |
|---|---|---|
| **Language** | Python 3.10+ | Core application runtime |
| **Web Framework** | Flask | Routing, template rendering, server |
| **ML / Forecasting** | Statsmodels (ARIMA) | Time-series model training & prediction |
| **Data Processing** | Pandas, NumPy | CSV ingestion, aggregation, interpolation |
| **Evaluation** | Scikit-learn | RMSE computation; MAPE via NumPy |
| **Visualization** | Chart.js (CDN) | Interactive dual-series line chart |
| **Templating** | Jinja2 | Server-side HTML rendering with data injection |
| **Styling** | Custom CSS | Dashboard layout, KPI cards, chart container |

---

## 📂 Dataset

| Property | Detail |
|---|---|
| **File** | `data/sales_data.csv` |
| **Rows** | ~438,400 transactional records |
| **Date Range** | 2019 – 2024 (daily granularity) |
| **Stores** | 10 |
| **Products** | 20 per store |
| **Columns** | `date`, `store_id`, `product_id`, `category`, `price`, `promotion`, `sales` |
| **Aggregation** | Daily multi-store records → monthly total sales (via `groupby + sum`) |
| **Generation** | Synthetic with realistic trend, weekly seasonality, yearly seasonality, and promotion effects |

### Data Pipeline Steps

```python
# 1. Parse dates
df['date'] = pd.to_datetime(df['date'])

# 2. Aggregate duplicates — sum sales across all stores/products per date
df = df.groupby('date', as_index=False)['sales'].sum()

# 3. Set monthly start frequency
df = df.asfreq('MS')

# 4. Fill gaps with linear interpolation
df['sales'] = df['sales'].interpolate(method='linear')
```

---

## 📈 Forecasting Model

### ARIMA(1, 1, 1)

| Parameter | Value | Meaning |
|---|---|---|
| **p** (AR) | 1 | 1 lagged observation as predictor |
| **d** (I) | 1 | 1st-order differencing for stationarity |
| **q** (MA) | 1 | 1 lagged forecast error in the model |
| **Forecast steps** | 6 | 6-month forward planning window |

**Why ARIMA?**
- Handles non-stationary sales data via differencing (`d=1`)
- Captures autocorrelation patterns in monthly aggregated demand
- Highly interpretable for business communication
- Computationally efficient on limited historical windows
- No external regressors required — runs on `date + sales` alone

### Evaluation Metrics

| Metric | Formula | Business Meaning |
|---|---|---|
| **RMSE** | √( mean( (actual − predicted)² ) ) | Error magnitude in original sales units |
| **MAPE** | mean( \|actual − predicted\| / actual ) × 100 | Forecast accuracy as a percentage — intuitive for all stakeholders |

Both metrics are computed on **in-sample fitted values** (not the 6-month forecast) and displayed live on the dashboard.

---

## 🔁 Application Routes

| Method | Route | Template | Description |
|---|---|---|---|
| `GET` | `/` | `index.html` | Landing page with dashboard CTA |
| `GET` | `/dashboard` | `dashboard.html` | Full ML pipeline → KPI cards + chart |
| `GET` | `/insights` | `insights.html` | Business interpretation of forecast |

---

## 🚀 Getting Started

### Prerequisites

- Python **3.10+**
- `pip` package manager

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-username/sales-demand-forecasting.git
cd sales-demand-forecasting

# 2. (Recommended) Create a virtual environment
python -m venv venv
source venv/bin/activate        # macOS / Linux
venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the application
python app.py
```

Open your browser at:

```
http://127.0.0.1:5000
```

Navigate to `/dashboard` to trigger the full ML pipeline and view forecasts.

### Dependencies

```
flask
pandas
numpy
matplotlib
statsmodels
scikit-learn
```

> `matplotlib` is imported with the `Agg` backend (`matplotlib.use("Agg")`) to prevent GUI conflicts in a headless Flask server environment.

---

## 📁 Project Structure

```
sales-demand-forecasting/
│
├── app.py                    # Flask app — routes & controller logic
├── forecast.py               # ARIMA model training, prediction, RMSE, MAPE
├── data_processing.py        # CSV loading, aggregation, interpolation, feature engineering
├── requirements.txt          # Python dependencies
│
├── data/
│   ├── sales_data.csv        # Primary dataset (~438k rows, 2019–2024)
│   └── generate_dataset.py   # Synthetic data generator (trend + seasonality + promotions)
│
├── templates/
│   ├── index.html            # Landing page
│   ├── dashboard.html        # KPI cards + Chart.js dual-series chart
│   └── insights.html         # Business insights page
│
└── static/
    ├── css/
    │   └── style.css         # All application styles
    └── images/
        └── forecast.png      # Static forecast image (supplementary)
```

---

## 📷 Screenshots

### 🏠 Home Page
![Home Page](https://github.com/user-attachments/assets/50c804aa-8f3e-4414-85ad-77c96f0c77e0)

### 📊 Forecast Dashboard — KPI Cards
![Dashboard KPI](https://github.com/user-attachments/assets/4994e150-6af0-433f-9e66-83f1323425b0)

### 📈 Interactive Forecast Chart — Historical vs Predicted
![Forecast Chart](https://github.com/user-attachments/assets/fb8a235e-c195-4373-86d5-ce9b2a2458e0)

### 🔭 Forecast Extension — Future Demand Projection
![Forecast Extension](https://github.com/user-attachments/assets/112e2224-aa90-49be-a7fe-32864c5ae11a)

### 📌 Business Insights Page
![Business Insights](https://github.com/user-attachments/assets/eb12a212-d3d0-471e-b04c-9afa9db416fb)

---

## 🧠 Software Engineering & ML Principles Applied

| Principle | Implementation |
|---|---|
| **Separation of Concerns** | `data_processing.py`, `forecast.py`, `app.py` — each owns one responsibility |
| **DRY Architecture** | Chart arrays assembled entirely in `app.py`; zero business logic in Jinja2 templates |
| **Headless-safe ML** | `matplotlib.use("Agg")` prevents GUI thread crashes in Flask context |
| **Business-safe imputation** | Linear interpolation preserves trend continuity without overfitting |
| **Scalable data pipeline** | `groupby + sum` aggregation handles any number of stores/products transparently |
| **Interpretable metrics** | RMSE (units) + MAPE (%) — one for engineers, one for stakeholders |

---

## 🔮 Roadmap

- [ ] **CSV Upload UI** — Allow users to upload their own sales dataset via the web interface
- [ ] **Auto ARIMA** — Use `pmdarima.auto_arima` to select optimal (p, d, q) parameters automatically
- [ ] **Multi-product Forecasting** — Generate individual forecasts per product or category
- [ ] **Advanced Models** — Facebook Prophet, SARIMA (seasonal), and LSTM comparisons
- [ ] **Forecast Confidence Intervals** — Display upper/lower bounds on the chart
- [ ] **Role-based Dashboards** — Separate views for Inventory Manager, Finance Lead, and Executive
- [ ] **Export Reports** — Download forecasts as PDF or Excel for offline use
- [ ] **Cloud Deployment** — Dockerized deployment to Render / Railway / AWS EC2
- [ ] **Model Persistence** — Save trained ARIMA model with `joblib` to avoid re-training on every page load

---

## 🎓 Academic & Internship Context

| Field | Detail |
|---|---|
| **Internship Track** | Machine Learning |
| **Organization** | Future Interns |
| **Task** | Task 1 — Sales & Demand Forecasting for Businesses |
| **Concepts Demonstrated** | Time-series analysis, ARIMA modeling, ML evaluation, full-stack ML deployment |
| **Developer** | M V Karthikeya |

---

## 📄 License

```
MIT License — Copyright (c) 2026 M V Karthikeya

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

> Developed for academic and internship evaluation purposes.

---

<div align="center">

Built with 🐍 Python · Powered by ARIMA + Flask + Chart.js · © 2026 M V Karthikeya

</div>
