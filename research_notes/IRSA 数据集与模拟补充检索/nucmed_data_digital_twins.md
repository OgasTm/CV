# IRSA supplementary sweep: nuclear-medicine imaging/dosimetry data, virtual trials and theranostic digital twins, PBPK/kinetics simulators (checked 2026-11-23)

Scope note (researcher). This sweep adds to two earlier notes: `IRSA 前期开发与公开数据集/physics_imaging_datasets.md` and `kinetics_data_and_hardware.md`. Items already covered there are not repeated. They include the Deep Blue SNMMI 177Lu-DOTATATE set, PSMA-PET-CT-Lesions, DEEP-PSMA, TotalSegmentator, the MRTDosimetry OSF project, the Claeys Belgian phantoms, Ogino/Daisaki 2026, radot p142, Iori, Arasi, EURADOS 2024, Schütze, Kurth, Monserrat Fuertes, Bailey, Turku, Levart, ICRP 137 and the Fele-Paranj SBML.

Verification method:
- Dataset metadata (title, licence, size, file list, date) was read from the Zenodo REST API and the DataCite REST API.
- The contents of three zip archives were listed without downloading them, by reading each zip's central directory with HTTP range requests.
- Software licences were read from LICENSE or DESCRIPTION files on raw.githubusercontent.com, from GitHub search-API metadata, and from the PyPI and CRAN (crandb) JSON APIs.
- TCIA content was checked with the NBIA REST API (`getSeries?Modality=PT`).
- Paper facts come from PMC full text where it was fetched. Anything taken only from a search-engine summary is tagged **[search-level, verify]**.

Module codes (same as the earlier notes, plus kinetics codes):
- M1 = detector-reading inversion; M1b = activity-distribution prior from SPECT or PET.
- M2 = external dose engine. M3 = Monte Carlo (MC) training data and surrogate. M4 = uncertainty quantification. M5 = validation.
- **K1** = priors on effective half-life (T_eff) and compartment parameters. **K2** = synthetic retained-activity trajectories, including voiding steps. **K3** = validation of the kinetics estimator.

---

## Q1. Additional open datasets: 177Lu SPECT/CT, 131I, 225Ac imaging, PSMA/SSTR PET, quantitative SPECT projections, multi-centre phantom and activity-meter intercomparisons

### Takeaway
New this round:
- **One open 177Lu-PSMA-617 patient SPECT/CT acquisition**: one patient, one time point, raw projections from 2 bed positions plus CT, CC BY 4.0. It ships inside the PyTomography/SlicerSPECTRecon tutorial deposits.
- **A Zenodo mirror of the MRTDosimetry four-organ 177Lu phantom series**: 1, 4, 24, 40, 72 and 144 h, with planar, raw SPECT, reconstructed SPECT and CT, CC BY 4.0.
- **Repeated-acquisition 177Lu and 225Ac phantom SPECT sets**, CC BY 4.0.
- **A CC BY multi-isotope SIMIND set** that includes the 225Ac daughters.
- **Lund XCAT-based simulated 177Lu-DOTATATE SPECT / 68Ga-SSTR PET images**, with no licence stated.

Gaps that remain after checking DataCite, Zenodo, TCIA (NBIA API), Figshare and Hugging Face:
- No open **131I** SPECT/CT or planar patient series.
- No **225Ac patient** imaging.
- No sizeable **SSTR PET** cohort.
- No **multi-time-point 177Lu-PSMA** patient data.
- No downloadable data from **IAEA CRP E23005**.

### Cited Findings

#### 1a. New 177Lu SPECT/CT and SPECT-simulation data

| Name | Contents | Size | Licence / commercial use | Access | Link | IRSA module | Caveats |
|---|---|---|---|---|---|---|---|
| **SlicerSPECTRecon Tutorial Data** (L. Polson, 2024) | (i) DICOM of **a patient receiving 177Lu-PSMA-617**: SPECT projection data from 2 bed positions (`FOV1.dcm`, `FOV2.dcm`) plus CT (304 DICOM files). (ii) SIMIND simulation of a Jaszczak phantom filled with 177Lu (3 energy windows plus attenuation map) | 95.9 MB zip, 319 entries | **CC BY 4.0** (commercial use allowed with attribution) | Open, Zenodo | [doi:10.5281/zenodo.14172228](https://doi.org/10.5281/zenodo.14172228); zip listing read by range request | M1b (reconstruct a real 177Lu-PSMA distribution), M2-MC source map, M3 | **Single time point, single patient**, so no kinetics. Acquisition time p.i. not stated in the metadata |
| **PyTomography SPECT Tutorial data** v2.1 (Polson; Zubal part by H. Wei, KCL) | `SPECT.zip` (1,266 entries) contains these folders:<br>• `Lu177-PSMA-GEDisc`: patient; bed1/bed2 projections, 304-slice CT, segmentations, PyTomography reconstruction<br>• `Lu177-NEMA-SymT2`: projections, CT, scanner reconstruction<br>• `Ac225-NEMA-SymT2`: projections, CT, PSF operator<br>• `SIMIND-Jaszak`: lu177, ac225, **fr221, bi213, tl209**, pb212, bi212, tl208 and y90, plus attenuation maps<br>• Tc-99m cardiac; Tc-99m NEMA (StarGuide)<br>Outside the zip, v2/v2.1 add 3D Slicer ROIs and a **simulated 177Lu-DOTATATE SPECT of the Zubal phantom with a simulated 68Ga-DOTATATE PET prior** | 1,455 MB, 7 files | **CC BY 4.0** | Open, Zenodo (v2.1 dated 2026-11-07; earlier version May 2025 [10.5281/zenodo.15314460](https://zenodo.org/records/15314460)) | [doi:10.5281/zenodo.24040979](https://doi.org/10.5281/zenodo.24040979); zip listing read by range request | M1b, M3, 225Ac daughter-window modelling | The `Lu177-PSMA-GEDisc` patient appears to be the same patient as in SlicerSPECTRecon: both have a 304-file CT and two bed positions (my inference). No I-131 folder |
| **53 Repeated Acquisitions of a Lu-177 NEMA Phantom** (Polson, 2024) | 53 repeated SPECT acquisitions. Six spheres (37–10 mm) at 10:1, initial 0.89 MBq/mL. 128×128 at 4.8 mm, 96 projections × 15 s, plus CT | 128.8 MB | **CC BY 4.0** | Open, Zenodo | [doi:10.5281/zenodo.14226289](https://doi.org/10.5281/zenodo.14226289) | M4 (noise and repeatability), M1b | Time span between repeats not stated |
| **MRTDosimetry 4-organ phantom imaging dataset** (Robinson, Denis-Bacelar, Tran-Gia, Tipping, Bardiès, Kayal) | GE DICOM of the 177Lu-filled four-organ phantom at **1, 4, 24, 40, 72 and 144 h**. Folders: `CT/<t>` (124–161 slices each), `Planar/<t>`, `Raw SPECT/<t>` and `Reconstructed SPECT/RR ON` | 172.3 MB zip, 875 entries | **CC BY 4.0** | Open, Zenodo | [doi:10.5281/zenodo.18891044](https://doi.org/10.5281/zenodo.18891044); `isPartOf` [OSF 69NGE](https://doi.org/10.17605/OSF.IO/69NGE); described in [doi:10.1016/j.ejmp.2023.102583](https://doi.org/10.1016/j.ejmp.2023.102583) | **K3** (known-truth multi-time-point series; planar images act as a whole-body-retention proxy), M1, M5 | Uploader's note: "Obtained from https://osf.io/9qr8x … Extract using 7zip, not unzip". The earlier notes covered only the OSF project |
| Preclinical SPECT images for 177Lu protocols (Tosoni, 2026) | Images from an EJNMMI Physics paper on preclinical SPECT optimisation for 177Lu | 323.7 MB | CC BY 4.0 | Open, Zenodo | [doi:10.5281/zenodo.19336881](https://doi.org/10.5281/zenodo.19336881) | Marginal | Preclinical only |
| **DPM-RPT vs GATE MC dosimetry data** (Gupta & Dewaraja, Deep Blue 2026) | Images for 4 of 8 patients in a voxel-dosimetry comparison of DPM-RPT and GATE across "four therapies". SNMMI 2026 abstract; full paper in preparation | not stated | **No licence in DataCite metadata.** Other Dewaraja Deep Blue sets are CC BY-NC 4.0 | Deep Blue | [doi:10.7302/me7k-ph50](https://doi.org/10.7302/me7k-ph50) | M3/M5 (MC cross-check) | The four therapies are not named in the metadata |
| **Lund XCAT-based simulated 177Lu-DOTATATE SPECT and 68Ga-SSTR PET images** (Gustafsson et al.) | MC-simulated images from improved XCAT patient models | not stated | **No licence**: provided "as is without any guarantee"; users are asked to cite the paper | Open link (lu.box.com) | [Lund resources page](https://www.msf.lu.se/en/research/resources); paper EJNMMI Phys 13 (2026), [doi:10.1186/s40658-026-00899-9](https://doi.org/10.1186/s40658-026-00899-9) | M1b, M3 (realistic simulated SSTR cohort) | Files not inspected. With no licence, ask the authors before redistributing |
| lu177-resolution-dosimetry (analysis code for Ogino/Daisaki 2026) | Reproduces all tables and figures from the MRTDosimetry data plus derived data | 0.2 MB | MIT | Zenodo | [doi:10.5281/zenodo.24355541](https://doi.org/10.5281/zenodo.24355541) | M1/M3 | The record says the code was written with LLM assistance and verified by the authors |
| simind-python-connector v2.0.2 (Porter, Varzakis, Thielemans) | Python driver for SIMIND with STIR/SIRF and PyTomography adaptors | 2.1 MB | **Apache-2.0** (LICENSE file; GitHub API says NOASSERTION) | Zenodo / GitHub | [doi:10.5281/zenodo.24110015](https://doi.org/10.5281/zenodo.24110015) | M3 (scripted virtual SPECT) | Needs SIMIND installed separately |
| SIMIND (Lund, M. Ljungberg) | MC SPECT simulator | — | **No OSI licence found.** The download page disclaimer says it is "intended exclusively for research, educational, and training purposes" and "not designed, validated, or approved for clinical diagnosis… treatment planning" (last updated 10 Nov 2026) | Download from Lund | [SIMIND downloads](https://www.msf.lu.se/en/research/simind-monte-carlo-program/downloads) | M3 | Commercial-use terms are unclear: ask Lund before using it in a product pipeline |
| YazdanSalimi/simind-pipeline | "Whole SPECT simulation pipeline" | — | MIT (GitHub API) | GitHub | [repo](https://github.com/YazdanSalimi/simind-pipeline) | M3 | Contents not inspected |

#### 1b. 131I

| Check | Result | Link |
|---|---|---|
| DataCite title search: (131I OR I-131 OR radioiodine) AND (SPECT, planar, whole-body, retention, dosimetry or excretion) | 12 hits. Rat 131I biodistribution supplements (2026); the Vivli tositumomab package (already known); figshare supplementary files of an FDG-PET vs 131I whole-body-scan paper (2022). **No patient image set** | [DataCite API query](https://api.datacite.org/dois?query=titles.title:(131I%20OR%20I-131%20OR%20radioiodine)) |
| Zenodo full-text search ("I-131" SPECT phantom; radioiodine whole-body retention) | No 131I SPECT/planar dataset. Only generic SPECT tutorials and an atmospheric I-131 dispersion set | [Zenodo search](https://zenodo.org/search?q=%22I-131%22%20SPECT) |
| PyTomography tutorial zip listing | No I-131 folder (see 1a) | [doi:10.5281/zenodo.24040979](https://doi.org/10.5281/zenodo.24040979) |
| Deep Blue (umich) via DataCite | 19 records; Lu-177 and Y-90 only, no I-131 | [DataCite](https://api.datacite.org/dois?query=client.id:umich.deepblue) |

#### 1c. 225Ac imaging (phantom only)

| Name | Contents | Size | Licence | Link | IRSA module | Caveats |
|---|---|---|---|---|---|---|
| **Ac-225 Scans: 34 Repeated Phantom Acquisitions** (Polson, 2024) | Cylindrical phantom, three spheres (60/28/22 mm) at 10:1, initial 1.37 kBq/mL. 128×128 at 4.8 mm, 96 projections × 60 s, plus CT | 88.6 MB | **CC BY 4.0** | [doi:10.5281/zenodo.14166469](https://doi.org/10.5281/zenodo.14166469) | 225Ac add-on (low-count SPECT and uncertainty) | Phantom only |
| PyTomography `Ac225-NEMA-SymT2` plus SIMIND daughter projections | See 1a | in the 1.4 GB zip | CC BY 4.0 | [doi:10.5281/zenodo.24040979](https://doi.org/10.5281/zenodo.24040979) | 225Ac daughter-window response | Phantom/simulated |
| Data underlying "Free 221Fr inflates 225Ac activity estimates in quantitative SPECT…" (Ennassiri & Touti, 2026) | Window-response matrices and raw MC counts (OpenGATE 10.0.3, Siemens Intevo model; ME and HE collimators) | 0 files visible | Declared CC BY 4.0, but **access "restricted"** | [doi:10.5281/zenodo.22055021](https://doi.org/10.5281/zenodo.22055021) | 225Ac daughter-equilibrium correction | Not downloadable without a request |

#### 1d. PSMA / SSTR PET beyond the earlier notes

| Check / item | Result | Licence | Link | Caveats |
|---|---|---|---|---|
| **TCIA PT inventory** via NBIA API (`getSeries?Modality=PT`, 2026-11-23) | 6,140 PT series across 44 collections. The only PSMA collection is **PSMA-PET-CT-Lesions (597)**. There is **no SSTR / DOTATATE collection**; NaF PROSTATE (139) and FDG collections dominate | per collection | [NBIA API](https://services.cancerimagingarchive.net/nbia-api/services/v1/getSeries?Modality=PT) | Confirms the earlier gap |
| MSKCC nnU-Net models for neuroendocrine tumour segmentation in DOTATATE PET/CT | Pretrained models trained on 915 DOTATATE PET/CT volumes. **Images not public** ("data will not be made public at this time") | Models listed as CC BY 4.0 (Zenodo 10.5281/zenodo.7843469) | [MSK data catalog 11202](https://datacatalog.mskcc.org/dataset/11202); paper [doi:10.1007/s11548-023-02968-1](https://doi.org/10.1007/s11548-023-02968-1) | **[search-level, verify]**. Usable as a segmentation tool, not as data |
| 64Cu-DOTATATE total-tumour segmentation (Carlsen et al., 2022) and 68Ga-DOTATATE PPGL segmentation (2024) | Figshare search returned only "Additional file" journal contributions, not image sets | CC BY (supplements) | [figshare 19923244](https://springernature.figshare.com/articles/journal_contribution/Additional_file_1_of_A_convolutional_neural_network_for_total_tumor_segmentation_in_64Cu_Cu-DOTATATE_PET_CT_of_patients_with_neuroendocrine_neoplasms/19923244); [figshare 27618100](https://doi.org/10.6084/m9.figshare.27618100.v1) | No downloadable SSTR images found |
| Hugging Face dataset search ("SPECT", "PSMA", "lutetium", "nuclear medicine", "PET/CT", "dosimetry") | No RPT-specific data. Generic PET/CT sets only, e.g. `dacthai2807/pet-ct` (CC BY-NC-SA 4.0) and `shorna/PET_CT_SEG` (no licence tag) | — | [HF API](https://huggingface.co/api/datasets?search=PET/CT) | Not useful for IRSA |
| PSMA-PET-CT-Lesions NIfTI v3 on FDAT | Cited in the samkiwi18 README as doi:10.57754/FDAT.g27kx-86t35 | not checked | [FDAT DOI](https://doi.org/10.57754/FDAT.g27kx-86t35) | **[not verified]**. The earlier notes flagged a CC BY-NC licence on the FDAT copy |

#### 1e. Multi-centre phantom and activity-meter intercomparisons; IAEA, EURADOS, EANM

| Item | What exists | Data deposited? | Link | Caveats |
|---|---|---|---|---|
| IAEA CRP **E23005** "Dosimetry in Molecular Radiotherapy for Personalized Patient Treatments" | Approved 13 Feb 2017, started 10 Jul 2017, concluded Aug 2024. It tested and standardised dosimetric methods; one published case was a 177Lu-DOTATATE patient from Tygerberg Hospital (South Africa). Follow-on CRP **E24027** "Audit system for RPT" began 25 Apr 2025 | **None found** | [IAEA CRP E23005](https://iaea.org/projects/crp/e23005); [E24027 news](https://www.iaea.org/newscenter/news/new-crp-audit-system-for-radiopharmaceutical-therapy) | **[search-level, verify]**: the IAEA page returned HTTP 402 to fetch |
| Italian national comparison ENEA-INMRI ILC-2 | Blind comparison with a circulating instrument; only 8 participants reported 177Lu | No | [Spagnuolo 2024 PDF](https://iris.uniroma1.it/retrieve/9dd0f962-30a2-4e05-be1f-2d9df56b79bd/Spagnuolo_National%20programme_2024.pdf) | [search-level, verify] |
| 32 radionuclide calibrators in 8 hospitals (NL/BE/DE) | Large variability by system, container and geometry | No | [EJNMMI Phys s40658-018-0213-z](https://link.springer.com/10.1186/s40658-018-0213-z) | [search-level, verify] |
| MRTDosimetry multicentre 177Lu SPECT/CT accuracy | Multicentre, multinational comparison | Phantom images via OSF/Zenodo (1a) | [PMC8302709](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8302709/) | — |
| European 7-hospital 177Lu quantitative-imaging intercomparison (2018) | Variability between protocols | No | [PMC6070453](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6070453/) | — |
| Claeys multicentre RNC + SPECT/CT preprint | RNC up to 11% between centres; SPECT up to 20% (cylinder) and 58% (NEMA) | Yes (already in the earlier notes) | [arXiv 2601.20549](https://arxiv.org/abs/2601.20549) | Preprint |
| EURADOS WG13 177Lu-PSMA-617 case study (Covens, EFOMP SMRD 2025 slides) | Ambient dose-equivalent rate measured at 1 m compared with simulations. The slides state that close-contact posture strongly affects results | No (slides only) | [Covens slides](https://smrd2025.efomp.org/wp-content/uploads/2025/12/Covens.pdf) | [search-level, verify]; no peer-reviewed 177Lu output found |

### Inferences
- The best fully open (CC BY) 177Lu building blocks for M1/M3 are:
  - the single 177Lu-PSMA-617 patient acquisition, giving a real PSMA uptake pattern and real projections;
  - the MRTDosimetry six-time-point phantom series, giving known-truth kinetics;
  - the 177Lu and 225Ac repeated phantom scans, giving noise models.

  Unlike the Deep Blue patient data (CC BY-NC), all of these allow commercial use with attribution.
- The MRTDosimetry planar images at 1–144 h are the cleanest open test bed for the "single-detector reading → retained activity" inversion (K3/M1), because the ground-truth activity is known at every time point.
- For SSTR, the realistic open route is simulation: the Lund XCAT 68Ga-SSTR/177Lu-PRRT images (no licence, so ask first) or the 14 DblurDoseNet PET-derived phantoms (CC BY-NC). No open SSTR patient PET cohort exists.

### Gaps
- No open 131I SPECT/CT or planar whole-body series; no 225Ac patient imaging; no multi-time-point 177Lu-PSMA patient data. All were confirmed absent in DataCite, Zenodo, TCIA (NBIA) and Hugging Face.
- **Not effectively searched**:
  - Kaggle (API needs authentication) and Synapse (not queried).
  - The grand-challenge.org and PhysioNet APIs ignored the `search` parameter and returned generic lists, so neither repository was searched by keyword in this session.
  - Mendeley Data was covered only through DataCite. The one Mendeley hit, "Developments in 177Lu-based RPT and Dosimetry" ([doi:10.17632/g2g3dcv2b5](https://doi.org/10.17632/g2g3dcv2b5)), is bibliometric text data, not imaging.
- No downloadable EANM/EARL 177Lu harmonisation data, NPL or PTB activity-meter comparison data, or IAEA E23005 data were found.
- The Lund XCAT image package and the DPM-RPT Deep Blue set were not opened, so their sizes, file formats and therapies are unknown.

---

## Q2. Virtual clinical trials and theranostic digital twins (2022–2026): released code, synthetic cohorts, licences

### Takeaway
The QURIT/Rahmim–Uribe group now has two public repositories:
- **PyTheraTwin** (MIT, pushed 2026-11-17): an end-to-end "theranostic digital twin" pipeline. CT → TotalSegmentator → PBPK time-activity curves (TACs) → synthetic lesions → SIMIND SPECT plus PyTomography reconstruction → OpenGATE dosimetry.
- **PyCNO**: a Python/SBML port of the Fele-Paranj PSMA PBRPK model. **It has no licence file.**

**No synthetic cohort is released** by any group:
- The 640-virtual-patient PBPK study is preprint-only.
- An independent MIT repository builds a PSMA virtual cohort from public PET/CT but ships only network weights (CC BY-NC).
- Glatting/Kletting-lineage work, such as the 177Lu-DOTATATE PBPK models and in-silico trials, has published equations and parameters but no code or cohort that I could find.

### Cited Findings

#### 2a. Released code

| Name | What it does | Licence / commercial use | Status | Link | IRSA module | Caveats |
|---|---|---|---|---|---|---|
| **qurit/PyTheraTwin** | "Creates patient-specific theranostic digital twins by combining CT-based anatomy/segmentation with PBPK kinetics and physics-based SPECT simulation/reconstruction … for Virtual Theranostic Trials." Phase 1: segmentation, synthetic lesions, PBPK TAC generation. Phase 2: SIMIND SPECT and OpenGATE dosimetry. Phase 3: post-processing. It has a browser UI and a CLI | **MIT** (LICENSE "Copyright (c) 2026 Qurit") | Pushed 2026-11-17; 5 stars | [GitHub](https://github.com/qurit/PyTheraTwin) | **K2** (organ TACs per virtual patient), M1b/M3 (paired activity maps and SPECT), M2-MC | SIMIND is an external dependency with unclear commercial terms (Q1). It uses PyCNO, which has no licence. Code search finds "bladder" only in ROI-naming/option JSON files, so a bladder-voiding model is not confirmed |
| **qurit/PyCNO** ("Python Computational Nuclear Oncology") | Python implementation of the Fele-Paranj 2024 PBRPK PSMA model, exported from SimBiology to SBML: an original model (lesions plus organs), a reduced key-organ model, and a PK/PD-coupled SBML. The README says it will add other radioligands and PD later | **No licence** (GitHub reports none; LICENSE and LICENSE.md return 404) | Pushed 2026-10-14; README warns of breaking changes | [GitHub](https://github.com/qurit/PyCNO); model paper [EJNMMI Radiopharm Chem 2024;9:6](https://link.springer.com/article/10.1186/s41181-023-00236-w) | K1/K2 (PSMA whole-body PBPK) | Without a licence, default copyright applies. GitHub code search found no "bladder" or "urine" terms (the index may skip large SBML files) |
| samkiwi18/lu177-psma-predictive-dosimetry | Virtual cohort built from public PSMA PET/CT (PSMA-PET-CT-Lesions) with the **Kletting 2016 PBPK**, GATE 10 MC doses, a SwinUNETR that predicts voxel-wise kinetic parameters with uncertainty, and six Bayesian single-SPECT update methods. Tested on the 2 Deep Blue 177Lu-DOTATATE patients. Main finding: one SPECT at 1–7 d cuts TIA median absolute log error from 0.36 to 0.14–0.18 | **Code MIT; weights, data and derived material CC BY-NC 4.0** (per `DATA_LICENSES.md`) | Pushed 2026-11-22; 3 commits; no paper linked | [GitHub](https://github.com/samkiwi18/lu177-psma-predictive-dosimetry) | K1 (Bayesian population-prior update logic transfers directly to IRSA's wearable update), M4 | "No data is included". `data/` and `output/` are empty; only `model.pth` is released. Individual, unreviewed project |
| Yenipertiwi16/VP-NLME-STP-Dosimetry | R scripts and NONMEM control streams for virtual-patient nonlinear mixed-effects modelling of single-time-point dosimetry in 177Lu-PSMA-617 | **No licence** | Pushed 2026-08-03 | [GitHub](https://github.com/Yenipertiwi16/VP-NLME-STP-Dosimetry) | K1 (NLME population priors) | README not retrievable; NONMEM is commercial |
| UNHSAILLab/rl_for_theranostics | PBPK, neural ODE/PINN and RL notebooks, including a "Lu-DOTATATE Therapy – Insilico Analysis" notebook, a PRRT planning tutorial and a copy of "UBC-PSMA-PBPK-Model-Matlab" | Apache-2.0 (repo) | Last push 2024-03-31 | [GitHub](https://github.com/UNHSAILLab/rl_for_theranostics) | K2 (toy PBPK) | **Licence ambiguity**: it bundles a copy of a UBC model whose original repo has no licence |
| Borealis-BioModeling/radioligand-therapy-model | Semi-mechanistic RLT PK/PD model in PySB (pysb-pkpd, pysb-units) | MIT | Pushed 2024-10-17 | [GitHub](https://github.com/Borealis-BioModeling/radioligand-therapy-model) | K2 | Generic, not fitted to a specific ligand (from the README) |
| **bertoletlab/topas-rpt** | OpenTOPAS 4.3.0 extension: samples decays across full branching chains with the correct time structure, and threads compartmental binding kinetics through the source term. Accepts DICOM activity maps | **MIT**; Zenodo [10.5281/zenodo.22651399](https://doi.org/10.5281/zenodo.22651399) | Pushed 2026-11-13; pinned to Geant4 11.3.2 | [GitHub](https://github.com/bertoletlab/topas-rpt) | M2-MC/M3 for **225Ac chains** with kinetics | Needs a patch to OpenTOPAS; method paper on 211At (IJROBP 2025) |
| qurit/LesionStudio | Synthetic lesion insertion in PET/SPECT | MIT | 2026-07-24 | [GitHub](https://github.com/qurit/LesionStudio) | M3 | — |
| qurit/xcat-sims | XCAT lymphatic-system extension and PET simulation/reconstruction; simulated lymphoma patients "freely accessed" | **No licence**; upgraded XCAT on request from Duke | 2026-09-27 | [GitHub](https://github.com/qurit/xcat-sims) | M3 | PET/lymphoma, not RPT |
| callmesobhan/rlt-sparse-dosimetry; callmesobhan/RadioNuclide_Uncertainty_Lab | MATLAB TIA and red-marrow dosimetry from sparse measurements; MC propagation and Sobol sensitivity for MIRD dosimetry | MIT; Apache-2.0 | 2026-11-17 | [repo 1](https://github.com/callmesobhan/rlt-sparse-dosimetry); [repo 2](https://github.com/callmesobhan/RadioNuclide_Uncertainty_Lab) | K1/M4 | Self-described "research code, seeking review" |
| SJTUzhou/Voxel-wise-Adjustment-of-Single-Time-Point-Dosimetry-for-Lu177-PSMA-RPT | STP dosimetry adjustment | MIT | 2026-02-10 | [GitHub](https://github.com/SJTUzhou/Voxel-wise-Adjustment-of-Single-Time-Point-Dosimetry-for-Lu177-PSMA-RPT) | K1 | Not inspected |
| lucasfffaria/Monte-Carlo-Dosimetry-of-Therapeutic-Radioligands | Description: "Monte Carlo assessment of ambient dose equivalent and effective d[ose] around patients treated with 177Lu, 161Tb, 212Pb and 225[Ac]" | No licence | **Repository empty** on 2026-11-23 | [GitHub](https://github.com/lucasfffaria/Monte-Carlo-Dosimetry-of-Therapeutic-Radioligands) | Watch item for M2/M5 (external H*(10) around patients, incl. 225Ac) | Nothing to use yet |

#### 2b. Literature (no code or cohort release confirmed)

| Paper | Content relevant to IRSA | Link | Code/cohort |
|---|---|---|---|
| Abdollahi et al., *Theranostics* 2024;14(9):3404 | Defines theranostic digital twins (TDTs) and a roadmap | [PMC11209714](https://pmc.ncbi.nlm.nih.gov/articles/PMC11209714) | — |
| Zaidi, Rahmim, Saboury et al., JNM 2025 (computational nuclear oncology) | TDT attributes: verification, validation, interoperability | [PMC12051771](https://pmc.ncbi.nlm.nih.gov/articles/PMC12051771/) | — |
| Strigari et al., JNM 2025;66(5):748 | Ethics and regulation for TDTs; data-sharing limits, federated learning | [JNM](https://jnm.snmjournals.org/content/66/5/748) | — |
| Ryhiner, Song, Saboury, Glatting, Rahmim, arXiv 2511.03755 (2025-11-04) | TDT framework and roadmap | [arXiv](https://arxiv.org/abs/2511.03755) | Not stated in the abstract |
| Zaid, Hardiansyah, Yusufaly, Rahmim, arXiv 2509.21697 (2025-09-25) | Verification, validation and uncertainty quantification (VVUQ) of PBPK models for TDTs | [arXiv](https://arxiv.org/abs/2509.21697) | Not stated in the abstract |
| **Abdollahi, Fowler, Uribe, Rahmim, arXiv 2510.21054 (v4)** | **640 virtual patients**, 15,360 tumour and organ TACs from PBPK simulation of F-18, Ga-68 and Cu-64 PSMA PET tracers paired with 177Lu-PSMA. ML predicts dose; Cu-64 PSMA-617 PET gives tumour-dose MAPE as low as 8% | [arXiv](https://arxiv.org/abs/2510.21054) | **Not confirmed**: the HTML full text returned 404 and the abstract has no code link |
| Uribe, Abdollahi, Yusufaly, McColl, Sabouri et al., arXiv 2602.10570 (2026-02-11) | "Virtual Theranostic Trials" as new approach methodologies (NAMs) and companion trials | [arXiv](https://arxiv.org/abs/2602.10570) | Perspective |
| Hardiansyah, Patrianesha, Shi K, Saboury, Rahmim, Glatting et al., arXiv 2509.09755 | Review of PBPK and PopPK in RPT | [arXiv](https://arxiv.org/abs/2509.09755) | Review. Kuangyu Shi is a co-author |
| Begum et al. 2019 (Kletting/Glatting) | 13 patient-individualised virtual mCRPC patients, PSMA PBPK in SimBiology | [PMC6934468](https://pmc.ncbi.nlm.nih.gov/articles/PMC6934468) | No code link (earlier notes) |
| Jiménez-Franco, Glatting et al., JNM 2021;62(1):92 | In-silico PBPK: minimal tumour perfusion and SSTR2 density for 177Lu-DOTATATE (TCP 99%); GFR and kidney SSTR2 effects | [JNM](https://jnm.snmjournals.org/content/62/1/92) | [search-level, verify] |
| Stenvall, Beer, Gleisner, Glatting, *Phys Med* 2024 | Whole-body **177Lu-DOTATATE PBPK** in 12 NET patients; SAAM II and SimBiology implementations give comparable TIAs | [LUP record](https://lup.lub.lu.se/record/059f169f-fa5b-4d3f-81b5-e6d4046bcf0a) | [search-level, verify]; code not found |
| Costa, Glatting et al. (Dec 2025) | Mouse-specific PBPK "digital twin" for [177Lu]Lu-rhPSMA-10.1 | [PMC12730921](https://pmc.ncbi.nlm.nih.gov/articles/PMC12730921/) | [search-level, verify]; preclinical |
| DosiTest (GATE-generated clinical 177Lu SPECT/CT) | Simulated clinical 177Lu SPECT/CT images planned for circulation to clinical departments | [PubMed 33957577](https://pubmed.ncbi.nlm.nih.gov/33957577/) | [search-level]; no public download found |

### Inferences
- **PyTheraTwin is the closest open match to IRSA's K2 need** (synthetic patients with organ TACs and paired images). Its MIT licence allows reuse, but two dependencies block a clean commercial chain: the PBPK engine (PyCNO, no licence) and SIMIND (research-only disclaimer). For a TC-project prototype it can be used now. For anything distributed, either swap in a re-implemented PBPK (e.g., from published Kletting/Fele-Paranj equations in tellurium or PySB, Q3) or get written permission from QURIT.
- None of these digital twins models what IRSA needs at the whole-body level: retained activity after discharge, bladder filling and voiding, and an external detector signal. That still has to be added. One option is a bladder compartment with voiding events on top of PyCNO/Kletting TACs, followed by a forward model to a torso-worn detector.
- The samkiwi18 result (one late SPECT plus a Bayesian update beats network priors) is analogous to IRSA's "population prior plus a calibrated 1 m reading plus a wearable update" design. Its code is MIT and could serve as a template for the update step.

### Gaps
- No released synthetic cohort (TACs or parameter sets) for 177Lu-PSMA, 177Lu-DOTATATE, 131I or 225Ac was found.
- Code availability for arXiv 2510.21054 and for the Glatting-group 177Lu-DOTATATE PBPK (Stenvall 2024) could not be confirmed.
- No XCAT-based RPT virtual trial with released cohort data was found beyond the Lund images. XCAT itself remains a paid licence (earlier notes).

---

## Q3. Open PBPK/compartmental software, SBML repositories and published parameters (177Lu-PSMA, 177Lu-DOTATATE, 131I-NaI, 225Ac-PSMA) including urinary excretion

### Takeaway
Licences, from most to least permissive:
- **Permissive** (product-friendly): tellurium (Apache-2.0), PySB (BSD-2), Chi (BSD-3), PKPDsim R (MIT).
- **Copyleft**: Open Systems Pharmacology (PK-Sim/MoBi/OSPSuite-R) uses **GPL-2.0 plus a clarifying addendum**; mrgsolve is GPL (≥2); nlmixr2 and rxode2 are GPL (≥3).

Ready radiopharmaceutical models remain rare. The PSMA PBRPK exists as SBML/Python with no licence. 177Lu-DOTATATE PBPK models are published but without code. No 225Ac PBPK with daughter compartments was found.

The FDA labels give citable population excretion priors:
- **Lutathera**: cumulative urinary excretion 44% within 5 h, 58% within 24 h, 65% within 48 h; terminal half-life 71 ± 28 h.
- **Pluvicto**: terminal half-life 41.6 h (CV 68.8%); CL 2.04 L/h; "primarily eliminated renally", with no percentages given.

### Cited Findings

#### 3a. Software

| Tool | Purpose | Licence | Version / activity | Link | IRSA fit |
|---|---|---|---|---|---|
| **PK-Sim / MoBi** (Open Systems Pharmacology) | Whole-body PBPK (PK-Sim) and multiscale model editing (MoBi) | **GPL-2.0 "plus clarifying addendum"** (LICENSE file, develop branch). GitHub reports NOASSERTION | Pushed 2026-11-19 / 2026-11-17 | [PK-Sim](https://github.com/Open-Systems-Pharmacology/PK-Sim); [MoBi](https://github.com/Open-Systems-Pharmacology/MoBi) | Offline generation of K2 priors. Copyleft matters only if the code is distributed |
| **OSPSuite-R** | R interface to OSP simulations | GPL-2 (DESCRIPTION: "GPL-2 \| file LICENSE"; same addendum) | Pushed 2026-11-21 | [GitHub](https://github.com/Open-Systems-Pharmacology/OSPSuite-R) | Batch virtual-population runs |
| **tellurium** | SBML/Antimony modelling in Python (libRoadRunner engine) | **Apache-2.0** (GitHub) | PyPI 2.2.13.1; pushed 2026-08-29 | [GitHub](https://github.com/sys-bio/tellurium) | Run PyCNO/Fele-Paranj SBML; build a bladder-voiding compartment model (K2) |
| libRoadRunner | High-performance SBML simulator | **Not verified** (PyPI licence field empty; GitHub record not returned) | PyPI 2.11.0 | [PyPI](https://pypi.org/project/libroadrunner/) | — |
| **PySB** | Rule-based systems-biology models in Python | **BSD-2-Clause** | PyPI 1.17.0; pushed 2026-05-19 | [GitHub](https://github.com/pysb/pysb) | Used by the Borealis RLT PK/PD model |
| **Chi** (`chi-drm`) | PKPD modelling and model-informed precision dosing (Bayesian inference) in Python | **BSD-3-Clause** | PyPI 1.0.3; pushed 2026-05-23 | [GitHub](https://github.com/DavAug/chi) | K1 (hierarchical priors, Bayesian per-patient update) |
| **mrgsolve** | ODE PopPK/PD and QSP simulation in R | GPL (≥2) | CRAN 2.0.1 (2026-05-20) | [GitHub](https://github.com/metrumresearchgroup/mrgsolve) | Fast virtual-population simulation |
| **nlmixr2** / **rxode2** | Nonlinear mixed-effects estimation / ODE solving in R | GPL (≥3) / GPL (≥3) | CRAN 7.0.1 (2026-08-05) / 5.1.8 (2026-10-13) | [nlmixr2](https://github.com/nlmixr2/nlmixr2) | Fit population T_eff/compartment priors if patient data are obtained |
| **PKPDsim** (InsightRX) | ODE PK-PD simulation in R | **MIT** ("MIT + file LICENSE") | CRAN 1.4.1 (2025-04-17); GitHub DESCRIPTION 1.5.0 | [GitHub](https://github.com/InsightRX/PKPDsim) | Permissive alternative to mrgsolve |
| BioModels (EBI) | SBML repository | — | — | [BioModels](https://www.ebi.ac.uk/biomodels/) | **Not checked**: the search API returned non-JSON in this session |

#### 3b. Published models and parameters relevant to K1/K2

| Agent | Source | Key content | Licence / access | Link | Caveats |
|---|---|---|---|---|---|
| 177Lu-DOTATATE (Lutathera) | US prescribing information (revised 10/2024) | Clearance 4.5 L/h (CV 31%). Terminal half-life 71 (±28) h. "**Cumulative excretion of 44% within 5 hours, 58% within 24 hours, and 65% within 48 hours**". Over 99% eliminated within 14 days. "Radiation can be detected in the urine for up to 30 days" | Public label | [Novartis Lutathera PI (PDF)](https://www.novartis.com/us-en/sites/novartis_us/files/lutathera.pdf) | Population means only |
| 177Lu-PSMA-617 (Pluvicto) | US prescribing information (revised 07/2026) | Terminal elimination half-life **41.6 h (68.8%)**; CL **2.04 L/h (31.5%)**; "primarily eliminated renally". AUC rises as creatinine clearance falls; CLcr < 54 mL/min not studied | Public label | [Novartis Pluvicto PI (PDF)](https://www.novartis.com/us-en/sites/novartis_us/files/pluvicto.pdf) | **No urinary-percentage figures in the label.** Use Kurth 2018 or de Bakker 2023 for excretion fractions |
| 177Lu-PSMA-617 | **de Bakker et al., EJNMMI 2023;50(12):3572** (Radboud) | Urine collected up to 24 h in 25 patients (28 cycles), plus 35 samples at 7–42 d in 9 patients.<br>• Excretion half-time over the first 20 h: 4.9 h overall; 4.4 h in the fast group (n = 19) vs 8.4 h in the slow group (n = 6).<br>• Predicted excretion by 48 h: 67 ± 18% (fast 75 ± 10%, slow 36 ± 10%).<br>• eGFR 50–65 vs > 65 mL/min: slower excretion (p < 0.01).<br>• 177Lu measurable in urine up to 18 d (abstract).<br>• Skin equivalent dose 50–145 mSv for contamination at 0–8 h | CC BY 4.0. Per-patient curves in figures only; data "available from the corresponding author on reasonable request" | [PMC10547615](https://pmc.ncbi.nlm.nih.gov/articles/PMC10547615/) | Gives a bimodal fast/slow excretion prior for the K2 voiding model |
| 177Lu-PSMA (PBPK) | Abdollahi, Fele-Paranj, Rahmim, *Cancers* 2024;16:3120; identical copy in UBC cIRcle | Effect of PBPK parameters on TAC-derived physical, biological and statistical measures | Open | [doi:10.14288/1.0445509](https://doi.org/10.14288/1.0445509) (= [10.3390/cancers16183120](https://doi.org/10.3390/cancers16183120)) | Parameter tables not extracted here |
| 177Lu-PSMA-I&T (PopPK/PD of PSA) | EJNMMI 2024, Additional file 1 "Individual model fit results" | 6.5 MB supplement | CC BY 4.0 | [doi:10.6084/m9.figshare.26711045](https://doi.org/10.6084/m9.figshare.26711045) | PSA dynamics, not activity kinetics |
| 177Lu-HA-DOTATATE / 68Ga-HA-DOTATATE | Semi-physiological population PK (2024), Additional file 1 | Goodness-of-fit plots; individual kidney and tumour predictions vs observations | CC BY 4.0 | [doi:10.6084/m9.figshare.26612495](https://doi.org/10.6084/m9.figshare.26612495) | Figures, not parameter tables |
| 68Ga-(HA-)DOTATATE | PBPK for whole-body distribution and tumour-sink effects in GEP-NET (2024), Additional file 1 | Table S1 (patient details) and liver/thyroid SUV figures | CC BY 4.0 | [doi:10.6084/m9.figshare.26566100](https://doi.org/10.6084/m9.figshare.26566100) | Diagnostic tracer |
| 177Lu-DOTATATE (PBPK) | Stenvall et al. 2024; Jiménez-Franco et al. 2021 (Q2b) | Whole-body PBPK structure and parameters in the papers | Journal | see Q2b | [search-level, verify]; no code |
| 225Ac-PSMA-I&T (daughter kinetics) | **Liubchenko et al., EJNMMI 2024;51(8):2504** (LMU) | 5 patients, 7.7 ± 0.2 MBq. SPECT/CT at 24 and 48 h with windows at 440 keV (213Bi), 218 keV (221Fr) and 78 keV.<br>• Urine 213Bi/225Ac ratio: 0.98 ± 0.15 (24 h) → 1.08 ± 0.09 (48 h).<br>• Kidney T_eff: 27 ± 6 h (213Bi), 24 ± 6 h (221Fr), 23 ± 7 h (78 keV). Lesions about 38–39 h.<br>• The kidney difference between 213Bi and 221Fr was significant (p = 0.0078) | CC BY 4.0; data "Please contact the corresponding author" | [PMC11178588](https://pmc.ncbi.nlm.nih.gov/articles/PMC11178588/) | No whole-body T_eff reported. Search summary: kidney dose 0.17 ± 0.06 Sv/MBq (RBE 5) [search-level] |
| 225Ac carriers (generic) | Yskaa91/AlphaRadPharmacokinetics | R routine for PK and dosimetry of 225Ac carriers | GPL-3.0 | [GitHub](https://github.com/Yskaa91/AlphaRadPharmacokinetics) | Last push 2017; README not retrievable |
| 131I-NaI | No new open parameter set found beyond ICRP 137 / Leggett 2010 (earlier notes) | — | — | — | Gap |

### Inferences
- **Permissive stack for an IRSA kinetics engine**: tellurium (Apache-2.0) or PySB (BSD-2) for model definition and simulation, plus Chi (BSD-3) for hierarchical Bayesian priors and per-patient updates. All three can be embedded in a distributed app. OSP/mrgsolve/nlmixr2 are GPL and should stay as offline prior-generation tools.
- **A citable 177Lu-DOTATATE excretion prior** now exists from the label: 44/58/65% at 5/24/48 h. Together with Kurth 2018 for PSMA-617 (30/50/70/76% at 2/4/12/24 h) and de Bakker 2023 (bimodal fast/slow, eGFR-dependent), it can parameterise the urinary-compartment and voiding-step generator in K2:
  - mixture weights of about 76% fast and 24% slow (19 of 25 patients, my calculation from de Bakker);
  - excretion half-times of about 4.4 h vs 8.4 h.
- The Pluvicto label's terminal half-life (41.6 h, CV 68.8%) is a regulator-reviewed population value. It matches Kurth's 40.5 ± 9.6 h T_eff, and its large CV supports a wide lognormal K1 prior.

### Gaps
- BioModels was not searched successfully, so the existence of SBML radiopharmaceutical models there is unconfirmed.
- No open PBPK code with a licence for 177Lu-DOTATATE, 131I-NaI or 225Ac-PSMA was found.
- No 225Ac-PSMA PBPK with explicit 221Fr/213Bi compartments was found. Pluvicto excretion percentages are not in the US label (EU SmPC not checked).
- libRoadRunner's licence was not verified.

---

## Q4. 225Ac: open biodistribution, excretion, contamination and dose-rate data; daughter dosimetry models

### Takeaway
Open, downloadable 225Ac data are limited to phantoms and simulations: the Polson phantom scans, the PyTomography Ac-225 and daughter SIMIND projections (both CC BY 4.0), and the radot p142 225Ac-chain dose rates (earlier notes).

Patient-level 225Ac kinetics exist only as published aggregates:
- Liubchenko 2024 (225Ac-PSMA-I&T: urine 213Bi/225Ac ratios, kidney T_eff per daughter).
- [225Ac]Ac-SibuDAB JNM 2025 (213Bi excess in urine).
- A 2026 JNM phase 1 of 225Ac-PSMA-I&T.

A named "CLEAN-225Ac" study was **not found**. A 2026 inpatient contamination wipe-test study (225Ac-PSMA-I&T/177Lu tandem and 225Ac-DOTATOC) matches the description, but only at search level. External dose rates for 225Ac-PSMA come from a Japanese MHLW model, not from measurements.

### Cited Findings

| Item | Data | Licence / access | Link | IRSA relevance | Caveats |
|---|---|---|---|---|---|
| Liubchenko et al. 2024, [225Ac]Ac-PSMA-I&T (n = 5) | See Q3b: urine 213Bi/225Ac ratios over time, daughter-specific kidney and lesion T_eff. Tables 1–5 in the main text, plus a DOCX supplement (261 KB) | CC BY 4.0; data on request | [PMC11178588](https://pmc.ncbi.nlm.nih.gov/articles/PMC11178588/) | K1 for 225Ac (kidney/lesion T_eff about 1–1.6 d); contamination modelling (213Bi in urine) | 2 time points only |
| [225Ac]Ac-SibuDAB, JNM 2025;66(8):1239 (3 patients) | Recoil 213Bi rapidly distributed into urine. A 20-min urine sample showed a **281% net 213Bi excess** over equilibrium; no 213Bi detected in blood | Journal | [PubMed 40473466](https://pubmed.ncbi.nlm.nih.gov/40473466/); [JNM figures](https://jnm.snmjournals.org/content/66/8/1239/tab-figures-data) | Urine-contamination source term for the IRSA hygiene module | [search-level, verify] |
| Phase 1 [225Ac]Ac-PSMA-I&T in mCRPC, JNM 2026;67(7):1109 | Recommended phase 2 dose 10 MBq. Dosimetry from whole-body planar images plus blood and urine samples | Journal | [JNM 67(7):1109](https://jnm.snmjournals.org/content/67/7/1109) | Future K1 source if the supplement has urine tables | [search-level, verify]; supplement not inspected |
| 2026 inpatient contamination study (225Ac-PSMA-I&T / 177Lu-PSMA-I&T tandem; 225Ac-DOTATOC) | Wipe samples in patient rooms. Contamination in 4 of 6 therapies; **12 of 128 post-therapy samples (9.4%)**, mainly floors and toilet seats, also faucets. Higher risk with urinary incontinence | Not determined | Search result pointing to [JNM node 128470](https://jnm.snmjournals.org/node/128470) | Home-contamination guidance (225Ac add-on) | **[search-level, verify]**: journal, date and the "CLEAN-225Ac" name not confirmed; no supplementary data seen |
| Japanese MHLW research report 2024 (202421004A) | Model inputs: Ac-225 effective dose-rate constant **0.03059 µSv·m²/(MBq·h)**; planned maximum 10 MBq (PSMA-617) and 20 MBq (PSMA-R2). Model outputs: caregiver about 0.32 mSv per treatment; public about 0.16 mSv/yr (PSMA-617) | Government report (PDF) | [MHLW report PDF](https://mhlw-grants.niph.go.jp/system/files/report_pdf/202421004A-buntan2.pdf) | M2-NRC-style baseline Γ for 225Ac | [search-level, verify]; model outputs, not measurements |
| 225Ac daughter redistribution (mouse, PSMA-I&T) | MIRDcalc analysis: unbound-progeny redistribution raised kidney and salivary-gland dose by factors of 1.3 and 2.5 (assuming 221Fr uptake and in-situ 213Bi). Another mouse study found no redistribution from tumour | Journals | Summarised in [PMC11178588](https://pmc.ncbi.nlm.nih.gov/articles/PMC11178588/) | Daughter-model uncertainty range | Preclinical |
| Open phantom/simulation data | Ac-225 34-scan phantom; PyTomography Ac225 NEMA plus fr221/bi213/tl209 SIMIND; radot p142 225Ac chain (earlier notes); Ennassiri 221Fr window response (restricted) | CC BY 4.0 (restricted for Ennassiri) | [zenodo.14166469](https://doi.org/10.5281/zenodo.14166469); [zenodo.24040979](https://doi.org/10.5281/zenodo.24040979); [zenodo.22055021](https://doi.org/10.5281/zenodo.22055021) | M3 for the 225Ac add-on | No patient data |
| topas-rpt | MC with time-resolved decay chains plus binding kinetics | MIT | [GitHub](https://github.com/bertoletlab/topas-rpt) | Simulate 225Ac chain external fields over time | Validation for 225Ac not stated |
| Radboud [225Ac]Ac-DOTA-MSC3 (preclinical radiobiology) | Preclinical CAIX tumour study data | CC0 | [doi:10.34973/pzkn-kw44](https://doi.org/10.34973/pzkn-kw44) | Marginal | Mouse model |

### Inferences
- For the 225Ac add-on, the external-dose problem is small: the MHLW Γ ≈ 0.031 µSv·m²/(MBq·h) at ≤ 10–20 MBq, and measured 225Ac-lintuzumab gives 0.3 µSv/h at 1 m (earlier notes). The dominant IRSA function for 225Ac would be **contamination/hygiene guidance**, driven by urinary 213Bi excess and incontinence. Kinetics priors can borrow Liubchenko's kidney/lesion T_eff (about 24–39 h) while the radionuclide physics follows the 9.9 d parent.
- A wearable gamma detector would see mostly the 218 keV (221Fr) and 440 keV (213Bi) lines at very low count rates. The Polson 225Ac phantom set and the PyTomography daughter projections are the only open data to test count-rate statistics and window choices.

### Gaps
- "CLEAN-225Ac" could not be verified under that name. Its supplementary data (if any) were not located.
- No open per-patient 225Ac excretion, whole-body retention or dose-rate time series was found.
- No human 225Ac-PSMA-617 PBPK with daughter compartments was found.

---

## Q5. Time-activity / excretion data in open supplementary tables (2015–2026) beyond those already listed

### Takeaway
This sweep found **no new patient-level whole-body retention, per-void urine or serial 1 m dose-rate tables** in open repositories. DataCite title searches for (dose rate AND patient AND 177Lu/131I) and for (urine/excretion/retention AND 177Lu/131I/225Ac) return only the OSF scoping-review protocol and the Maulidina 131I deposit, both already known.

The new citable aggregates are:
- the **Lutathera label excretion curve**;
- **de Bakker 2023**: 177Lu-PSMA-617 urine kinetics, fast and slow groups, eGFR effect, long-term urine activity;
- **Liubchenko 2024** for 225Ac.

The **MRTDosimetry six-time-point planar and SPECT phantom series** (now on Zenodo, CC BY) is the only open multi-time-point ground-truth retention dataset.

### Cited Findings
- DataCite title search for dose-rate patient datasets returned 3 records: OSF k4e2q (protocol) and two versions of Maulidina 2024 (131I). Both were covered earlier — [DataCite query](https://api.datacite.org/dois?query=titles.title:(%22dose%20rate%22%20OR%20%22dose-rate%22)%20AND%20titles.title:(patient%20OR%20patients))
- DataCite title search for (urine OR urinary OR excretion OR retention) AND (177Lu OR lutetium OR 131I OR radioiodine OR 225Ac) returned only OSF k4e2q — [DataCite query](https://api.datacite.org/dois?query=titles.title:(urine%20OR%20urinary%20OR%20excretion%20OR%20retention)%20AND%20titles.title:(177Lu%20OR%20lutetium%20OR%20131I%20OR%20radioiodine%20OR%20225Ac))
- Lutathera label: 44% at 5 h, 58% at 24 h and 65% at 48 h cumulative urinary excretion; terminal half-life 71 ± 28 h; urine radioactivity detectable up to 30 days — [Lutathera PI](https://www.novartis.com/us-en/sites/novartis_us/files/lutathera.pdf)
- de Bakker 2023: fast vs slow excretion groups (4.4 h vs 8.4 h half-time); 48 h excretion 75 ± 10% vs 36 ± 10%; eGFR effect; per-patient curves shown only in Fig. 1–3; data on request — [PMC10547615](https://pmc.ncbi.nlm.nih.gov/articles/PMC10547615/)
- Pluvicto label: terminal half-life 41.6 h (CV 68.8%), CL 2.04 L/h (31.5%) — [Pluvicto PI](https://www.novartis.com/us-en/sites/novartis_us/files/pluvicto.pdf)
- MRTDosimetry four-organ phantom: planar, raw and reconstructed SPECT, plus CT at 1, 4, 24, 40, 72 and 144 h (zip directory listing) — [doi:10.5281/zenodo.18891044](https://doi.org/10.5281/zenodo.18891044)
- Negative check: the CC BY "Additional file 2" of the 2025/26 EJNMMI Phys 177Lu-PSMA-617 bone-marrow dosimetry paper (123 KB xlsx) contains red-marrow **energy-absorption fractions** by bone-volume fraction (sheets "BVF 0.03" to "BVF 0.35"), not patient TACs — [doi:10.6084/m9.figshare.31023349](https://doi.org/10.6084/m9.figshare.31023349) (supplement to [doi:10.1186/s40658-025-00816-6](https://doi.org/10.1186/s40658-025-00816-6))
- EURADOS WG13 has a 177Lu-PSMA-617 case study comparing measured 1 m ambient dose-equivalent rate with simulation (slides only) — [Covens, EFOMP SMRD 2025](https://smrd2025.efomp.org/wp-content/uploads/2025/12/Covens.pdf) [search-level, verify]

### Inferences
- The K2 voiding-step generator can now be fully parameterised from open, citable aggregates:
  - Lutathera label (DOTATATE cumulative urinary fraction);
  - Kurth 2018 (PSMA-617 cumulative fraction);
  - de Bakker 2023 (between-patient bimodality and the eGFR covariate);
  - Schütze/Monserrat Fuertes (late T_eff, earlier notes).

  Patient-level raw data still need a data-sharing request. Candidates: de Bakker/Radboud, Kurth, Schütze, Bailey.
- The MRTDosimetry phantom series lets the team test the whole chain: activity at time t → simulated body-worn detector reading → inversion → T_eff. Ground truth is known at every step and the licence is CC BY.

### Gaps
- No open per-patient serial 1 m dose-rate data, per-void urine activity or whole-body retention table was found for 177Lu or 131I in this sweep.
- The supplements of Schütze 2026, Kurth 2018, the JNM 2026 225Ac phase 1 and Liubchenko 2024 were not opened.
- EU SmPC excretion figures for Pluvicto were not checked.

---

## Q6. Synthetic patient generators (Synthea and others) adaptable to nuclear medicine

### Takeaway
**Synthea is Apache-2.0** (GitHub, pushed 2026-11-20, about 3.5k stars). Its module library has cancer modules (lung, breast, colorectal, veteran lung, veteran prostate) and a hypothyroidism module. **No module mentions iodine, lutetium, radioiodine, PSMA or nuclear medicine** (GitHub code search, 0 hits each), and its radiation content is external-beam radiotherapy only. It can generate demographic and comorbidity context (age, BMI, renal function proxies, household), but RPT episodes would need a custom Generic Module Framework JSON module. No other open RPT-specific synthetic patient generator was found.

### Cited Findings
- Synthea licence Apache-2.0; last push 2026-11-20; description "Synthetic Patient Population Simulator" — [GitHub synthetichealth/synthea](https://github.com/synthetichealth/synthea)
- Module files with "cancer" in the filename: `lung_cancer.json`, `breast_cancer.json`, `colorectal_cancer.json`, `veteran_lung_cancer.json`, `veteran_prostate_cancer.json` (plus `lung_cancer/lung_cancer_probabilities.json`). A "thyroid" search finds only `hypothyroidism.json` — GitHub code search of `src/main/resources/modules`, 2026-11-23 ([modules folder](https://github.com/synthetichealth/synthea/tree/master/src/main/resources/modules))
- Code search of the modules for "iodine", "lutetium", "radioiodine", "nuclear medicine" and "PSMA" returned 0 results. Radiation strings in `lung_cancer.json` and `breast_cancer.json` refer to EBRT ("External beam radiation therapy (EBRT)…", "NSCLC Radiation") — same source
- PyTheraTwin (MIT) is the closest RPT-specific "synthetic patient" generator. It builds digital twins from real CTs rather than sampling demographics — [GitHub](https://github.com/qurit/PyTheraTwin)

### Inferences
- A practical IRSA synthetic-cohort recipe:
  1. Sample patient covariates with Synthea (Apache-2.0) or from published distributions: age, sex, BMI, eGFR, continence, household composition.
  2. Map them to kinetics parameters: T_eff and urinary fast/slow class from de Bakker/Kurth/Turku; BMI → body attenuation from Bellamy (earlier notes).
  3. Simulate trajectories with a tellurium/PySB compartment model with voiding events.
  4. Optionally attach anatomy and activity maps via PyTheraTwin for physics-layer training.

  Every component except PyCNO and SIMIND is permissively licensed.
- Synthea's veteran prostate cancer module could be extended with a "177Lu-PSMA cycle" state (6 cycles every 6 weeks per the Pluvicto label). This would generate realistic treatment timelines for app-agent testing (e.g., reminder and guidance logic).

### Gaps
- No Synthea community module for nuclear medicine or RPT was found. Only the main repository was searched, not forks or the module-builder gallery.
- Other generators (e.g., commercial virtual-population tools in PK-Sim's population module) were not compared.
