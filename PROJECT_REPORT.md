# K.R. MANGALAM UNIVERSITY
### SCHOOL OF ENGINEERING & TECHNOLOGY
**DEPARTMENT OF COMPUTER SCIENCE & ENGINEERING**

---

# CAPSTONE PROJECT REPORT
## MINI-COURIER DELIVERY & ROUTE OPTIMIZATION PLANNER

**Course Code:** ETCCDA302  
**Course Title:** Design & Analysis of Algorithms (DAA)  
**Academic Year / Semester:** 3rd Semester, B.Tech CSE (FSD) – Section B  

```
========================================================================================
SUBMITTED BY:                                SUBMITTED TO:
Krishna Gupta                                Prof. Anjali Pal
Roll No: 2501350048                          Department of Computer Science & Engineering
Branch: B.Tech CSE (FSD)                     School of Engineering & Technology
Section: B                                   K.R. Mangalam University, Gurugram
========================================================================================
```

---

## CANDIDATE DECLARATION

I hereby declare that the capstone project work entitled **"Mini-Courier Delivery & Route Optimization Planner"** submitted to the Department of Computer Science & Engineering, School of Engineering & Technology, **K.R. Mangalam University**, is an authentic record of my own work carried out under the supervision and guidance of **Prof. Anjali Pal**.

The matter embodied in this assignment has not been submitted by me for the award of any other degree or diploma.

**Date:** 24 November 2026  
**Place:** Gurugram, Haryana  

**Krishna Gupta**  
Roll No: `2501350048`  
B.Tech CSE (FSD) – 3rd Semester, Section B  

---

## ACKNOWLEDGEMENT

I express my sincere gratitude and indebtedness to my esteemed course instructor, **Prof. Anjali Pal**, Department of Computer Science & Engineering, for her invaluable guidance, constructive advice, and insightful feedback throughout the duration of the course **Design & Analysis of Algorithms (ETCCDA302)**.

I would also like to thank the **School of Engineering & Technology, K.R. Mangalam University** for providing the academic infrastructure and computing environment that enabled the successful development and empirical benchmarking of this project.

Finally, I thank my peers and family for their unwavering support and encouragement.

---

## TABLE OF CONTENTS

1. [Executive Summary / Abstract](#1-executive-summary--abstract)
2. [Problem Statement & Mathematical Formulation](#2-problem-statement--mathematical-formulation)
3. [System Architecture & Algorithmic Workflow](#3-system-architecture--algorithmic-workflow)
4. [Phase-wise Implementation & Algorithm Design](#4-phase-wise-implementation--algorithm-design)
   - [Phase 1: Multi-Attribute Sorting (Merge Sort)](#phase-1-multi-attribute-sorting-merge-sort)
   - [Phase 2: Constrained Payload Selection (0/1 Knapsack DP)](#phase-2-constrained-payload-selection-01-knapsack-dp)
   - [Phase 3: Graph Modeling & Shortest Paths (Dijkstra)](#phase-3-graph-modeling--shortest-paths-dijkstra)
   - [Phase 4: Tour Construction (Nearest Neighbour TSP Approximation)](#phase-4-tour-construction-nearest-neighbour-tsp-approximation)
5. [Theoretical Complexity Analysis](#5-theoretical-complexity-analysis)
6. [Comprehensive Dry Run on Baseline Dataset](#6-comprehensive-dry-run-on-baseline-dataset)
   - [Step 1: Input Data & Stable Sort State](#step-1-input-data--stable-sort-state)
   - [Step 2: DP Table Computation & Backtracking](#step-2-dp-table-computation--backtracking)
   - [Step 3: Dijkstra Road Distance Matrix](#step-3-dijkstra-road-distance-matrix)
   - [Step 4: Greedy TSP Route Formulation](#step-4-greedy-tsp-route-formulation)
7. [Comparative Paradigms: Why Heuristics & DP are Essential](#7-comparative-paradigms-why-heuristics--dp-are-essential)
   - [DP vs Greedy Knapsack (Sub-optimality Counterexample)](#dp-vs-greedy-knapsack-sub-optimality-counterexample)
   - [Nearest Neighbour vs Exhaustive Permutations (NP-Hardness)](#nearest-neighbour-vs-exhaustive-permutations-np-hardness)
8. [Automated Verification & Unit Testing](#8-automated-verification--unit-testing)
9. [Empirical Runtime Evaluation & Scaling](#9-empirical-runtime-evaluation--scaling)
10. [Instructions to Run the Project](#10-instructions-to-run-the-project)
11. [Conclusion & Future Enhancements](#11-conclusion--future-enhancements)

---

## 1. EXECUTIVE SUMMARY / ABSTRACT

In contemporary logistics and last-mile supply chain management, delivery operations face two critical optimization bottlenecks:
1. **Capacity & Priority Knapsack:** A delivery van has a finite physical weight capacity ($W$), while the central depot holds dozens or hundreds of parcels with varying urgencies, delivery deadlines, and priority rankings.
2. **Routing & Travelling Salesperson Dilemma:** Once parcels are chosen, dispatchers must plan an efficient driving tour that minimizes total travel distance, begins and terminates at the central depot, and visits each unique destination exactly once.

This capstone project implements the **Mini-Courier Delivery & Route Optimization Planner**, an end-to-end algorithmic pipeline engineered in pure Python (standard library only) without external optimization black-boxes. The solution integrates four foundational algorithms:
- **Merge Sort (Divide-and-Conquer):** Pre-sorts consignments hierarchically by deadline ascending and priority descending in $\mathcal{O}(n \log n)$ time with guaranteed stability.
- **0/1 Knapsack via Bottom-Up Dynamic Programming:** Maximizes cumulative priority score within van capacity $W$ in $\mathcal{O}(n \cdot W)$ time, avoiding the proven sub-optimal traps of greedy heuristics.
- **Dijkstra's Algorithm (Greedy Single-Source Shortest Paths):** Computes exact shortest physical distances between the depot and all selected stops across arbitrary non-negative road networks.
- **Nearest Neighbour Heuristic (Greedy TSP Approximation):** Rapidly constructs a Hamiltonian delivery tour in $\mathcal{O}(k^2)$ polynomial time, sidestepping the $\mathcal{O}(k!)$ intractability of exact TSP permutations.

The system is validated through an automated 11-point test suite, empirical performance profiling across scaling inputs up to $n = 1000$ parcels, and comprehensive dry runs demonstrating mathematical optimality.

---

## 2. PROBLEM STATEMENT & MATHEMATICAL FORMULATION

### 2.1 The Last-Mile Delivery Problem
Let a courier distribution depot store a set of $n$ candidate parcels:
$$\mathcal{P} = \{p_1, p_2, \dots, p_n\}$$
Each parcel $p_i$ is characterized by the 5-tuple:
$$p_i = (\text{id}_i, \text{dest}_i, w_i, v_i, d_i)$$
Where:
- $\text{id}_i$: Unique identifier string (e.g., $P_1, P_2, \dots$).
- $\text{dest}_i \in \mathcal{V}$: Designated destination drop-off node.
- $w_i \in \mathbb{Z}^+$: Consignment mass in kilograms ($w_i > 0$).
- $v_i \in [1, 10]$: Normalized priority weight (10 = critically urgent).
- $d_i \in \mathbb{Z}^+$: Delivery SLA deadline in operating hours.

The delivery vehicle has a strict maximum gross payload weight $W$.

### 2.2 Sub-Problem 1: Optimal Payload Selection (0/1 Knapsack)
Find a subset of parcels $\mathcal{S} \subseteq \mathcal{P}$ that maximizes the total value:
$$\max \sum_{p_i \in \mathcal{S}} v_i$$
Subject to the physical capacity constraint:
$$\sum_{p_i \in \mathcal{S}} w_i \le W$$
Where parcel inclusion is strictly binary: $x_i \in \{0, 1\}$.

### 2.3 Sub-Problem 2: Tour Minimization (TSP over Shortest Paths)
Let $\mathcal{G} = (\mathcal{V}, \mathcal{E}, \omega)$ be an undirected, weighted graph representing the urban road network where $\mathcal{V}$ denotes road junctions/landmarks, $\mathcal{E}$ denotes bidirectional roadways, and $\omega(u, v) \in \mathbb{R}^+$ denotes road length in kilometers.

Let $\mathcal{K} \subseteq \mathcal{V}$ be the set of unique delivery stops required by the selected payload $\mathcal{S}$:
$$\mathcal{K} = \{\text{dest}_i \mid p_i \in \mathcal{S}\} \cup \{\text{Depot}\}$$
Find an ordered sequence (tour) $\mathcal{T} = (\tau_0, \tau_1, \tau_2, \dots, \tau_k, \tau_0)$ where $\tau_0 = \text{Depot}$, every node in $\mathcal{K}$ appears exactly once, such that the total traversed metric distance is minimized:
$$\min \mathcal{D}(\mathcal{T}) = \sum_{j=0}^{k-1} \text{dist}(\tau_j, \tau_{j+1}) + \text{dist}(\tau_k, \tau_0)$$
Where $\text{dist}(u, v)$ is the true shortest road path between $u$ and $v$ in $\mathcal{G}$.

---

## 3. SYSTEM ARCHITECTURE & ALGORITHMIC WORKFLOW

The program follows a modular five-stage data pipeline:

```
[ parcels.csv ]                                        [ roads.csv ]
       |                                                     |
       v                                                     v
+-----------------------------+               +-----------------------------+
|    Phase 1: Merge Sort      |               |      Graph Construction     |
| Order: deadline asc,        |               | Adjacency List:             |
|        priority desc        |               | u -> [(v, dist), ...]       |
+-----------------------------+               +-----------------------------+
       |                                                     |
       v                                                     |
+-----------------------------+                              |
|   Phase 2: 0/1 Knapsack     |                              |
| Bottom-Up Dynamic Prog.     |                              |
| Output: Packing Plan S      |                              |
+-----------------------------+                              |
       |                                                     |
       | Unique Delivery Stops K                             |
       +------------------------------------+                |
                                            |                |
                                            v                v
                               +-----------------------------------------+
                               |    Phase 3: Dijkstra Shortest Paths     |
                               | Single-Source all-pairs from Depot & K  |
                               +-----------------------------------------+
                                            |
                                            v
                               +-----------------------------------------+
                               |     Phase 4: Nearest Neighbour TSP      |
                               | Greedy Tour Formulation & Back to Depot |
                               +-----------------------------------------+
                                            |
                                            v
                               +-----------------------------------------+
                               | Phase 5: Tour Breakdown & Metrics Output|
                               | Turn-by-turn road paths & diagnostics   |
                               +-----------------------------------------+
```

---

## 4. PHASE-WISE IMPLEMENTATION & ALGORITHM DESIGN

### Phase 1: Multi-Attribute Sorting (Merge Sort)
- **File:** `sorting.py`
- **Rationale:** Standard quicksort exhibits worst-case $\mathcal{O}(n^2)$ behavior and is unstable in typical implementations. Merge Sort guarantees strict $\mathcal{O}(n \log n)$ time across all permutations and preserves the initial file relative order for ties (stable).
- **Comparator Logic:**
  ```python
  def comes_first(p1, p2):
      if p1["deadline"] != p2["deadline"]:
          return p1["deadline"] < p2["deadline"]  # earlier deadline first
      return p1["priority"] >= p2["priority"]     # higher priority tie-breaker
  ```
- **Recurrence:** $T(n) = 2T(n/2) + \Theta(n) \implies \Theta(n \log n)$ by Master Theorem Case 2.

### Phase 2: Constrained Payload Selection (0/1 Knapsack DP)
- **File:** `knapsack.py`
- **Rationale:** Fractional knapsack is solvable greedily by value/weight ratio, but 0/1 knapsack is NP-complete. Dynamic programming yields the optimal subset in pseudo-polynomial time.
- **State Definition:** Let $DP[i][w]$ denote the maximum priority achievable considering the first $i$ sorted parcels under residual capacity $w$.
- **Bellman Recurrence Relation:**
  $$DP[i][w] = \begin{cases} 
  0 & \text{if } i = 0 \text{ or } w = 0 \\
  DP[i-1][w] & \text{if } w_i > w \\
  \max\Big(DP[i-1][w], \; DP[i-1][w - w_i] + v_i\Big) & \text{if } w_i \le w 
  \end{cases}$$
- **Backtracking Reconstruction:** Scans backwards from $DP[n][W]$ to $DP[0][0]$: if $DP[i][w] \neq DP[i-1][w]$, parcel $p_i$ was included; we deduct $w_i$ from $w$ and decrement $i$.

### Phase 3: Graph Modeling & Shortest Paths (Dijkstra)
- **File:** `route.py`
- **Representation:** Adjacency list via Python dictionary mapping nodes to lists of tuples `(neighbour, distance)`:
  $$\text{Space Complexity} = \Theta(V + E)$$
- **Algorithm:** Dijkstra's single-source shortest path using edge relaxation. Since all edge weights represent real-world physical distances ($\omega(e) \ge 0$), negative cycles cannot exist.
- **Predecessor Tracking:** Maintains `previous[v]` mapping to reconstruct complete step-by-step vertex paths between any pair of nodes.

### Phase 4: Tour Construction (Nearest Neighbour TSP Approximation)
- **File:** `route.py`
- **Rationale:** Finding the absolute minimal Hamiltonian cycle across $k$ stops is NP-hard. Generating all permutations has factorial complexity $\mathcal{O}(k!)$, which becomes intractable for $k \ge 12$.
- **Greedy Approximation Algorithm:**
  1. Start at `current = Depot`, mark `Depot` visited.
  2. While unvisited stops remain:
     - Scan all unvisited stops $u \in \text{Unvisited}$.
     - Choose stop $u^*$ that minimizes $\text{dist}(\text{current}, u^*)$.
     - Append $u^*$ to tour, accumulate distance, set `current = u^*`.
  3. Append `Depot` to close the loop, adding $\text{dist}(\text{current}, \text{Depot})$.
- **Approximation Ratio:** Guarantees an $\mathcal{O}(\log k)$ factor approximation for general metric TSP.

---

## 5. THEORETICAL COMPLEXITY ANALYSIS

| Pipeline Stage | Algorithm Employed | Time Complexity (Best) | Time Complexity (Average) | Time Complexity (Worst) | Space Complexity |
|---|---|---|---|---|---|
| **Parcel Prioritization** | Merge Sort | $\Omega(n \log n)$ | $\Theta(n \log n)$ | $\mathcal{O}(n \log n)$ | $\mathcal{O}(n)$ auxiliary |
| **Payload Optimization** | 0/1 Knapsack (DP) | $\Omega(n \cdot W)$ | $\Theta(n \cdot W)$ | $\mathcal{O}(n \cdot W)$ | $\mathcal{O}(n \cdot W)$ table |
| **Shortest Path Tree** | Dijkstra (Array/Dict) | $\Omega(V^2)$ | $\Theta(V^2)$ | $\mathcal{O}(k \cdot V^2)$ total | $\mathcal{O}(V)$ |
| **TSP Tour Generation** | Nearest Neighbour | $\Omega(k^2)$ | $\Theta(k^2)$ | $\mathcal{O}(k^2)$ | $\mathcal{O}(k)$ |
| **Overall Execution** | Composite System | $\mathcal{O}(n \log n + nW + kV^2 + k^2)$ | $\mathcal{O}(n \log n + nW + kV^2 + k^2)$ | $\mathcal{O}(n \log n + nW + kV^2 + k^2)$ | $\mathcal{O}(nW + V)$ |

*Where:*  
- $n = |\mathcal{P}|$ (total parcel inventory)  
- $W$ = Maximum vehicle carrying capacity (kg)  
- $V = |\mathcal{V}|$ (road network vertices/intersections)  
- $E = |\mathcal{E}|$ (road network edges/links)  
- $k = |\mathcal{K}|$ (unique delivery stops visited)  

---

## 6. COMPREHENSIVE DRY RUN ON BASELINE DATASET

### Step 1: Input Data & Stable Sort State
**Van Capacity:** $W = 20\text{ kg}$  
**Raw Parcels (Unsorted):**
- $P_1$: Dest=C, Wt=6, Pri=8, Ddl=2
- $P_2$: Dest=A, Wt=3, Pri=5, Ddl=4
- $P_3$: Dest=E, Wt=9, Pri=9, Ddl=3
- $P_4$: Dest=B, Wt=2, Pri=3, Ddl=6
- $P_5$: Dest=F, Wt=5, Pri=7, Ddl=1
- $P_6$: Dest=D, Wt=8, Pri=6, Ddl=5
- $P_7$: Dest=A, Wt=4, Pri=6, Ddl=2
- $P_8$: Dest=E, Wt=7, Pri=4, Ddl=8
- $P_9$: Dest=C, Wt=1, Pri=2, Ddl=7
- $P_{10}$: Dest=B, Wt=5, Pri=5, Ddl=4

**After Merge Sort (Deadline Ascending, Priority Descending):**
```
Index  ID   Dest  Weight  Priority  Deadline  Sorting Reason
1      P5   F     5       7         1         Earliest deadline (1h)
2      P1   C     6       8         2         Deadline 2h, Priority 8 > 6
3      P7   A     4       6         2         Deadline 2h, Priority 6
4      P3   E     9       9         3         Deadline 3h
5      P2   A     3       5         4         Deadline 4h, preserved initial order
6      P10  B     5       5         4         Deadline 4h, stable sort tie
7      P6   D     8       6         5         Deadline 5h
8      P4   B     2       3         6         Deadline 6h
9      P9   C     1       2         7         Deadline 7h
10     P8   E     7       4         8         Deadline 8h
```

---

### Step 2: DP Table Computation & Backtracking

**Computed Dynamic Programming Table ($DP[i][w]$):**
Rows = Parcels ($1 \dots 10$), Columns = Capacity ($0 \dots 20\text{ kg}$)

```
capacity ->   0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20
no parcel     0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0
P5 (w=5,p=7)  0  0  0  0  0  7  7  7  7  7  7  7  7  7  7  7  7  7  7  7  7
P1 (w=6,p=8)  0  0  0  0  0  7  8  8  8  8  8 15 15 15 15 15 15 15 15 15 15
P7 (w=4,p=6)  0  0  0  0  6  7  8  8  8 13 14 15 15 15 15 21 21 21 21 21 21
P3 (w=9,p=9)  0  0  0  0  6  7  8  8  8 13 14 15 15 15 16 21 21 21 22 23 24
P2 (w=3,p=5)  0  0  0  5  6  7  8 11 12 13 14 15 18 19 20 21 21 21 26 26 26
P10 (w=5,p=5) 0  0  0  5  6  7  8 11 12 13 14 15 18 19 20 21 21 23 26 26 26
P6 (w=8,p=6)  0  0  0  5  6  7  8 11 12 13 14 15 18 19 20 21 21 23 26 26 26
P4 (w=2,p=3)  0  0  3  5  6  8  9 11 12 14 15 16 18 19 21 22 23 24 26 26 29
P9 (w=1,p=2)  0  2  3  5  7  8 10 11 13 14 16 17 18 20 21 23 24 25 26 28 29
P8 (w=7,p=4)  0  2  3  5  7  8 10 11 13 14 16 17 18 20 21 23 24 25 26 28 29
```

**Backtracking Trace:**
- Start at cell $(P_8, 20) = 29$. Row above is $(P_9, 20) = 29 \implies P_8$ **NOT taken**.
- At cell $(P_9, 20) = 29$. Row above is $(P_4, 20) = 29 \implies P_9$ **NOT taken**.
- At cell $(P_4, 20) = 29$. Row above is $(P_6, 20) = 26$. Differ! $\implies$ **$P_4$ TAKEN**.  
  Residual capacity $w = 20 - 2 = 18$.
- At cell $(P_6, 18) = 26$. Row above is $(P_{10}, 18) = 26 \implies P_6$ **NOT taken**.
- At cell $(P_{10}, 18) = 26$. Row above is $(P_2, 18) = 26 \implies P_{10}$ **NOT taken**.
- At cell $(P_2, 18) = 26$. Row above is $(P_3, 18) = 21$. Differ! $\implies$ **$P_2$ TAKEN**.  
  Residual capacity $w = 18 - 3 = 15$.
- At cell $(P_3, 15) = 21$. Row above is $(P_7, 15) = 21 \implies P_3$ **NOT taken**.
- At cell $(P_7, 15) = 21$. Row above is $(P_1, 15) = 15$. Differ! $\implies$ **$P_7$ TAKEN**.  
  Residual capacity $w = 15 - 4 = 11$.
- At cell $(P_1, 11) = 15$. Row above is $(P_5, 11) = 7$. Differ! $\implies$ **$P_1$ TAKEN**.  
  Residual capacity $w = 11 - 6 = 5$.
- At cell $(P_5, 5) = 7$. Row above is $0$. Differ! $\implies$ **$P_5$ TAKEN**.  
  Residual capacity $w = 5 - 5 = 0$.

**Packing Plan Result:**
- **Selected:** $P_5, P_1, P_7, P_2, P_4$
- **Total Payload Weight:** $5 + 6 + 4 + 3 + 2 = 20 / 20\text{ kg}$ (100% capacity utilization)
- **Total Maximized Priority:** $7 + 8 + 6 + 5 + 3 = 29$
- **Deferred to Next Trip:** $P_3, P_{10}, P_6, P_9, P_8$

---

### Step 3: Dijkstra Road Distance Matrix

Road topology contains edges:  
`Depot-A(4)`, `Depot-B(2)`, `A-B(1)`, `A-C(5)`, `B-C(8)`, `B-D(10)`, `C-D(3)`, `C-E(2)`, `D-E(4)`, `D-F(6)`, `E-F(3)`

**Shortest Path Calculations from Depot:**
- **To B:** Direct edge $\text{Depot} \to \text{B} = 2\text{ km}$.
- **To A:** Via B: $\text{Depot} \to \text{B}(2) \to \text{A}(1) = \mathbf{3\text{ km}}$ (Shorter than direct road of 4 km!).
- **To C:** $\text{Depot} \to \text{B}(2) \to \text{A}(1) \to \text{C}(5) = \mathbf{8\text{ km}}$.
- **To D:** $\text{Depot} \to \text{B}(2) \to \text{A}(1) \to \text{C}(5) \to \text{D}(3) = \mathbf{11\text{ km}}$.
- **To E:** $\text{Depot} \to \text{B}(2) \to \text{A}(1) \to \text{C}(5) \to \text{E}(2) = \mathbf{10\text{ km}}$.
- **To F:** $\text{Depot} \to \dots \to \text{E}(10) \to \text{F}(3) = \mathbf{13\text{ km}}$.

---

### Step 4: Greedy TSP Route Formulation

Selected destinations: **F, C, A, B** (Note: A has two parcels $P_7, P_2$ but is visited only once).

1. **At Depot:** Distances to stops $\to \text{B}=2, \text{A}=3, \text{C}=8, \text{F}=13$.  
   $\implies$ Minimum is **B** ($2\text{ km}$).
2. **At B:** Distances to remaining $\{A, C, F\} \to \text{A}=1, \text{C}=6, \text{F}=11$.  
   $\implies$ Minimum is **A** ($1\text{ km}$).
3. **At A:** Distances to remaining $\{C, F\} \to \text{C}=5, \text{F}=10$.  
   $\implies$ Minimum is **C** ($5\text{ km}$).
4. **At C:** Distances to remaining $\{F\} \to \text{F}=5$ (via path $C \to E \to F$).  
   $\implies$ Visit **F** ($5\text{ km}$).
5. **At F:** All delivery stops completed. Return to Depot: $\text{dist}(\text{F}, \text{Depot}) = 13\text{ km}$.

**Final Delivery Route Summary:**
$$\text{Depot} \xrightarrow{2\text{km}} \text{B} \xrightarrow{1\text{km}} \text{A} \xrightarrow{5\text{km}} \text{C} \xrightarrow{5\text{km}} \text{F} \xrightarrow{13\text{km}} \text{Depot}$$
- **Total Route Distance:** $2 + 1 + 5 + 5 + 13 = \mathbf{26\text{ km}}$
- **Full Physical Waypoint Path:**  
  `Depot -> B -> A -> C -> E -> F -> E -> C -> A -> B -> Depot`

---

## 7. COMPARATIVE PARADIGMS: WHY HEURISTICS & DP ARE ESSENTIAL

### DP vs Greedy Knapsack (Sub-optimality Counterexample)
If a dispatcher uses a Greedy heuristic sorting by priority:
- Parcel $P_3$ has the highest priority (9), but weighs 9 kg.
- Greedy chooses $P_3$ (wt=9, pri=9).
- Next highest: $P_1$ (wt=6, pri=8), remaining space = 5 kg.
- Next highest: $P_5$ (wt=5, pri=7), remaining space = 0 kg.
- **Greedy Total Priority:** $9 + 8 + 7 = \mathbf{24}$.
- **Dynamic Programming Optimal Total:** $\mathbf{29}$ ($P_5 + P_1 + P_7 + P_2 + P_4$).  
**Conclusion:** Greedy yields a $17.2\%$ inferior payload because it ignores packing combinations. Dynamic Programming is strictly mathematically necessary.

### Nearest Neighbour vs Exhaustive Permutations (NP-Hardness)
- For $k = 4$ stops: $(4)! = 24$ permutations (checked in $\sim 0.05\text{ ms}$). Nearest Neighbour matches the exact optimum ($26\text{ km}$).
- For $k = 10$ stops (Medium Dataset):
  - Nearest Neighbour: $105\text{ km}$ computed in **0.03 ms**.
  - Brute Force ($10! = 3,628,800$ permutations): $95\text{ km}$ computed in **9354 ms** (over 9 seconds!).
  - **Trade-off:** Nearest Neighbour achieved a solution within $10.5\%$ of theoretical optimum while executing **300,000 times faster**.

---

## 8. AUTOMATED VERIFICATION & UNIT TESTING

The test suite (`test_cases.py`) validates system invariants through 11 automated assertions:

```
============================================================
                      TEST CASES
============================================================
PASS --> Merge sort order is same as Python sorted() for 300 parcels
PASS --> Knapsack DP answer is same as brute force (20 random cases)
PASS --> Knapsack with capacity 0 selects nothing
PASS --> Knapsack with very big capacity selects all parcels
PASS --> Small dataset : knapsack picks P5, P1, P7, P2, P4 (priority 29)
PASS --> Dijkstra distances from Depot are same as hand calculation
PASS --> Dijkstra gives infinity for a place that is not connected
PASS --> Route starts and ends at Depot
PASS --> Route visits every stop exactly once
PASS --> Small dataset : total route length is 26 km
PASS --> Small dataset : nearest neighbour route is same as the best route
------------------------------------------------------------
Passed : 11   Failed : 0
```

---

## 9. EMPIRICAL RUNTIME EVALUATION & SCALING

Execution benchmarking across scaling synthetic benchmarks ($n = 10$ to $1000$ parcels):

```
Parcels  Points  Capacity  Merge Sort(ms)  Knapsack DP(ms)  Dijkstra(ms)  NN-TSP(ms)  Total(ms)
10       6       20        0.014           0.025            0.036         0.004       0.078
50       10      50        0.062           0.262            0.146         0.007       0.476
100      20      100       0.177           1.224            0.670         0.020       2.091
500      50      500       1.014           35.529           7.978         0.157       44.678
1000     100     1000      2.180           148.893          51.625        0.485       203.183
```

### Key Observations:
1. **Knapsack DP dominates runtime at scale:** As both $n$ and $W$ scale by $10\times$ ($100 \to 1000$), table operations scale quadratically by $\sim 100\times$ ($1.22\text{ ms} \to 148.89\text{ ms}$).
2. **Merge Sort exhibits ideal scalability:** Scaling from 10 to 1000 parcels requires barely $2.18\text{ ms}$, validating the low constants of $\mathcal{O}(n \log n)$.
3. **Nearest Neighbour TSP is negligible in runtime:** Requires $< 0.5\text{ ms}$ even for 100 delivery stops.

---

## 10. INSTRUCTIONS TO RUN THE PROJECT

### 10.1 System Requirements
- Python 3.8+ (Tested on Python 3.12)
- Operating System: Windows, macOS, or Linux
- External Dependencies: None required for core solver (optional `matplotlib` for chart visualization).

### 10.2 Command Execution

1. **Execute Default Baseline Pipeline:**
   ```bash
   python main.py
   ```

2. **Execute with Custom Datasets & Capacity:**
   ```bash
   python main.py data/parcels_medium.csv data/roads_medium.csv 60
   ```

3. **Run Full Verification Test Suite:**
   ```bash
   python test_cases.py
   ```

4. **Run Synthetic Benchmark & Scaling Generator:**
   ```bash
   python performance_test.py
   ```

5. **Generate New Synthetic Datasets:**
   ```bash
   python generate_data.py
   ```

---

## 11. CONCLUSION & FUTURE ENHANCEMENTS

### Conclusion
This project successfully solves the combined knapsack packing and vehicle routing challenge by synthesizing four core algorithms studied in **ETCCDA302 (Design & Analysis of Algorithms)**. The integration of stable Merge Sort, exact Dynamic Programming, single-source Dijkstra, and heuristic Nearest Neighbour produces a mathematically sound, highly performant, and resilient courier scheduling system.

### Future Scope & Algorithmic Improvements
1. **Min-Heap Priority Queue for Dijkstra:** Replacing linear scan with Python's `heapq` reduces Dijkstra complexity from $\mathcal{O}(V^2)$ to $\mathcal{O}((V + E) \log V)$, offering massive speedups on sparse continental-scale road networks.
2. **2-Opt Tour Post-Processing:** Applying 2-opt edge-exchange heuristics to the Nearest Neighbour route untangles crossing paths and closes the $10.5\%$ optimality gap on larger graphs.
3. **1D Space-Optimized Knapsack:** Using rolling 1D state arrays reduces knapsack auxiliary memory from $\mathcal{O}(n \cdot W)$ to $\mathcal{O}(W)$.
4. **Time-Window Constrained Routing (VRPTW):** Incorporating arrival time constraints into route sequencing to guarantee delivery before parcel deadlines expire.

---

**Report Prepared By:**  
**KRISHNA GUPTA**  
Enrollment No: `2501350048`  
B.Tech CSE (FSD), 3rd Semester, Section B  
K.R. Mangalam University  
Course: ETCCDA302 (DAA)  
Faculty: Prof. Anjali Pal  
