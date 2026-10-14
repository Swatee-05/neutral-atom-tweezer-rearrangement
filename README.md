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

- **Autonomous Execution:** Solves bipartite matching and routes atoms sequentially using center-outward heuristic scheduling and perimeter-buffered breadth-first search.
- **Manual Control:** Click any static atom (Red) and select an open target trap (Blue ring) to manually execute collision-free shuttling trajectories.

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

## Physical Foundations: Optical Dipole Traps (Tweezers)

An optical tweezer is an attractive potential well created by focusing a Gaussian laser beam through a high-numerical-aperture ($\text{NA} \ge 0.5$) microscope objective down to a diffraction-limited beam waist $w_0 \approx 0.7 - 1.0\,\mu\text{m}$.

### The Optical Dipole Force (AC Stark Shift)
When an electric field $\mathbf{E}(\mathbf{r}, t) = \hat{\mathbf{e}} E(\mathbf{r}) e^{-i\omega_L t} + \text{c.c.}$ interacts with an isolated neutral alkali atom (e.g., $^{87}\text{Rb}$, $^{133}\text{Cs}$) or alkaline-earth atom ($^{171}\text{Yb}$), it induces an oscillating atomic electric dipole moment:

$$\mathbf{d}(\mathbf{r}, t) = \alpha(\omega_L) \mathbf{E}(\mathbf{r}, t)$$

where $\alpha(\omega_L)$ is the dynamic atomic polarizability at laser frequency $\omega_L$. The interaction between the induced dipole and the electric field yields the optical dipole potential (the light shift or AC Stark shift):

$$U_{\text{dip}}(\mathbf{r}) = -\frac{1}{2} \langle \mathbf{d} \cdot \mathbf{E} \rangle = -\frac{1}{2\epsilon_0 c} \text{Re}[\alpha(\omega_L)] I(\mathbf{r})$$

For a two-level atomic transition with resonant frequency $\omega_0$, natural decay rate $\Gamma$, and laser detuning $\Delta = \omega_L - \omega_0$, the potential evaluates to:

$$U_{\text{dip}}(\mathbf{r}) = \frac{3\pi c^2}{2 \omega_0^3} \left( \frac{\Gamma}{\Delta} \right) I(\mathbf{r})$$

- **Far Red Detuned ($\omega_L < \omega_0 \implies \Delta < 0$):** The dipole potential is negative ($U_{\text{dip}} < 0$). Atoms are conservatively attracted toward intensity maxima (the laser focal center).
- **Photon Scattering Rate:** The accompanying spontaneous emission rate scales as:

$$\Gamma_{\text{sc}}(\mathbf{r}) = \frac{3\pi c^2}{2\hbar \omega_0^3} \left( \frac{\Gamma}{\Delta} \right)^2 I(\mathbf{r}) = \frac{\Gamma}{\hbar \Delta} U_{\text{dip}}(\mathbf{r})$$

By choosing large red detuning ($\vert{}\Delta\vert{} \gg \Gamma$), experimentalists maintain deep trapping potentials ($U_0 / k_B \sim 0.5 - 1.5\,\text{mK}$) while suppressing photon scattering to rates below $1\,\text{s}^{-1}$, isolating the atomic quantum state.

### Harmonic Oscillator Approximation
The intensity profile of a Gaussian laser propagating along $\hat{z}$ is:

$$I(r, z) = \frac{2P}{\pi w(z)^2} \exp\left( -\frac{2r^2}{w(z)^2} \right), \quad w(z) = w_0 \sqrt{1 + \left(\frac{z}{z_R}\right)^2}$$

where $P$ is optical power and $z_R = \frac{\pi w_0^2}{\lambda}$ is the Rayleigh length. Expanding $U_{\text{dip}}(r, z)$ near the focal center $(r = 0, z = 0)$ via second-order Taylor expansion gives a 3D harmonic well:

$$U(r, z) \approx -U_0 \left[ 1 - 2\left(\frac{r}{w_0}\right)^2 - \left(\frac{z}{z_R}\right)^2 \right]$$

with radial and axial trapping harmonic frequencies:

$$\omega_r = \sqrt{\frac{4 U_0}{m w_0^2}}, \qquad \omega_z = \sqrt{\frac{2 U_0}{m z_R^2}}$$

where $m$ is the atomic mass. Typically, $\omega_r \sim 2\pi \times (50 - 150)\,\text{kHz}$ and $\omega_z \sim 2\pi \times (10 - 30)\,\text{kHz}$.

---

## The Core Challenge: Light-Assisted Collisional Blockade

To perform gate-based quantum computing, quantum registers require **exactly one atom per trap site**. However, loading neutral atoms directly into microscopic optical tweezers from a cold magneto-optical trap (MOT) or optical molasses is fundamentally stochastic.

### The Two-Body Loss Rate Equation
The atom number distribution $N(t)$ within a microscopic tweezer volume is governed by:

$$\frac{dN}{dt} = R - \gamma N - \beta' N(N - 1)$$

where:
- $R$ is the single-atom capture rate from the surrounding atomic cloud.
- $\gamma$ is the one-body background gas collision rate ($\gamma \approx 1/\tau_{\text{vac}} \sim 0.04\,\text{s}^{-1}$).
- $\beta'$ is the two-body light-assisted collision parameter.

### Collisional Parity Projection
Because the optical tweezer volume is microscopic ($\sim 1\,\mu\text{m}^3$), the effective two-body collision rate is large: $\beta' \sim 10^4 - 10^5\,\text{s}^{-1} \gg R, \gamma$.

When two atoms occupy the same potential well in the presence of near-resonant cooling light:

The atoms absorb a laser photon and form a transient repulsive or attractive quasi-molecular state.

Inelastic decay converts electronic excitation energy into kinetic energy: $\Delta E_{\text{kinetic}} \sim 1 - 10\,\text{K}$.

Because the trap depth is only $U_0 / k_B \sim 1\,\text{mK}$, **both colliding atoms are ejected simultaneously**.

Consequently, the steady-state occupancy projects onto the parity of the number of entering atoms:

$$N_{\text{final}} = N_{\text{initial}} \pmod 2 \in \{0, 1\}$$

This caps the passive loading probability of an isolated tweezer at:

$$P_{\text{load}} \approx 0.50 - 0.55$$

### The Exponential Register Scaling Bottleneck
For an $N_{\text{target}}$-qubit register, the joint probability of loading all traps simultaneously without active defect-correction is:

$$P_{\text{success}} = (P_{\text{load}})^{N_{\text{target}}} \approx (0.50)^{N_{\text{target}}}$$

- For an $8 \times 8$ register ($N_{\text{target}} = 64$):
  
$$P_{\text{success}} = (0.5)^{64} \approx 5.42 \times 10^{-20}$$

Even running at a $100\,\text{Hz}$ repetition rate, loading an $8 \times 8$ register by chance would require **over 580 million years**. Deterministic optical rearrangement is the prerequisite for scalable neutral atom quantum computing.

---

## Hardware Architecture: SLM + AOD Integration

The experimental apparatus utilizes a dual-path optical control system:

---

<hr />

<h2>Physical Foundations: Optical Dipole Traps (Tweezers)</h2>
<p>
  An optical tweezer is an attractive potential well created by focusing a Gaussian laser beam through a high-numerical-aperture ($\text{NA} \ge 0.5$) microscope objective down to a diffraction-limited beam waist $w_0 \approx 0.7 - 1.0\,\mu\text{m}$.
</p>

<h3>The Optical Dipole Force (AC Stark Shift)</h3>
<p>
  When an oscillating electric field $\mathbf{E}(\mathbf{r}, t) = \hat{\mathbf{e}} E(\mathbf{r}) e^{-i\omega_L t} + \text{c.c.}$ interacts with an isolated neutral alkali atom (e.g., <sup>87</sup>Rb, <sup>133</sup>Cs) or alkaline-earth atom (<sup>171</sup>Yb), it induces an oscillating atomic electric dipole moment:
</p>
<p align="center">
  $$\mathbf{d}(\mathbf{r}, t) = \alpha(\omega_L) \mathbf{E}(\mathbf{r}, t)$$
</p>
<p>
  where $\alpha(\omega_L)$ is the dynamic atomic polarizability at laser frequency $\omega_L$. The interaction between the induced dipole and the electric field yields the optical dipole potential (the AC Stark shift):
</p>
<p align="center">
  $$U_{\text{dip}}(\mathbf{r}) = -\frac{1}{2} \langle \mathbf{d} \cdot \mathbf{E} \rangle = -\frac{1}{2\epsilon_0 c} \text{Re}[\alpha(\omega_L)] I(\mathbf{r})$$
</p>
<p>
  For a two-level atomic transition with resonant frequency $\omega_0$, natural linewidth $\Gamma$, and laser detuning $\Delta = \omega_L - \omega_0$, the potential evaluates to:
</p>
<p align="center">
  $$U_{\text{dip}}(\mathbf{r}) = \frac{3\pi c^2}{2 \omega_0^3} \left( \frac{\Gamma}{\Delta} \right) I(\mathbf{r})$$
</p>
<ul>
  <li><b>Far Red Detuned ($\omega_L &lt; \omega_0 \implies \Delta &lt; 0$):</b> The potential is negative ($U_{\text{dip}} &lt; 0$). Atoms are attracted toward laser intensity maxima (the focal center).</li>
  <li><b>Photon Scattering Rate:</b> The accompanying spontaneous emission rate scales as:
    <p align="center">
      $$\Gamma_{\text{sc}}(\mathbf{r}) = \frac{3\pi c^2}{2\hbar \omega_0^3} \left( \frac{\Gamma}{\Delta} \right)^2 I(\mathbf{r}) = \frac{\Gamma}{\hbar \Delta} U_{\text{dip}}(\mathbf{r})$$
    </p>
  </li>
</ul>
<p>
  By maintaining large red detuning ($\vert{}\Delta\vert{} \gg \Gamma$), experimental setups sustain deep trapping potentials ($U_0 / k_B \sim 0.5 - 1.5\,\text{mK}$) while suppressing photon scattering rates below $1\,\text{s}^{-1}$, preserving atomic coherence.
</p>

<h3>Harmonic Oscillator Approximation</h3>
<p>
  The intensity profile of a Gaussian beam propagating along $\hat{z}$ is given by:
</p>
<p align="center">
  $$I(r, z) = \frac{2P}{\pi w(z)^2} \exp\left( -\frac{2r^2}{w(z)^2} \right), \quad w(z) = w_0 \sqrt{1 + \left(\frac{z}{z_R}\right)^2}$$
</p>
<p>
  where $P$ is optical laser power and $z_R = \frac{\pi w_0^2}{\lambda}$ is the Rayleigh length. Expanding $U_{\text{dip}}(r, z)$ near the focal center $(r = 0, z = 0)$ via second-order Taylor expansion yields a 3D harmonic oscillator:
</p>
<p align="center">
  $$U(r, z) \approx -U_0 \left[ 1 - 2\left(\frac{r}{w_0}\right)^2 - \left(\frac{z}{z_R}\right)^2 \right]$$
</p>
<p>with radial and axial trapping harmonic frequencies:</p>
<p align="center">
  $$\omega_r = \sqrt{\frac{4 U_0}{m w_0^2}}, \qquad \omega_z = \sqrt{\frac{2 U_0}{m z_R^2}}$$
</p>
<p>
  where $m$ is the atomic mass. Typically, $\omega_r \sim 2\pi \times (50 - 150)\,\text{kHz}$ and $\omega_z \sim 2\pi \times (10 - 30)\,\text{kHz}$.
</p>

<hr />

<h2>The Core Challenge: Light-Assisted Collisional Blockade</h2>
<p>
  Scalable quantum computation requires <b>exactly one atom per tweezer site</b>. However, initial trap loading from a magneto-optical trap (MOT) or optical molasses is fundamentally stochastic.
</p>

<h3>The Two-Body Loss Rate Equation</h3>
<p>
  The time-dependent atom number distribution $N(t)$ within an individual microscopic optical trap is governed by:
</p>
<p align="center">
  $$\frac{dN}{dt} = R - \gamma N - \beta' N(N - 1)$$
</p>
<ul>
  <li>$R$ is the capture rate of cold atoms into the trap volume.</li>
  <li>$\gamma$ is the single-atom loss rate due to background gas collisions ($\gamma \approx 1/\tau_{\text{vac}} \sim 0.04\,\text{s}^{-1}$).</li>
  <li>$\beta'$ is the two-body light-assisted collision rate coefficient.</li>
</ul>

<h3>Collisional Parity Projection</h3>
<p>
  Because the tweezer focal volume is microscopic ($\sim 1\,\mu\text{m}^3$), the two-body loss rate is massive: $\beta' \sim 10^4 - 10^5\,\text{s}^{-1} \gg R, \gamma$. When two atoms occupy the same potential well under cooling illumination:
</p>
<p>
  The atoms absorb a photon and form a transient molecular state.
</p>
<p>
  Inelastic decay converts electronic excitation into kinetic energy ($\Delta E \sim 1 - 10\,\text{K}$).
</p>
<p>
  Because trap depths are only $U_0 / k_B \sim 1\,\text{mK}$, <b>both colliding atoms are immediately ejected from the trap</b>.
</p>
<p>
  Consequently, final trap occupancy projects onto the parity of the loaded atom count:
</p>
<p align="center">
  $$N_{\text{final}} = N_{\text{initial}} \pmod 2 \in \{0, 1\}$$
</p>
<p>
  This fixes the passive loading probability of an isolated tweezer at $P_{\text{load}} \approx 0.50 - 0.55$.
</p>

<h3>Exponential Register Scaling Bottleneck</h3>
<p>
  For an $N_{\text{target}}$-qubit register, the probability of concurrently loading all sites by chance is:
</p>
<p align="center">
  $$P_{\text{success}} = (P_{\text{load}})^{N_{\text{target}}} \approx (0.50)^{N_{\text{target}}}$$
</p>
<p>
  For an $8 \times 8$ register ($N_{\text{target}} = 64$), $P_{\text{success}} = (0.5)^{64} \approx 5.42 \times 10^{-20}$. At a $100\,\text{Hz}$ experimental cycle rate, loading this register by chance would require over 580 million years. Active rearrangement is therefore necessary.
</p>

<hr />

<h2>Hardware Architecture: SLM + AOD Integration</h2>
<p>
  The experimental layout uses a dual-path optical control architecture:
</p>

<pre>
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
</pre>

<p>
  <b>Static Trapping Array (SLM):</b> A phase-only Spatial Light Modulator applies a computed 2D phase hologram $\Phi(x, y)$ calculated via the Gerchberg-Saxton algorithm, generating the static $12 \times 12$ reservoir grid.
</p>
<p>
  <b>Dynamic Shuttling Tweezers (AOD):</b> A pair of orthogonal Acousto-Optic Deflectors driven by RF acoustic waves steer dynamic tweezers via Bragg diffraction:
</p>
<p align="center">
  $$\theta(t) \approx \frac{\lambda_{\text{laser}}}{v_{\text{acoustic}}} f_{\text{RF}}(t)$$
</p>
<p>
  Chirping $f_{\text{RF}}(t)$ translates the optical focal point with sub-nanometer precision.
</p>
<p>
  <b>Adiabatic Handshake:</b> To transfer atoms between static and dynamic traps without heating, trap depths are ramped simultaneously:
</p>
<p align="center">
  $$U_{\text{tot}}(\mathbf{r}, t) = [1 - \lambda(t)] U_{\text{SLM}}(\mathbf{r}) + \lambda(t) U_{\text{AOD}}(\mathbf{r}), \quad \lambda(t) \in [0, 1]$$
</p>

<hr />

<h2>Algorithmic Pipeline &amp; Mathematical Formulations</h2>

<pre>
Stochastic Array Loading (P ~ 0.55)
                 │
                 ▼
     [EMCCD Fluorescence Readout]
                 │
                 ▼
   [Minimum-Cost Bipartite Matching]  ──&gt; Kuhn-Munkres (Hungarian) O(M³)
                 │
                 ▼
    [Center-Outward Path Scheduler]   ──&gt; Topological Deadlock Avoidance
                 │
                 ▼
  [Perimeter-Buffered BFS Path Router] ──&gt; Collision-Free Waypoints
                 │
                 ▼
        Hardware Execution
</pre>

<h3>Occupancy Detection &amp; Coordinate Mapping</h3>
<p>
  An EMCCD camera detects fluorescence under optical molasses. Thresholding generates a binary occupancy matrix $\mathcal{M} \in \{0, 1\}^{N \times N}$.
</p>
<ul>
  <li><b>Reservoir Coordinates:</b> $\mathcal{S} = \{\mathbf{s}_1, \dots, \mathbf{s}_M\} \subset \mathbb{Z}^2$, where $\mathcal{M}(\mathbf{s}_i) = 1$ and $M \ge K$.</li>
  <li><b>Target Coordinates:</b> $\mathcal{T} = \{\mathbf{t}_1, \dots, \mathbf{t}_K\} \subset \mathbb{Z}^2$, where $K = 64$ for an $8 \times 8$ register centered within the $12 \times 12$ array with offset $\delta = (12 - 8) / 2 = 2$.</li>
</ul>

<h3>Global Cost Optimization (Hungarian Assignment)</h3>
<p>
  The cost matrix $C \in \mathbb{R}^{M \times K}$ uses squared Euclidean distance to penalize large single-atom displacements:
</p>
<p align="center">
  $$C_{ij} = \|\mathbf{s}_i - \mathbf{t}_j\|_2^2 = (x_{s,i} - x_{t,j})^2 + (y_{s,i} - y_{t,j})^2$$
</p>
<p>The assignment is formulated as an Integer Linear Program:</p>
<p align="center">
  $$\min_{X} \sum_{i=1}^M \sum_{j=1}^K C_{ij} X_{ij}$$
</p>
<p align="center">
  $$\text{subject to} \quad \sum_{j=1}^K X_{ij} \le 1, \quad \sum_{i=1}^M X_{ij} = 1, \quad X_{ij} \in \{0, 1\}$$
</p>
<p>
  This is solved in $\mathcal{O}(M^3)$ via the Kuhn-Munkres algorithm using dual variables $u_i, v_j$:
</p>
<p align="center">
  $$\max_{u, v} \left( \sum_{i=1}^M u_i + \sum_{j=1}^K v_j \right) \quad \text{subject to} \quad u_i + v_j \le C_{ij}$$
</p>

<h3>Center-Outward Heuristic Motion Scheduling</h3>
<p>
  To prevent target-boundary sites from sealing off central vacancies, target sites are prioritized by radial distance from the centroid $\mathbf{c}_T = \left(\delta + \frac{T - 1}{2}, \delta + \frac{T - 1}{2}\right)$:
</p>
<p align="center">
  $$\mathcal{R}(\mathbf{t}_j) = \|\mathbf{t}_j - \mathbf{c}_T\|_2^2 = \left(x_{t,j} - c_{T,x}\right)^2 + \left(y_{t,j} - c_{T,y}\right)^2$$
</p>
<p>
  Sorting assignments by $\mathcal{R}(\mathbf{t}_j)$ fills central positions first, leaving perimeter pathways accessible for later transfers.
</p>

<h3>Perimeter-Buffered Collision-Free BFS Router</h3>
<p>
  During transport, moving atoms must not intersect occupied traps. The array is embedded inside an expanded grid graph $\mathcal{V} = \{(r, c) \in \mathbb{Z}^2 \mid -1 \le r \le N_{\text{res}}, -1 \le c \le N_{\text{res}}\}$, where boundary lines $r, c \in \{-1, N_{\text{res}}\}$ define a clear perimeter corridor.
</p>
<p>Traversability for a graph node $\mathbf{v}$ is evaluated as:</p>
<p align="center">
  $$\text{Free}(\mathbf{v}) = \begin{cases} \text{True} & \text{if } \mathbf{v} = \mathbf{t}_j \\ \text{True} & \text{if } \mathbf{v} \notin [0, N_{\text{res}}-1]^2 \quad (\text{Perimeter Corridor}) \\ \text{True} & \text{if } \mathcal{M}(\mathbf{v}) = 0 \quad (\text{Empty Static Trap}) \\ \text{False} & \text{if } \mathcal{M}(\mathbf{v}) = 1 \quad (\text{Occupied Trap Obstacle}) \end{cases}$$
</p>
<p>
  Breadth-First Search finds the minimal-hop collision-free path $P = (\mathbf{p}_0, \dots, \mathbf{p}_L)$. The source site is cleared at departure ($\mathcal{M}(\mathbf{s}_i) \leftarrow 0$) and the destination marked occupied upon delivery ($\mathcal{M}(\mathbf{t}_j) \leftarrow 1$).
</p>

<hr />

<h2>Physical AMO Metrics &amp; Performance Benchmarks</h2>

<h3>Transport Kinematics &amp; Timing</h3>
<ul>
  <li><b>Total Distance:</b> $D_{\text{total}} = \sum_{k=1}^{N_{\text{moves}}} (L(P_k) - 1) \cdot a$, where $a = 4.5\,\mu\text{m}$.</li>
  <li><b>Translation Time:</b> $t_{\text{trans}} = \frac{D_{\text{total}}}{v_{\text{AOD}}}$ with velocity $v_{\text{AOD}} = 60\,\mu\text{m/ms}$.</li>
  <li><b>Ramp Overhead:</b> $t_{\text{overhead}} = 2 \cdot N_{\text{moves}} \cdot t_{\text{ramp}}$ with $t_{\text{ramp}} = 80\,\mu\text{s}$.</li>
  <li><b>Total Assembly Duration:</b> $T_{\text{cycle}} = t_{\text{trans}} + t_{\text{overhead}}$.</li>
</ul>

<h3>Survival Probability Formulations</h3>
<ul>
  <li><b>Vacuum Background Survival:</b> $P_{\text{vac}} = \exp\left( -\frac{T_{\text{cycle}}}{\tau_{\text{vac}}} \right)$ where $\tau_{\text{vac}} \approx 25.0\,\text{s}$.</li>
  <li><b>Transport Survival:</b> $P_{\text{transfer}} = (1 - \epsilon_{\text{loss}})^{N_{\text{moves}}}$ with single-move transfer loss $\epsilon_{\text{loss}} = 0.002$.</li>
  <li><b>Net Assembly Yield:</b> $\mathcal{F}_{\text{net}} = P_{\text{vac}} \times P_{\text{transfer}}$.</li>
</ul>

<h3>System Benchmarks ($12 \times 12 \to 8 \times 8$)</h3>

<table>
  <thead>
    <tr>
      <th align="left">Metric</th>
      <th align="left">Simulated Benchmark</th>
      <th align="left">Physical Significance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><b>Reservoir Dimension</b></td>
      <td>$12 \times 12$ ($144$ traps)</td>
      <td>Over-provisioning ratio of $2.25:1$ against $55\%$ loading</td>
    </tr>
    <tr>
      <td><b>Target Register Dimension</b></td>
      <td>$8 \times 8$ ($64$ traps)</td>
      <td>Defect-free target computing register</td>
    </tr>
    <tr>
      <td><b>Trap Pitch ($a$)</b></td>
      <td>$4.5\,\mu\text{m}$</td>
      <td>Matches optical resolution of $\text{NA} \approx 0.55$ objectives</td>
    </tr>
    <tr>
      <td><b>AOD Translation Velocity</b></td>
      <td>$60\,\mu\text{m/ms}$</td>
      <td>Adiabatic limit avoiding vibrational excitation</td>
    </tr>
    <tr>
      <td><b>Total Assembly Time</b></td>
      <td>$\approx 35 - 55\,\text{ms}$</td>
      <td>Completed well within atom trap lifetimes</td>
    </tr>
    <tr>
      <td><b>Vacuum Survival ($P_{\text{vac}}$)</b></td>
      <td>$&gt; 99.8\%$</td>
      <td>Negligible loss against residual background gas</td>
    </tr>
    <tr>
      <td><b>Register Assembly Yield</b></td>
      <td>$&gt; 90.0\%$</td>
      <td>High single-shot preparation probability</td>
    </tr>
  </tbody>
</table>

<hr />

<h2>Software Architecture</h2>
<ul>
  <li><b>Matching Engine:</b> Computes optimal source-to-target assignment via Kuhn-Munkres cost minimization (<code>scipy.optimize.linear_sum_assignment</code>).</li>
  <li><b>Motion Planner:</b> Graph traversal router with dynamic obstacle masking and perimeter-corridor graph expansion.</li>
  <li><b>Diagnostics Module:</b> Computes physical distance traveled ($\mu\text{m}$), AOD ramp delays, vacuum survival probabilities, and register assembly yields.</li>
  <li><b>Interactive Client:</b> Browser-based vector rendering canvas written in standards-compliant JavaScript and HTML5, deployed directly on GitHub Pages.</li>
</ul>

<hr />

## 🧒 Explanation for Kids: How It Works Simply

Imagine trying to build a microscopic computer using individual atoms as Lego bricks, using lasers as your only tools!

### What is an Optical Tweezer?
Atoms are small and float around like dust specks in the air. They cannot be picked up with mechanical tools.

Instead, a laser beam is focused through a microscope lens down to a tight focal point. This point acts like a **tractor beam**: it creates a localized bowl of light that attracts and holds exactly one atom in place.

### The Problem: "Two's a Crowd"
Catching atoms is like a carnival game:
- A cold cloud of atoms is washed over the laser array.
- If **0 atoms** fall into a trap, it stays empty.
- If **1 atom** lands in a trap, it stays trapped.
- If **2 atoms** land in the same trap at the same time, the laser light causes them to bump together with high energy. Both atoms get kicked out immediately, leaving the trap empty.

Because of this rule, every trap has roughly a **50% coin-flip chance** of catching an atom. Getting 64 traps to all catch an atom at the exact same instant by pure chance is roughly $1$ in $1.8 \times 10^{19}$.

### The Strategy: Over-Provision and Rearrange
Instead of relying on luck:

**Cast a wider net:** Shine $144$ laser traps ($12 \times 12$). With 50% odds, 75 to 80 atoms are caught across the grid.

**Take a picture:** A sensitive camera identifies which traps are filled (Red dots) and which are empty (Gray dots).

**Use a moving laser crane:** A secondary movable laser beam picks up atoms from the edges and places them into the center to build a full $8 \times 8$ grid (Blue rings).

### The Three Rules of the Sorting Engine

- **No Collisions (The Outer Hallway):** Atoms cannot pass through each other. If a moving atom touches a stationary one, both are lost. If an atom gets blocked by neighbors, it moves out into the perimeter buffer corridor (the purple dashed boundary) to bypass traffic and re-enter safely.
- **Center-Outward Parking:** If outer target positions are filled first, they form a wall that blocks atoms trying to reach the center. The algorithm fills the dead center of the target square first and works outward.
- **Speed vs. Lifetime:** Trapped atoms have a finite lifetime before colliding with stray background gas particles. The algorithm finds the shortest possible paths so the entire assembly finishes in under $0.05$ seconds (50 milliseconds), maintaining a $>99.8\%$ survival rate.

Stochastic Array Loading (P ~ 0.55)
                 │
                 ▼
     [EMCCD Fluorescence Readout]
                 │
                 ▼
   [Minimum-Cost Bipartite Matching]  ──> Kuhn-Munkres (Hungarian) O(M³)
                 │
                 ▼
    [Center-Outward Path Scheduler]   ──> Topological Deadlock Avoidance
                 │
                 ▼
  [Perimeter-Buffered BFS Path Router] ──> Collision-Free Waypoints
                 │
                 ▼
        Hardware Execution


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
