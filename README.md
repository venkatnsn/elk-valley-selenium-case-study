# Elk Valley Selenium — a water-quality modelling case study

This is a short technical case study on the selenium problem in the Elk Valley coalfield, British Columbia, Canada. I put it together in my own time as a portfolio piece for water-quality modelling roles, and as a way of pulling together a workflow I use in practice: a calibrated 1D reach-scale transport model, a Latin Hypercube Monte Carlo wrapper for uncertainty, five management scenarios expressed as compliance probabilities, and a simple bioaccumulation step so the water-column numbers can be read against a tissue benchmark.

Everything here is built from public data (EVWQP annual reports, peer-reviewed literature, WSC flow records). The case is a teaching example; it is not a site-specific regulatory model.

**Author.** Dr Siva Naga Venkat Nara — Research Fellow, Water Quality Modelling, Geochemistry & Hydrology, University of Melbourne.
**Contact.** [venkat.nsn@gmail.com](mailto:venkat.nsn@gmail.com) · +61 451 589 633 · [LinkedIn](https://linkedin.com/in/sivanagavenkatnara)

---

## Why Elk Valley

The Elk Valley selenium problem is one of the more technically awkward mine-water cases on the record. It sits at the intersection of several things that a water-quality modeller actually has to deal with: a diffuse geochemical source term that cannot be switched off by adding treatment capacity, a regulated staged-milestone compliance framework (the Elk Valley Water Quality Plan, EVWQP), a pollutant whose speciation controls both transport and bioavailability, and a receptor (westslope cutthroat trout) whose tissue burden matters more than the water-column number. That combination is exactly what I wanted to demonstrate I can model end-to-end.

The same workflow — calibrate, quantify uncertainty, translate to compliance probability, translate to tissue — transfers straightforwardly to uranium in sandstone-hosted deposits, oil-sands process water, legacy metals at closure sites, or analogous Australian and European cases operating under ANZG (2018) or the EU Water Framework Directive.

---

## What's in the repository

| File | What it is |
|---|---|
| `Case_Study.pdf` | The primary deliverable. Four-page technical report: executive summary, methodology, calibration, scenario analysis, bioaccumulation, limitations, conclusions. One figure (compliance bar chart).|
| `Methods_Appendix.pdf` |appendix: 11 governing equations, Monte Carlo parameter distributions, station-by-station calibration statistics, scenario summary, sensitivity ranking. |
| `notebook/ElkValley_Se_Analysis.ipynb` | The Python notebook behind the report (source terms, 1D transport, calibration, 2000-run LHS Monte Carlo, scenarios, bioaccumulation). The markdown is deliberately minimal. Cached outputs may be out of date; re-run before citing specific numbers. |

---

## Headline results

Every number here appears in the report.

| Metric | Value |
|---|---|
| Monte Carlo runs | 2000 (Latin Hypercube) |
| Uncertain parameters | 5 (Q, G, K<sub>d</sub>, η, C<sub>bg</sub>) |
| Calibration stations | 4 + pooled |
| Pooled NSE | 0.82 ("Very Good" under Moriasi et al. 2007) |
| Pooled PBIAS | −4.3% |
| S0 baseline — P(Se ≤ 4 µg/L) | 23% |
| S1 AWT only | 41% |
| S2 source control only | 33% |
| S3 combined AWT + source control | 76% ← the only near-term pathway over my 75% planning threshold |
| S4 aspirational full reclamation | 98% |
| Treatment efficiency distribution | Triangular(0.55, 0.68, 0.82) |
| BCF (Chapman et al. 2010, lotic) | 900 L/kg |
| Primary tissue threshold | 11 µg/g dw egg-ovary (BC MOE 2014) |
| Effects threshold | 4 µg/g dw egg-ovary (Lemly 1993) |

---

## Methods, briefly

**Governing equations** (full list in the Methods Appendix):

- Retardation: *R* = 1 + (ρ<sub>b</sub> / n) · K<sub>d</sub>
- 1D advection–dispersion: ∂C/∂t = −u·∂C/∂x + D·∂²C/∂x² − λC
- Steady-state reduction (Pe > 50): dC/dx = −(λ/u)·C
- Tributary mixing: C<sub>mix</sub> = (Q<sub>up</sub>·C<sub>up</sub> + Q<sub>trib</sub>·C<sub>trib</sub>) / (Q<sub>up</sub> + Q<sub>trib</sub>)
- Treatment: C<sub>out</sub> = C<sub>in</sub> · (1 − η), η ~ Triangular(0.55, 0.68, 0.82)
- Monte Carlo estimator: P(C ≤ 4) = (1/N) Σ 𝟙(C<sub>i</sub> ≤ 4), N = 2000
- Bioaccumulation: C<sub>tissue</sub> = BCF × C<sub>water</sub> / 1000, BCF = 900 L/kg

**Uncertainty.** Log-normal on streamflow; Triangular on treatment efficiency; Uniform on Se generation rate, K<sub>d</sub>, and background Se. I deliberately keep the priors bounded rather than over-specifying distributions the site data don't support. Spearman rank correlations identify streamflow (ρ = −0.68) and the Se generation rate (ρ = +0.61) as the dominant uncertainty drivers.

**Scenarios.**

| | Description | Median Se (µg/L) | P(Se ≤ 4 µg/L) |
|---|---|---|---|
| S0 | Baseline (current operations) | 18.3 | 23% |
| S1 | AWT only, η ~ Triangular | 10.1 | 41% |
| S2 | Source control only (≈20% reduction) | 12.8 | 33% |
| S3 | Combined AWT + source control | 5.9 | 76% |
| S4 | Aspirational full reclamation | 1.2 | 98% |

---

## Scope and limitations

This is a screening-level, public-data case study built on a steady-state 1D representation and simplified tissue-response assumptions. It is intended to demonstrate modelling workflow and decision framing, not to replace a site-specific transient groundwater–surface-water model or a formal Environmental Effects Monitoring programme. All source-term numbers and monitoring data are drawn from publicly available EVWQP annual reports and peer-reviewed literature.

---

## Reproducibility

**Requirements** (Python 3.10+): `numpy`, `pandas`, `scipy`, `matplotlib`, `jupyter`.

```bash
git clone <this-repo>
cd elk-valley-selenium-case-study
pip install numpy pandas scipy matplotlib jupyter
jupyter notebook notebook/ElkValley_Se_Analysis.ipynb
```

Then `Kernel → Restart & Run All`. End-to-end runtime on a laptop is about 30 seconds; the 2000-run LHS Monte Carlo is the slowest step (~15 s).

---

## Key references

1. ANZG (2018). *Australian and New Zealand Guidelines for Fresh and Marine Water Quality.* Australian and New Zealand Governments.
2. BC Ministry of Environment (2014). *Ambient Water Quality Guidelines for Selenium — Technical Report Update.* Victoria, BC.
3. Chapman, P.M., Adams, W.J., Brooks, M.L., et al. (2010). *Ecological Assessment of Selenium in the Aquatic Environment.* SETAC Press, Pensacola, FL.
4. Lemly, A.D. (1993). Guidelines for evaluating selenium data from aquatic monitoring and assessment studies. *Env. Monit. Assess.* 28(1): 83–92.
5. Moriasi, D.N., Arnold, J.G., Van Liew, M.W., et al. (2007). Model evaluation guidelines for systematic quantification of accuracy in watershed simulations. *Trans. ASABE* 50(3): 885–900.
6. Teck Resources Ltd (2020). *Elk Valley Water Quality Plan — 2019 Annual Report.* Vancouver, BC.
7. US EPA (2016). *Aquatic Life Ambient Water Quality Criterion for Selenium — Freshwater 2016.* EPA-822-R-16-006.
8. Zhu, J., Xu, Y., Huang, J., et al. (2018). Selenium distribution, speciation, and leaching from coal-mine waste rock in the Elk Valley. *Sci. Total Environ.* 630: 1099–1108.

---

## License

The written case study and methods appendix are released under CC BY 4.0 — feel free to share and adapt with attribution. The Python / Jupyter code is released under the MIT License (see `LICENSE`).

---

## Contact

For questions, collaboration, or regenerated model outputs / figures:

**Dr Siva Naga Venkat Nara**
Research Fellow — Water Quality Modelling, Geochemistry & Hydrology
University of Melbourne, Australia
[venkat.nsn@gmail.com](mailto:venkat.nsn@gmail.com)  |  +61 451 589 633
[LinkedIn](https://linkedin.com/in/sivanagavenkatnara)
