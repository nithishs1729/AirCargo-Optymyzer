# Air Cargo Route Optymyzer

An interactive, multi-objective decision-support dashboard for airline logistics and network route optimization.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Tech: HTML5, JavaScript, CSS](https://img.shields.io/badge/Tech-HTML5%20%7C%20JavaScript%20%7C%20CSS-orange.svg)](#technology-stack)
[![Platform: Web](https://img.shields.io/badge/Platform-Web-1877F2.svg)](#getting-started)

## Overview

Air Cargo Route Optymyzer is a self-contained browser dashboard for exploring air-cargo route alternatives across an illustrative airport network. Enter a shipment profile and aircraft class, set relative priorities, and explicitly run the route heuristic to compare feasible paths.

### Key features

- **Interactive global route map:** Leaflet.js displays the airport network and selected routes as great-circle paths. Routes include direct and hub-and-spoke alternatives.
- **Multi-parametric optimization engine:** Priority sliders balance estimated Direct Operating Cost (DOC), transit time, fuel-related CO₂ emissions, and SLA exposure.
- **On-demand optimization:** The **Optimize Network Route** button evaluates candidate paths; editing inputs does not recalculate or replace the last results until the button is clicked.
- **Interactive input tooltips:** Keyboard-focusable and hoverable `i` icons explain the impact of each shipment, cargo, aircraft, and priority input.
- **Pareto and route analytics:** Chart.js visualizes feasible cost/time trade-offs and the selected route's relative cost, fuel, emissions, transit, and SLA exposure.
- **Dark, Facebook-blue interface:** Responsive dark-mode UI with blue accents for desktop and mobile screens.
- **Transparent model notes:** An expandable explanation documents the objective, feasibility constraints, approximations, and limitations.

## Mathematical engine and objective function

For each selected origin-destination pair, the heuristic enumerates feasible paths and ranks them using a min-max-normalized weighted-sum scalarization:

$$
\min_{p \in \mathcal{P}} J(p) =
w_{\text{cost}} \cdot \hat{C}(p) +
w_{\text{time}} \cdot \hat{T}(p) +
w_{\text{co2}} \cdot \hat{E}(p) +
w_{\text{sla}} \cdot \hat{R}(p)
$$

Here, \(\mathcal{P}\) is the finite set of loop-free candidate paths connecting origin \(O\) and destination \(D\), after aircraft range, shipment payload, cargo-handling capability, and hard maximum-transit constraints have been applied. The path \(p\) may be direct or include up to three intermediate hubs (four flight legs).

Each metric is normalized across the currently feasible candidate set:

$$
\hat{C}(p) = \frac{C(p)-C_{\min}}{C_{\max}-C_{\min}}, \quad
\hat{T}(p) = \frac{T(p)-T_{\min}}{T_{\max}-T_{\min}}
$$

$$
\hat{E}(p) = \frac{E(p)-E_{\min}}{E_{\max}-E_{\min}}, \quad
\hat{R}(p) = \frac{R(p)-R_{\min}}{R_{\max}-R_{\min}}
$$

When every feasible path has the same value for a metric, its normalized value is set to zero to avoid division by zero. All four raw slider values are dynamically normalized into nonnegative weights that sum to \(1\). If every slider is set to zero, equal weights (25% each) are used. The engine ranks paths in ascending \(J(p)\) and breaks score ties by estimated cost.

### Metric approximations

- **Cost, \(C(p)\):** The displayed DOC estimate combines a base block-hour operating-cost component, payload-sensitive fuel cost, distance-based overflight fees, and landing/handling fees at endpoints and intermediate hubs. The simplified conceptual breakdown is:

  $$
  DOC(p) \approx \text{Fuel Cost}(p) +
  \text{Landing/Handling Fees}(p) +
  \text{Overflight Fees}(p)
  $$

  The implementation additionally includes block-hour cost. Fuel burn per kilometre varies with shipment mass; all aircraft coefficients, fuel prices, and fee amounts are synthetic estimates, not live carrier tariffs.

- **Transit time, \(T(p)\):** Great-circle route distance is divided by the selected aircraft's nominal cruise velocity \(V\). The estimate adds 0.75 hours per flight leg for block/turnaround allowance, two hours for each intermediate hub, and 1.5 hours at each endpoint. The user-entered maximum transit time remains a hard feasibility limit.

- **Emissions, \(E(p)\):** Estimated fuel burn is converted to CO₂ using an illustrative factor of 3.16 kg CO₂ per kg of fuel.

- **SLA exposure, \(R(p)\):** The dashboard uses the deterministic proxy \(R(p)=T(p)/T_{\text{SLA}}+0.10N_{\text{transfers}}(p)\), where \(T_{\text{SLA}}\) is the maximum transit time and \(N_{\text{transfers}}\) is the number of intermediate hubs. This reflects used SLA buffer and transfer exposure; it is **not** a predicted delay probability or contractual penalty estimate.

### Engine scope and assumptions

- Normalization is calculated across the complete set of currently feasible candidate paths for the selected origin-destination pair before the displayed list is limited to the top 40.
- Slider normalization is dynamic and sums to exactly \(1.0\); the all-zero fallback is equal weighting.
- The engine is a route-choice and feasibility heuristic over an illustrative network graph, not a global fleet-wide integer-programming schedule generator.
- Airport connectivity, capacity, flight performance, operating rates, and special-cargo handling flags are demonstration assumptions, not live schedules or operational dispatch data.
- The Pareto comparison considers cost, transit time, CO₂, and SLA exposure; route scores are utility rankings rather than guaranteed commercial or operational outcomes.

## Getting started

### Run locally

1. Clone the repository:

   ```bash
   git clone https://github.com/nithishs1729/AirCargo-Optymyzer.git
   cd AirCargo-Optymyzer
   ```

   If using your own fork, replace the repository URL with your clone URL.

2. Open `index.html` directly in a modern browser, or serve the project locally. For example:

   ```bash
   python3 -m http.server 8000
   ```

   Then open <http://localhost:8000>.

The page has no build step or package installation. Leaflet, Chart.js, and MathJax are loaded from CDNs, and the map uses OpenStreetMap tiles, so an internet connection is required for those resources.

## Technology stack

- **Frontend:** HTML5, responsive CSS embedded in `index.html`, and vanilla JavaScript.
- **Mapping:** Leaflet.js with OpenStreetMap tiles and client-side great-circle path rendering.
- **Charts:** Chart.js.
- **Mathematical notation:** MathJax.
- **Optimization logic:** JavaScript path enumeration, hard feasibility filters, min-max normalization, and weighted multi-objective heuristic ranking.

This project does **not** currently use Tailwind CSS or D3.js; its styling is custom embedded CSS, and its map is implemented with Leaflet.

## Project structure

```text
.
├── .gitignore
├── index.html
├── LICENSE
└── README.md
```

## License

Distributed under the MIT License. See [LICENSE](LICENSE).

## Initialize and push to GitHub

Run the following commands from the project directory. If this folder is already a Git repository, skip `git init`; if `origin` already exists, update it with `git remote set-url origin https://github.com/nithishs1729/AirCargo-Optymyzer.git` instead of adding it again.

```bash
git init
git add .
git commit -m "feat: initial commit of Air Cargo Route Optymyzer dashboard"
git branch -M main
git remote add origin <YOUR GITHUB REPOSITORY>
git push -u origin main
```
