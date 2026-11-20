# IRSA context and human-behaviour layer: open datasets catalogue (checked 2026-11-20)

Scope: open or easily accessible data that a small team can use now for (1) contact-distance/duration priors and scenario simulation, (2) coarse location/venue context, (3) room geometry and wearable-motion artefacts. IRSA module codes used in tables:

- **M1 Contact prior / scenario simulator** (who is near the patient, how close, how long)
- **M2 Coarse location / venue context** (home vs transport vs workplace vs public venue; background density)
- **M3 Home geometry** (room sizes, bed/sofa/toilet layout, crowding)
- **M4 Wearable motion artefacts** (posture, walking, toilet, sleep: events that shift a body-worn detector's reading)
- **M5 Calibration / validation targets** (radiation-protection occupancy assumptions and measured caregiver behaviour)

Verification legend: **V** = license/size confirmed on the primary page in this session; **P** = partially confirmed (existence confirmed, license or details not); **U** = not verified in this session (prior knowledge only, flagged in Gaps).

---

## 1. Social contact and proximity data (distance, duration, age, setting)

### Takeaway
Diary-based contact surveys (POLYMOD and the 48-survey Zenodo "Social contact data" community, which includes LMIC surveys from the Gambia, Zimbabwe, Zambia/South Africa, Peru, Vietnam, Thailand, China and Somaliland) give **duration categories, physical-vs-conversational contact, setting and age** but **no distance**. Sensor datasets (SocioPatterns, Copenhagen, Reality Mining) give **fine-grained duration** and a fixed proximity threshold (or raw RSSI), but **no calibrated distance**. Only the exposure-notification BLE datasets (MIT PACT / MITRE range-angle / NIST) give **labelled ground-truth distance**, and they hold no behaviour data. IRSA therefore has to combine the two: survey data for "who and how long", plus an assumed distance per setting, checked against radiation-specific caregiver studies (Section 3). Several key contact datasets are **non-commercial (CC BY-NC)**.

### Cited Findings

**Table 1a: Diary-based contact surveys**

| Name | Contents | Size / coverage | License / commercial use | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| POLYMOD (Mossong et al. 2008) | One-day paper diary (5 a.m.–5 a.m.); contact = skin-to-skin (physical) or 2-way conversation of ≥3 words; **duration**: <5 min, 5–15 min, 15 min–1 h, 1–4 h, ≥4 h; **location**: home, work, school, leisure, transport, other; frequency (daily … first time); 5-yr age bands to 70+ | 7,290 participants, 97,904 contacts (13.4/person/day), 8 European countries (BE, DE, FI, GB, IT, LU, NL, PL), May 2005–Sep 2006; ~7.8 MB of files | Zenodo v2 record shows **CC BY-NC 4.0** (non-commercial) | Open download (CSV + dictionary); R `socialmixr`/`contactsurveys` | [Zenodo 10.5281/zenodo.3874557](https://zenodo.org/records/3874557); [paper](https://journals.plos.org/plosmedicine/article?id=10.1371/journal.pmed.0050074) | M1 | No distance; Europe-only; ~75% of home contacts were physical vs ~50% at school/leisure | V |
| Zenodo "Social contact data" community (48 surveys) | Same POLYMOD-style schema (participant, contact, household, sday CSVs) | Includes LMIC/middle-income: **Peru** (zenodo.3874805), **Zimbabwe** (3886638), **Vietnam** (3874802), **Zambia & South Africa CODA** (3874675), **China mainland** (3878754), **China before/during COVID** (7326686), **Thailand** (4086739; 2015: 4739777), **Russia** (3874674), **Hong Kong** (3874808), **Taiwan** (6385759), **Somaliland IDPs 2019** (7071876), **the Gambia BHDSS/FWHDSS 2022** (13101862); also **older adults 70+ NL 2021** (7751724), **EPICURUS older individuals** (20271335), **Household contact survey Belgium** (7965594), **US health-care personnel** (14156576), MixIT Italy post-pandemic (17579537), Reconnect (21218377) | **Per record; check each.** Gambia 2022 = **CC BY 4.0**; POLYMOD = CC BY-NC 4.0 | Open; programmatic via R `contactsurveys::list_surveys()/download_survey()` | [contactsurveys vignette (full list)](https://cran.nics.utk.edu/cran/web/packages/contactsurveys/vignettes/contactsurveys.html); [Gambia record](https://zenodo.org/records/13101862) | M1 (LMIC priors) | Licenses differ by survey; Gambia record ~3.4 MB, reports all direct contacts in prior 24 h plus classroom observation subset | V (list, Gambia, POLYMOD); P (others) |
| CoMix (COVID-era, 2020–2023) | Repeated panel contact diaries during pandemic restrictions | Zenodo records for UK (13684044), BE (10549953), NL, AT, DK, ES, FR, IT, PT, PL, GR, SI, HR, SK, HU, EE, LT, FI, CH; CoMix 2.0 (7331926); last round BE/CH/NL/UK (11154066) | Not checked per record | Open via Zenodo / `contactsurveys` | [list in vignette](https://cran.nics.utk.edu/cran/web/packages/contactsurveys/vignettes/contactsurveys.html) | M1 (sensitivity: "restricted behaviour" scenario) | Pandemic-distorted; useful as analogue for "patient told to limit contact" | P |
| BBC Pandemic (Klepac et al.) | Age-specific mixing matrices by setting (home, work, school, other) and type (conversational vs physical) | 40,177 UK participants who completed the study | Not stated; matrices as XLS via medRxiv; LSHTM Data Compass lists "No files available" | medRxiv supplementary | [medRxiv 2020.02.16.20023754](https://www.medrxiv.org/content/10.1101/2020.02.16.20023754v2.external-links); [LSHTM Data Compass](https://datacompass.lshtm.ac.uk/id/eprint/1615/) | M1 (UK matrices) | Matrices only (no individual records found); 4,007 users dropped for slider default at age 50 | P |
| Prem et al. 2021 synthetic contact matrices | Projected age × setting (home, work, school, other) matrices, with urban/rural where data exist | 177 geographical regions | **License not confirmed** | GitHub `kieshaprem/synthetic-contact-matrices`; Zenodo 10.5281/zenodo.4889499 | [PMC8354454](https://pmc.ncbi.nlm.nih.gov/articles/PMC8354454); [correction](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11410204/) | M1 (default priors for countries with no survey) | Modelled, not observed; use corrected code version | P |

**Table 1b: Wearable / phone proximity sensor datasets (duration-rich, distance-poor)**

| Name | Contents | Size / coverage | License | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| SocioPatterns datasets (Génois & Barrat Zenodo bundle) | Six face-to-face contact networks + six co-presence networks + node group metadata | 66.7 MB total | **CC BY-NC 4.0** | Open | [Zenodo 10.5281/zenodo.1117884](https://zenodo.org/record/1117884) | M1 (contact duration distributions) | Page does not name deployments in the bundle; proximity is a threshold, not a distance | V |
| SocioPatterns hospital ward (Lyon) | Face-to-face contacts | 75 people (29 patients, 46 health-care workers), Dec 2010 | See above / site | sociopatterns.org | [Génois/Barrat review arXiv 1305.3532](https://arxiv.org/pdf/1305.3532) | M1 (clinic/hospital visit scenario) | `sociopatterns.org/datasets/` returned 404 during this session | P |
| SocioPatterns primary school (Lyon) | Face-to-face contacts | 242 people (232 children, 10 teachers), Oct 2009 | See above | sociopatterns.org | [arXiv 1305.3532](https://arxiv.org/pdf/1305.3532) | M1 (child-contact scenario) | High-income setting | P |
| Rural Malawi village (Ozella et al. 2021) | Wearable proximity sensors, face-to-face contacts within and between households | 86 participants, 13 complete days, Mdoliro village, Dowa district, 16 Dec 2019–10 Jan 2020 (UNICEF Malawi funded) | Article CC BY 4.0; **dataset license/hosting not confirmed** | Not confirmed | [EPJ Data Science 10.1140/epjds/s13688-021-00302-w](https://link.springer.com/article/10.1140/epjds/s13688-021-00302-w); [arXiv 2012.10983](https://arxiv.org/pdf/2012.10983); [later analysis citing 86 participants/13 days](https://arxiv.org/pdf/2402.03333) | M1 (LMIC household contact durations; most relevant sensor dataset for TC member states) | Need to locate data (check paper's data-availability statement / sociopatterns.org) | P |
| Copenhagen Networks Study | Bluetooth proximity (**RSSI in dBm**) every 5 min, calls, SMS, Facebook friendships, gender | 706 users in `bt.csv`; 4 weeks; DTU students | Article CC BY 4.0; **data-file license not confirmed** (Figshare page returned 403) | Figshare | [Figshare 10.6084/m9.figshare.7267433](https://figshare.com/articles/dataset/The_Copenhagen_Networks_Study_interaction_data/7267433); [Sci Data paper PMC6906316](https://pmc.ncbi.nlm.nih.gov/articles/PMC6906316/) | M1 (RSSI-as-distance-proxy experiments; duration of co-location) | Relative timestamps only; young adults; RSSI not distance-calibrated | P |
| MIT Reality Mining (2004–2005) | Call logs, Bluetooth devices in proximity (5-min scans, ~4 M proximity events), cell tower IDs, app use | ~100 subjects | **Not confirmed**; copy listed as CRAWDAD `mit/reality` on IEEE DataPort (access may require account) | IEEE DataPort / MIT | [IEEE DataPort CRAWDAD mit/reality](https://ieee-dataport.org/open-access/crawdad-mitreality); [ReadMe](https://www.cse.iitk.ac.in/users/nsrivast/HCC/RealityMining_ReadMe.pdf) | M1 (low priority) | Old, US, no distance | P |

**Table 1c: Exposure-notification BLE datasets with ground-truth distance**

| Name | Contents | Size / coverage | License | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| NIST TC4TL challenge data (2020, with MIT PACT) | Phone RSSI series + other phone sensors; task: estimate distance and time between two phones | 15,552 training, 8,423 test, 936 dev files; train = MITRE Range-Angle Structured; dev/test = MIT Matrix; distance classes 1.2, 1.8, 3.0, 4.5 m | Not stated; registration closed | Underlying sets via MIT PACT GitHub | [NIST TC4TL](https://www.nist.gov/node/1617526); [TC4TL ML paper arXiv 2203.04307](https://arxiv.org/pdf/2203.04307) | M1 (if IRSA ever uses phone-to-phone BLE to sense a companion's distance) | Phone-to-phone only; no behaviour | P |
| MIT PACT datasets page (12 sets) | BLE RSSI static (6 ft, many carry positions/environments); EN beacons; H0H1 (far >10 ft vs close <6 ft); KRI anechoic chamber; **MIT Matrix** (3–15 ft); **MITRE Range-Angle structured / unstructured** (12 MITRE staff at home); Range Angle; phone/chipset catalogue; **SocializeME** (11 h indoor social interaction, 15 volunteers, wristband BLE tags, Mendeley) | ZIPs on GitHub | Each repo has LICENSE.md/DISTRIBUTION.md; **not checked** | Open download | [mitll.github.io/PACT/datasets.html](https://mitll.github.io/PACT/datasets.html) | M1 | Licenses per repo unverified | P |
| NIST BLE RSSI for proximity detection (mds2-2827) | Line-of-sight (person standing between radios) and non-line-of-sight (walls of different materials) scenarios with measured separation | ZIP + README | **NIST open license** (public) | Open | [data.gov record](https://catalog.data.gov/dataset/bluetooth-low-energy-ble-received-signal-strength-indicator-rssi-data-for-proximity-detect); [ZIP](https://data.nist.gov/od/ds/mds2-2827/BLE_RSSI_Data.zip) | M1 (body-shadowing and wall attenuation of RF: rough analogue for how bodies/walls perturb distance sensing) | Issued Oct 2022 | V |

### Inferences
- For M1 priors, the most useful **open, commercially usable** LMIC survey confirmed is **the Gambia 2022 (CC BY 4.0)**. POLYMOD and the SocioPatterns Zenodo bundle are **CC BY-NC 4.0**, which is fine for an IAEA research prototype but must be revisited if IRSA ever becomes a commercial product.
- Survey duration categories (<5 min … ≥4 h) plus the "physical contact" flag can be mapped to IRSA distance bins (physical ≈ <0.3 m; conversational ≈ 0.5–1.5 m) as an **assumption** to be calibrated against Levart 2025 and the NRC scenarios (Section 3). No dataset found records true distance together with real-life behaviour.
- The Malawi village sensor study is the closest open analogue to TC-member-state households and should be chased for raw data (contact the authors or check SocioPatterns).
- Older-adult and household-specific surveys (NL 70+, EPICURUS, Belgian household survey) suit the typical radiopharmaceutical-therapy patient (older adults with thyroid cancer, NET or prostate cancer).

### Gaps
- Licenses for most individual Zenodo surveys, CoMix records, Prem matrices, MIT PACT repos, Copenhagen data files and Reality Mining were **not confirmed**.
- Kenya (Kilifi, Kiti et al.) and Uganda contact surveys did **not** appear in the Zenodo community list; their hosting was not checked.
- No household dataset with **measured distance** (as opposed to proximity thresholds or RSSI) was found.
- The SocioPatterns website dataset page returned 404, so the per-dataset terms (Kenya households, South Africa, etc.) could not be listed.

---

## 2. Time-use surveys and caregiving time (time at home, with family, on transport)

### Takeaway
Time-use diaries give the **daily time budget** (home, travel, sleep, care) that bounds total exposure time. ATUS (US) is the easiest to get (free, single and multi-year files through **2025**). IPUMS harmonises ATUS, AHTUS and MTUS (free for research). HETUS microdata are restricted to accredited researchers, and HETUS 2020 microdata are not expected before 2027. UKTUS 2014–15 needs UK Data Service registration. China's NBS publishes only aggregates as far as this session found. For cancer caregivers, published surveys give **~33 h/week** of care on average.

### Cited Findings

| Name | Contents | Size / coverage | License / access terms | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|
| American Time Use Survey (ATUS, BLS) | 24-h diaries; single-year files 2003–**2025**; multi-year 2003–2025 and 2003–2024; modules: Well-Being, Leave, Eating & Health | US, nationally representative | Free download from BLS (terms page not opened this session) | [bls.gov/tus/data.htm](https://www.bls.gov/tus/data.htm) | M1 (time at home/travel, by age; default scenario durations) | US only; "who was present" file and eldercare items not confirmed this session | P |
| IPUMS Time Use (ATUS v3.4, AHTUS v1.3, MTUS v1.5) | Harmonised time-diary extracts; MTUS = multi-country harmonised diaries | ATUS since 2003; AHTUS 1930–present; MTUS multiple countries (list not on page) | "Free individual-level time use data for research purposes"; terms at ipums.org/about/terms (not read) | [ipums.org/projects/ipums-time-use](https://www.ipums.org/projects/ipums-time-use) | M1 (cross-country time budgets) | Commercial and redistribution terms not confirmed | P |
| HETUS (Eurostat) 2010 round | Harmonised European diaries | Scientific-use files for 17 countries: AT, BE, DE, EE, EL, ES, FI, FR, HU, IT, LU, NL, PL, RO, UK, NO, RS (surveys 2008–2015) | Universities/certified research centres only; confidentiality declaration; statistical/scientific use only; secure-use files in Eurostat safe centre; **HETUS 2020 microdata not before 2027**; public gets aggregate Eurobase tables | [Eurostat HETUS microdata](https://ec.europa.eu/eurostat/web/microdata/harmonised-european-time-use-surveys); [metadata](https://ec.europa.eu/eurostat/cache/metadata/en/tus_00_esms.htm) | M1 (European default priors from aggregates) | Restricted; slow access | V |
| UK Time Use Survey 2014–2015 (SN 8128) | Diaries for people aged 8+; added mobile-device and enjoyment fields | UK; fieldwork Apr 2014–Dec 2015 (NatCen for CTUR) | UK Data Service End User Licence; registered users; **commercial use requires data-owner approval** | [DOI 10.5255/UKDA-SN-8128-1 (b2find)](https://b2find.eudat.eu/dataset/686f6564-b83e-5fe4-a7e9-24367928fa2e); [timeuse.org](https://www.timeuse.org/node/10831) | M1 | "With whom" variable not confirmed | P |
| China 2nd National Time Use Survey 2018 (NBS) | 2-day diaries (weekday + weekend), 15-min slots, residents 15+ | 11 provinces/municipalities; 20,226 households, 48,580 people; daily averages: 11 h 53 min personal needs, 4 h 24 min paid work, 2 h 42 min unpaid work | Aggregates published; **microdata access not found** | [NBS communiqué 2019](https://www.stats.gov.cn/xxgk/sjfb/zxfb2020/201901/t20190125_1768278.html); [UNSD presentation](https://unstats.un.org/unsd/demographic-social/meetings/2019/newyork-egm-tus/May%2021_China.pdf) | M1 (aggregate priors for China) | 3rd national TUS communiqué Oct/Nov 2024 also exists | P |
| China 3rd National Time Use Survey (2024) | Communiqués No. 1 and No. 2 | National | Aggregates | [NBS English No.1](https://www.stats.gov.cn/english/PressRelease/202411/t20241115_1957435.html); [No.2](https://www.stats.gov.cn/english/PressRelease/202411/t20241115_1957436.html) | M1 | Content not read in detail | P |
| Zimbabwe contact + time-use survey (Melegaro et al. 2017, Manicaland) | Contact diary paired with time use | Zimbabwe | Zenodo record (license not checked) | [Zenodo 3886638](https://zenodo.org/record/3886638) | M1 (LMIC time-at-home and contact duration together) | License to check | P |

**Caregiving time (cancer):**
- Caregiving in the U.S. 2015 (National Alliance for Caregiving): cancer caregivers average **32.9 h/week**; **32% provide ≥41 h/week** — [NAC Cancer Caregiving report 2016](https://www.caregiving.org/wp-content/uploads/2022/11/CancerCaregivingReport_FINAL_June-17-2016.pdf).
- CanCORS (lung and colorectal cancer, 2005–2006): ~22.5% of caregivers gave ≤1 h/week, 25.4% 1–10 h, 26% 11–35 h, 20.1% >35 h; **24.5 h/week in terminal phase vs ~15 h earlier** — [PMC4479404](https://pmc.ncbi.nlm.nih.gov/articles/PMC4479404); [NCI PDQ Informal Caregivers](https://www.cancer.gov/about-cancer/coping/family-friends/family-caregivers-hp-pdq).

### Inferences
- A simple M1 prior: total daily time near the patient ≤ (time both at home awake) + (shared sleep). Time-use diaries supply the first term by age/employment. The caregiving figures (≈4.7 h/day average for cancer caregivers; >5.8 h/day for the 32% top group) bracket "dependent patient" scenarios. Note that care hours ≠ hours at 1 m.
- For LMIC deployment, published aggregates (China NBS, Eurostat tables) and the Zimbabwe contact/time-use survey may be the only practical inputs. IRSA should treat these as weak priors and rely on patient-reported routines.

### Gaps
- Japan's Survey on Time Use and Leisure Activities (2021) and its microdata terms were **not verified**.
- ATUS "Who" (co-presence) file contents, ATUS eldercare questions, IPUMS commercial/redistribution terms and MTUS country list were **not confirmed** on primary pages this session.
- No time-use dataset specific to **patients after radionuclide therapy** or their households was found.

---

## 3. Radiation-protection occupancy factors and measured caregiver behaviour (calibration targets)

### Takeaway
Regulatory defaults assume **occupancy factor (OF) 0.25 at 1 m** (NRC RG 8.39), and NRC's own literature review says the realistic maximum is **~0.5**, with 0.25 often not feasible in small homes or with children or infirm patients. The best measured behavioural target found is **Levart et al. 2025 (EJNMMI Physics, UK, 177Lu-DOTATATE)**: over 4 weeks, family members spent a median **39.5 h at 1 m and 19.9 h at <1 m**, with huge spread (up to ~194 h and ~230 h). Diary data are available only on request.

### Cited Findings
- **NUREG-1556 Vol. 9 Rev. 3** ("Program-Specific Guidance About Medical Use Licenses") is the latest revision, published **September 2019**, PDF accession **ML19256C219** — [NRC NUREG-1556 Vol. 9 page](https://www.nrc.gov/reading-rm/doc-collections/nuregs/staff/sr1556/v9/index.html). (Appendix U text was not retrieved this session.)
- NRC RES literature review (SECY-18-0015 Enclosure 1, Feb 2018): OF is "the fraction of the time a person is close to the patient" at an average of 1 m; **RG 8.39 default OF = 0.25 at 1 m**; if the patient sleeps alone 8 h/day, the max OF ≈ **0.7**, and allowing ~4 h/day of unrelated tasks the **max reasonable OF ≈ 0.5**; "a number of patients live in trailers or … very small apartments where maintaining occupancy of 0.25 at 1 meter is difficult or not feasible"; moving from 1 m to 0.5 m raises dose rate ×4; 1 m → 1.5 m lowers it ×2.3 — [NRC ML17279B142](https://www.nrc.gov/docs/ML1727/ML17279B142.pdf).
- Same NRC review, scenario results: hotel scenarios do not exceed 1 or 5 mSv at 100% occupancy up to 10 days; nursing-home person **seated ~30 cm from the bed edge** exceeds 1 mSv (100 mrem) if OF > **0.2 (4.8 h/day)** after thyrotoxicosis therapy or **0.25 (6 h/day)** after thyroid-cancer therapy; roommates in adjacent beds do not exceed limits — [NRC ML17279B142](https://www.nrc.gov/docs/ML1727/ML17279B142.pdf).
- RG 8.39 (as surfaced in search): for half-lives ≤1 day, 0.25 is "difficult to justify" and **0.75–1.0** may be more appropriate — [search surfaced NRC documents, e.g. forms.nrc.gov ML003739575](https://forms.nrc.gov/docs/ML0037/ML003739575.pdf) (exact document holding this sentence not confirmed).
- **Levart D, Wicks S, Heraghty N, Kalogianni E, Mulholland N. EJNMMI Physics 2025** (King's College Hospital, UK): 12 family members (11 partners, 1 adolescent) of 177Lu-DOTATATE patients (median 7,515 MBq); LiF TLD pin badges worn at waist **continuously for 4 weeks**; median effective dose **0.038 mSv** (0.014–0.111); **time at 1 m: median 39.5 h (0–194.3 h); at <1 m: median 19.9 h (0–229.8 h)** over 4 weeks. By period (median h, at 1 m / <1 m): days 1–2: 0.2 / 0.1; week 1: 2.3 / 0.0; week 2: 2.3 / 1.6; week 3: 8.8 / 5.4; week 4: 11.0 / 3.8. Week 4 (restrictions lifted) diaries: **1.6 h/day at 1 m (0–7.0), 0.5 h/day at <1 m (0–14.6)**. Discharge dose rate median 15 µSv/h at 1 m, 150 µSv/h at 0.1 m. Restrictions advised (<1 m): children <2 y 17 d; pregnant / children 2–5 y 13 d; children >5 y 10 d; other adults 6 d; **no bed sharing 16 d**; **public transport ≤1 h for 1 week**; double flushing 1 week. Contact model assumed 8 h sleep at 0.1 m. Data "available from the corresponding author on reasonable request"; article **CC BY-NC-ND 4.0** — [PMC12595148](https://pmc.ncbi.nlm.nih.gov/articles/PMC12595148/).
- Earlier 177Lu-DOTATATE restriction study (EJNMMI Physics 2019): dose rates at 0.1 m and 1 m; ~8 days day-contact restriction and separate sleeping for inpatient therapy (9 days outpatient mean); estimated partner dose 6.2 mSv (2.0–17.9) without restrictions — [PMC6484059](https://pmc.ncbi.nlm.nih.gov/articles/PMC6484059).
- Tanzanian I-131 caregiver modelling study (2025): totally dependent patients modelled at **10 cm** "close" distance; transport scenarios <1 mSv/yr for short durations — [VJOL article](https://vjol.info.vn/mostc/article/download/136456/111618/).
- Graves' disease outpatient I-131: household contacts mean dose 0.258 mSv — [ScienceDirect S0378603X18301748](https://www.sciencedirect.com/science/article/pii/S0378603X18301748).
- A systematic review/meta-analysis of radiation exposure to family members of I-131 patients exists — [ResearchGate 370594653](https://www.researchgate.net/publication/370594653_Radiation_exposure_to_family_members_of_patients_treated_with_radioactive_iodine_131I_a_systematic_review_and_meta-analysis) (not read).

### Inferences
- **Calibration check for M1:** Levart week-4 values (1.6 h/day at 1 m ≈ OF 0.07; 0.5 h/day at <1 m) are far below the 0.25 default for the median person, but the maxima (~7 h/day at 1 m, ~14.6 h/day at <1 m) exceed it. A scenario simulator should reproduce this **heavy-tailed, person-specific** distribution, not a single OF.
- The NRC "30 cm from bed edge, 4.8–6 h/day" and "8 h sleep at 0.1 m" assumptions are ready-made **dependent-patient / shared-bed** scenario parameters.
- Levart's restriction list (bed-sharing, children by age, public transport ≤1 h) maps directly onto IRSA's scenario library (grandchildren visit, shared bed, commute).

### Gaps
- The exact NUREG-1556 Vol. 9 Rev. 3 Appendix U wording (OF values by half-life) was not retrieved (PDF download not completed). RG 8.39 Rev. 1 text not read directly.
- Levart individual diary data are not public (request only). No public, per-person caregiver distance/duration dataset was found.
- ICRP 94 / IAEA SRS-63 occupancy assumptions not checked this session.

---

## 4. Geographic context data (maps, buildings, population, transit, crowding)

### Takeaway
Open, commercially usable global layers exist and cover LMICs well: **Google Open Buildings** (1.8 B buildings, Africa / S & SE Asia / LatAm; CC BY 4.0 *or* ODbL), **Microsoft Global ML Building Footprints** (CDLA-Permissive-2.0, updated Nov 2026), **Overture** (places CDLA-Permissive-2.0 / Apache / CC0 per source; buildings/transport/base ODbL), **Foursquare OS Places** (Apache 2.0, 100 M+ POIs, gated download), and **CC BY 4.0 population grids** (GHSL, WorldPop, LandScan). Live crowd data is **not** available through official Google APIs, and the workarounds are unofficial or commercial. This supports demoting crowd data to background context.

### Cited Findings

| Name | Contents | Size / coverage | License / commercial use | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| Overture Maps: Places | POIs from Meta, Microsoft, Foursquare, AllThePlaces, others | Global | Mostly **CDLA-Permissive-2.0**; Foursquare-sourced = **Apache 2.0** (+ NOTICE.txt); AllThePlaces = **CC0 1.0** | Open (cloud Parquet) | [docs.overturemaps.org/attribution](https://docs.overturemaps.org/attribution/) | M2 (venue category near patient location, on-device lookup) | Attribution per source | V |
| Overture Maps: Buildings / Transportation / Base / Divisions | Buildings merge OSM + Microsoft + Google Open Buildings + Esri + others; transport from OSM/TomTom | Global | **ODbL** (share-alike for derived databases) | Open | [docs.overturemaps.org/attribution](https://docs.overturemaps.org/attribution/) | M2, M3 (building footprint → dwelling size proxy) | ODbL share-alike obligations | V |
| Overture Addresses | National/regional open address sources | Varies by country | Mixed (CC BY 4.0, CC0, government licences) | Open | [same](https://docs.overturemaps.org/attribution/) | M2 | Sparse in LMICs (inference) | V |
| Foursquare Open Source Places | 100 M+ POIs, 22 core attributes, monthly updates | Global | **Apache 2.0** | Gated on Hugging Face (share contact info) / token + Iceberg catalogue | [Foursquare blog](https://foursquare.com/resources/blog/products/foursquare-open-source-places-a-new-foundational-dataset-for-the-geospatial-community/); [HF](https://huggingface.co/datasets/foursquare/fsq-os-places) | M2 | OSM community reports stale places (closed 10+ years) | V |
| Microsoft Global ML Building Footprints | Building polygons from Bing imagery 2014–2024 (+ Vexcel 2021–2026); some heights | Intro says 1.4 B buildings (FAQ says 999 M polygons: inconsistent); 225 regions, 30,108 tiles; Sudan, Ethiopia, Kenya, Nigeria in recent updates; last update **12 Nov 2026** | **CDLA Permissive 2.0** (GitHub); Overture lists it under ODbL inside its buildings theme | GeoJSONL.gz via `dataset-links.csv`; Planetary Computer | [GitHub](https://github.com/microsoft/GlobalMLBuildingFootprints) | M2, M3 | Missing where imagery predates 2014 | V |
| Google Open Buildings v3 | 1.8 B building detections with confidence ≥0.65, area, Plus Code | Africa, South Asia, SE Asia, Latin America & Caribbean (≈100 countries; 58 M km²); polygons 178 GB, points 48 GB | **CC BY 4.0 or ODbL v1.0 (user's choice)** | CSV (S2 cells), Earth Engine, HDX, gsutil | [sites.research.google/gr/open-buildings](https://sites.research.google/gr/open-buildings/) | M2, M3 (footprint area → home-size prior in LMICs) | Conflict zones excluded; separate temporal dataset not detailed | V |
| OpenStreetMap | Roads, POIs, buildings, transit | Global; completeness uneven in LMICs | **ODbL** | Open | [openstreetmap.org/copyright](https://www.openstreetmap.org/copyright) | M2 | License page not re-fetched this session | U |
| GHSL GHS-POP R2023A | Multitemporal population grids 1975–2030 (100 m, 1 km per JRC catalogue) | Global | **CC BY 4.0** | JRC / AWS Open Data / Earth Engine | [AWS registry](https://registry.opendata.aws/ghs-pop/); [GHSL Data Package 2023](https://ghsl.jrc.ec.europa.eu/documents/GHSL_Data_Package_2023.pdf); [GHS-POP page](https://human-settlement.emergency.copernicus.eu/ghs_pop2023.php) | M2 (background density) | Residential (night-time) population | V |
| WorldPop | 100 m country rasters; Global 2015–2030 R2024A 1 km mosaics | Global, LMIC focus | **CC BY 4.0** | Open | [WorldPop licence.txt](https://worldpop-public-data.soton.ac.uk/GIS/Population/Individual_countries/TZA/United_Republic_of_Tanzania_100m_Population/licence.txt); [R2024A release statement](https://worldpop-public-data.soton.ac.uk/repo/prj/Global_2015_2030/R2024A/Mosaicked_1km/v1/release_statement.pdf) | M2 | Modelled | V |
| LandScan Global | **Ambient** (24-h average) population, 30 arc-second (~1 km); 2000–2024 | Global | **CC BY 4.0** since ORNL's 2022 public release (2024 record also shows rights "Public") | ORNL portal (portal content not readable via fetch); Earth Engine community catalogue | [ORNL 2022 news](https://www.ornl.gov/news/public-release-ornl-global-population-distribution-data-aids-humanitarian-support); [BTAA 2024 record](https://geo.btaa.org/catalog/88-0001_gl_2024.dc_xml); [GEE catalogue](https://gee-community-catalog.org/projects/landscan/) | M2 (ambient ≈ daytime exposure context) | License confirmed via secondary catalogues, not ORNL's own terms page | P |
| Meta / Data for Good high-resolution population density (HRSL) | ~30 m population incl. demographic splits | Many countries | Believed CC BY 4.0 | HDX | [HDX Meta org](https://data.humdata.org/organization/meta) | M2 | HDX page returned 403; license not verified | U |
| Mobility Database | Catalogue of GTFS, GTFS-RT and GBFS feeds; API | 6,000+ feeds, 99+ countries | Catalogue terms not read; **each feed has its own licence** (not verified) | Open API | [mobilitydatabase.org/about](https://mobilitydatabase.org/about) | M2 (commute duration/vehicle type for transport scenario) | LMIC feed coverage is thin (inference); many informal transit systems have no GTFS | P |
| Google Popular Times / live busyness | Relative hourly busyness; live where enough data | Venue-dependent | **Not provided by official Places API**; shown only when Google has sufficient visit data | Only via unofficial scrapers (e.g. `populartimes`), which may conflict with Maps terms | [Google Business help](https://support.google.com/business/answer/6263531?hl=en); [populartimes GitHub](https://github.com/m-wrzr/populartimes) | M2 (drop) | Relative 0–100, opaque, sparse in LMICs | V |
| BestTime.app (commercial) | Hourly forecasts normalised 0–100, live busyness where available | Venue-dependent | Commercial API (pricing/terms not checked) | Paid | [BestTime blog](https://blog.besttime.app/popular-times-api-alternative-for-developers/) | M2 (optional) | Vendor-claimed ±5% vs Google; not official | P |
| Google Maps Platform caching terms | Place IDs cacheable; lat/lng for some APIs cacheable up to 30 consecutive days, then delete or replace | — | Restrictive: no long-term storage of most Maps content | — | [Maps Service Specific Terms (archived 2024)](https://cloud.google.com/maps-platform/terms/maps-service-terms/index-20240422); [2025 archive](https://cloud.google.com/archive/maps-platform/terms/maps-service-terms-20250501) | M2 | Current Places-specific storage rule not confirmed | P |

### Inferences
- For privacy-by-default on-device context, a small offline bundle of **Overture/Foursquare POI categories + building footprints + one population grid** (all permissive or CC BY/ODbL) lets the phone classify "home / transit / workplace / public venue" without sending location to a server. Google Maps content cannot legally be cached for this.
- In LMICs, **Google Open Buildings + Microsoft footprints** are far more complete than OSM buildings. Footprint area divided by DHS household size (Section 5) gives a rough "space per person" crowding prior.
- LandScan's *ambient* population is the better fit for "how many people around during the day"; GHSL/WorldPop are residential (night-time).

### Gaps
- OSM ODbL and Meta HRSL licenses not re-verified this session; Mobility Database catalogue licence and per-feed licences unverified.
- Exact current Google Places API storage/caching clause and any "high-risk / health use" restriction in the Maps Platform Terms were not confirmed.
- BestTime pricing and terms (storage, health use) not checked.

---

## 5. Indoor geometry and room-size statistics (home scenarios)

### Takeaway
For home-scenario simulation the only confirmed **permissive** large floor-plan dataset is **Swiss Dwellings v3 (CC BY 4.0; 45,176 apartments, including toilets/bathtubs as features)**. CubiCasa5K is **CC BY-NC-SA 4.0** and ZInD is academic non-commercial. RPLAN, Structured3D and HouseExpo licenses were not confirmed. All of these come from high-income or East Asian housing. For LMIC crowding, the DHS **persons per sleeping room** (hv216) is the practical proxy.

### Cited Findings

| Name | Contents | Size / coverage | License | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| Swiss Dwellings v3.0.0 (Archilyse) | 2D geometries: ~1.7 M walls/railings, ~715 k doors/windows, ~520 k areas (rooms, bathrooms, kitchens), ~315 k features (sinks, **toilets**, bathtubs); simulations (view, daylight, noise, connectivity) | 45,176 apartments, ~370 k rooms, ~3,100 buildings (Switzerland); 931.9 MB | **CC BY 4.0** | Open | [Zenodo 10.5281/zenodo.7788422](https://zenodo.org/records/7788422) | M3 (room-to-room distances, bedroom–bathroom distance, apartment area distributions) | Swiss housing only; no furniture | V |
| Modified Swiss Dwellings (MSD) | ML-ready derivative of Swiss Dwellings v3 | — | "License info not available" on TU Delft listing | TU Delft | [doi 10.4121/e1d89cb5-…](https://doi.org/10.4121/e1d89cb5-6872-48fc-be63-aadd687ee6f9); [arXiv 2407.10121](https://arxiv.org/pdf/2407.10121) | M3 | License unclear | P |
| CubiCasa5K | Floor-plan images with polygon annotations in 80+ object categories | 5,000 plans; 5.5 GB | **CC BY-NC-SA 4.0** | Open | [Zenodo 2613548](https://zenodo.org/records/2613548); [paper arXiv 1904.01920](https://arxiv.org/pdf/1904.01920) | M3 | Non-commercial, share-alike; origin country not stated on page | V |
| Zillow Indoor Dataset (ZInD) | Annotated floor plans with 360° panoramas | (CVPR 2021) | **ZInD Terms of Use: free for academic, non-commercial use; not licensed for commercial use**; non-academic users must submit a proposal; code Apache | GitHub request | [github.com/zillow/zind](https://de.github.com/zillow/zind); [CVPR paper](https://openaccess.thecvf.com/content/CVPR2021/papers/Cruz_Zillow_Indoor_Dataset_Annotated_Floor_Plans_With_360deg_Panoramas_and_CVPR_2021_paper.pdf) | M3 | US homes; restrictive | V |
| RPLAN | Real residential floor plans (Asia) | 80 k+ (some sources say 120 k) | **Not confirmed** | Request form | [review arXiv 2504.09694](https://arxiv.org/pdf/2504.09694) | M3 | License unverified | P |
| Structured3D | Photo-realistic synthetic house designs | 3.5 k house designs | **Not confirmed** | Agreement form | [review arXiv 2504.09694](https://arxiv.org/pdf/2504.09694) | M3 | License unverified | P |
| HouseExpo | 2D floor plans (SUNCG-derived) | — | **Not found** in searches | — | — | M3 | Unverified | U |
| DHS crowding indicator | Persons per sleeping room = usual household members / rooms used for sleeping (**hv216**); ≥3 persons per room flagged as crowded; households with 0 sleeping rooms handled separately | DHS surveys (many LMICs) | DHS registration (terms not checked) | DHS Program | [DHS User Forum (Stata code)](https://userforum.dhsprogram.com/pdf.php?msg=19811); [forum reply on sleeping space](https://userforum.dhsprogram.com/pdf.php?msg=24722) | M3 (LMIC bed/room-sharing prior), M1 | Variable definition can differ by survey; check each survey's documentation | P |
| Other survey sleeping-room variables | e.g. World Bank Microdata Library `H_SLEEPROOMS`; UNHCR microdata catalogue | Country surveys | Per catalogue | World Bank / UNHCR microdata | [World Bank variable](https://microdata.worldbank.org/index.php/catalog/3161/variable/H/H_SLEEPROOMS) | M3 | Heterogeneous | P |

### Inferences
- A practical M3 approach: sample apartment layouts from **Swiss Dwellings** (permissive) to derive distributions of bedroom-to-living-room and bedroom-to-toilet distances. Then **scale area down** using LMIC footprint areas (Google/Microsoft buildings) and DHS persons-per-sleeping-room, and force **shared-bed / shared-room** scenarios where DHS indicates ≥3 per sleeping room. Sofa-to-sofa and bed-partner distances need assumptions (e.g. bed partner 0.1–0.3 m, consistent with Levart's 0.1 m sleep assumption and NRC's 30 cm bedside scenario in Section 3).

### Gaps
- No open dataset with **furniture positions** (sofa, bed) and true inter-person distances in homes was confirmed. CubiCasa5K category list (beds/sofas) not confirmed on the page.
- No LMIC floor-plan dataset found. RPLAN, Structured3D and HouseExpo licenses unverified.
- American Housing Survey, EU-SILC overcrowding and IPUMS-International census room counts not checked.

---

## 6. Wearable motion / activity datasets (posture, walking, toilet, sleep artefacts)

### Takeaway
**Capture-24 (CC BY, 151 people, free-living wrist accelerometer, 100 Hz, ~24 h each)** is the best open free-living set for sleep, sitting, walking and other daily activities. **ExtraSensory** is the only confirmed open set with explicit **toilet (2,655 min, 43 users)** and **shower** labels plus "lying down", "sleeping", "in car" and "on bus", from phone + watch in the wild. It is free with citation, but no formal license is named. PAMAP2, WISDM and HHAR are clean **CC BY 4.0** lab datasets for basic posture and gait.

### Cited Findings

| Name | Contents | Size / coverage | License | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| Capture-24 (Oxford, 2021) | Axivity AX3 **wrist** accelerometer, 3-axis, **100 Hz**; ground truth from body camera (1 image / 20 s, images not released) and sleep diaries; 200+ Compendium-of-Physical-Activity labels, mapped to Willetts/Walmsley label sets | 151 participants, ~24 h each; zip 6.4 GB (~7 GB) | **CC BY** | ORA download / repo code | [ORA record, DOI 10.5287/bodleian:NGx0JOMP5](https://ora.ox.ac.uk/objects/uuid:99d7c092-d865-4a19-b096-cc16440cd001); [GitHub OxWearables/capture24](https://github.com/OxWearables/capture24) | M4 (sleep, sitting, walking, free-living artefact model) | Wrist, not torso/neck; UK adults | V |
| ExtraSensory (UCSD) | Phone accel, gyro, magnetometer, location, audio MFCC, phone state; **watch accel + compass**; context labels incl. **TOILET (2,655 examples / 43 users)**, **shower (2,087 / 37)**, **sleeping (83,055 / 53)**, **lying down (104,210 / 58)**, at home (152,892 / 57), **in a car (6,083 / 33)**, **on a bus (1,794 / 31)** | 60 users (34 iPhone, 26 Android), 300 k+ one-minute examples; features + labels 215 MB; raw phone accel 6.1 GB, watch accel 800 MB | "Publicly available", free; **must cite original publication**; no formal license named on page | Open download | [extrasensory.ucsd.edu](http://extrasensory.ucsd.edu/) | M4 (toilet / bathing / lying / vehicle event detection), M2 (transport context) | Self-reported labels; students; minute-level | V |
| PAMAP2 (UCI) | 3 IMUs + heart-rate monitor, 18 activities | 9 subjects | **CC BY 4.0** | UCI | [UCI dataset 231](https://archive.ics.uci.edu/dataset/231) | M4 (posture/gait classes; IMU on chest/wrist/ankle) | Lab protocol, small n | V |
| WISDM Smartphone & Smartwatch Activity and Biometrics (UCI, 2019) | Phone + watch accelerometer and gyroscope, **20 Hz**; 18 activities × 3 min | 51 subjects; 15.6 M instances; 295 MB | **CC BY 4.0** | UCI | [UCI dataset 507](https://archive.ics.uci.edu/dataset/507/wisdm+smartphone+and+smartwatch+activity+and+biometrics+dataset) | M4 | Scripted activities | V |
| HHAR – Heterogeneity Activity Recognition (UCI) | Smartphone + smartwatch accel/gyro across devices; 6 activities | (multiple device models) | **CC BY 4.0** | UCI | [UCI dataset 344](https://archive.ics.uci.edu/dataset/344/heterogeneity+activity+recognition) | M4 (device heterogeneity: relevant to varied phones in TC states) | Scripted | V |

### Inferences
- M4 artefact model: train a small on-device classifier on **Capture-24 (free-living, sleep)** + **ExtraSensory (toilet, shower, lying, vehicle)**, with PAMAP2/WISDM/HHAR for robustness across devices. Use detected toilet visits (urine excretion → contamination/dose-rate step changes), lying/sleep (detector geometry changes; potential bed-partner proximity) and vehicle travel (transport scenario) as **context flags**, not as dose estimates.
- None of these datasets has a body-worn **radiation** detector. The link between posture and dosimeter reading still has to be measured or simulated (e.g. phantom geometry), which is a likely IRSA pilot-study item.
- **Cross-cutting starter kit (all confirmed open, mostly commercially usable):** M1 = Gambia 2022 social contact (CC BY 4.0) + POLYMOD/NL 70+/EPICURUS (check NC terms) + Levart 2025 and NRC SECY-18-0015 scenario numbers as calibration targets. M2 = Overture places/buildings + Google Open Buildings + LandScan/WorldPop (CC BY 4.0), with crowd APIs excluded. M3 = Swiss Dwellings (CC BY 4.0) + DHS hv216 crowding. M4 = Capture-24 (CC BY) + ExtraSensory + PAMAP2/WISDM/HHAR (CC BY 4.0). Datasets to avoid for any commercial path: POLYMOD (CC BY-NC), SocioPatterns bundle (CC BY-NC), CubiCasa5K (CC BY-NC-SA), ZInD (academic only), UKTUS/HETUS (restricted microdata), Google Maps content (no storage).

### Gaps
- Datasets with smart-home **toilet / bed-to-toilet** events (e.g. CASAS), sleep-accelerometry datasets on PhysioNet (e.g. Walch et al.), OPPORTUNITY, UK Biobank / NHANES accelerometry: **not verified** this session (UK Biobank is known to be restricted access).
- No open dataset with older-adult or cancer-patient free-living wearable data plus toilet labels was confirmed.
- ExtraSensory has no formal license, so redistribution terms beyond "free with citation" are unclear.
