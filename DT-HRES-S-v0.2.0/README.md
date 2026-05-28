# DT-HRES-S
### Digital Twin for Hybrid Renewable Energy Systems — Student-Led Open-Source

> Open-source AI-driven digital twin for designing Hybrid Renewable Energy Systems (HRES) in Mexican indigenous communities. Built by students, for students and communities.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Aaron-Cuevas/DT-HRES-S/blob/main/notebooks/11_digital_twin_prototype.ipynb)
[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Methodology: 4D](https://img.shields.io/badge/methodology-4D%20Technical%20Arts-orange.svg)](docs/4D_methodology/)

---

## 🌍 Project Context

This repository is part of the **EPICS in IEEE** funded project *"Student-Led Open-Source Digital Twin for Hybrid Energy Systems in Indigenous Communities – Mexico"*, led by Dr. Rasikh Tariq at Tecnológico de Monterrey, in collaboration with UADY, Fundación Internacional de Innovación Social y Sustentabilidad, and Villanova University.

The DT-HRES-S replaces commercial software like HOMER and PVsyst — which require paid subscriptions and specialized technical skills — with an **accessible, AI-driven alternative** that runs entirely on Google Colab.

---

## 🧭 Built with the 4D Methodology

This project follows the **4D Methodology** developed by Technical Arts (ITESM Student Chapter) for building digital twins from scratch. The methodology ensures all developers — regardless of background — share a common conceptual framework.

[![Wath video](https://img.youtube.com/vi/iXTr-tumegM/maxresdefault.jpg)](https://www.youtube.com/watch?v=iXTr-tumegM)

| Dimension | What it answers | Status |
|---|---|---|
| **1D — Concept** 🥚 | What is HRES and what do we predict? | ✅ Complete |
| **2D — Body** 🥚 | What do we optimize and for whom? | 🟡 In progress |
| **3D — Mind** 💪 | What sensors/data feed the twin? | 🟡 In progress |
| **4D — Spirit** 💪 | In what physical arrangement does it live? | 🔴 Awaiting field data |
| **HRES 4** 🧠 | Digital Shadow with synthetic data | ✅ Complete (this is v0.2.0) |
| **HRES 5** 🧠 | Pipeline replacement with real data | 🔴 Pending Ixil field campaign |
| **HRES 6** 👻 | Auto-correction | 🟡 Scaffold ready |
| **HRES 7** 👻 | ML predicts the future | 🟡 Now-casting works |

📖 **Full methodology overview:** [`docs/4D_methodology/`](docs/4D_methodology/)

### Where are we today? **Digital Shadow stage (HRES 4 → 5 transition)**

```
Digital Model  →  Digital Shadow  ←── YOU ARE HERE
   (static)        (1-way data)
                                    →  Digital Twin   →  Digital Thread
                                       (2-way data)      (auto-evolving)
```

---

## 🎯 What this repository delivers (Output 3.1)

| Deliverable | Status | Location |
|---|---|---|
| 📊 Open-access structured datasets | ✅ | `data/processed/` |
| 📓 Colab simulation prototype | ✅ | `notebooks/11_digital_twin_prototype.ipynb` |
| 🤖 Validated DT-HRES-S ML model | ✅ | `src/ml_models.py` (RF wins: R²=1.00, CV-RMSE=5.4%) |
| 📄 Technical report scaffold | 🟡 | `docs/`, `results/reports/` |

---

## 📦 The data

The `data/raw/SolarDataofMexicanCities.xlsx` file contains **Typical Meteorological Year (TMY)** data — 8,737 hourly records — for four Mexican cities representing distinct climate zones (Köppen classification):

| City | State | Köppen | GHI (kWh/m²/yr) | T̄ (°C) | v̄_wind (m/s) |
|---|---|---|---|---|---|
| **Monterrey** | Nuevo León | BSh — semi-arid hot | 1,878 | 25.1 | 4.5 |
| **Campeche** | Campeche | Aw — tropical savanna | 1,887 | 26.6 | 3.4 |
| **Mexico City** | CDMX | Cwb — subtropical highland | 1,840 | 16.4 | 2.6 |
| **San Ignacio** | Baja California Sur | BWh — hot desert | 2,155 | 21.6 | 2.2 |

📖 Full data dictionary: [`data/metadata/data_dictionary.md`](data/metadata/data_dictionary.md)

---

## 🏗 Repository structure

```
DT-HRES-S/
├── README.md
├── LICENSE
├── CITATION.cff
├── requirements.txt
├── data/                              ← TMY datasets
│   ├── raw/SolarDataofMexicanCities.xlsx
│   ├── processed/                     ← 4 cities clean CSVs
│   └── metadata/data_dictionary.md
├── notebooks/                         ← Colab notebooks
│   ├── 11_digital_twin_prototype.ipynb     ← END-TO-END demo
│   ├── 12_community_interface.ipynb        ← 2D ipywidgets sketch
│   └── 13_4D_methodology_walkthrough.ipynb ← Methodology tutorial
├── src/                               ← Python modules
│   ├── data_loader.py                 ← TMY loading
│   ├── pv_model.py                    ← PV physics (1D theory)
│   ├── wind_model.py                  ← Wind physics
│   ├── battery_model.py               ← Battery dynamics
│   ├── hres_simulator.py              ← Full HRES engine (HRES 4)
│   ├── ml_models.py                   ← DT/RF/SVM/NN (HRES 7)
│   ├── sensors.py                     ← 3D Mind: data abstraction
│   ├── telemetry.py                   ← Logging & tracking
│   ├── auto_correction.py             ← HRES 6: drift detection
│   └── uncertainty.py                 ← Confidence intervals
├── docs/
│   ├── architecture.md
│   └── 4D_methodology/                ← Conceptual framework
│       ├── README.md
│       ├── 01_concept_HRES.md
│       ├── 02_body_optimization.md
│       ├── 03_mind_sensors.md
│       ├── 04_spirit_physical_arrangement.md
│       ├── 05_universe_principles.md
│       ├── 06_roadmap_HRES1_to_7.md
│       └── 07_team_responsibilities.md
├── tests/
├── results/
└── community/
```

---

## 🚀 Quick start

### Option A — Google Colab (recommended)

Click the badge above, or open directly:
```
https://colab.research.google.com/github/Aaron-Cuevas/DT-HRES-S/blob/main/notebooks/11_digital_twin_prototype.ipynb
```

### Option B — Local installation
```bash
git clone https://github.com/Aaron-Cuevas/DT-HRES-S.git
cd DT-HRES-S
pip install -r requirements.txt
jupyter notebook notebooks/
```

---

## 👥 Task 3 team

Each member's primary responsibility is mapped to a 4D dimension. Full detail with concrete deliverables: [`docs/4D_methodology/07_team_responsibilities.md`](docs/4D_methodology/07_team_responsibilities.md).

| Member | Role | Dimension |
|---|---|---|
| **Víctor Cardeña** | Simulation Lead | 1D · Concept — physical models (PV, wind, battery) |
| **Daniel Leiva** | Module Leader | 1D · Concept — concept docs & cross-module consistency |
| **Aaron Cuevas** | Technical Lead | 2D · Body — repo structure & module integration |
| **Carlos Rodríguez Tenorio** | Data & Design Engineering | 2D · Body — community interface & objective metrics |
| **Samuel Canul** | Data Lead | 3D · Mind — NASA POWER source & Ixil TMY |
| **José Llashag** | ML Systems & Deployment | 3D · Mind — physical sensor source & deployment |
| **Arturo Cruz** | Integration & Reproducibility | 4D · Spirit — the 5 Universe principles |
| **Emilio Urbina** | Baseline DT/RF | 4D · Spirit — baseline models on HRES 4 |
| **Félix Valadez** | Simulation analytics | 4D · Spirit — simulation series analysis |
| **Regina Muñoz** | ML & Validation Lead | HRES 6–7 — validation & drift detection |
| **Miguel Garduño** | Benchmarking Engineer | HRES 6–7 — model comparison & physical checks |

*Luis Benvenuto and Roberto Pérez: to be assigned (pending profile).*

---

## 📄 License

Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).

## 🙏 Acknowledgments

Funded by **IEEE EPICS in IEEE Program (2025-2026)**. Methodology adapted from **Technical Arts (ITESM Student Chapter)** 4D framework for digital twins.
