# 🏋️ GymFlow: Hybrid CSP + RL Workout Planner
**M.Tech AI Project | IIT Kharagpur**

An intelligent scheduling system that balances safety (Constraint Satisfaction) with performance (Q-Learning). It doesn't just give you a plan; it learns what works by cross-referencing your profile against 970+ real-world gym members.

[**Launch on Colab**](https://colab.research.google.com/drive/1FdLjSaljkcomQuZ0G6JxOZLe_mN9cIge?usp=sharing) | [**View Results**](https://github.com/your-username/your-repo-name)

---

### 💡 The Core Problem
Generic workout apps usually fail at two things:
1. They ignore **safety constraints** (like "don't do HIIT two days in a row").
2. They aren't **evidence-based** (they use hardcoded rules rather than real human data).

**GymFlow** solves this by running a three-stage pipeline that ensures your plan is physically safe, then optimizes it using a reward signal derived from a dataset of actual fitness outcomes.

---

### 🛠 How the Engine Works

The pipeline is split into three distinct "brains":

#### 1. The Safety Brain (CSP)
We treat the week as a **Constraint Satisfaction Problem**. 
* **The Goal:** Find a 7-day sequence that is physically possible.
* **The Rules:** We use **AC-3 arc consistency** to prune impossible options (e.g., if you're injured, HIIT is purged from your "domain").
* **Result:** A baseline schedule that won't overtrain you.

#### 2. The Optimizer Brain (Q-Learning)
Once we have a safe plan, we refine it. An RL agent experiments with actions like `increase_frequency` or `add_yoga_recovery`.
* **The Reward Signal:** Instead of guessing if a plan is "good," we use a **K-Nearest Neighbors (KNN)** lookup on our dataset. We find 10 people most similar to you and see what worked for them.
* **The Math:** Rewards are calculated based on heart rate improvements, calorie burn, and adherence:

$$R = 0.004\bar{C} - 0.03\bar{B}_{rest} - 0.05\bar{F} + 0.8\bar{E} - 0.15|\bar{f} - 4|$$

#### 3. The Refiner
The optimized policy from the RL agent is fed back into the CSP solver. It re-ranks the workout types so that the most effective options are picked first, without ever breaking the original safety rules.

---

### 📊 Performance at a Glance
We tested the system on 10 diverse synthetic candidates (from injured beginners to elite athletes).

* **Reliability:** 100% success rate in finding valid schedules.
* **Safety:** **Zero** constraint violations (no back-to-back HIIT sessions).
* **Efficiency:** Average reward improvement of **+16.6%** after just 25 training episodes.
* **Speed:** The entire pipeline runs in **~1.5 seconds** per user.

---

### 📂 Project Architecture

* **The Data (Input):** A dataset of 970+ real-world gym members (`gym_members_exercise_tracking.csv`).
* **The Engine (Process):** A single notebook containing the CSP solver and Q-Learning agent (`AI_Project_.ipynb`). 
* **The Results (Output):** The final, personalized schedules generated for the test candidates (`final_gym_schedule_results.csv`).

---

### 🏃 Quick Start

1. **Open** the notebook in Colab.
2. **Upload** the `gym_members_exercise_tracking.csv` file when prompted.
3. **Run All**. The system will automatically generate schedules for the 10 test cases and plot the RL learning curves.

---

### 🏗 The Team
**IIT Kharagpur | Dept. of Artificial Intelligence**
* **Aishik Das** (25AI60R23)
* **Sarthak Dey** (25AI60R20)
* **Ritish Bhatt** (25AI60R11)
* **Adarsh Raj** (25AI60R25)
