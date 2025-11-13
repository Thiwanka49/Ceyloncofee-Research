# Coffee AI Platform

> Smart tools to detect coffee plant diseases, predict yield & prices, and optimise labor & transport for Sri Lankan coffee producers.

![CI](https://img.shields.io/badge/status-active-brightgreen)

## Table of Contents

- [About](#about)
- [Key Components](#key-components)
- [Features](#features)
- [Architecture](#architecture)
  - [Mermaid Diagram](#mermaid-diagram)
  - [ASCII Diagram](#ascii-diagram)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Run locally](#run-locally)
- [Folder Structure](#folder-structure)
- [Models & Data](#models--data)
- [API / Endpoints](#api--endpoints)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

---

## About

This project bundles a set of AI-powered modules tailored for Sri Lankan coffee production and supply chains. It includes image-based disease classification, yield prediction under disease stress, price & demand forecasting, labor & transport resource planning, and bean type & grade identification.

## Key Components

- **Disease Classification** — CNN-based image classifier for leaf/bean diseases (e.g., Leaf Rust, Cercospora).
- **Yield Prediction** — Regression model to estimate yield per plant or plot.
- **Price & Demand Prediction** — Time-series and econometric models for short/medium-term forecasting.
- **Labor & Transport Forecasting** — Scheduling and optimization for workers and vehicles.
- **Bean Type & Grade Identification** — Image model to classify bean variety and grade.
- **Frontend & Dashboard** — User-facing web/mobile app for uploading images, viewing predictions, alerts and optimization dashboards.
- **Backend & ML Pipeline** — API services, model serving, databases, and background workers.

## Features

- Upload leaf/bean images and get disease & grade predictions.
- Batch yield forecasts and plantation-level aggregation.
- Price and demand forecasts with explanatory factors (weather, macro indicators).
- Labor/vehicle scheduling suggestions and alerts.
- Simple exportable reports for exporters and farm managers.

---

## Architecture

The architecture below maps the main components and data flow (ML training, inference, and operational tooling).

### Mermaid Diagram

```mermaid
flowchart LR
  subgraph U[Users]
    Web[Web Dashboard / Admin] -->|HTTPS| GW[API Gateway]
    Mobile[Mobile App / Field App] -->|HTTPS| GW
    FieldDevice[Field Camera/Phone] -->|HTTPS| GW
  end

  subgraph S[Services]
    GW --> Auth[Auth Service (JWT/OAuth)]
    GW --> API[API Service (Inference, Aggregation)]
    API --> ML[Model Serving (TF/ONNX/GPU)]
    API --> DB[(Postgres / TimescaleDB)]
    API --> Cache[Redis Cache]
    API --> MQ[Message Queue (RabbitMQ / SQS / Kafka)]
    MQ --> Worker[Workers: Preprocessing, Batch Jobs]
    Worker --> FeatureStore[Feature Store]
    ML --> FeatureStore
    ML --> ModelRepo[Model Registry]
    Worker --> Storage[S3 / Object Storage (images, artifacts)]
  end

  subgraph External[External Data]
    Weather[Weather API] --> ETL[Data Ingest]
    Market[Market / Price Feeds] --> ETL
    ETL --> DB
  end

  subgraph Ops[Monitoring & CI/CD]
    CI[CI/CD Pipeline] --> ModelRepo
    Prom[Prometheus / Grafana] <-- API
  end

  Auth --> DB
  API --> Storage
  FeatureStore --> DB
  style U fill:#f9f,stroke:#333,stroke-width:1px
  style S fill:#f2f2f2,stroke:#333,stroke-width:1px
  style External fill:#fff3cd,stroke:#333,stroke-width:1px
  style Ops fill:#e8f6ff,stroke:#333,stroke-width:1px
