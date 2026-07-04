#  dynamic-hospital-triage-astar

> An intelligent, time-sensitive emergency department queue optimizer that evolves the traditional Manchester Triage System using Artificial Intelligence.

![Python](https://img.shields.io/badge/python-3.8+-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![AI](https://img.shields.io/badge/AI-Bayesian%20Networks%20%7C%20A*%20Search-orange)

---

##  Overview

In real-world hospitals, the **Manchester Triage System** assigns static priority colors to patients based on initial vital signs. However, this system is inherently **static**: it does not automatically recalculate patient priority as they sit in the waiting room, which can lead to critical patient deterioration.

This project introduces a **Dynamic Manchester Triage System**. It uses a **Bayesian Network** to assess initial clinical severity based on discrete vital signs, combined with an **A* Search Algorithm** that models patient risk as an **exponential curve over time**. The system continuously predicts future deterioration to find the absolute mathematically optimal treatment agenda.

---

##  Benchmark Results

The chart below shows the **Total Accumulated Risk** across the different strategies. A lower index means less patient suffering and a safer waiting room environment.

![Strategy Comparison Chart](image_a150a9.png)

### Key Insights from the Data:
*   **Small Scale (5 Patients):** Even with few patients, A\* achieves the safest schedule with a total risk score of **8.93**, outperforming both FIFO (11.58) and Greedy Search (11.28).
*   **Large Scale (20 Patients):** In a crowded ER scenario, the traditional FIFO baseline scales dangerously up to a massive risk index of **1273.91**. 
*   **The AI Advantage:** The **A\* algorithm cuts the total risk by more than half** compared to FIFO, dropping it down to **610.33**. It also drastically outperforms the Greedy Search (917.22), proving that anticipating exponential timeline deterioration is mathematically safer than just treating the most severe current case.

---

##  How It Works

The architecture relies on three core AI and data structure components to process each patient's parameters (**Fever**, **Oxygen Saturation**, **Blood Pressure**, and **Initial Wait Time**):

### 1. Clinical Severity Prediction (Bayesian Network)
Instead of relying on rigid manual rules, vital signs are fed into a Bayesian Inference Network. This calculates a dynamic probability of high severity:
$$P(\text{High Severity} \mid \text{Evidence})$$

### 2. The Exponential Deterioration Engine
Patient risk is not linear. A patient with moderate symptoms waiting for 2 hours might suddenly become a higher priority than a severe patient who just walked through the door. The core decision engine drives this behavior via the following exponential formula:

$$\text{Risk} = P(\text{High Severity}) \times e^{\frac{\text{Initial Wait} + \text{Simulated Time}}{\tau}}$$

*   **$\tau$ (tau):** Coefficient controlling the acceleration speed of the deterioration curve.

### 3. Chronological Decision Tree (A* Search + Beam Search)
The A* algorithm maps out an optimal scheduling tree where each node represents a 10-minute treatment interval. It evaluates paths using:
$$f(n) = g(n) + h(n)$$

*   **$g(n)$ (Accumulated Cost):** The real mathematical harm/risk already suffered by all patients left waiting during previous steps.
*   **$h(n)$ (Heuristic):** The predicted future risk of the remaining patients in the queue (`self.pacientes`).
*   **Beam Search Optimization:** Since scheduling grows factorially ($n!$), evaluating 20 patients generates over $2.43 \times 10^{18}$ combinations. The algorithm uses a `BEAM_WIDTH = 100` boundary constraint to prune the tree, keeping the execution lightning-fast and memory-efficient.

---

##  Search Strategies Breakdown

| Algorithm | Strategy / Core Metric | Pros | Cons |
| :--- | :--- | :--- | :--- |
| **FIFO (Baseline)** | Chronological order of arrival (`tempo_espera_inicial`). No heuristic. | Fair by arrival time; trivial to implement. | Completely blind to clinical severity. |
| **Greedy Search** | Immediate highest severity ($P(\text{Alta})$). Uses custom inverted negative logic on a `heapq`. | Extremely fast; handles the single worst case instantly. | Short-sighted (myopic). Completely ignores time, allowing older moderate patients to exponentially deteriorate. |
| **A* + Beam Search** | Minimizes total cumulative risk over time ($g(n) + h(n)$). | Mathematically balances clinical urgency with waiting room time. | Requires state-space search scaling constraints (Pruned via Beam Search). |

>  **The `heapq` Twist:** Python’s built-in `heapq` module acts as our priority queue manager. Because a `heapq` naturally pops the *smallest* value first, the Greedy Search algorithm passes a **negative sum of probabilities** (`-sum(p.p_alta)`) to inverted-index the heap structure. This effortlessly forces the heaviest clinical risks straight to the top.

---

##  Getting Started

### Prerequisites
Make sure you have Python 3.8+ installed along with the required scientific computing and probabilistic graphical model libraries:

```bash
pip install numpy pgmpy
