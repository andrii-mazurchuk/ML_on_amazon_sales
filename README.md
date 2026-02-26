# Amazon ML Engineering Project

End-to-end machine learning engineering project built on the Amazon Sales Dataset.

Stage 0 focuses on environment setup, dependency management, and repository structure.

---

## 📦 Project Setup

### 1. Clone the repository

```bash
git clone https://github.com/yourname/amazon-ml-engineering.git
cd amazon-ml-engineering
```

---

### 2. Create virtual environment

```bash
python -m venv .venv
```

Activate it:

**Windows (PowerShell):**

```bash
.venv\Scripts\Activate.ps1
```

**Linux / Mac:**

```bash
source .venv/bin/activate
```

---

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

---

## 🧪 Smoke Test

Verify environment works:

```bash
python -c "import pandas; import sklearn"
```

If no errors appear → setup is successful.

---

## 📁 Repository Structure (Stage 0)

```
amazon-ml-engineering/
├── data/
├── notebooks/
├── src/
├── models/
├── reports/
├── tests/
├── requirements.txt
└── README.md
```

---

## 🎯 Stage Goal

Establish a reproducible Python environment and baseline project structure for future ML pipeline development.

---

## 🛠️ Tech Stack (Initial)

* Python 3.11+
* pandas
* NumPy
* scikit-learn
* matplotlib

---

## ✅ Status

Stage 0 — Environment & Tooling Baseline ✔️
    