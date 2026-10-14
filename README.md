# Collision-Free Optical Tweezer Rearrangement Engine

[![Launch Live Simulator](https://img.shields.io/badge/PLAY_LIVE-Interactive_Simulation-0284c7?style=for-the-badge&logo=googlechrome&logoColor=white)](https://Swatee-05.github.io/neutral-atom-tweezer-rearrangement/)

A motion-planning and routing pipeline for assembling defect-free neutral atom quantum registers from stochastically loaded optical tweezer arrays.

<p align="center">
  <img src="tweezer_game.gif" width="550" alt="Neutral Atom Rearrangement Simulation" />
</p>

---

## Interactive Simulation

Test the rearrangement solver live in your browser:  
👉 **[Launch Interactive Web Application](https://Swatee-05.github.io/neutral-atom-tweezer-rearrangement/)**

* **Auto-Solve Mode:** Solves matching and routes all atoms sequentially using the center-outward Hungarian + BFS algorithm.
* **Manual Control:** Click any loaded atom (Red) and select an empty target site (Blue) to test manual collision-free shuttling trajectories.

---

## Visual Key & System Schematics

| Marker | Element | Description |
| :---: | :--- | :--- |
| <span style="color:#38bdf8;">**O**</span> | **Target Register Site** | Designated locations for the target $8 \times 8$ (64-qubit) defect-free array. |
| <span style="color:#f43f5e;">**●**</span> | **Neutral Atom (Static)** | Atom trapped at rest inside a stationary optical tweezer dipole trap. |
| <span style="color:#10b981;">**●**</span> | **AOD Moving Shuttle** | Atom trapped in a dynamic optical tweezer steered via Acousto-Optic Deflectors. |
| <span style="color:#475569;">**·**</span> | **Reservoir Tweezer** | Empty optical dipole trap within the $12 \times 12$ reservoir grid. |
| <span style="color:#9333ea;">**--**</span> | **Perimeter Corridor** | Buffer lane outside static array boundaries for collision-free diversion. |

---

## Physical Motivation

In neutral atom quantum architectures, atoms are caught in microscopic optical dipole traps. Trap loading is fundamentally limited by **light-assisted collisional blockades**:
* When multiple atoms enter a focal volume under cooling light, pairwise collisions eject both atoms from the potential well.
* This results in an initial stochastic loading probability of $P \approx 50\% - 55\%$.

To construct a scalable, defect-free register without exponential state-preparation overhead, hardware protocols over-provision an initial static reservoir ($12 \times 12 = 144$ traps), image occupied sites, and rearrange atoms into a dense target geometry ($8 \times 8 = 64$ traps) using dynamic AOD lasers.

---

## Algorithmic Architecture

1. **Global Cost Optimization:** Solves the linear sum assignment problem via the Hungarian algorithm using squared Euclidean distance penalties:
   $$\min \sum_{i} \sum_{j} \|\mathbf{s}_i - \mathbf{t}_j\|_2^2 \cdot X_{ij}$$
2. **Center-Outward Motion Scheduling:** Sorts moves from target center outward to keep perimeter target slots clear of transit obstacles.
3. **Perimeter-Buffered BFS Routing:** Computes collision-free shortest paths avoiding occupied traps. If interior bottlenecks occur, trajectories route through an external perimeter corridor.

---

## Simulated AMO Hardware Benchmarks

| Metric | Simulated Benchmark | Physical Significance |
| :--- | :--- | :--- |
| **Array Geometry** | $12 \times 12 \rightarrow 8 \times 8$ | Over-provisioning ratio of $2.25:1$ |
| **Trap Pitch ($a$)** | $4.5\,\mu\text{m}$ | Standard objective resolution limit |
| **AOD Translation Speed** | $60\,\mu\text{m/ms}$ | Sub-motional heating adiabatic velocity |
| **Total Assembly Time** | $\approx 35 - 55\,\text{ms}$ | Completed well within trap coherence windows |
| **Vacuum Lifetime Survival** | $> 99.8\%$ | Minimal loss against background gas ($\tau_{\text{vac}} = 25\,\text{s}$) |
