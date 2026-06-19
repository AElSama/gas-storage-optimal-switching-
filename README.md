# Optimal Switching for Gas Storage Valuation

Numerical valuation of a natural gas storage facility, modelled as a multi-regime **optimal switching** problem and solved by backward dynamic programming with Monte Carlo regression (Least Squares Monte Carlo).

This project reproduces and extends the gas-storage approach of **Carmona & Ludkovski (2010)**, where the operator's decisions — *inject*, *store*, *withdraw* — are the three switching regimes, and the optimal policy is driven primarily by the gas price.

> Master's project, **El Karoui programme** (Sorbonne Université / École Polytechnique), supervised by **Idris Kharroubi**.
> Full report (FR) and defence slides (FR) are included in this repository.

---

## The problem in one picture

A gas storage facility is a real option: buy and inject when prices are low, withdraw and sell when they are high, while respecting inventory limits and switching costs. Formally, this is a stochastic control problem with an inventory state variable, recast as an **iterated optimal stopping** problem across three regimes.

![Optimal control regions](figures/control_regions.png)

*Optimal control regions at 3 and 6 months to maturity, in the (price, inventory) plane: inject at low prices (left), withdraw at high prices (right), wait in the transition band (centre). Reproduction of Figure 2 of Carmona & Ludkovski (2010).*

---

## What is in this repository

| File | Description |
|------|-------------|
| `notebooks/` | Jupyter notebooks: price simulation, backward solver, control-region plots |
| `report_FR.pdf` | Full written report (French) |
| `slides_FR.pdf` | Defence slides (French) |
| `figures/` | Generated figures (control regions, inventory paths, convergence) |

---

## Method

The state is the pair **(gas price, inventory)**. The price follows a mean-reverting (Ornstein–Uhlenbeck-type) dynamic; the inventory is path-dependent and controlled by the chosen regime.

The solver works **backward in time**:

1. Simulate price paths forward.
2. At each step, for each regime, estimate the continuation value by regression on a polynomial basis of the state.
3. Choose the regime maximising immediate cash-flow (minus switching cost) plus discounted continuation value.
4. Propagate value functions back to *t = 0*.

Two schemes from the paper are considered:

- **MITvR** — a fixed inventory grid with a price regression at each inventory level. Simple and robust. **Implemented and validated.**
- **BLSM** — a bivariate regression on (price, inventory) with quasi-simulation of the inventory paths. More elegant but numerically delicate (see below).

---

## Results

**MITvR — Example 1 (single-factor OU price).**
Estimated storage value **≈ 9.75 M\$**, against a reference range of **9.35–9.86 M\$** in the paper. The optimal control regions (figure above) reproduce the expected geometry: the decision is governed primarily by price, with injection at low prices and withdrawal at high prices.

The reflected-BSDE / analytical benchmark (Pham–Vath) is matched to within **~2%** on the reproduction cases.

---

## On the BLSM scheme — an honest note

The BLSM scheme is presented **conceptually** in the report but is *not* claimed as a fully validated implementation here, and that is a deliberate choice rather than an omission.

The difficulty is concentrated in one specific point: the **quasi-simulation of the backward inventory trajectories**. With a naive single inventory vector, all simulated paths collapse toward a single inventory level in the backward pass, so the value function cannot be evaluated across the full inventory range. A correct implementation requires maintaining **one inventory vector per current regime** and alternating LSM / TvR updates — a construction that is itself the subject of dedicated literature (Denault, Simonato & Stentoft 2013; Balata & Palczewski 2018).

Rather than ship an unstable BLSM that "sometimes" returns the right number, the repository keeps the **validated MITvR** as the working method and documents BLSM as a known hard problem with the right references. This reflects how the method should be understood, not a gap glossed over.

---

## References

- R. Carmona, M. Ludkovski (2010). *Valuation of energy storage: an optimal switching approach.* Quantitative Finance.
- B. Djehiche, S. Hamadène, A. Popier (2009). *A finite horizon optimal multiple switching problem.* SIAM J. Control Optim.
- M. Denault, J.-G. Simonato, L. Stentoft (2013). *A simulation-and-regression approach for stochastic dynamic programs with endogenous state variables.* Computers & Operations Research.
- A. Balata, J. Palczewski (2018). *Regress-Later Monte Carlo for optimal control of Markov processes.*

---

## Running the code

```bash
pip install numpy matplotlib pandas
jupyter notebook notebooks/
```

The notebooks are self-contained: run top to bottom to regenerate the figures.
