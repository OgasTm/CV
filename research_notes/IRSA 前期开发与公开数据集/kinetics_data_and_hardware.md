# IRSA early prototype: public biokinetic / dose-rate data and prototyping-ready BLE dosimeter hardware

Scope note (researcher): checked 2026-11-20. Where a full text was read (Europe PMC XML, PMC, GitHub/PyPI/Zenodo/DataCite APIs), figures are quoted from it. Where only a search-engine summary was available, the item is tagged **[search-summary, verify]**. GitHub "license: none" means the GitHub API reports no license file. Without one, default copyright applies and the code is not legally reusable without permission.

**Correction to the brief:** "Levart 2025 EJNMMI Physics" exists, but it is a **caregiver TLD study** (177Lu-DOTATATE, n = 12 family members). It is not a time-activity data set, so it is filed under question 4.

---

## Q1. Published time-activity curves / serial dose-rate data after 177Lu-PSMA, 177Lu-DOTATATE and 131I (open data or supplementary tables)

### Takeaway
No public deposit of **per-patient external dose-rate time series** for 177Lu or 131I therapy was found. Every clinical dose-rate study checked says its data are "available on reasonable request". Usable today:
- Published summary statistics: means, SD, ranges and percentiles of Teff at defined time points, from Schütze 2026, Kurth 2018, Bailey 2026 and Turku 2024.
- One per-patient table of early and late Teff (Monserrat Fuertes 2023, n = 20, CC BY).
- Imaging-based open sets: SNMMI/Deep Blue 177Lu SPECT/CT with 2 patients × 4 time points, and the MRTDosimetry phantom set.

That is enough to build priors and a synthetic-data simulator. It is not enough to validate a filter on real raw wearable traces, which would need a data-sharing request or IRSA's own pilot data.

### Cited Findings

**Table Q1-a: Clinical serial dose-rate / Teff sources**

| Name | Contents | Size | License / access | Link | IRSA module served | Caveats |
|---|---|---|---|---|---|---|
| Schütze C et al., *Nuklearmedizin* 2026;65:60–68 (multicentre, Germany) | 8 departments; 210 dose-rate measurement sets; Pluvicto, Lutathera, in-house PSMA (ihPSMA), in-house RRT (ihRRT); mean activity 7149 ± 522 MBq. Measured at 2 m with Berthold LB 123/UMO LB123, Thermo FH-40, Step Sensor DLMon (wall) and Automess 6150AD (Table 1). Time bins: 0–6, 12–20, 24–30, 36–42, 44–48, 72–96, 144–168 h p.i. | Aggregate only | Article CC BY (open access); no data-availability statement found in the full text | [PMC13046430](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML); DOI 10.1055/a-2784-7468 | Priors on Teff per compound; tests of how the fit window biases the estimate | Summary statistics only. Fits restricted to ≤48 h give shorter Teff than fits to ≤168 h |
| Kurth J et al., *EJNMMI Res* 2018;8:32 | 50 patients, 177Lu-PSMA-617, 6.3 ± 0.5 GBq, 2 centres. Dose rate at 2 m at 2, 4, 24, 48, 72 h. Whole-body excretion measured in 25 patients at 7 time points | Aggregate tables (mean/SD/min/max) | CC BY 4.0; data on reasonable request | [PMC5897276](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/) | Population curve for 177Lu-PSMA; excretion prior (see Q3) | 72 h data available for only 25 patients |
| Monserrat Fuertes T et al., *EJNMMI Phys* 2023 | 20 patients, 177Lu-DOTATATE 7.4 GBq × 4 cycles. Six whole-body dose-rate measurements at about 0, 2, 6, 24, 48, 168 h; cycles 1 and 4 for 16 patients. **Table 2 gives per-patient Teff(6,24 h) and Teff(24,48,168 h) with uncertainties** | 20 rows | CC BY 4.0; raw data on request | [PMC10477145](https://pmc.ncbi.nlm.nih.gov/articles/PMC10477145/); DOI 10.1186/s40658-023-00570-7 | Validation of "early-window vs late-window" Teff estimation; prior on the bi-exponential slow phase | Raw dose-rate values not tabulated. Discharge at <20 µSv/h at 1 m after a 24 h stay |
| Bailey DL et al., *Asia Ocean J Nucl Med Biol* 2026;14(1):1–9 | 250 subjects, 131I thyroid ablation, 1–6 GBq. Ceiling-mounted GM detector (LND7231 tube), 1.8 m above bed, **1-minute sampling**, CSV output | Aggregate + example traces | CC BY 3.0; **no data-availability statement** | [PMC12854189](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/) | Artefact model (posture, voiding, shower); 131I Teff prior | Device is commercial (Globalsonics "AustralRad PMS"); authors declare a conflict of interest |
| Turku DTC study, *EJNMMI Phys* 2024 (s40658-024-00701-8) | 110 differentiated thyroid cancer patients (135 treatments in Table 1); ceiling remote dose-rate meter; 1.11 or 3.70 GBq | Aggregate | CC BY 4.0; data on reasonable request | [PMC11604915](https://pmc.ncbi.nlm.nih.gov/articles/PMC11604915/) | Covariate-informed prior for 131I Teff (BMI, GFR, diabetes) | Inpatient phase only |

**Key numbers from these sources**
- **Schütze 2026, fits to ≤48 h**: Teff Pluvicto 1.30 d, Lutathera 1.40 d, ihPSMA 1.45 d, ihRRT 1.97 d. **Fits to ≤168 h**: 1.60, 1.89, 2.10 and 2.85 d. Differences were significant for all compounds except Lutathera — [Schütze 2026](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)
- **Schütze 2026, ≤168 h detail**:

  | Compound | Mean ± SD (d) | Median (d) | Range (d) | 90th percentile (d) |
  |---|---|---|---|---|
  | Lutathera | 1.89 ± 1.14 | 1.86 | 0.66–3.66 | 2.59 |
  | ihPSMA | 2.10 ± 0.98 | 2.04 | **0.35–6.65** | 3.08 |
  | ihRRT | 2.85 ± 1.12 | 2.72 | 0.90–5.82 | 3.88 |

  The "0.35–6.65 d" range quoted in the IRSA brief is the **ihPSMA ≤168 h subgroup**, not all 177Lu — [Schütze 2026](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)
- **Schütze 2026, ≤48 h detail**:

  | Compound | Mean ± SD (d) | Median (d) | Range (d) | 90th percentile (d) |
  |---|---|---|---|---|
  | Lutathera | 1.40 ± 0.75 | 1.11 | 0.58–3.06 | 2.35 |
  | ihPSMA | 1.45 ± 0.76 | 1.20 | 0.35–4.84 | 2.58 |
  | ihRRT | 1.97 ± 1.23 | 1.67 | 0.40–6.19 | 2.44 |

  — [Schütze 2026](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)
- **Schütze 2026, recommended conservative Teff (Table 3)**: ihRRT 4.0 d, ihPSMA 3.5 d, Lutathera 3.0 d, Pluvicto 2.0 d. Estimated follow-up dose to third parties per application: 0.31, 0.17, 0.10 and 0.07 mSv — [Schütze 2026](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)
  - The abstract calls these the "90th percentile", but they are higher than the 90th percentiles in the results text (e.g., ihPSMA 3.5 vs 3.08 d). Treat them as rounded-up recommendations.
- **Schütze 2026, methods advice**: the authors stress including at least one terminal-phase data point. They suggest supplementing early-discharge fits with population-based decay constants. This is direct support for a Bayesian prior plus wearable update design — [Schütze 2026](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)
- **Kurth 2018 (Table 3), dose rate at 2 m**:

  | Time p.i. | Mean ± SD (µSv/h) | Range (µSv/h) |
  |---|---|---|
  | 2 h | 3.6 ± 0.7 | 2.2–5.0 |
  | 3.9 h | 2.8 ± 0.6 | not recorded in these notes |
  | 24.2 h | 1.6 ± 0.6 | not recorded in these notes |
  | 45.7 h | 1.1 ± 0.5 | not recorded in these notes |
  | 67.7 h | 0.7 ± 0.4 | not recorded in these notes |

  Teff 40.5 ± 9.6 h (21.7–85.7 h). Maximum dose to a member of the public per cycle: about 250 ± 55 µSv if discharged at 48 h, about 190 ± 36 µSv at 72 h — [Kurth 2018](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/)
- **Monserrat Fuertes 2023 (Table 2), median (min, Q1, Q3, max)**: Teff(6,24 h) = 25 h (12, 21, 36, 92); Teff(24,48,168 h) = 58 h (35, 47, 67, 119). The per-patient pairs show that early two-point Teff systematically underestimates the late phase. Example: patient 10 has 53 ± 32 h early vs 119 ± 19 h late — [Monserrat Fuertes 2023](https://pmc.ncbi.nlm.nih.gov/articles/PMC10477145/)
- **Bailey 2026**:
  - Mean 131I Teff 11.9 ± 3.2 h (range 5.0–23.1 h, n = 250). Under 55 years: 11.5 ± 3.0 h; 55 or older: 14.4 ± 7.4 h (P = 0.0007, univariate t-test) — [Bailey 2026](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/)
  - Artefacts: highest readings with the patient in bed under the detector; lower plateaus when sitting in a recliner; **stepwise drops attributed (not verified) to voiding**; long low periods attributed to showering.
  - The authors did not edit the data. A pilot removal of out-of-bed sections barely changed the fitted Teff.
  - Calibrated with a 137Cs point source to read Sv/h at 1 m, with no distance correction.
- **Turku 2024**: median 131I Teff 12.60 h (Q1 10.35, Q3 14.75). Longer Teff with higher BMI (p = 0.004), lower GFR (p < 0.001) and diabetes (p = 0.007). Age and treatment cycle not significant — [Turku 2024](https://pmc.ncbi.nlm.nih.gov/articles/PMC11604915/)
- **Other 131I Teff sources [search-summary, verify]**:
  - Remy et al., *J Nucl Med* 2008;49(9):1445: whole-body retention in 254 patients and urinary excretion in 30. Mean Teff about 10.5 h with rhTSH (n = 36) vs about 15.8 h after hormone withdrawal (n = 218) — [JNM tables](https://jnm.snmjournals.org/content/49/9/1445/tab-figures-data)
  - Health Physics 2001: more than 250 administrations, median Teff ≥14 h — [PubMed 11513465](https://pubmed.ncbi.nlm.nih.gov/11513465/)
  - Hyperthyroid/thyrotoxicosis: body-burden Teff about 111.4 h, release at <10 µSv/h at 1 m — [PMC3003283](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3003283/)
  - Bi-exponential 1 m dose-rate model with a thyroid uptake fraction — [Diagnostics 2024;14(10):1010](https://www.mdpi.com/2075-4418/14/10/1010)

**Table Q1-b: Open imaging / simulation data sets (indirect use)**

| Name | Contents | Size | License / access | Link | IRSA module served | Caveats |
|---|---|---|---|---|---|---|
| Deep Blue "Lu-177 DOTATATE Anonymized Patient Datasets: Multi-Time Point Lu-177 SPECT/CT Scans" (Dewaraja & Van, 2021), used for the SNMMI Dosimetry Challenge | 2 patients; SPECT/CT at 4 time points in the week after therapy; Siemens Intevo; DICOM | Not reported by DataCite | **License not stated in DataCite metadata; landing page returned HTTP 403 to automated fetch** | [DOI 10.7302/0n8e-rz46](https://doi.org/10.7302/0n8e-rz46) ([DataCite record](https://api.datacite.org/dois/10.7302/0n8e-rz46)) | Organ TACs from which to synthesise "external dose-rate" traces with a forward model | Only 2 patients. Exact hours not confirmed |
| SNMMI challenge results papers | 2021 JNM supplement; 2025 JNM 66(3):441; 2026 "part 5: segmentation" with figshare additional file (CC BY 4.0) | — | Open access; figshare CC BY 4.0 | [JNM 62 Suppl 3 36S](https://jnm.snmjournals.org/content/62/Supplement_3/36S); [JNM 66(3):441](https://jnm.snmjournals.org/content/66/3/441); [figshare c.8432063](https://doi.org/10.6084/m9.figshare.c.8432063) | Spread of time-integration methods across participants | Imaging-based, not wearable-like |
| Community code for the challenge | carluri/snmmi_dosimetry_challenge (MIT); luisacbscs/SNMMIDosimetryChallenge (task 4, no license) | — | MIT / none | [carluri](https://github.com/carluri/snmmi_dosimetry_challenge); [luisacbscs](https://github.com/luisacbscs/SNMMIDosimetryChallenge) | Starter code for TAC fitting | Small, unmaintained repos (last push 2023/2024) |
| MRTDosimetry (EMPIR) reference database | 3D-printed ICRP-110-based phantom filled following a realistic 177Lu-DOTATATE biokinetic; multi-time-point SPECT/CT; pharmacokinetic curves | — | Public on OSF; **license not verified** | [OSF 69NGE](https://osf.io/69nge); [paper PMC10165308](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10165308/) | Ground-truth biokinetics for testing the estimator | Phantom, not patient |
| Derived data for "Isolating the effect of spatial resolution… 177Lu-DOTATATE" (Zenodo, 2026-11-18) | Noise-free ground-truth activity maps of the MRTDosimetry phantom at **1, 4, 24, 40, 72, 144 h** | 1.2 MB zip | CC BY 4.0 | [10.5281/zenodo.24343110](https://doi.org/10.5281/zenodo.24343110) | Known-truth multi-time-point activity for simulator unit tests | Linked article "submitted" |
| radot OpenGATE validation data, patient p142 (Zenodo, 2026-10-30) | Anonymised trunk CT, liver mask, OpenGATE 10 voxel dose rates for 177Lu, 90Y, 131I, 225Ac (decay chain included) | 119 MB | CC BY 4.0 | [10.5281/zenodo.23809698](https://doi.org/10.5281/zenodo.23809698); library [github.com/dsarrut/radot](https://github.com/dsarrut/radot) | Monte Carlo infrastructure that could be adapted to compute body-surface/1 m fluence per organ source | Internal dose rate, not external. radot license not checked |
| Iori et al. "Real-Time Detection of Extravasation Events…" (Zenodo, 2026-04-24) | Portable gamma-detector monitoring during 18F/68Ga injections and 177Lu-DOTATATE administrations | 0.5 MB zip | CC BY 4.0 | [10.5281/zenodo.19251295](https://doi.org/10.5281/zenodo.19251295) | Real detector count-rate time series for testing step/anomaly detection | Covers minutes during infusion, not days post-therapy. File contents not inspected |

### Inferences
- The minimum-viable kinetics layer can be specified from published aggregates:
  - Use a lognormal prior on Teff per compound. Schütze ≤168 h: median about 1.9–2.7 d, range 0.35–6.65 d. Kurth: 40.5 ± 9.6 h for PSMA-617. Bailey/Turku: about 12 h for 131I DTC.
  - Add a bi-exponential structure, per Monserrat Fuertes (fast and slow phases).
  - Add an explicit "early-fit bias" term. Schütze's ≤48 h vs ≤168 h gap and the per-patient pairs in Monserrat Fuertes quantify it.
- The Bailey artefact descriptions are qualitative but specific enough to drive a synthetic generator:
  - Posture/distance changes appear as multiplicative level shifts that revert.
  - Voiding appears as irreversible downward steps.
  - Showering or absence appears as long low gaps.
  - On a body-worn sensor, distance artefacts mostly vanish, but body-habitus and position artefacts (sensor vs bladder geometry) will appear instead. This last point is our inference and is not tested.
- The most useful real-data request is to Bailey et al. (CSV at 1-minute resolution, 250 subjects) and to Kurth or Schütze (multicentre sets at 2 m).

### Gaps
- No open per-patient external dose-rate time series was found for 177Lu-PSMA, 177Lu-DOTATATE or 131I on Zenodo, Figshare, Deep Blue or as supplementary tables. Searches covered Zenodo (dataset type) and DataCite (umich.deepblue, creator Dewaraja).
- No VISION, TheraP or NETTER-1 dose-rate or retention sub-study data were found to be public. Not searched exhaustively.
- The exact SPECT time points (hours) and license of the Deep Blue SNMMI set could not be confirmed: the page blocked fetch and the DataCite rights field is empty.
- The hyperthyroid 131I figures rest on search summaries only.
- A 2026 Frontiers in Nuclear Medicine article (fnume.2026.1887752) appeared in results but was not read.

---

## Q2. Biokinetic / PBPK models with published parameters usable as priors (ICRP 128, ICRP iodine, PSMA PBPK, open code)

### Takeaway
- **177Lu**: no ICRP reference model exists. ICRP 128 is diagnostic-only. The usable priors are PSMA PBPK models (Kletting 2016 → Begum 2019 → Fele-Paranj 2024). The one public machine-readable implementation is the Fele-Paranj SBML/SimBiology model on GitHub, which **has no license file**.
- **131I**: the ICRP OIR iodine systemic model (Publication 137) plus Leggett's 2010 physiological model are the standard priors. Coefficients are distributed through the free ICRP OIR data viewer.

### Cited Findings
- **ICRP Publication 128** covers diagnostic nuclear medicine, "not … therapeutic applications". It compiles ICRP Publications 53, 80 and 106 and their amendments, so it is not a source for 177Lu therapy kinetics — [search summary of ICRP 128 scope; SAGE listing](https://uk.sagepub.com/en-gb/mst/icrp-publication-128/book248515); [ICRP TG36 poster](https://www.icrp.org/admin/2023_Poster_TG-36.pdf) [search-summary, verify]
- **ICRP Publication 137 (OIR Part 3)** covers iodine, among other elements — [ICRP page](https://icrp.org/page.asp?id=374)
- **ICRP OIR data viewers**: free desktop viewers with an electronic annex for inhalation, ingestion and direct uptake to blood (Windows installer) — [ICRP Dose Viewers](https://icrp.org/page.asp?id=505)
  - The Publication 141 electronic supplement combines OIR Parts 2–4 with a free viewer — [ICRP](https://www.icrp.org/page.asp?id=483) [search-summary, verify]
- **Leggett 2010** proposes a physiological systems model for iodine and compares it with the ICRP model — [NRC ML110200086 PDF](https://www.nrc.gov/docs/ML1102/ML110200086.pdf); DOI [10.1667/RR2243.1](https://www.doi.org/10.1667/RR2243.1)
- **PSMA PBPK model lineage**
  - Kletting et al. 2016 is the base PBPK model for PSMA ligands. Physiological parameters come from the literature; individual parameters were fitted to planar 177Lu-PSMA I&T data from 5 patients. Begum et al. 2019 extended it to 13 virtual mCRPC patients in SimBiology/MATLAB, with no code link — [Begum 2019 PMC6934468](https://pmc.ncbi.nlm.nih.gov/articles/PMC6934468) [search-summary, verify]
  - Hardiansyah et al. 2021 ran a global sensitivity analysis on that model; code release not confirmed — [search summary](https://pmc.ncbi.nlm.nih.gov/articles/PMC5017739)
  - **Fele-Paranj, Saboury, Uribe, Rahmim, *EJNMMI Radiopharm Chem* 2024 ("PBRPK")**:
    - More than 150 ODEs in a "reaction graph" notation, implemented in MATLAB SimBiology.
    - Parameters are population values as in Kletting 2016, listed in Additional file 1, section S.4. Validated against Kletting 2016 data.
    - SBML file at github.com/alifele/Computational-Physics/tree/main/PSMA-PBPK-Model-Matlab.
    - Article license CC BY 4.0 — [PMC10803696](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC10803696/fullTextXML)
  - The GitHub repo `alifele/Computational-Physics` reports **no license** (last push 2024-12-20). The mirror `qurit/PSMA-PBPK-Model-Matlab` also reports **no license** (last push 2024-12-10) — GitHub API queried 2026-11-20 ([alifele](https://github.com/alifele/Computational-Physics), [qurit](https://github.com/qurit/PSMA-PBPK-Model-Matlab))
  - Abdollahi, Fele-Paranj & Rahmim, *Cancers* 2024: "Model-Informed RPT Optimization: impact of PBPK model parameters … in 177Lu-PSMA therapy" — [PMC11430653](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11430653/)
  - arXiv 2510.21054: PBPK plus machine-learning predictive dosimetry for PSMA RPT — [arXiv](https://arxiv.org/pdf/2510.21054)
- **Empirical priors from Q1 that can replace a PBPK model at the whole-body level**: Schütze per-compound Teff distributions, Kurth's 177Lu-PSMA-617 Teff and excretion fractions, Monserrat Fuertes's per-patient bi-exponential phases, and the Turku covariate model for 131I (BMI, GFR, diabetes) — sources as in Q1.

### Inferences
- A wearable measures **whole-body** retention, convolved with geometry. A full >150-ODE PBPK model is over-parameterised for that signal. A practical prior is:
  - 2–3 compartments (blood/fast renal, slow tumour/kidney retention, bladder holding compartment with voiding events).
  - Parameters matched to the Kurth excretion curve and the Schütze/Monserrat Fuertes Teff distributions.
  - The Fele-Paranj SBML can be used offline to generate realistic whole-body plus bladder curves for training. Redistributing it requires the authors' permission, since it has no license.
- For 131I, ICRP OIR systemic coefficients describe thyroid, blood and urinary bladder. Ablation patients (little remnant tissue) and hyperthyroid patients (high uptake, Teff about days) need separate parameter sets.

### Gaps
- No open Python or R implementation of a PSMA or DOTATATE PBPK model with a permissive license was found.
- The exact ICRP 128 / ICRP 53 iodide model content and the OIR viewer file format were not verified.
- No public PBPK or compartment parameter set specific to 177Lu-DOTATATE was located in this pass.
- 225Ac: not researched.

---

## Q3. Excretion data (urine fraction vs time) for 177Lu and 131I

### Takeaway
For 177Lu-PSMA-617, Kurth 2018 gives the best open table: about 30% of activity excreted by 2 h, 50% by 4 h, 70% by 12 h and about 78% by 72 h, mainly via urine. For 131I, quantitative urine-fraction tables were not retrieved in full text. Only search-level summaries are available.

### Cited Findings
- **Kurth 2018, Table 2**: excreted activity between time points in 25 patients. Mean cumulative fraction of administered activity: **30% at 2 h, 50% at 4 h, 70% at 12 h, 76% at 24 h, 77% at 48 h, 78% at 72 h**, "primarily via urine". Measured as whole-body activity before first voiding and at 2, 4, 12, 24, 48, 72 h p.i. — [PMC5897276](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/)
  - Min and max per interval are also tabulated. Example: 2 h excretion ranged 13–42%.
- **Levart 2025**: median 35.8% of administered 177Lu-DOTATATE retained at discharge (about 6.2 h), measured by quantitative SPECT/CT. This implies about 64% eliminated by about 6 h — [PMC12595148](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC12595148/fullTextXML)
- **Remy et al. 2008 (131I DTC)**: urinary excretion measured in 30 patients. The longer Teff after hormone withdrawal is attributed mainly to delayed renal excretion — [JNM 49(9):1445](https://jnm.snmjournals.org/content/49/9/1445/tab-figures-data) [search-summary, verify]
- **Radiat Prot Dosimetry 2013 (131I DTC)**, urine-based Teff:
  - 18.4 ± 1.8 h within the first 24 h; 64 ± 2.7 h between 48 and 120 h.
  - Release at <20 µSv/h at 1 m reached after 48 h for 3.7 and 5.55 GBq, and after 72 h for 7.4 GBq.
  - [PubMed 23535081](https://pubmed.ncbi.nlm.nih.gov/23535081/) [search-summary, verify]
- **Bailey 2026**: stepwise drops in ceiling-detector traces are attributed (unverified) to voiding urine or faeces — [PMC12854189](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/)

### Inferences
- A voiding-step model for 177Lu-PSMA can be calibrated to Kurth's curve: about 50% of activity leaves in the first 4 h. During that window the patient is usually still in hospital, so the at-home wearable phase sees mostly the slow component, with smaller steps.
- Voiding steps will be most visible to the wearable during day 1. Step-detection tests should use day-1 synthetic data at high step amplitude and day 2+ at low amplitude.
- For hygiene guidance, the Kurth table supports saying that most urinary excretion happens in the first 12–24 h. This is an inference from the population mean, not a per-patient guarantee.

### Gaps
- No open per-void urine activity data were found for 177Lu-DOTATATE or for 131I.
- No 131I hyperthyroid excretion table was located.
- 177Lu faecal fraction not quantified in sources read.

---

## Q4. Open data from caregiver / family dosimetry studies (validation endpoints)

### Takeaway
Caregiver data exist only as **published summary statistics**:
- Levart 2025: TLD worn 4 weeks, n = 12 family members of 177Lu-DOTATATE patients, median 0.038 mSv, with time-at-distance diaries.
- A 2025 Japanese 177Lu-PSMA study: home exposure mostly 0 µSv.

Raw per-person data are available on request only. Levart's license is CC BY-NC-ND, which restricts reuse of adapted tables.

### Cited Findings
- **Levart D et al., *EJNMMI Phys* 2025 (DOI 10.1186/s40658-025-00776-x)**:
  - 12 adult family members wore TLDs continuously for 4 weeks. Patients received median 7515 MBq 177Lu-DOTATATE and stayed about 6.2 h.
  - Median dose rate at discharge: **150 µSv/h at 0.1 m and 15 µSv/h at 1 m** (anterior abdomen).
  - Family median effective dose **0.038 mSv over 4 weeks**. Median time with the patient: 39.5 h at 1 m and 19.9 h at <1 m.
  - Table 1 gives time at 1 m and at <1 m for days 1–2 and weeks 1–4, as medians with ranges.
  - License **CC BY-NC-ND 4.0**; data "available from the corresponding author on reasonable request".
  - [PMC12595148](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC12595148/fullTextXML)
- **177Lu-PSMA staff and family study**, *Ann Nucl Med* 2025 (s12149-025-02079-8):
  - 13 medical staff plus family members; 8 measurement sites including patients' homes.
  - Home exposure 0 µSv for 6 of 7 patients and 60 µSv for 1.
  - Dosimeter type for family members not confirmed.
  - [Springer](https://link.springer.com/10.1007/s12149-025-02079-8) [search-summary, verify]
- **Kim et al., JNM 2021 Suppl. 1 abstract 1573**: "Radiation exposure to family caregivers…". Values not retrieved — [JNM](https://jnm.snmjournals.org/content/62/supplement_1/1573/tab-article-info)
- **Kurth 2018**: modelled public dose per cycle, about 250 ± 55 µSv for 48 h discharge, from dose-rate integrals. This is a model-based comparator, not a measured endpoint — [PMC5897276](https://pmc.ncbi.nlm.nih.gov/articles/PMC5897276/)
- **WIDMApp group, 2026 conference abstract**: uses WIDMApp-derived curves in Geant4 to estimate caregiver and staff exposure — [Indico CVUT 2026](https://indico.fjfi.cvut.cz/event/376/contributions/6628/) [search-summary, verify]

### Inferences
- Levart gives a ready **end-to-end validation case**: discharge dose rate at 1 m, time-at-distance diary and measured 4-week caregiver dose. IRSA's exposure estimator can be checked against median 0.038 mSv using the published median inputs. Individual-level validation needs the raw data. Under ND licensing, our own re-derivations should be cited, not redistributed as adapted tables.

### Gaps
- No open individual-level caregiver data from TLDs or electronic personal dosimeters were found for 177Lu-PSMA or 131I.
- Caregiver studies for 131I hyperthyroid outpatients were not located in this pass.

---

## Q5. Off-the-shelf / open-source gamma detectors with Bluetooth and an open API (prototype hardware)

### Takeaway
**Radiacode 10x is the only device found that combines BLE, a mature MIT-licensed Python library, and a real-time dose-rate and spectrum API.**
- Detector: CsI(Tl). Price on the official store 2026-11-20: **US$269** (discounted from $319).
- It carries **no metrological or medical certification** on its product page.

Other options:
- **Open Gamma Detector** (GPL-3.0, NaI(Tl)+SiPM+Pi Pico): fully open hardware, but BLE is not documented.
- **GQ GMC** counters: published serial protocol, no confirmed BLE.
- Certified electronic personal dosimeters (Polimaster RadFlash/PoliSimeter, Thermo RadEye PRD-ER4, Kromek D3S, Mirion Instadose+): have Bluetooth but **no public API** found.

### Cited Findings

**Table Q5: Candidate hardware**

| Device | Detector / specs | Price | API / openness | Certification | Link | IRSA module served | Caveats |
|---|---|---|---|---|---|---|---|
| **Radiacode 103** (also 101/102/103G/110) | CsI(Tl), about 1 cm³ (103). Registered photons 0.02–3 MeV. Dose rate 0.01–1000 µSv/h ±15% (137Cs), 0.5 s update (series-level). Sensitivity 30 cps per µSv/h (137Cs). Resolution 8.4% ±0.3% FWHM at 662 keV (103) | US$269 (sale; list $319) | BLE + USB. Python library **cdump/radiacode, MIT**: PyPI 0.4.0 (2026-08-11), repo pushed 2026-11-15, 244 stars. `data_buf()` returns RealTimeData with dose_rate; `spectrum()`; `energy_calib()`; `dose_reset()`. Linux (BlueZ), Windows/macOS (Bleak; macOS by UUID) | No certification statement on the product page (consumer "science gadget") | [Product](https://www.radiacode.com/products/radiacode-103); [specs (search result)](https://radiacode.com/docs/en/devices/technical-specification); [cdump/radiacode](https://github.com/cdump/radiacode); [PyPI](https://pypi.org/project/radiacode/) | Wearable prototype sensor; spectrum allows radionuclide check (113/208 vs 364 keV) | Spec values come from search snippets: the official spec page URLs returned 404 or redirected on fetch. One Radiacode page reportedly gives 100 keV–3 MeV for dose with indication from 30 keV, which conflicts with the 20 keV figure. Battery life not verified |
| Radiacode ecosystem | ckuethe/radiacode-tools (MIT); 303Bryan/ha-radiacode (MIT, Home Assistant BLE); mkgeiger/RadiaCode (MIT, Arduino/ESP32 BLE); bsharper/radiacode.js (MIT, Web Bluetooth); darkmatter2222/Open-RadiaCode-Android (no license); **lucamors/G4Radiacode (GPL-3.0, Geant4 digital twin of the CsI(Tl) crystal)** | — | Open source | — | [GitHub search results 2026-11-20](https://github.com/search?q=radiacode&type=repositories) | ESP32 BLE bridge; browser app; Geant4 model for energy/angular response at 113/208/364 keV | Community-maintained |
| Radiacode Zero | BLE 5.2; 0.5 s display update; IP64 | Not verified | Assumed compatible with the Radiacode app; Python library support not verified | — | [Zero tech specs](https://radiacode.com/docs/de/zero/device/tech-specs) | Smaller form factor | Detector and range not verified |
| **Open Gamma Detector** (OpenGammaProject) | NaI(Tl) scintillator + SiPM + Raspberry Pi Pico; rev 4.2 uses Pico 2 | DIY (BOM not verified) | **GPL-3.0**, 392 stars, last push 2026-06-17; OSHWA-certified (AT000005); USB serial. BLE not documented | None | [GitHub](https://github.com/OpenGammaProject/Open-Gamma-Detector); [OSHWA](https://certification.oshwa.org/at000005.html) | Fully modifiable reference design (could add BLE via Pico W or ESP32) | Known Pico ADC nonlinearity: 4 channels much more sensitive. NaI is hygroscopic and bulkier |
| GQ Electronics GMC-500/600/800 | GM tube | Not verified | Serial protocol GQ-RFC1801 published (GMC-500/500+/600/600+); GMC-800 "open communication protocol" over USB; WiFi on some models; **pygmc** Python package lists GMC-800 as working | None found | [RFC1801](https://www.gqelectronicsllc.com/download/GQ-RFC1801.txt); [downloads](https://www.gqelectronicsllc.com/downloads); [pygmc](https://github.com/ctoth/pygmc) | Cheap bench reference / logger | No BLE confirmed. GM energy over-response at low energy (see Q6). pygmc license not checked |
| Kromek D3S (PRD / ID) | CsI(Tl) 16 cm³ (D3S PRD); 237 g; 1450 mAh LiPo | Not verified | Bluetooth/USB to a pre-paired smartphone app; CIRIS software for other Kromek PRDs; **no public BLE API found** | Security-market PRD | [Kromek PRDs](https://www.kromek.com/prds/); [PNNL D3S spec](https://d5.pnnl.gov/sites/default/files/2022-06/D3SID%20A4%20SPF-DOC-1250%20Rev8.pdf) | Higher-sensitivity reference | Would need vendor SDK |
| Polimaster RadFlash; PoliSimeter ER/ERB; PM1703MO-II BT; PM1605BT; PM1610 | RadFlash: real-time X/gamma EPD with **Bluetooth**, 30 keV–3 MeV, IEC 61526:2024. PoliSimeter ER: 20 keV–10 MeV, IEC 61526:2024. PM1703MO-II BT: CsI(Tl)+SiPM plus GM, Bluetooth 4.0, IP65, Polismart iOS/Android app. PM1610: GM, 20 keV–10 MeV, IEC 61526:2010 (older material), USB or ISO15693, no Bluetooth found | Not verified | Vendor apps; **no public API found** | IEC 61526 (stated by vendor) | [RadFlash](https://polimaster.com/products/dosimeters/radflash/); [PoliSimeter ER](https://polimaster.com/products/dosimeters/polisimeter-er/); [PM1610](https://polimaster.com/products/dosimeters/pm1610/); [PM1703MO-II BT](https://www.berkeleynucleonics.com/pm1703mo-ii-bt/); [PM1605BT](https://polimaster.com/eu/product/x-ray-and-gamma-radiation-dosimeters/electronic-dosimeters/electronic-dosimeter-pm1605_bt/) | Certified comparator; route to a regulated product later | Would need a vendor custom-design agreement (Polimaster advertises custom dosimeter development) |
| Mirion Instadose+ / Instadose VUE | Integrating dosimeter; BLE upload to Instadose Companion app or InstaLink hotspot; reads triggered by button press | Service model | Proprietary algorithm and portal; **no developer API found** | Occupational dosimetry service | [Mirion Instadose](https://www.mirion.com/products/medical/occupational-dosimetry/dosimetry-services/whole-body-area-dosimeters/on-demand-dosimetry-products/instadose-wireless-dosimeter) | Not suited to streaming dose rate | — |
| Thermo RadEye PRD4 / PRD-ER4 | Dose-rate range 10 nSv/h–250 µSv/h (PRD-ER4). Energy 58 keV–6 MeV (current catalog) vs 60 keV–1.3 MeV (older listing). **Bluetooth optional** (adapter sold separately) | Not verified | No public API found | Professional PRD | [Thermo PRD4/PRD-ER4](https://www.thermofisher.com/order/catalog/product/425067127) | Low-dose-rate reference | Energy-range figures conflict across listings. "NetDose" product not found |
| Better Geiger S-1 | Solid scintillator; 73 × 26 × 118 mm | Not verified | BLE/app **not verified** | — | [S-1 manual](https://manuals.online/better-geiger/measuring-instrument/s-1/user-manual/139oq) | — | Unverified |
| Atom Fast (KB Radar) | — | Not verified | AtomConnect app (GammaSpectacular); brochure says "free open source software"; **no protocol documentation found** | — | [AtomConnect](https://www.gammaspectacular.com/blue/software-downloads/atomconnect); [brochure](https://www.gammaspectacular.com/download/pdf_brochures/ATOM-FAST_Brochure.pdf) | — | Unverified |
| Technonia Smart Geiger Pro | Smartphone-attached sensor, used in O'Callaghan 2022 | Low cost (not quantified) | Smartphone app | — | [Sensors 2022](https://www.mdpi.com/1424-8220/22/23/9392) | Precedent for low-cost patient-side 131I measurement | Specs not retrieved |
| Safecast bGeigie Nano (kit) | GM-based mobile survey kit | Not verified | Open repo, last push 2025-10-01; license not shown by GitHub API | — | [bGeigieNanoKit](https://github.com/Safecast/bGeigieNanoKit) | Background logging | Not wearable on the torso |

### Inferences
- **Recommended prototype path**:
  - Radiacode 103 (or 102) as the patient-worn sensor.
  - Read it with `cdump/radiacode` on a phone gateway, or an ESP32 BLE bridge (`mkgeiger/RadiaCode`), or Web Bluetooth (`radiacode.js`).
  - Log the spectrum as well as dose rate. Spectral windows at 113/208 keV vs 364 keV allow nuclide verification and scatter-aware count-to-activity models.
  - Use G4Radiacode to simulate body-worn geometry.
  - Keep a certified electronic personal dosimeter (e.g., an IEC 61526 Polimaster or Thermo unit) as the comparator in phantom and pilot work.
- For relative Teff estimation (IRSA's main job), the absolute energy response matters less than **stability and linearity over time**. The CsI count rate is the natural observable; absolute level comes from the calibrated 1 m measurement at discharge, as in IRSA's design.

### Gaps
- Radiacode battery life, exact dose-measurement energy range and BLE data rate were not confirmed from an official page: the official spec URLs returned 404 or a redirect on fetch.
- No price list was obtained for Kromek, Polimaster, Thermo or Mirion.
- X-Z Lab RadTarge, Mirion RDS-31 and Thermo "NetDose" were not verified.
- No BLE GATT documentation is public for any certified electronic personal dosimeter.

---

## Q6. Energy / angular response at 113/208 keV vs 364 keV, low-dose-rate performance, IEC 61526 / IEC 62387, calibration sources

### Takeaway
- **IEC 61526:2024 (ed. 4)** is the current type-test standard for active personal dosemeters. An AMD1:2026 consolidated version exists.
- **IEC 62387** covers passive integrating systems: 0.01 mSv–10 Sv. For Hp(10), the mandatory photon range is 80 keV–1.25 MeV and type tests are possible over 12 keV–7 MeV (edition-1 table).
- 177Lu's photon lines (113 keV at about 6%, 208 keV at about 11%) sit in the region where uncompensated GM tubes and small scintillators deviate most from 137Cs calibration.
- 133Ba (356 keV) is a validated solid surrogate for 131I (364 keV) in **SPECT/CT** calibration, but a poor one in dose calibrators.
- No published energy or angular response evaluation of Radiacode at 177Lu energies was found.

### Cited Findings
- **IEC 61526:2024 (edition 4.0, published 2024-03-27)**:
  - Applies to active personal dosemeters for Hp(10), Hp(3) and Hp(0.07) from X, gamma, neutron and beta radiation; dose rate may be measured for alarm purposes.
  - Replaces the 2010 edition. Adds hybrid dosemeters and software requirements; linearity harmonised with IEC 62387.
  - Excludes pulsed radiation and ambient/directional dose-equivalent meters.
  - [IEC webstore](https://webstore.iec.ch/publication/66825); [BSI](https://knowledge.bsigroup.com/products/radiation-protection-instrumentation-measurement-of-personal-dose-equivalents-for-x-gamma-neutron-and-beta-radiations-active-personal-dosemeters-2)
  - IEC 61526:2024+AMD1:2026 consolidated version listed — [EVS](https://www.evs.ee/en/iec-61526-2024-amd1-2026-csv-consolidated)
- **IEC 62387** (ed. 2.0 2020; EN IEC 62387 2022): passive photon/beta dosimetry systems, 0.01 mSv–10 Sv.
  - Edition-1 Table 1: Hp(10)/H*(10) mandatory 80 keV–1.25 MeV, maximum 12 keV–7 MeV. Hp(3) mandatory 30–250 keV.
  - Tests not matching the standard for dosemeters with active parts defer to IEC 61526.
  - [IEC 62387 preview](https://webstore.ansi.org/preview-pages/IEC/preview_iec62387%7Bed1.0%7Db.pdf); [IEC 62387:2020](https://webstore.iec.ch/publication/66492)
- **177Lu emissions**: gamma lines at 208 keV (11%) and 113 keV (6%); beta Emax 0.498 MeV; T½ 6.7 d — [ClinicalTrials.gov protocol NCT03525392](https://cdn.clinicaltrials.gov/large-docs/92/NCT03525392/Prot_000.pdf); [NNDC ENSDF 177Lu](https://www.nndc.bnl.gov/ensnds/177/Lu/adopted.pdf)
- **133Ba as a 131I surrogate**:
  - A European multicentre study concluded that traceable solid 133Ba sources can stand in for liquid 131I in SPECT/CT calibration (356 vs 364 keV) — [Tran-Gia et al. 2023, PMC10665282](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10665282/)
  - A NIST study found Capintec dose-calibrator response to 133Ba about 3× that of the same 131I activity — [NIST](https://www.nist.gov/node/1095576)
- **Clinical dose-rate meter calibration practice**: in Schütze 2026, sites used officially calibrated (authority-certified) or in-house verified instruments at 2 m. One ceiling meter at about 2.6 m was calibrated and corrected for distances up to 2.0 m, with annual checks — [Schütze 2026 Table 1](https://www.ebi.ac.uk/europepmc/webservices/rest/PMC13046430/fullTextXML)
  - Bailey's ceiling GM was factory-calibrated with 137Cs in Sv/h at 1 m, with no distance or distributed-source correction — [PMC12854189](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/)
- **O'Callaghan 2022**: evaluated the effect of activity, photon energy, patient size, sensor placement and motion on the Smart Geiger Pro's dose-rate accuracy, and gives recommended use conditions. 4 I-131 patients matched a hospital thyroid uptake probe. CC BY 4.0 — [Sensors 22(23):9392](https://www.mdpi.com/1424-8220/22/23/9392)

### Inferences
- **Calibration protocol suggestion** (inference, not a published protocol):
  1. Calibrate count rate to air kerma or H*(10) with 137Cs (the vendor reference).
  2. Measure the energy-response ratio with 133Ba (356 keV, 131I surrogate) and with **sealed or vial 177Lu** (no long-lived surrogate matches 113/208 keV well). 57Co (122 keV) is a plausible proxy for the 113 keV line but was not checked here.
  3. Characterise body-worn geometry on an anthropomorphic phantom filled with 177Lu or 131I, using a certified IEC 61526 electronic personal dosimeter and a calibrated survey meter as references.
- Because IRSA anchors absolute level to the calibrated 1 m discharge measurement, the wearable needs mainly (a) linear count-rate response across about 0.1–200 µSv/h-equivalent and (b) energy stability. Spectral drift as the source mix changes (scatter build-up, bladder emptying) is a systematic effect to model.

### Gaps
- Exact IEC 61526:2024 tolerances (energy response, dose-rate range, angular response) were not retrievable: the standard is paywalled.
- No independent test of Radiacode, Kromek or GQ energy or angular response at 113/208/364 keV was found.
- No PTB, NPL or other national-lab service description for 177Lu/131I dose-rate meter calibration was retrieved. ISO 4037 reference radiation qualities were not checked.

---

## Q7. Open environmental dose-rate data sets for background subtraction

### Takeaway
**Safecast** is CC0, needs no API key and offers a bulk archive, so it is the easiest open background source. **EURDEP** offers a public map of hourly gamma dose-rate averages from about 5000 European stations (up to 35 days back), but no general public bulk download or API was confirmed.

### Cited Findings
- **Safecast**:
  - Data published under **CC0** — [Safecast data page](https://safecast.org/data/); [AWS Open Data registry](https://registry.opendata.aws/safecast/)
  - API at api.safecast.org with no account or key needed — [Safecast API docs](https://simplemap.safecast.org/docs/); [Openness page](https://safecast.org/?p=10645)
  - Full archive at api.safecast.org/system/measurements.tar.gz: about 4.5 GB compressed and 16.1 GB CSV as of 2020 — [EGU2020 presentation](https://presentations.copernicus.org/EGU2020/EGU2020-4944_presentation.pdf)
  - S3 bucket accessible without an AWS account — [AWS registry](https://registry.opendata.aws/safecast/)
- **EURDEP (JRC REMON)**:
  - Public map of hourly gamma dose-rate averages up to 35 days back from about 5000 stations; some countries delay public release — [JRC Rad Data Exchange](https://remon.jrc.ec.europa.eu/About/Rad-Data-Exchange)
  - Event-specific downloadable datasets exist, e.g., the Ru-106 event — [JRC data catalogue](https://data.jrc.ec.europa.eu/dataset/0f9f3e2d-c8d7-4f46-bbe7-eacf3eed1560)
  - Data description paper — [ESSD 12:109 (2020)](https://essd.copernicus.org/articles/12/109/2020/)

### Inferences
- The more practical background for a body-worn CsI detector is a **device-specific, location-specific baseline**: recorded before administration, or after the signal returns to plateau. Safecast and EURDEP mainly help set priors (typical 0.05–0.2 µSv/h, our inference) and flag anomalous locations such as granite areas or flights. In the first days after therapy, the patient's own signal dominates background by orders of magnitude.

### Gaps
- Current Safecast API rate limits and EURDEP programmatic access terms were not confirmed.

---

## Q8. Research groups with BLE / wearable dosimeter prototypes for nuclear-medicine patients (open designs or code)

### Takeaway
Several groups have built patient-worn or home detectors:
- University of Washington PODD/vest (GAGG+SiPM probes with BLE).
- WIDMApp (Rome; multi-channel garment).
- O'Callaghan et al. (Smart Geiger Pro for 131I).
- An Italian CNR wireless wearable for thyroid patients.
- The Bailey/Globalsonics ceiling GM (commercial).

**None publishes open hardware or code.** The open building blocks remain Radiacode's community software and the Open Gamma Detector.

### Cited Findings
- **UW Portable Organ Dosimetry Device (Miyaoka, Hauck; with W. Hunter)**:
  - 16 probes, each a GAGG scintillator with SiPM and tungsten pinhole collimator.
  - Time-over-threshold analog board; MIKROE Clicker 2 controller; **BLE to a mobile app**.
  - Goal: replace daily hospital visits for 177Lu-DOTATATE dosimetry. No open code or hardware stated.
  - [UW project page](https://people.ece.uw.edu/hauck/Radiology/)
  - 2021 SNMMI report on the home vest: half-life estimates within 4% of truth, maximum error 3.8% at 21 days — [JNM 2021 Suppl](https://jnm.snmjournals.org/content/61/supplement_1/139) [search-summary, verify]
  - Press coverage — [AuntMinnie](https://www.auntminnie.com/clinical-news/molecular-imaging/nuclear-medicine/article/15628641/patient-vest-monitors-dose-after-radionuclide-therapy)
- **WIDMApp (Sapienza/INFN Rome; Mancini-Terracciano, Mirabelli et al.)**:
  - Multi-channel wearable detector garment plus processing to collect patient time-activity data.
  - Monte Carlo design paper in *Medical Physics* 2021 — [PMC9298698](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9298698/)
  - Phantom test with 18F/99mTc/64Cu spheres: time-integrated activity uncertainty under 15% — [CERN Indico 2023](https://indico.cern.ch/event/1247911/contributions/5394524) [search-summary, verify]
  - Also presented at [ESTRO 2022](https://estro.org/Congresses/ESTRO-2022/664/dosimetry/11184/widmapp-aninnovativeapproachforindividualdosemonit). No open design found.
- **O'Callaghan, Cody, Cooke, *Sensors* 2022;22(23):9392 (CC BY 4.0)**: Technonia Smart Geiger Pro for at-home pre-therapy 131I dosimetry; 4 patients at St James's Hospital, Dublin — [MDPI](https://www.mdpi.com/1424-8220/22/23/9392)
- **Italian feasibility study (CNR)**: wireless wearable device for residual-activity assessment in thyroid cancer patients, compared with a home device — [IRIS CNR PDF](https://iris.cnr.it/retrieve/1b3917d8-a105-4f47-9e1f-a24539c0627a/Assessment%20of%20residual%20radioactivity%20by%20a%20comprehensive%20wireless%2c%20wearable%20device%20in%20thyroid%20cancer%20patients%20undergoing%20radionuclide%20therapy%20and%20comparison%20with%20the%20results%20of%20a%20home%20device-%20a%20feasibility%20study..pdf) [search-summary, verify]
- **Patents (freedom-to-operate check needed)**:
  - US 12377289, "Devices, systems, and methods for personalized dosimetry" — [USPTO](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/12377289)
  - US 12011248, "Multi-detector personalized home dosimetry garment" — [USPTO](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/12011248)
- **Bailey 2026 ceiling monitor**: commercial Globalsonics "AustralRad PMS"; PoE; Python browser app on a LattePanda; CSV output. Not open-sourced — [PMC12854189](https://pmc.ncbi.nlm.nih.gov/articles/PMC12854189/)

### Inferences
- IRSA's novelty relative to these groups: they target **organ-level dosimetry** (multi-probe, collimated) or **inpatient room monitoring**. IRSA targets a **single low-cost body-worn sensor** for **whole-body Teff plus third-party exposure guidance**. The UW 2021 result (half-life within 4%) suggests body-worn measurement of washout is feasible.
- The two patents above cover garment and multi-detector home dosimetry. A freedom-to-operate review is advisable before any IAEA technical-cooperation deployment.

### Gaps
- No open-source firmware, app code or hardware files were found from any patient-wearable dosimetry group.
- Clinical validation of any wearable for 177Lu at home beyond conference abstracts was not found.
