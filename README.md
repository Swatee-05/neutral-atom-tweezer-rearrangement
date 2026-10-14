# Collision-Free Optical Tweezer Rearrangement Engine for Neutral Atom Quantum Registers

[![Launch Live Simulator](https://img.shields.io/badge/PLAY_LIVE-Interactive_Simulation-0284c7?style=for-the-badge&logo=googlechrome&logoColor=white)](https://Swatee-05.github.io/neutral-atom-tweezer-rearrangement/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

A deterministic motion-planning and routing pipeline designed to assemble defect-free $N$-qubit neutral atom quantum registers from stochastically loaded optical tweezer arrays.

<p align="center">
  <img src="tweezer_game.gif" width="550" alt="Neutral Atom Rearrangement Simulation" />
</p>

---

## Interactive Client & Web Visualizer

Run and test the algorithmic engine directly in your browser:  
👉 **[Launch Interactive Web Simulator](https://Swatee-05.github.io/neutral-atom-tweezer-rearrangement/)**

* **Autonomous Execution:** Solves bipartite matching and routes atoms sequentially using center-outward heuristic scheduling and perimeter-buffered breadth-first search.
* **Manual Control:** Click any static atom (Red) and select an open target trap (Blue ring) to manually execute collision-free shuttling trajectories.

---

## Visual Key & System Schematics

| Marker | Nomenclature | Physical Representation |
| :---: | :--- | :--- |
| <span style="color:#38bdf8; font-weight:bold;">O</span> | **Target Register Site** | Designated spatial coordinates for the $8 \times 8$ ($K = 64$) defect-free quantum register. |
| <span style="color:#f43f5e; font-weight:bold;">●</span> | **Neutral Atom (Stationary)** | Individual neutral atom held at rest within a static optical dipole potential well. |
| <span style="color:#10b981; font-weight:bold;">●</span> | **Dynamic AOD Shuttle** | Atom transferred into a mobile optical tweezer actively translated across the array. |
| <span style="color:#475569; font-weight:bold;">·</span> | **Empty Reservoir Trap** | Vacant optical potential well within the $12 \times 12$ ($M = 144$) stochastic reservoir. |
| <span style="color:#fbbf24; font-weight:bold;">●</span> | **Selected Atom** | Active entity designated for manual trajectory execution in user-interaction mode. |
| <span style="color:#9333ea; font-weight:bold;">- -</span> | **Perimeter Buffer Corridor** | Exterior boundary transit corridor ($\Delta r = \pm 1$) bypassing interior potential wells. |

---

## 1. Physical Foundations: Optical Dipole Traps (Tweezers)

An optical tweezer is an attractive potential well created by focusing a Gaussian laser beam through a high-numerical-aperture ($\text{NA} \ge 0.5$) microscope objective down to a diffraction-limited beam waist $w_0 \approx 0.7 - 1.0\,\mu\text{m}$.

### 1.1 The Optical Dipole Force (AC Stark Shift)
When an electric field $\mathbf{E}(\mathbf{r}, t) = \hat{\mathbf{e}} E(\mathbf{r}) e^{-i\omega_L t} + \text{c.c.}$ interacts with an isolated neutral alkali atom (e.g., $^{87}\text{Rb}$, $^{133}\text{Cs}$) or alkaline-earth atom ($^{171}\text{Yb}$), it induces an oscillating atomic electric dipole moment:

$$\mathbf{d}(\mathbf{r}, t) = \alpha(\omega_L) \mathbf{E}(\mathbf{r}, t)$$

where $\alpha(\omega_L)$ is the dynamic atomic polarizability at laser frequency $\omega_L$. The interaction between the induced dipole and the electric field yields the optical dipole potential (the light shift or AC Stark shift):

$$U_{\text{dip}}(\mathbf{r}) = -\frac{1}{2} \langle \mathbf{d} \cdot \mathbf{E} \rangle = -\frac{1}{2\epsilon_0 c} \text{Re}[\alpha(\omega_L)] I(\mathbf{r})$$

For a two-level atomic transition with resonant frequency $\omega_0$, natural decay rate $\Gamma$, and laser detuning $\Delta = \omega_L - \omega_0$, the potential evaluates to:

$$U_{\text{dip}}(\mathbf{r}) = \frac{3\pi c^2}{2 \omega_0^3} \left( \frac{\Gamma}{\Delta} \right) I(\mathbf{r})$$

* **Far Red Detuned ($\omega_L < \omega_0 \implies \Delta < 0$):** The dipole potential is negative ($U_{\text{dip}} < 0$). Atoms are conservatively attracted toward intensity maxima (the laser focal center).
* **Photon Scattering Rate:** The accompanying spontaneous emission rate scales as:

$$\Gamma_{\text{sc}}(\mathbf{r}) = \frac{3\pi c^2}{2\hbar \omega_0^3} \left( \frac{\Gamma}{\Delta} \right)^2 I(\mathbf{r}) = \frac{\Gamma}{\hbar \Delta} U_{\text{dip}}(\mathbf{r})$$

By choosing large red detuning ($\vert{}\Delta\vert{} \gg \Gamma$), experimentalists maintain deep trapping potentials ($U_0 / k_B \sim 0.5 - 1.5\,\text{mK}$) while suppressing photon scattering to rates below $1\,\text{s}^{-1}$, isolating the atomic quantum state.

### 1.2 Harmonic Oscillator Approximation
The intensity profile of a Gaussian laser propagating along $\hat{z}$ is:

$$I(r, z) = \frac{2P}{\pi w(z)^2} \exp\left( -\frac{2r^2}{w(z)^2} \right), \quad w(z) = w_0 \sqrt{1 + \left(\frac{z}{z_R}\right)^2}$$

where $P$ is optical power and $z_R = \frac{\pi w_0^2}{\lambda}$ is the Rayleigh length. Expanding $U_{\text{dip}}(r, z)$ near the focal center $(r = 0, z = 0)$ via second-order Taylor expansion gives a 3D harmonic well:

$$U(r, z) \approx -U_0 \left[ 1 - 2\left(\frac{r}{w_0}\right)^2 - \left(\frac{z}{z_R}\right)^2 \right]$$

with radial and axial trapping harmonic frequencies:

$$\omega_r = \sqrt{\frac{4 U_0}{m w_0^2}}, \qquad \omega_z = \sqrt{\frac{2 U_0}{m z_R^2}}$$

where $m$ is the atomic mass. Typically, $\omega_r \sim 2\pi \times (50 - 150)\,\text{kHz}$ and $\omega_z \sim 2\pi \times (10 - 30)\,\text{kHz}$.

---

## 2. The Core Challenge: Light-Assisted Collisional Blockade

To perform gate-based quantum computing, quantum registers require **exactly one atom per trap site**. However, loading neutral atoms directly into microscopic optical tweezers from a cold magneto-optical trap (MOT) or optical molasses is fundamentally stochastic.

### 2.1 The Two-Body Loss Rate Equation
The atom number distribution $N(t)$ within a microscopic tweezer volume is governed by:

$$\frac{dN}{dt} = R - \gamma N - \beta' N(N - 1)$$

where:
* $R$ is the single-atom capture rate from the surrounding atomic cloud.
* $\gamma$ is the one-body background gas collision rate ($\gamma \approx 1/\tau_{\text{vac}} \sim 0.04\,\text{s}^{-1}$).
* $\beta'$ is the two-body light-assisted collision parameter.

### 2.2 Collisional Parity Projection
Because the optical tweezer volume is microscopic ($\sim 1\,\mu\text{m}^3$), the effective two-body collision rate is large: $\beta' \sim 10^4 - 10^5\,\text{s}^{-1} \gg R, \gamma$.

When two atoms occupy the same potential well in the presence of near-resonant cooling light:
1. The atoms absorb a laser photon and form a transient repulsive or attractive quasi-molecular state.
2. Inelastic decay converts electronic excitation energy into kinetic energy: $\Delta E_{\text{kinetic}} \sim 1 - 10\,\text{K}$.
3. Because the trap depth is only $U_0 / k_B \sim 1\,\text{mK}$, **both colliding atoms are ejected simultaneously**.

Consequently, the steady-state occupancy projects onto the parity of the number of entering atoms:

$$N_{\text{final}} = N_{\text{initial}} \pmod 2 \in \{0, 1\}$$

This caps the passive loading probability of an isolated tweezer at:

$$P_{\text{load}} \approx 0.50 - 0.55$$

### 2.3 The Exponential Register Scaling Bottleneck
For an $N_{\text{target}}$-qubit register, the joint probability of loading all traps simultaneously without active defect-correction is:

$$P_{\text{success}} = (P_{\text{load}})^{N_{\text{target}}} \approx (0.50)^{N_{\text{target}}}$$

* For an $8 \times 8$ register ($N_{\text{target}} = 64$):
  
$$P_{\text{success}} = (0.5)^{64} \approx 5.42 \times 10^{-20}$$

Even running at a $100\,\text{Hz}$ repetition rate, loading an $8 \times 8$ register by chance would require **over 580 million years**. Deterministic optical rearrangement is the prerequisite for scalable neutral atom quantum computing.

---

## 3. Hardware Architecture: SLM + AOD Integration

The experimental apparatus utilizes a dual-path optical control system:

```text
Cold Atom Cloud (MOT)
          │
          ▼
┌─────────────────────────┐          ┌─────────────────────────┐
│  Spatial Light          │          │  2D Acousto-Optic       │
│  Modulator (SLM)        │          │  Deflector (AOD)        │
└───────────┬─────────────┘          └───────────┬─────────────┘
            │                                    │
            ▼                                    ▼
Static Trapping Array                Dynamic Shuttling Tweezers
(12x12 = 144 sites, fixed)           (Frequency-chirped beam, mobile)
            │                                    │
            └───────────────┬────────────────────┘
                            │
                            ▼
              High-NA Microscope Objective
                            │
                            ▼
             Atom Rearrangement in Vacuum
