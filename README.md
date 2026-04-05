# 🏋️ Personalized Gym Scheduling via a Hybrid CSP and Reinforcement Learning Framework

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-blue?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/Jupyter-Notebook-orange?style=for-the-badge&logo=jupyter&logoColor=white"/>
  <img src="https://img.shields.io/badge/scikit--learn-1.x-f7931e?style=for-the-badge&logo=scikit-learn&logoColor=white"/>
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge"/>
</p>

<p align="center">
  <a href="https://colab.research.google.com/drive/1FdLjSaljkcomQuZ0G6JxOZLe_mN9cIge?usp=sharing">
    <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab" height="28"/>
  </a>
</p>

> An AI-powered workout planner that combines **Constraint Satisfaction (CSP)** for safe, feasible scheduling with **Q-Learning (RL)** optimised by **similarity-based reward estimation** from a real gym population dataset.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [How It Works](#-how-it-works)
- [Pipeline Architecture](#-pipeline-architecture)
- [Dataset](#-dataset)
- [Key Features](#-key-features)
- [Results](#-results)
- [Installation](#-installation)
- [Usage](#-usage)
- [Project Structure](#-project-structure)
- [Team](#-team)

---

## 🧠 Overview

Most generic workout plans fail because they ignore the individual — their injury history, recovery capacity, available days, fitness level, and personal goals. Professional personal trainers who account for all of this are expensive and inaccessible to most people.

This project builds an **automated, personalised gym scheduler** that:

1. Generates a **feasible** weekly workout plan that respects hard safety constraints (no consecutive HIIT, injury restrictions, frequency limits).
2. **Optimises** that plan using Reinforcement Learning guided by evidence from 973 real gym members — so recommendations are grounded in what actually works for people with similar profiles.
3. Produces a **human-readable strategy interpretation** explaining the recommended workout style, frequency, and session structure.

---

## ⚙️ How It Works

The system runs a **three-stage hybrid pipeline** for each candidate:

### Stage 1 — Personalised CSP (Feasibility)
- Each day of the week is a **CSP variable** with a domain of `{Rest, Strength, Cardio, HIIT, Yoga}`.
- Domains are **personalised** based on the candidate's fitness level, injury flag, and recovery need (e.g., injured candidates have HIIT removed entirely).
- **AC-3** arc consistency prunes domains before search begins.
- **Backtracking with MRV heuristic** finds the first feasible 7-day schedule.

### Stage 2 — Q-Learning Policy Optimisation
- The initial schedule is converted into a **policy vector** (workout frequency, session duration, workout-type proportions).
- A **Q-learning agent** explores 8 actions: `keep_same`, `increase/decrease_frequency`, `increase/decrease_duration`, `swap_to_cardio`, `swap_to_strength`, `add_yoga_recovery`.
- **Reward** is estimated by retrieving the top-10 most similar gym members (by joint physical + strategy distance) and computing a health outcome metric from their records:

$$R = 0.004\,\bar{C} - 0.03\,\bar{B}_{rest} - 0.05\,\bar{F} + 0.8\,\bar{E} - 0.15\,|\bar{f} - 4|$$

### Stage 3 — RL-Guided CSP Re-Solve (Optimality)
- The RL-updated target policy **re-orders** each day's domain so preferred workout types appear first (soft guidance).
- A best-first backtracker finds the **highest-scoring feasible schedule** — hard constraints remain fully intact.

---

## 🏗️ Pipeline Architecture

```
Candidate Profile
       │
       ▼
┌─────────────────────────┐
│  Stage 1: CSP Solver    │  ──►  Initial Feasible Schedule S₀
│  (AC-3 + Backtracking)  │
└─────────────────────────┘
       │
       ▼  policy vector
┌─────────────────────────┐
│  Stage 2: Q-Learning    │  ──►  Optimised Target Policy π*
│  + KNN Reward from      │
│    Gym Dataset          │
└─────────────────────────┘
       │
       ▼  guided domains
┌─────────────────────────┐
│  Stage 3: RL-Guided     │  ──►  Final Optimised Schedule S*
│  CSP Re-Solve           │       + Strategy Interpretation τ
└─────────────────────────┘
```

---

## 📊 Dataset

| Property | Value |
|---|---|
| Source | Gym Members Exercise Tracking |
| Records | 973 members |
| Features | 15 (Age, Gender, Weight, BMI, BPM metrics, Calories, Workout Type, Frequency, Experience, etc.) |
| Physical feature dimensions | 10 |
| Strategy feature dimensions | 6 (after one-hot encoding) |
| Preprocessing | StandardScaler applied separately to physical and strategy spaces |

The dataset drives the **reward signal** — no simulated fitness model is needed. The system learns from what real people with similar bodies and strategies actually achieved.

---

## ✨ Key Features

- **Hard constraint enforcement** — zero violations across all generated schedules
  - No consecutive HIIT days
  - No consecutive high-intensity (Strength + HIIT) sessions
  - Frequency ceiling and floor respected
  - Injury and recovery restrictions applied at domain level

- **Personalised domains** — Beginners, Intermediate, and Advanced candidates each get a different allowable workout set per day

- **Evidence-based reward** — uses real gym population data, not hand-crafted fitness rules

- **Interpretable output** — strategy is expressed in plain English (dominant style, secondary style, target frequency)

- **Modular pipeline** — each stage (CSP, RL, retrieval) is independently testable

---

## 📈 Results

### Strategy Summary (10 Synthetic Candidates)

| Candidate | Fitness Level | Goal | Initial Freq | RL Target Freq | Dominant Style |
|---|---|---|:---:|:---:|---|
| C1 | Advanced | General Fitness | 4 | 6 | Cardio |
| C2 | Advanced | General Fitness | 4 | 2 | Cardio |
| C3 | Beginner | Weight Loss | 5 | 6 | Yoga |
| C4 | Advanced | Weight Loss *(injured)* | 4 | 4 | Cardio |
| C5 | Intermediate | General Fitness | 5 | 6 | Strength |
| C6 | Beginner | Muscle Gain | 5 | 6 | Yoga |
| C7 | Intermediate | Muscle Gain | 3 | 2 | Yoga |
| C8 | Intermediate | Weight Loss | 5 | 6 | Strength |
| C9 | Intermediate | General Fitness | 3 | 2 | Yoga |
| C10 | Intermediate | Weight Loss | 4 | 4 | HIIT |

### Performance Highlights

| Metric | Value |
|---|---|
| Pipeline success rate | **100%** (10/10 candidates) |
| Hard constraint violations | **0** |
| Reward improvement (C1, 25 episodes) | **+16.6%** (3.98 → 4.65) |
| RL training time per candidate | < 2 seconds |
| Total runtime (10 candidates) | ≈ 15 seconds |

### Notable Behaviours
- **C4 (injured, prefers HIIT)** — HIIT correctly blocked; RL recommends Cardio + Yoga
- **C5 (low recovery)** — RL introduces Yoga on alternating days despite low recovery flag, matching population evidence
- **C9 (Yoga-preferring, moderate recovery)** — RL converges to a light 2-day routine (Yoga + Cardio) to avoid overtraining
- **C10 (HIIT seeker, Weight Loss)** — HIIT-Cardio schedule maintained with minor Strength substitution

---

## 🚀 Installation

### Run on Google Colab (Recommended — no setup needed)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1FdLjSaljkcomQuZ0G6JxOZLe_mN9cIge?usp=sharing)

Click the badge above. The notebook is fully self-contained — just upload `gym_members_exercise_tracking.csv` when prompted, then run all cells.

### Run Locally

```bash
# Clone the repository
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name

# Install dependencies
pip install pandas numpy scikit-learn matplotlib seaborn

# Launch the notebook
jupyter notebook AI_Project_.ipynb
```

> **Python 3.10+** recommended. No additional packages beyond the standard data science stack are required.

---

## 💻 Usage

All logic lives inside `AI_Project_.ipynb`. The notebook is structured linearly — run all cells top to bottom.

**To run the full pipeline for all 10 candidates:**
```python
# Already wired up in Block 31 — just run the cell:
all_results = []
for i, candidate_row in candidates.iterrows():
    result = run_full_pipeline_for_candidate(candidate_row)
    if result:
        all_results.append(result)
```

**To run for a single custom candidate:**
```python
my_candidate = pd.Series({
    "candidate_id": "C_custom",
    "age": 28,
    "fitness_level": "Intermediate",   # Beginner / Intermediate / Advanced
    "goal": "Weight Loss",             # Weight Loss / Muscle Gain / Endurance / General Fitness
    "preferred_workout": "Cardio",     # Strength / Cardio / HIIT / Yoga
    "available_days": 4,
    "injury_flag": 0,                  # 0 or 1
    "recovery_need": "Medium"          # Low / Medium / High
})

result = run_full_pipeline_for_candidate(my_candidate)
```

**To inspect the RL learning history:**
```python
hist_df = result["history"]
plt.plot(hist_df["episode"], hist_df["reward"], marker="o")
plt.title("Reward Progression")
plt.show()
```

---

## 📁 Project Structure

```
📦 repo-root/
├── 📓 AI_Project_.ipynb                  # Main notebook (all code)
├── 📄 gym_members_exercise_tracking.csv  # Input dataset
├── 📄 final_gym_schedule_results.csv     # Output: all candidate schedules
└── 📄 README.md                          # This file
```

### Notebook Block Structure

| Block | Description |
|---|---|
| 1–2 | Imports |
| 3–4 | Load & inspect dataset |
| 5–6 | Clean columns, create candidate profiles |
| 7–13 | CSP: domain construction, AC-3, backtracking |
| 14–21 | Feature engineering, scalers, similarity retrieval |
| 22–28 | RL: state/action/reward, Q-learning loop |
| 29–35 | RL-guided CSP re-solve, full pipeline runner |
| 36–37 | Run all candidates, display results |
| 38 | Strategy summary table |
| 39–40 | RL history inspection & reward plot |
| 41–43 | Schedule comparison tables, CSV export |
| 44–54 | Per-candidate deep-dive analysis (C4, C5, C8, C9, C10) |

---

## 👥 Team

| Name | Roll Number | Department |
|---|---|---|
| Student Name 1 | Roll No. 1 | Department 1 |
| Student Name 2 | Roll No. 2 | Department 2 |
| Student Name 3 | Roll No. 3 | Department 3 |
| Student Name 4 | Roll No. 4 | Department 4 |

---

## 📄 License

This project is licensed under the MIT License.
