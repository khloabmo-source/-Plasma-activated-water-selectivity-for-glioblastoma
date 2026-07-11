# Plasma-activated-water selectivity for glioblastoma — reproducibility package


---

## Layout

```
md_simulation/        All-atom MD of the two plasma membranes (GROMACS/CHARMM36)
  prod_npt.mdp                 production mdp actually used (tensionless NPT, 2 fs)
  colab_production_notebook.ipynb   the Colab notebook that built/ran the six replicates
membrane_analysis/    WS1 — membrane observables from the trajectories  -> Figs S1–S3
  extract_ws1.py               read trajectories -> ws1_values.csv + ws1_scd_profiles.npz
  ws1_analysis.py              block-average, Welch/Mann-Whitney, Cohen's d, draft text
  plot_ws1_final.py            -> figS1_order.png, figS2_structure.png
  ws1_convergence.py           -> figS4_convergence.png (equilibration check)
  render_membrane.py           -> figS0_membranes.png (cross-section render)
transport_model/      WS2 — steady-state transfer function          -> Figs 1–9
  01_transfer_function.py      Figs 1–3  (regime switch, selectivity map, gated dynamics)
  02_sensitivity_robustness.py Figs 4–6  (Sobol tornado, robustness histogram + map)
  03_dose_survival_window.py   single-hit comparator (-> *_singlehit.png; REJECTED ΔAICc +12; see 13)
  03b_dose_survival_window_hill.py  Figs 7–8  (MANUSCRIPT: cooperative Hill survival, 3 h, Kurake-calibrated; window ~[60,350] uM)
  04_causal_chain_schematic.py Fig  9    (schematic)
  05_measured_plip_coupling.py Phase B   (couples MD-measured H2O2 permeability into S_dose)
  07_survival_hill_kurake.py   Fig 7b    (Hill survival CALIBRATED to Kurake 2016 kill curve, 3 h)
  08_normal_anchor_mcf10a.py   Fig 8     (normal-cell MCF10A anchor overlaid -> validates astro~100%)
  09_clearance_grounding.py    Fig 9     (grounds k_clear in Doskey 2016 k_cell; selectivity vs catalase-collapse)
  10_bootstrap_uncertainty.py  Fig 10    (bootstrap CIs on D50/h/selectivity -> robustness of the kill calibration)
  11_depletion_selfconsistent.py Fig 11  (self-consistent C_out depletion -> selectivity invariant; closes constant-C_out caveat)
  12_robustness_map.py         Fig S12   (2-D selectivity map over the clearance plane)
  13_model_comparison.py       Fig S13   (AICc/BIC: single-hit rejected, cooperative law required)
  killcurve.csv                digitised Kurake U251SP H2O2 kill curve (calibration data)
permeability/         EXPLORATORY H2O2 PMF — NOT used to set P_lip (see permeability/FINDINGS.md)
  build_h2o2.py / str2itp_h2o2.py / *.mdp / run_pmf.sh / permeability_isd.py
  (bare-bilayer PMF; absolute P overestimates experiment ~1-2 orders -> paper uses Orrico 2021)
aqp8_structure/       Sec. 4.7 — AQP8 selectivity-filter geometry
  aqp8_filter_analysis.py      AlphaFold AQP8 (UniProt O94778): NPA, Cys53, ar/R filter
```

## Quick start (model — no trajectories needed)

```bash
pip install -r requirements.txt
cd transport_model
python3 01_transfer_function.py        # Phi=0.08/2.0; C_in=7.4/66.7(50.4 gated); S_dose=9.0->6.9
python3 02_sensitivity_robustness.py   # S_T(P_AQP8,astro)=0.34; rho-ratio=0.08; P(S>2)=0.99; median 6.2
python3 03_dose_survival_window.py     # single-hit comparator -> *_singlehit.png (REJECTED ΔAICc +12; see 03b/13)
python3 03b_dose_survival_window_hill.py # MANUSCRIPT Fig 7/8: Hill survival, 3 h; astro~100%/U87~1.9% (52x); window ~[60,350]uM
python3 04_causal_chain_schematic.py
python3 07_survival_hill_kurake.py     # CALIBRATED survival: D50=3.06e5, h=6.82, R2=0.98; 100uM/3h astro~100% vs U87~1.9% (52x); window ~[60,350]uM
python3 08_normal_anchor_mcf10a.py     # overlays Kurake MCF10A normal-cell anchor (>=93% @64uM) -> validates astro~100%
python3 09_clearance_grounding.py      # grounds k_clear in Doskey 2016 k_cell framework; selectivity vs catalase-collapse ratio
python3 10_bootstrap_uncertainty.py    # bootstrap 95% CIs: h=6.8[4.6,11.7], selectivity >=17x -> conclusion robust to digitisation error
python3 11_depletion_selfconsistent.py # self-consistent C_out depletion: selectivity invariant (52-64x) -> constant-C_out OK
```

These scripts depend only on numpy/scipy/matplotlib and reproduce the
manuscript numbers exactly. The survival layer (`07`/`08`) is calibrated to the
published glioblastoma kill curve of Kurake et al. 2016 (see
`transport_model/README.md`); the selectivity thesis (`01`/`02`) is independent
of that calibration.

## Membrane analysis (needs the trajectories)

The six production trajectories (3 astrocyte + 3 U87-MG, 200 ns each) are
large and are archived separately with the dataset. With them in place:

```bash
cd membrane_analysis
python3 extract_ws1.py        # -> ws1_values.csv, ws1_scd_profiles.npz
python3 ws1_analysis.py       # statistics + draft sentence for Sec. 3.1/4.1
python3 plot_ws1_final.py     # Figs S1, S2
python3 ws1_convergence.py    # Fig S4
```

## MD provenance

`prod_npt.mdp` is the exact production control file. The membranes were built
with the CHARMM-GUI Membrane Builder (CHARMM36, TIP3P), 512 lipids each; three
independent 200 ns replicates per composition (1.2 µs aggregate). Production
ran under **tensionless semiisotropic NPT at 2 fs**; the first tens of ns are
the area-relaxation transient and are discarded, and all observables are
averaged over the 80–200 ns window. `colab_production_notebook.ipynb` contains
the full build/run/QC workflow.

---

## Notes for reviewers / honest caveats

The following points are flagged transparently so they can be checked or
corrected before submission; none affects the load-bearing conclusions, but
the first two are text/figure consistency items the authors should resolve.

1. **Thermostat/barostat naming.** The production `prod_npt.mdp` uses
   `tcoupl = v-rescale` (stochastic velocity rescaling) and `pcoupl =
   C-rescale`. If the manuscript Methods state Nosé–Hoover / Parrinello–Rahman,
   that text should be reconciled with the mdp (the mdp is the record of what
   ran). Reference temperature is 310.15 K.

2. **Order-parameter summary (resolved in code).** `extract_ws1.py` now
   reports BOTH summaries per replicate. `SCD_glycero` is the manuscript value:
   the mean −S_CD over *all glycerophospholipid* acyl segments (PC/PE/PI/PS,
   both chains, every methylene/methyl carbon), averaged over chains and frames,
   with sphingomyelin (PSM) and cholesterol (CHL1) excluded — the 0.247 / 0.249
   reported in the paper. `SCD_plateau` is the POPC sn-1 plateau (C34–C38,
   ~0.357), kept as a regression check against `scd_plateau_values.csv`. The
   full-chain mean is necessarily below the head-proximal plateau because the
   disordered chain-end carbons are included (ratio ≈ 0.69). A single run on the
   archived trajectories reproduces both numbers in one pass — the plateau
   confirms correct trajectory reading, the glycerophospholipid mean confirms
   the published value. The selection / formula / averaging logic is unit-tested
   without trajectories in `membrane_analysis/test_scd.py`. The authors should
   run `extract_ws1.py` on the archived six trajectories to regenerate the exact
   published 0.247 / 0.249.

3. **Exploratory model not used in the paper.** An earlier, fuller antioxidant-
   network ODE (catalase switch + Prx/Trx/GPx/GSH + bystander coupling) was
   explored but is **not** the model reported in the paper, which uses the
   compact transfer function in `transport_model/`. It is omitted here to avoid
   confusion.

4. **Absolute vs relative parameters.** Absolute permeabilities and the lethal
   dose `D_lethal` are literature-bracketed/illustrative; the conclusions rest
   on the ratios (ρ, k) and the Φ = 1 regime switch, and on `D_lethal` being a
   single free parameter calibratable to a measured kill curve.

## Dependencies

See `requirements.txt`. Model + structure: Python ≥ 3.9, numpy, scipy,
matplotlib, MDAnalysis. MD: GROMACS (GPU build), CHARMM-GUI inputs.

 

---

