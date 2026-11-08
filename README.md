# 3D Container Loading: Classical Heuristic vs QUBO Hybrid

A small proof-of-concept project. I try to load rectangular packages into one container so that the **total weight loaded is as high as possible**. I start with a simple idea, check what goes wrong, and improve it step by step. At the end I compare a classical method with a QUBO (quantum-inspired) hybrid method, using the same data.

> **Please note:** this is a research prototype. It is **not** a certified cargo-loading system. The "quantum" part is a **classical simulated annealer**, so no quantum hardware is used and **no quantum advantage is claimed**.

---

## 1. What is in this folder

| File / folder | What it is |
|---|---|
| `container_optimisation_story.ipynb` | The main notebook. It tells the whole story: idea, code, result, what I observed, conclusion, next idea. |
| `README.md` | This file. |
| `Report` (separate document) | The detailed written report with all results and explanations. |
| `results/test_dataset.json` | The test data: three containers with their packages (small, medium, large). |
| `results/packages_small.csv`, `packages_medium.csv`, `packages_large.csv` | The same packages as simple tables. |
| `results/all_runs.csv` | Every single run: method, seed, packages packed, volume %, weight %, time, violations. |
| `results/summary.csv` | The final comparison table (mean and best over runs). |
| `results/journey_large_set.csv` | One line per version on the large dataset, in the order I tried them. |
| `results/packing_medium.png`, `results/packing_large.png` | 3D pictures of the final layouts. |
| `results/comparison_chart.png` | Bar charts: volume %, weight %, violations. |

The `results/` folder is created by the notebook when you run it.

---

## 2. How to run it

### Option A: Google Colab (easiest)

1. Open [colab.research.google.com](https://colab.research.google.com) and choose **Upload**, then select `container_optimisation_story.ipynb`.
2. Click **Runtime > Run all**.
3. Wait about **3 to 5 minutes**. Your first cell installs what is needed (it needs internet).
4. The tables and 3D pictures appear inside the notebook. The files appear in the `results/` folder on the left side.

### Option B: On your own computer

You need **Python 3.10 or newer**.

```bash
pip install numpy pandas matplotlib dimod dwave-neal ortools jupyter
jupyter notebook container_optimisation_story.ipynb
```

Then choose **Kernel > Restart & Run All**.

> If the first cell (`%pip install ...`) gives an error on your computer, install the packages with the `pip` line above and then skip that cell.

### What you should see

* All numbers should be the **same as in the report**, because the data and random seeds are fixed. Only the **seconds** will be different on your machine.
* The checker should show **0 violations** for v1.5, v2 and the QUBO methods, and **overlaps for v1**.

---

## 3. What the notebook does, step by step

| Step | What I try | Short result (large dataset: 80 packages) |
|---|---|---|
| 0 | Make repeatable test data (small, medium, large) | Same data every run |
| 1 | **v1**: a moving pointer places boxes one by one | Claims 99.9% volume, but **185 boxes overlap**, so it is not valid |
| 1b | Build the **checker** (the judge for every layout) | Used for all later versions |
| 2 | **v1.5**: remember placed boxes, pick rotation by a score | Valid, but only 51 of 80 packed (70.1% volume) |
| 3 | **v2**: remember the **empty space**, not the boxes | **63 packed, 87.2% volume, 0 violations**, same result every run |
| 3b | v2 with only the density order | Worse, so I dropped it |
| 3c | Compare with an **exact solver** on small cases | v2 matched the true best in all 5 cases |
| 4 | **QUBO v2**: annealer chooses *which* packages, v2 places them | Small (98 qubits) and valid, but not better than v2 |
| 5 | **QUBO v2 + density** in the energy | No clear gain |
| 6 | Final comparison: 3 test sets, 5 seeds, runtime, 3D plots | v2 is as good or better on weight |
| 7 | Conclusions and next ideas | See the report |

---

## 4. The three methods I compare at the end

| Tag | Method | Is it random? |
|---|---|---|
| **A** | v1 | No |
| **B** | v2 (my finalised classical method) | No, same input always gives same output |
| **C** | QUBO v2, weight only | Yes, so I run **5 seeds** (0 to 4) |
| **D** | QUBO v2, weight + density | Yes, so I run **5 seeds** (0 to 4) |

Every layout from every method is checked by the same **checker**, which counts:

* boxes sticking out of the container,
* overlapping pairs of boxes,
* total weight over the limit,
* a rotation that is not allowed,
* the same package placed twice.

---


## 5. Settings you can change

Everything is in the notebook cells. The main knobs are:

| Setting | Where | What it does |
|---|---|---|
| `TESTSETS` | Step 0 cell | Number of packages, container size, data seed, rotation share, weight limit |
| `min_support` | `deterministic_pack(...)` | 0.0 = off. 0.5 or 0.7 = at least that share of a box's bottom must rest on something. Higher is more stable but packs less. |
| `density_weight` (lambda) | `hybrid_pack(...)` | 0 = reward by weight only. Bigger = prefer packages that carry a lot of weight in little space. |
| `fracs` | `hybrid_pack(...)` | Volume budgets the annealer tries (default 1.0, 0.9, 0.8, 0.7 of the container). |
| `num_reads`, `num_sweeps` | `qubo_select(...)` | How much work the annealer does. More = slower, maybe better. |
| `QUBO_SEEDS` | Step 6 cell | Which random seeds to run (default 0 to 4). |

To try a **different solver** (for example a real quantum or other annealing backend), replace the one `neal.SimulatedAnnealingSampler().sample(...)` call inside `qubo_select`. Everything else stays the same.

---


## 6. Reproduction details

* Language: Python 3.10+
* Libraries: `numpy`, `pandas`, `matplotlib`, `dimod`, `dwave-neal`, `ortools`
* Data seeds: small = 11, medium = 22, large = 80
* QUBO seeds: 0, 1, 2, 3, 4 (all recorded in `results/all_runs.csv`)
* Same seed gives the same layout (the notebook checks this).
