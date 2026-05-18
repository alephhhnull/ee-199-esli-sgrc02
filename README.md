# ESLI Fault Localization — HMI

A desktop application for running and analyzing fault localization simulations using a modified Event Source Location Identification (ESLI) algorithm on radial distribution networks.

---

## Background

The **Event Source Location Identification (ESLI)** algorithm is a PMU-based method for locating the source of power system events in radial distribution feeders. Applied to fault localization, it uses pre- and post-fault differential phasor measurements from PMUs to identify the faulted bus and its precise position within a line segment — without relying on current flow assumptions or labeled training data.

This HMI implements two modifications to the base ESLI algorithm:

- **Source-Rooted Topological Propagation (SRTP):** Restricts each PMU's propagation to its Minimum Spanning Tree (MST) path toward the source bus, eliminating cross-branch error accumulation in branched multi-PMU feeders.
- **DER Fault Current Correction (ΔI_DER):** Extends the bus injection model to account for PV inverter fault current contributions, improving accuracy in solar-integrated networks.

Simulations are run against OpenDSS-based network models via a Python FastAPI backend. The ESLI math (propagation, precise sweep, Monte Carlo) runs in a compiled C++ library.

---

## Installation

Download the latest `.msi` installer from the [Releases](https://github.com/alephhhnull/ee-199-esli-sgrc02/releases) page and run it. The app installs to your system and creates a Start Menu shortcut.

**Requirements:**
- Windows 10 or later (64-bit)
- Python backend server must be running before launching the HMI (see below)

### Starting the Backend

The HMI connects to a local Python FastAPI server at `http://127.0.0.1:8000`. Start it before opening the app:

```bash
python server.py
```

The app will wait up to 90 seconds for the server to respond. If it cannot connect, an error screen is shown.

---

## Getting Started

On first launch, the app shows a **Welcome Screen** with an **Open Case** button. A *case* is a pre-configured simulation setup that defines the network, fault types, fault positions, and PMU buses.

Click **Open Case** to select a case from the registered list. Each case opens in its own **tab** at the top of the window. Multiple cases can be open simultaneously.

---

## Navigation

The left sidebar has four pages. Navigation between pages does not lose any form state or results within a tab.

---

### Overview

A dashboard for the currently open case.

**Case Configuration** shows the static setup: network name, DSS file path, fault types, fault position percentages, alpha sweep steps, Monte Carlo trials, and selected PMU buses.

**Session Run History** is a live table of every simulation run since the app was opened. Columns show the run type, line, fault type, fault position, model used, FBLSR, MAFLE, and elapsed time. FBLSR ≥ 80% is highlighted green; MAFLE < 5% is highlighted green. The table can be exported as a CSV.

**Results History** loads batch results previously saved to disk. Select an ESLI model from the dropdown, then:
- **Load Results** — displays the scenario table (line, fault type, position, FLE, estimated bus)
- **Download CSV** — saves the results file locally
- **View SLD** — displays the Single-Line Diagram image for a selected fault type (Balanced, DLG, LL, SLG)

---

### System

Shows the network topology and element tables for the current case.

**Topology tab** — an interactive graph of the distribution network. Bus nodes are color-coded:

| Color | Bus type |
|---|---|
| Orange | Source bus |
| Blue | Load bus |
| Green | PV bus |
| Purple | Load + PV bus |
| Gray | Plain (no load) |

**Click any bus node to toggle PMU placement.** Selected PMU buses appear as chips below the graph and are used in all subsequent simulations. The total PMU count is shown in the header badge.

**Elements tab** — tabular view of all buses and lines in the network (name, type, connectivity).

---

### Simulation

The main workspace for running fault localization scenarios. Four run modes are available from the mode bar at the top.

#### Single
Run one fault scenario at a time.

Inputs: line, fault type (Balanced / DLG / LL / SLG), fault position (0 / 25 / 50 / 75 / 100 %), ESLI model, alpha steps.

Output: estimated fault bus, FLE (%), and whether the correct bus was identified (FBI).

#### Batch
Run all fault scenarios defined in the case configuration in sequence.

Inputs: ESLI model, alpha steps, and an optional checkbox to save results to disk as JSON and CSV.

Output: full scenario table with FLE and aggregate FBLSR / MAFLE metrics.

#### Compare
Run all 8 ESLI model variants on a single fault scenario and compare their results side by side.

Inputs: line, fault type, fault position, alpha steps.

Output: table of all 8 models ranked by FLE (best first), showing FLE and estimated bus for each.

The 8 models correspond to combinations of: SRTP on/off × DER correction on/off × load model (Constant Impedance / dynamic).

#### Monte Carlo
Sensitivity analysis using Gaussian noise perturbation on algorithm inputs.

Inputs: line, fault type, fault position, test type (Line Impedance or PMU measurement noise), ESLI model, alpha steps, number of trials, noise level (% σ).

Output: distribution of FLE across all trials, FBLSR, and MAFLE under the specified noise level.

Note: large trial counts (e.g. >1000) may take several minutes. The run timeout is 15 minutes.

---

**Tip:** Toggle **Show Network** in the Simulation page header to open a split-pane view of the network topology alongside the form. This lets you reference bus positions and connections while configuring a scenario.

---

### Settings

**Simulation Defaults** — set global defaults for ESLI model, alpha steps, Monte Carlo trials, and noise level. These pre-fill all simulation forms but can be overridden per run. Click **Save** to apply; **Reset to defaults** to revert.

**Appearance** — choose an accent color (Amber, Blue, Sage, Teal, Lavender, Rose) that changes the header, active tab, and button highlights. Toggle fullscreen with the button or press **F11**.

---

## What to Expect

| Metric | What it means |
|---|---|
| **FBI** | 1 if the correct bus was identified, 0 otherwise |
| **FBLSR** | Percentage of scenarios where the correct bus was identified |
| **FLE** | Signed error in estimated fault position (% of segment length) |
| **MAFLE** | Mean absolute FLE across all identified cases |

A FBLSR of 88.9% and MAFLE of 0% is the expected result on the 28-bus feeder with SRTP enabled under Constant Impedance loads and no DER, consistent with the published results.

Under DER conditions, FBLSR may vary with PV penetration level and configuration. Line impedance uncertainty is the dominant sensitivity factor — accuracy degrades faster with impedance noise than with PMU measurement noise or DER injection uncertainty.

---

## Authors

Raingel Vryse Mendoza, Grentson Suguitan, and Ludwig Huntley Vengco  
Electrical and Electronics Engineering Institute  
University of the Philippines Diliman

**Paper:** *Fault Localization Using Modified Event Source Location Identification on Distribution Networks*
