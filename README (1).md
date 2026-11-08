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
| **A** | v1, the cursor idea | No |
| **B** | v2, free-space method (my finalised classical method) | No, same input always gives same output |
| **C** | QUBO v2, weight only | Yes, so I run **5 seeds** (0 to 4) |
| **D** | QUBO v2, weight + density | Yes, so I run **5 seeds** (0 to 4) |

Every layout from every method is checked by the same **checker**, which counts:

* boxes sticking out of the container,
* overlapping pairs of boxes,
* total weight over the limit,
* a rotation that is not allowed,
* the same package placed twice.

---

## 5. Main results (final comparison)

Mean over runs. "Weight %" is packed weight divided by the container's weight limit. "Violations" is the total over all runs.

| Test set | Method | Packed | Volume % | Weight % | Time (s) | Violations |
|---|---|---|---|---|---|---|
| small (12 pkgs) | A v1 | 7 | 62.1 | 100.0 | ~0 | **8** |
| | B v2 | 8 | 62.7 | 100.0 | 0.01 | 0 |
| | C QUBO (weight) | 8.2 | 62.2 | 99.3 | 0.38 | 0 |
| | D QUBO (weight+density) | 8.0 | 58.3 | 99.5 | 0.40 | 0 |
| medium (20 pkgs) | A v1 | 13 | 67.6 | 99.4 | ~0 | **18** |
| | B v2 | 13 | 71.1 | 100.0 | 0.07 | 0 |
| | C QUBO (weight) | 14.2 | 70.2 | 99.9 | 0.60 | 0 |
| | D QUBO (weight+density) | 14.0 | 69.9 | 99.6 | 0.63 | 0 |
| large (80 pkgs) | A v1 | 64 | 99.9 | 78.0 | ~0 | **185** |
| | B v2 | 63 | 87.2 | 76.6 | 4.3 | 0 |
| | C QUBO (weight) | 66.0 | 81.0 | 73.1 | 6.0 | 0 |
| | D QUBO (weight+density) | 65.2 | 81.1 | 73.6 | 5.5 | 0 |

**In one sentence:** v1 gives invalid layouts, v2 is valid, fast and repeatable, and the QUBO hybrid is also valid but did **not** beat v2 on quality or on speed.

---

## 6. Settings you can change

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

## 7. Known limits (please read)

* Only **3 test sets** and **5 seeds**. Differences of a few points are a hint, not a proof.
* My methods maximise **weight**. The project brief measures **volume**. The ranking could change if I optimise volume. The tables show both.
* The QUBO "volume budget" is only a rough guess of whether packages will fit. The real fit is decided by v2.
* The annealer is **classical simulated annealing**, not a quantum computer.
* No load-bearing physics, no centre of gravity, no multiple containers. Support is a simple area check and is **off** in the experiments.
* Units are plain numbers. The containers are cubes, not the 5.90 x 2.35 x 2.39 m container from the brief.
* Run times depend on the machine.
* In the 3D pictures, some package ids are hidden behind other boxes and are tiny on the 80-package picture.

---

## 8. Small dictionary

| Word | Meaning in simple terms |
|---|---|
| **Heuristic** | A rule of thumb that gives a good answer fast, but with no guarantee it is the best. |
| **Deterministic** | Same input always gives the same output. No randomness. |
| **Seed** | A starting number for a random generator. Same seed gives the same "random" numbers, so results can be repeated. |
| **Checker / validator** | A separate judge that looks at a finished layout and counts rule violations. |
| **Free space** | An empty box-shaped region inside the container where a package may still go. |
| **Support** | How much of a box's bottom rests on the floor or on other boxes. |
| **Density** | Weight divided by volume: how much weight a package carries for the space it takes. |
| **QUBO** | A way to write a problem with yes/no variables and one energy formula. Lowest energy is the best answer. Rules become penalties. |
| **Qubit** | Here, simply one yes/no variable in the QUBO. |
| **Slack qubits** | Extra yes/no variables that soak up leftover room under a limit, so "at most 800" can be written as a simple sum. |
| **Annealer** | A solver that searches for the lowest energy, a bit like slowly cooling metal: early on it accepts random changes, later only good ones. |
| **Exact solver (CP-SAT)** | A solver that finds the true best answer on small cases. I use it as a yardstick. |
| **Volume % / Weight %** | Packed volume / container volume, and packed weight / weight limit. |

---

## 9. Reproduction details

* Language: Python 3.10+
* Libraries: `numpy`, `pandas`, `matplotlib`, `dimod`, `dwave-neal`, `ortools`
* Data seeds: small = 11, medium = 22, large = 80
* QUBO seeds: 0, 1, 2, 3, 4 (all recorded in `results/all_runs.csv`)
* Same seed gives the same layout (the notebook checks this).
