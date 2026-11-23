# IRSA supplementary search: in-home behaviour, proximity and household-contact datasets and simulators (checked 2026-11-23)

Scope: new items not in `IRSA 前期开发与公开数据集/context_behaviour_datasets.md` (2026-11-20), plus licence gaps that file left open. Items already catalogued there (POLYMOD, Zenodo contact community, SocioPatterns Zenodo bundle, Copenhagen, NIST TC4TL/MITRE, MIT PACT, ATUS/IPUMS/HETUS/UKTUS, Overture, Foursquare, building footprints, population grids, Mobility Database, Swiss Dwellings, CubiCasa5K, ZInD, DHS hv216, Capture-24, ExtraSensory, PAMAP2, WISDM, HHAR) are not repeated except where this round resolved their licence.

IRSA module codes (same as the earlier file, plus M6):
- **M1** Contact prior (who is near the patient, how close, how long)
- **M3** Home geometry (rooms, floor plans, bed/toilet positions)
- **M4** Activity / artefact detection (toilet, shower, sleep, lying: events that shift a body-worn detector's reading)
- **M5** Calibration / validation targets (measured caregiver–patient distance and duration)
- **M6** Scenario simulation engine (synthetic household + schedule + positions → distance-time traces)

Verification legend: **V** = licence/size confirmed on the primary page or repository API in this session; **P** = existence confirmed, licence or key details not; **U** = surfaced only in search summaries, primary page not opened.

How licences were checked: Zenodo REST API (`zenodo.org/api/records/<id>`), Figshare API, raw `LICENSE` files on GitHub, Hugging Face dataset API, PhysioNet pages, sociopatterns.org dataset pages, and the Netzschleuder catalogue's "upstream license" field.

---

## 1. Smart-home activity datasets with bathroom / toilet / shower / sleep information

### Takeaway
The strongest new open resource is the **CASAS release on Zenodo (2025, CC BY 4.0)**: 189 community homes, 2007–2024, PIR and door sensors named by room (Bathroom, Bedroom, Bed), activity labels for some homes, and **floor plans**. It gives toilet and bed-occupancy timing priors and home layouts, but it is ambient, not wearable. **SPHERE house scripted v2.0 (CC BY 4.0)** adds a wrist accelerometer and BLE RSSI room location. **TIHM (Zenodo CC BY 4.0, 56 people with dementia)** adds an under-mattress sleep mat (in/out of bed), though one source says its terms bar commercial use. Orange4Home has explicit "Using the toilet" and "Showering" labels but is request-only with no licence named. ARAS, UCAmI, MARBLE and MuRAL have no stated licence; Kasteren's original host was not found; "HomeSense" was not found at all.

### Cited Findings

**Table 1a: Smart-home / ambient datasets**

| Name | Contents | Size / coverage | Licence / commercial use | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| CASAS free-living (WSU), "motion, door, activity labels" | PIR motion + magnetic door sensors, ceiling-mounted, named by room (Bathroom, Bedroom, Bed; some "Area" whole-room sensors); one file per home; activity labels from an external annotator for **some** homes; **floorplans.zip** | **189 homes, 2007–2024**; 3.0 GB (data.zip 2.7 GB, labeled_data.zip 236 MB, floorplans.zip 15.1 MB) | **CC BY 4.0** (commercial OK with attribution) | Open download | [Zenodo 15708568](https://zenodo.org/records/15708568); [Sandia RAIICE listing, 2026-09-29](https://www.sandia.gov/raiice/2026/09/29/casas-smart-home-dataset-free-living-motion-door-activity-labels/) | M4 (bathroom/bed occupancy timing, night toilet visits), M3 (floor plans), M1 (time in shared rooms, multi-resident homes) | Label names (e.g. bed-to-toilet) **not listed on the Zenodo page**; ambient sensors cannot say who triggered an event in multi-resident homes; US homes | V |
| CASAS raw (49 homes) | Unannotated ambient PIR/door data, sensor IDs by room | 49 homes; rawdata.zip 1.5 GB | **CC BY 4.0** | Open | [Zenodo 16343844](https://zenodo.org/records/16343844) | M4 | No labels | V |
| CASAS Aruba / Cairo / Milan / Tulum (labelled) | PIR, door, ambient temperature; annotated activities | 4 homes; 33 MB | **CC BY 4.0** | Open | [Zenodo 17180309](https://zenodo.org/records/17180309) | M4 | Label list and resident count not shown on page | V |
| CASAS scripted multi-resident | Scripted ADLs "performed separately and collaboratively by multiple residents" | Smart-apartment testbed; published 2025-06-22 | **CC BY 4.0** | Open | [Zenodo 15713378](https://zenodo.org/records/15713378) | M1 (two people in one apartment), M4 | Scripted, lab apartment | V |
| CASAS scripted "separated and interweaved" / "with and without activity errors" | Scripted ADL/iADL tasks | Published 2025-06-21 | **CC BY 4.0** | Open | [Zenodo 15713190](https://zenodo.org/records/15713190); [Zenodo 15712834](https://zenodo.org/records/15712834) | M4 | Scripted tasks (medication, plant watering), little bathroom content | V |
| SPHERE house scripted v2.0 (Bristol) | Wrist tri-axial accelerometer streamed over BLE to in-house receivers; **RSSI per receiver (indoor localisation)**; environmental and video-derived data; files include `acceleration.csv`, `rssi.csv` | v2.0 deposited 4 Nov 2022 with "extensive additional data" | **CC BY 4.0** | data.bris (returned HTTP 503 this session) | [data.bris DOI 10.5523/bris.2h0wyctxrd69j2oqccsi45hy1p](https://data.bris.ac.uk/data/dataset/2h0wyctxrd69j2oqccsi45hy1p); [Bristol research portal](https://research-information.bris.ac.uk/en/datasets/sphere-house-scripted-dataset-a-multi-sensor-dataset-with-annotat-2/); [2016 dataset paper](https://arxiv.org/html/1603.00797) | M4 (wrist accel + room location), M1 (room co-location from RSSI) | Scripted in one house; activity list not re-read this session | P (licence V via portal/search; page 503) |
| SPHERE 100 Homes study | Multi-year deployment; guide covers wearable localisation (RSSI) data | System ran in ~50 homes, each up to a year (per 2018 guide) | **Access route not found** | — | [Guide arXiv 1805.11907](https://arxiv.org/abs/1805.11907v2) | M4, M1 | Release status unclear | P |
| TIHM (Surrey, Sci Data 2023) | PIR + door sensors per room; **under-mattress sleep mat (sleep, in/out of bed)**, 1-min sampling; physiology (BP, HR, temperature, weight, hydration); labelled health events | **56 people with dementia**, mean 50 days each, 2,803 days; Zenodo files 6 MB | Zenodo API: **CC BY 4.0**; a search summary of the same record says terms limit use to research and patient benefit and **bar commercial use**, with acknowledgement of Surrey and Borders NHS Trust | Open (Zenodo + GitHub notebooks) | [Zenodo 7622128](https://zenodo.org/records/7622128); [paper PMC10492790](https://pmc.ncbi.nlm.nih.gov/articles/PMC10492790) | M4 (bed exits, night-time activity in older adults), M5 | **Licence conflict**: read the Zenodo description before commercial use | V (licence field) / conflict flagged |
| Smart Home environment data across 4 European countries | PIR, door contact, temperature, humidity, light, CO2, VOC, sound (2–5 min); rooms incl. bathroom, bedroom, kitchen, living room | **62 users aged 65+ living alone** (DK, PT, RO, CH); 16 May 2023–8 May 2024 (359 days); 516 MB | **CC BY 4.0** | Open | [Zenodo 14243470](https://zenodo.org/records/14243470); [paper PMC12145531](https://pmc.ncbi.nlm.nih.gov/articles/PMC12145531/) | M4 (bathroom-visit timing priors for older adults living alone) | **No activity labels** (PIR used as proxy); no PIR/door data for Switzerland | V |
| Orange4Home (Orange Labs / Inria Amiqual4Home) | 236 data sources; 20 activity classes, 493 instances labelled in situ, incl. **"Using the toilet"**, **"Showering"**, **"Napping"** | 1 occupant, ~180 h, 30 Jan–24 Feb 2017 | **No licence named**; must cite Cumin et al. UCAmI 2017 | **Email request** (MySQL dump) | [amiqual4home.inria.fr/orange4home](https://amiqual4home.inria.fr/orange4home/); [HAL](https://hal.archives-ouvertes.fr/hal-01639673) | M4 | Single person; redistribution terms unknown | V (page) / licence absent |
| ARAS (Boğaziçi Univ.) | Ambient sensors with ground-truth activity labels | 2 houses, multiple residents, 1 month per house | **Not confirmed** (site HTTP 503) | cmpe.boun.edu.tr/aras | [ARAS page](https://www.cmpe.boun.edu.tr/aras); [paper](https://eudl.eu/doi/10.4108/icst.pervasivehealth.2013.252120) | M4, M1 (2 residents) | Licence unknown | P |
| MARBLE (Univ. Milan) | Smartwatch inertial + magnetic door/drawer sensors, pressure mats on chairs/sofas, plug sensors; ground truth links each sensor event to the resident | 12 subjects, up to 4 residents at once, ~16 h, 13 ADLs (cooking, eating, TV, medicines…; **no toilet/sleep**) | **No licence on page**; cite Arrotta et al. 2022 | Open | [EveryWare Lab page](https://everywarelab.di.unimi.it/index.php/research/datasets/220-marble-dataset-multi-inhabitant-activities-of-daily-living-combining-wearable-and-environmental-sensors-data); [DOI 10.13130/RD_UNIMI/VGLD0Y](https://doi.org/10.13130/RD_UNIMI/VGLD0Y) | M1 (who-sits-on-which-sofa, multi-resident attribution) | Scripted; no bathroom labels | V (page) / licence absent |
| MuRAL (Grenoble, 2025–2026) | Multi-resident ambient data with resident identities, natural-language activity annotations | 1 smart home, 21+ h, 21 sessions; arXiv v1 Apr 2025, v2 Mar 2026 | **Not stated** | mural.imag.fr | [arXiv 2504.20505](https://arxiv.org/abs/2504.20505) | M1, M4 | Small; licence unknown | P |
| UCAmI Cup 2018 (UJAmI SmartLab, Jaén) | Binary sensors, proximity, acceleration; 24 activity classes, 246 instances | 1 inhabitant | **Not stated**; access by email after registration | Request | [UJAmI repository](https://ceatic.ujaen.es/ujami/en/repository); [Cup paper](https://www.mdpi.com/2504-3900/2/19/1267) | M4 | Licence unknown | P |
| Kasteren (UvA, 2008) / "Synthetic Smart Home Dataset" | Original: House-B labels incl. Sleeping, Showering, Toileting; synthetic derivative: Markov-model daily activity sequences | Synthetic: 1 MB | Original: **host/licence not found**; synthetic: **CC BY 4.0** | Synthetic open | [Zenodo 12686003](https://zenodo.org/records/12686003); [label table, Sensors 2020](https://pmc.ncbi.nlm.nih.gov/articles/PMC7601290/table/sensors-20-05770-t001) | M6 (activity-sequence generator seed), M4 | Synthetic file is a work in progress | V (synthetic) / P (original) |
| UCI "HAR from Continuous Ambient Sensor Data" (id 506) | Ambient PIR, door/temperature, light-switch sensors in volunteers' homes | ~13.9 M instances | Not checked (UCI usually CC BY 4.0) | UCI | [UCI 506](https://archive.ics.uci.edu/dataset/506/human+activity+recognition+from+continuous+ambient+sensor+d) | M4 | Licence not verified | U |
| AgeInPlace-Synth v1; AgeSafe-SynthHome | Fully synthetic smart-home sensor data for older adults (calibrated longitudinal; anomaly) | — | Not checked (IEEE DataPort) | IEEE DataPort | [AgeInPlace-Synth](https://ieee-dataport.org/documents/ageinplace-synth-v1-calibrated-longitudinal-smart-home-sensor-dataset-older-adults-living); [AgeSafe-SynthHome](https://ieee-dataport.org/documents/agesafe-synthhome-fully-synthetic-smart-home-activity-anomaly-dataset) | M6 | Synthetic; DataPort may need account | U |
| Multimodal older-adult homecare dataset (Preprints.org, 2026) | Motion, door, environmental, wearable, under-mattress sleep | 10 participants, ~1 month | Not confirmed; not peer reviewed | — | [Preprints 202609.0112](https://www.preprints.org/manuscript/202609.0112) | M4 | Unverified | U |

**Table 1b: Wearable sleep datasets (complement to Capture-24)**

| Name | Contents | Licence | Access | Link | IRSA module | Verif. |
|---|---|---|---|---|---|---|
| Walch et al. "Motion and heart rate from a wrist-worn wearable and labeled sleep from polysomnography" | Apple Watch raw acceleration + PPG heart rate, PSG sleep labels | **Open Data Commons Attribution v1.0** (anyone may access) | Open | [PhysioNet sleep-accel 1.0.0](https://physionet.org/content/sleep-accel/1.0.0/) | M4 (sleep/lying onset from wrist motion) | V |
| DREAMT | Empatica E4 wrist signals + PSG, expert sleep stages, clinical metadata (v2.2.0) | **PhysioNet Restricted Health Data License 1.5.0** + signed DUA | Credentialed | [PhysioNet DREAMT](https://physionet.org/content/dreamt/) | M4 | V |

### Inferences
- For **toilet-visit and night-time bathroom priors** (when a patient leaves the bed and voids, which matters for both contamination and detector artefacts), the CASAS 189-home release and the 62-home European older-adult set are both CC BY 4.0 and large enough to estimate per-night bathroom-visit counts and times. They are ambient, so they serve as priors, not as training data for a phone/wearable classifier.
- For a **wearable classifier**, the only new open sets pairing body-worn motion with room context are SPHERE scripted (wrist accel + RSSI room) and the Walch sleep set. ExtraSensory (from the earlier file) remains the only open wearable set with explicit toilet/shower labels.
- CASAS `floorplans.zip` gives US home layouts with sensor positions, a second M3 source next to Swiss Dwellings.

### Gaps
- CASAS activity label vocabulary (e.g. whether "Bed_Toilet_Transition" or "Personal_Hygiene" are present in the 2025 Zenodo files) was not visible on the Zenodo pages.
- ARAS, Orange4Home, MARBLE, MuRAL, UCAmI and the original Kasteren data have **no formal licence** found; redistribution and commercial terms need author confirmation.
- A dataset named **"HomeSense" was not found**; it may be a different spelling or an unindexed paper.
- SPHERE 100 Homes data access and the DOMUS multi-resident dataset (toilet-lid sensor, surfaced only in a search summary) were not verified.
- No open smart-home dataset with a body-worn **radiation** detector exists in what this round found.

---

## 2. In-home and close-range interpersonal proximity data (measured distance or calibrated threshold)

### Takeaway
SocioPatterns now hosts **three African household datasets**: **South Africa SA-S-HTS (88 households, CC0)**, **rural Kenya (5 households, CC0)** and **rural Malawi (86 people / 28 households, CC BY-NC-SA)**. The PHIRST South Africa set (60 households, 307 people, 3 waves of 10–14 days) has data on GitHub but **no licence file**. All of these use a **~1–1.5 m attenuation threshold with 20-s resolution**, not a continuous distance. The only datasets found with **continuous measured distance between family members at home** come from UWB wearables (**TotTag**, 1 Hz ranging, child–caregiver), and their data licence is unclear. The new **Wedding UWB badge dataset (2026, CC BY 4.0)** is open but thresholded at 1.5 m and not a home.

### Cited Findings

**Table 2a: Household proximity-sensor datasets (LMIC and family settings)**

| Name | Contents | Size / coverage | Licence / commercial use | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| South African households (SA-S-HTS, NICD + SocioPatterns) | Close-range contacts between household members; a contact = consecutive 20-s intervals with ≥1 packet exchanged (within ~1.5 m) | **88 households**, Klerksdorp and Soweto; sensors for ~2 weeks per household, Nov 2020–Aug 2021; 70,271 edges | **CC0 (Creative Commons Public Domain Dedication)**; cite Kleynhans et al. eLife 2023 | Open (GitHub, linked from page) | [sociopatterns.org dataset page](https://sociopatterns.org/datasets/south-african-households-contact-networks/); [Netzschleuder sp_south_african_households](https://networks.skewed.de/net/sp_south_african_households); [eLife 10.7554/eLife.84753](https://doi.org/10.7554/eLife.84753) | M1 (within-household contact duration by age, LMIC urban) | Households followed after an adult SARS-CoV-2 index case, so behaviour may be altered by illness | V |
| Rural Kenya households (Kilifi HDSS) | Face-to-face proximity contacts, daytime 7:00–20:00; node age/sex; edge duration, day, hour | 5 households, 3 consecutive days each, 24 Apr–12 May 2012; network has **47 nodes**, 32,643 contact events | **CC0 1.0** (upstream licence per Netzschleuder); Figshare supplement (PDF) **CC BY 4.0** | Open | [Netzschleuder sp_kenyan_households](https://networks.skewed.de/net/sp_kenyan_households); [Figshare collection](https://figshare.com/collections/Quantifying_social_contacts_in_a_household_setting_of_rural_Kenya_using_wearable_proximity_sensors/3697636/1); [EPJ Data Sci 2016](https://epjdatascience.springeropen.com/track/pdf/10.1140/epjds/s13688-016-0084-2) | M1 | Figshare description mentions **75 individuals** vs 47 nodes in the network file (unresolved); no night-time | V (CC0 via catalogue) |
| Rural Malawi village (Mdoliro, Dowa) | Close-range proximity, attenuation threshold **−75 dBm (~1–1.5 m)**, 20-s intervals; sensors removed overnight; night-time and non-moving-sensor periods removed | **86 people from 28 households** (children, adolescents, adult caregivers), 16 Dec 2019–10 Jan 2020; 102,293 edges | **CC BY-NC-SA** (sociopatterns.org "Terms and Conditions"; Netzschleuder lists v3.0) → **non-commercial, share-alike** | Open download (contact list) | [sociopatterns.org Malawi page](https://sociopatterns.org/datasets/contact-patterns-in-a-village-in-rural-malawi/); [Netzschleuder sp_malawi_village](https://networks.skewed.de/net/sp_malawi_village) | M1 (LMIC within/between-household durations) | No night data (shared-bed time is missing) | V (**resolves earlier gap**) |
| PHIRST SocioPatterns (South Africa 2018) | Wearable proximity badges (pinned or lanyard), ~1.5 m, 20-s; household type, head gender, age/sex/employment per person; contact rates per wave | **60 households, 307 people** (28 rural Agincourt, 32 urban Klerksdorp); 3 waves (Feb, Apr, Jun), 10–14 days each | **No licence file** in repo (default: all rights reserved); paper PLOS ONE 21(3) e0344732 (2026) | GitHub `Data/` folder | [GitHub PHIRST_SocioPatterns](https://github.com/lorenzodallamico/PHIRST_SocioPatterns); [arXiv 2503.22239v4](https://arxiv.org/html/2503.22239v4); [PLOS ONE PDF](https://www.isi.it/wp-content/uploads/2026/03/journal.pone_.0344732.pdf) | M1 (household-type-specific priors, child caregiving, seasonality) | Ask authors for a licence before reuse; companion household matrices in `PHIRST_CM` repo (PLOS ONE 2024) | V (data present) / licence absent |
| Infant households, Italy (Ozella et al. 2018) | RFID face-to-face contacts, ~1–1.5 m; night (21:00–07:00) excluded | 16 households with infants <6 months; 55 people (16 infants, 4 siblings, 31 parents, 4 grandparents); 5,958 contact events; 2–5 days | **Data availability not found** | — | [PMC5991752](https://pmc.ncbi.nlm.nih.gov/articles/PMC5991752) | M1 (infant–mother contact share, grandparent visits) | Mother accounts for most infant contact | P |
| Hospital ward, Lyon (Vanhems et al. 2013) | 20-s face-to-face contacts, roles NUR/PAT/MED/ADM | 46 health-care workers, 29 patients, 6–10 Dec 2010, geriatric unit | **CC BY-NC-SA 3.0** (upstream licence per Netzschleuder) | Open | [Netzschleuder sp_hospital](https://networks.skewed.de/net/sp_hospital) | M1 (inpatient/clinic scenario) | Non-commercial | V (via catalogue) |
| i-Bird long-term care facility (Berck-sur-Mer, France) | RFID proximity among patients and staff | 200-bed facility; 329 patients, 261 staff; Jul–Oct 2009 | **Access not confirmed** (only summary tables on PLOS Figshare) | — | [PMC5786108](https://pmc.ncbi.nlm.nih.gov/articles/PMC5786108); [PLOS Figshare summary table](https://plos.figshare.com/articles/dataset/Summary_of_network_characteristics_for_the_observed_i-Bird_total_network_patient-patient_subgraph_staff-staff_subgraph_and_patient-staff_subgraph_/26034910) | M1 (nursing-home scenario) | Likely restricted | U |

Netzschleuder also lists these other SocioPatterns networks (not reviewed here): `sp_colocation`, `sp_high_school`, `sp_high_school_new`, `sp_hypertext`, `sp_infectious`, `sp_office`, `sp_office_2015`, `sp_primary_school`, `sp_sfhh_conference`, `sp_baboons` — [Netzschleuder API listing](https://networks.skewed.de/api/nets).

**Table 2b: Measured-distance (ranging) datasets and devices**

| Name | Contents | Size / coverage | Licence | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| TotTag (Vanderbilt / UCSD; SociTrack lineage) | **UWB time-of-flight ranging + BLE discovery**, 1 Hz, median of several ToF samples, ±3-s smoothing; ground-truth SD ~0.3–1.6 ft at 1/3/6 ft; indoor LOS 20–30 m; passes drywall and bodies | Case study: 2 families, 2 weekend days, 12-h windows; "touching distance" ≤3 ft; **no dyad spent >16% of the day within touching distance**. Feasibility preprint: 125 families, 7 days | Data + scripts in GitHub `vanderbiltsealab/tottag_methods` (**no licence file found**); article reuse "academic research for non-commercial purposes"; firmware `lab11/socitrack` **Apache 2.0** | GitHub | [Salo et al. 2022, PMC8907335](https://pmc.ncbi.nlm.nih.gov/articles/PMC8907335); [feasibility preprint](https://sciety.org/articles/activity/10.31234/osf.io/tejdg_v1); [lab11/socitrack](https://github.com/lab11/socitrack) | **M5** (only measured continuous in-home family distance found), M1 | Children, not patients; tiny public sample; participants removed tags for baths | V |
| SociTrack (Berkeley) | BLE + UWB wearable; pairwise distances among all wearers up to **16 Hz**, **14.8 cm 99th-percentile accuracy** | System paper | Firmware Apache 2.0 (above) | — | [Biri et al. SociTrack](https://lab11.eecs.berkeley.edu/content/pubs/biri20socitrack.pdf) | Hardware option for an IRSA companion-distance pilot | No public dataset found | P |
| Wedding Cocktail Hour Contact Webs (2026) | **Ultra-Wideband** badges detecting other badges within ~1.5 m every 5 s, coarsened to 10-s bins; relationship-group labels | 95 adults; 7,213 contact events, 2,760 dyads; files: node list, temporal edge list, build script | **CC BY 4.0** | Open | [Zenodo 20430824](https://zenodo.org/records/20430824); [arXiv 2605.30291](https://arxiv.org/abs/2605.30291) | M1 (family vs non-family contact durations at a gathering; "family event" scenario) | Thresholded, not continuous distance; outdoor social event | V |
| Residential UWB channel-impulse-response dataset (Bocus & Piechocki 2022) | Passive (device-free) UWB CIR for localisation and activity recognition in a home | ~1.6 h annotated | Not checked | Figshare | [PMC9587989](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9587989/); [Figshare c.6078021](https://doi.org/10.6084/m9.figshare.c.6078021.v1) | M4 (device-free room activity), M3 | No wearables, no inter-person distance | P |
| LocoVR (ICLR 2025) | **7,000+ two-person trajectories** with social motion behaviour, collected in VR | 130+ indoor home environments | Repo **MIT** licence | GitHub | [kt2024-hal/LocoVR](https://github.com/kt2024-hal/LocoVR); [arXiv 2410.06437](https://arxiv.org/abs/2410.06437v2) | M6 (two-person in-home distance traces), M3 | VR, not real homes; short goal-directed walks, not day-long routines | V |
| MI-Motion | 3D multi-person motion, 217 sequences (201 three-person, 16 six-person), 20 keypoints | General indoor scene | Not checked | — | [arXiv 2306.13566](https://arxiv.org/pdf/2306.13566) | M6 | Not a household | U |

### Inferences
- **Commercially usable LMIC household contact data now exists**: SA-S-HTS (88 households) and Kenya (5 households) are **CC0**. Malawi and the Lyon hospital ward are **non-commercial share-alike**, fine for the IAEA research prototype but not for a product. PHIRST has the richest household-type metadata but no licence.
- None of these give metres. All SocioPatterns household data effectively answer "how long within ~1.5 m, daytime". For IRSA, that maps to the "<1 m to 1.5 m" exposure bin; shared-bed time (night) is missing in Malawi, Kenya and the Italian infant study, so sleep exposure still has to come from assumptions (e.g. 0.1 m × 8 h used by Levart 2025, earlier file).
- The TotTag result that no parent–child dyad spent >16% of a 12-h day within ≤3 ft is a useful **upper-bound sanity check** for "caring for a small child" scenarios.
- A UWB wearable pair (SociTrack/TotTag class, Apache-2.0 firmware) is the most direct way for an IRSA pilot to **measure** companion distance, since BLE RSSI alone is not distance-calibrated.

### Gaps
- TotTag case-study data licence (no LICENSE file) and the 125-family feasibility data availability were not resolved.
- The Italian infant-household data, i-Bird data and PHIRST licences need author contact.
- No proximity dataset involving **adult patients with a serious illness and their household** was found.

---

## 3. Synthetic population / household generators and agent-based contact simulators

### Takeaway
For a reusable, commercially safe stack, the permissive options are **SPW synthetic populations (CC BY 4.0 data, households + daily activities + locations)**, **IDM SynthPops / Covasim / Starsim (MIT)**, **PopulationSim (BSD-3)**, **pyEpiabm (BSD-3)** and **Mesa (Apache 2.0)**. **OpenABM-Covid19, JUNE and CovidSim are GPL-3** (copyleft), **FRED v5 is non-commercial only**, and **RTI's 2010 US synthetic population is free for non-commercial use only**. None of the epidemic ABMs model **distance**; they model contacts as daily membership in household / workplace / community layers, so IRSA would have to add a distance and duration model on top.

### Cited Findings

**Table 3a: Synthetic population / household generators**

| Name | Contents | Coverage | Licence / commercial use | Access | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|---|
| SPW — Synthetic Populations for regions of the World (UVA Biocomplexity) | Synthetic people with demographics, **organised into households**, **assigned daily activities** and activity **locations** | 21 Zenodo records (v0.7), e.g. US states (New Mexico, Virginia, Washington, Florida, Alabama) and Indian states (Delhi, Manipur, Punjab); NSF RAPID-funded | **CC BY 4.0** | Open | [Zenodo community synpop_world](https://zenodo.org/communities/synpop_world); [example record 6505934](https://zenodo.org/records/6505934) | M6 (household composition + daily schedule) | "coarse" model; country coverage limited to the 21 records | V |
| SPEW (CMU) | R package generating synthetic ecosystems: agents in households, optionally schools/workplaces | Working paper: >3.5 B individuals, 50+ countries | **No licence file** found in `leerichardson/spew` | GitHub | [arXiv 1701.02383](https://arxiv.org/pdf/1701.02383); [CMU working paper](https://www.stat.cmu.edu/~brian/rm/764-2016/week06/papers/Unnamed.pdf) | M6 | Licence unclear; appears unmaintained (inference) | P |
| RTI U.S. Synthetic Household Population (2010) / RTI SynthPop | Synthetic US households and persons, geolocated | US national + state | **Free for non-commercial use** (contact RTI); commercial SynthPop under negotiated terms; RTI 2019-based dataset code is **CC BY-NC-SA 4.0** | Request / FRED mirror | [RTI page](https://www.rti.org/impact/rti-us-synthetic-household-populationtm); [RTI SynthPop](https://www.rti.org/focus-area/rti-synthpoptm); [FRED population files](https://fred.publichealth.pitt.edu/syn_pops) | M6 | US only; non-commercial | P |
| SynthPops (IDM) | Synthetic populations with household, school, workplace contact layers (Covasim companion) | Configurable | **MIT** (©2020–2021 IDM) | GitHub | [InstituteforDiseaseModeling/synthpops](https://github.com/InstituteforDiseaseModeling/synthpops) | M6 | Development appears to have stopped in 2021 (inference from licence years) | V |
| PopulationSim (ActivitySim) | Population synthesis by list balancing to control totals | Any region with census tables | **BSD-3-Clause** | GitHub | [ActivitySim/populationsim](https://github.com/ActivitySim/populationsim) | M6 | Needs seed microdata (e.g. IPUMS) | V |
| synthpop (R, CRAN) | Synthetic versions of sensitive microdata (disclosure control), not a spatial household generator | — | **GPL-2 \| GPL-3**; v1.9-3, published 2026-09-10 | CRAN | [CRAN synthpop](https://cran.r-project.org/web/packages/synthpop/) | M6 (synthesising patient-household survey microdata for sharing) | Different purpose from population synthesis | V |

**Table 3b: Agent-based contact / epidemic simulators**

| Name | Household / setting layers | Duration or distance? | Licence | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|---|
| Covasim (IDM) | Households, schools, workplaces, community | Contact counts per layer; no distance | **MIT** (©2020–2023 Gates Foundation) | [GitHub covasim](https://github.com/InstituteforDiseaseModeling/covasim) (layer list from a search summary; source page not opened) | M6 (layer structure) | COVID-specific | V (licence) / U (layers) |
| Starsim (successor framework) | Generic multi-disease ABM with network layers | No distance | **MIT** (©2023–2026) | [GitHub starsim](https://github.com/starsimhub/starsim) | M6 | — | V |
| OpenABM-Covid19 (Oxford BDI) | **Household network** (all members interact every day), **occupation network** (small-world, daily sample; primary/secondary school, workers, retired, elderly), **random network** (transport, shopping, gatherings; rebuilt daily) | Interactions per day; no distance | **GPL-3** | [GitHub](https://github.com/BDI-pathogens/OpenABM-Covid19); [network figure, PMC8328312](https://pmc.ncbi.nlm.nih.gov/articles/PMC8328312/figure/pcbi.1009146.g001) | M6 (household/transport layers map to IRSA scenarios) | Copyleft | V |
| JUNE (Durham/UCL) | Census-built virtual England (~53 M people assigned households); households, schools, workplaces, **care homes**, other activities with social mixing matrices | Time-step activity schedule; no distance | **GPL-3** | [GitHub IDAS-Durham/JUNE](https://github.com/IDAS-Durham/JUNE); [RSOS 2021](https://discovery-pp.ucl.ac.uk/10132387/1/rsos.210506.pdf) | M6 (care-home and household scenarios) | UK-specific inputs; copyleft | V |
| FRED v5 (Univ. Pittsburgh) | Synthetic-population ABM (uses RTI 2010 population) | No distance | **Pitt EULA: non-commercial, educational or research purposes only** | [GitHub PublicHealthDynamicsLab/FRED](https://github.com/PublicHealthDynamicsLab/FRED) | M6 | Restrictive | V |
| CovidSim (Imperial / MRC-IDE) | Households, schools, workplaces, spatial community | No distance | **GPL-3** | [GitHub mrc-ide/covid-sim](https://github.com/mrc-ide/covid-sim) | M6 | Copyleft; large C++ code base | V |
| pyEpiabm (Oxford SABS) | Python re-implementation of CovidSim-style household/place model | No distance | **BSD-3-Clause** | [GitHub epiabm](https://github.com/SABS-R3-Epidemiology/epiabm) | M6 | — | V |
| Mesa | General Python ABM framework with grid/continuous space (can carry x,y positions) | Distance possible if modelled | **Apache 2.0** | [GitHub projectmesa/mesa](https://github.com/projectmesa/mesa) | **M6** (build an in-home agent model with explicit positions) | Generic; IRSA must write the model | V |

**Table 3c: Home-activity simulators from the smart-home / HAR literature**

| Name | What it does | Licence | Link | IRSA module | Verif. |
|---|---|---|---|---|---|
| AgentSense (2025) | LLM-generated personas → daily routines executed in VirtualHome; 250 days of virtual sensor data across 22 home layouts | Not checked | [arXiv 2506.11773](https://arxiv.org/pdf/2506.11773v1) | M6, M4 (synthetic training data) | U |
| MoSen (2021) | Synthetic multi-occupancy (2–5 residents) home behaviour built from real single-occupancy traces and floor plans | Not checked | [ar5iv 2101.00235](https://ar5iv.labs.arxiv.org/html/2101.00235) | M6 | U |
| Procedural home human-activity generator (2023) | Template/schedule-driven daily human movement in home layouts | Not checked | [arXiv 2311.15400](https://arxiv.org/abs/2311.15400v1) | M6 | U |
| Patterns-of-Life simulation (2026) | City-scale OSM-based agent mobility calibrated to trip counts and radius of gyration | Not checked | [arXiv 2601.01219](https://arxiv.org/pdf/2601.01219v2) | M6 (commuting / out-of-home scenarios) | U |

### Inferences
- A pragmatic M6 engine: **SPW or SynthPops** households → **ATUS-style schedules** (earlier file) → room assignment on **CASAS / Swiss Dwellings** floor plans → **Mesa** agents with positions → distance-time traces per household member. Covasim/Starsim (MIT) can supply the out-of-home layers (work, school, community, transport) without copyleft obligations.
- GPL-3 tools (OpenABM, JUNE, CovidSim) are fine for internal research runs, but linking their code into a distributed IRSA app would trigger copyleft; FRED and RTI data block commercial use.
- Epidemic ABMs treat every household member as "in contact every day". IRSA needs more resolution (who sleeps with whom, hours at <1 m), which no ABM here provides; this is where the SocioPatterns household durations (Section 2) and Levart/NRC values (earlier file) must calibrate the model.

### Gaps
- "Hypothesis/EpiSim" and "Pathogen ABMs" named in the brief could not be identified as specific tools; ASU PopGen and a named Eurostat-based household generator were **not found or checked** this round.
- Whether SPEW outputs are still downloadable, and their licence, were not confirmed.

---

## 4. Embodied multi-human home simulators and human-motion datasets

### Takeaway
Code for the main embodied simulators is permissive (**VirtualHome MIT, Habitat-lab/sim MIT, BEHAVIOR-1K/OmniGibson MIT, iGibson MIT, ProcTHOR and AI2-THOR Apache 2.0**), but the **human assets and motion data are non-commercial**: Habitat humanoid avatars (CC BY-NC / NC-SA, conflicting), HSSD scenes (CC BY-NC 4.0), BEHAVIOR dataset (separate licence, encrypted), AMASS (non-commercial research), BABEL (academic only), HumanML3D (derived from AMASS). VirtualHome is the only one built around **multi-agent household activity programs**; LocoVR (MIT) is the only open dataset of **two-person trajectories inside home layouts**. None of them outputs day-long household distance-time traces out of the box.

### Cited Findings

| Name | Multi-human in homes? | Licence (code / assets) | Link | IRSA module | Caveats | Verif. |
|---|---|---|---|---|---|---|
| VirtualHome (MIT/Toronto) | Yes: "Multi-Agent Household Simulator"; activities written as programs; Unity executable | Code **MIT** (©2022 MIT); paper record CC BY-NC-SA 4.0 (paper only) | [GitHub xavierpuigf/virtualhome](https://github.com/xavierpuigf/virtualhome); [paper arXiv 1806.07011](https://arxiv.org/abs/1806.07011); [DSpace record](https://dspace.mit.edu/handle/1721.1/118051) | M6 (scripted daily routines of 2 agents → positions over time) | Unity build licence for assets not confirmed; some actions unsupported in Unity | V (code) |
| Habitat 3.0 (Meta) | Humanoid avatars + robots in homes (social navigation, rearrangement) | habitat-lab and habitat-sim **MIT**; **habitat_humanoids**: YAML card says **CC BY-NC-SA 4.0**, body text says avatars **CC BY-NC 4.0**, motion files under SMPL Body Motion File Licence (CC BY 4.0; commercial via Meshcapade); **HSSD scenes CC BY-NC 4.0** | [habitat-lab](https://github.com/facebookresearch/habitat-lab); [habitat-sim](https://github.com/facebookresearch/habitat-sim); [HF habitat_humanoids](https://huggingface.co/datasets/ai-habitat/habitat_humanoids); [HF hssd-hab](https://huggingface.co/datasets/hssd/hssd-hab); [arXiv 2310.13724](https://arxiv.org/abs/2310.13724) | M6, M3 | Avatars/scenes non-commercial; licence conflict on humanoids | V |
| BEHAVIOR-1K / OmniGibson (Stanford) | Household activities (1,000 tasks) for robots | Code **MIT** (©2023); dataset under a separate "BEHAVIOR Data Bundle License Agreement", **encrypted** because most assets come from ShapeNet/TurboSquid | [GitHub BEHAVIOR-1K](https://github.com/StanfordVL/BEHAVIOR-1K); [docs: important concepts](https://behavior.stanford.edu/getting_started/important_concepts.html) | M3 (furnished home scenes) | Requires NVIDIA Isaac Sim; asset licence text not read | P |
| iGibson (Stanford) | Interactive home scenes | **MIT** (©2017) | [GitHub iGibson](https://github.com/StanfordVL/iGibson) | M3 | Superseded by OmniGibson (inference) | V |
| ProcTHOR / AI2-THOR (AI2) | Procedurally generated houses; agent navigation | **Apache 2.0** (both) | [allenai/procthor](https://github.com/allenai/procthor); [allenai/ai2thor](https://github.com/allenai/ai2thor) | M3 (procedural floor plans with furniture), M6 | Human-avatar support not confirmed this session | V |
| AMASS (MPI) | Large mocap archive (SMPL) | "Dataset Copyright License for **non-commercial scientific research** purposes" | [nghorbani/amass LICENSE](https://github.com/nghorbani/amass) | M4 (synthetic IMU from mocap: posture/lying artefact simulation) | Non-commercial | V |
| HumanML3D | 14,616 motions, 44,970 text descriptions, 28.59 h; from HumanAct12 + AMASS | Repo **MIT**; underlying motions inherit AMASS/HumanAct12 terms (inference) | [GitHub HumanML3D](https://github.com/EricGuo5513/HumanML3D) | M4, M6 (text-to-motion for "lying on sofa", "sitting on toilet") | Single-person clips | V |
| BABEL (MPI) | Frame-level action labels on AMASS | **Academic research only**; PS:License 1.0 (2021-06-18); needs AMASS separately | [babel.is.tue.mpg.de](https://babel.is.tue.mpg.de/); [MPI code page](https://is.mpg.de/en/code/babel-bodies-action-and-behavior-with-english-labels) | M4 | Non-commercial | P |
| LocoVR | Two-person trajectories in 130+ home layouts (see Table 2b) | **MIT** | [kt2024-hal/LocoVR](https://github.com/kt2024-hal/LocoVR) | M6 | VR-collected | V |

### Inferences
- These simulators **can** produce inter-person distance-time traces if IRSA scripts two agents' daily programs (VirtualHome) or places humanoids in scenes (Habitat 3.0), then logs agent positions. This is a build task, not a ready dataset, and the realism of durations would still come from time-use and contact data.
- For a commercial path, prefer VirtualHome code + ProcTHOR/AI2-THOR houses (Apache/MIT) and avoid HSSD, Habitat humanoids, BEHAVIOR assets, AMASS/BABEL-derived data.
- AMASS-type mocap is more useful for **M4** (simulate how lying, bending and sitting change detector-to-source geometry and generate synthetic IMU signals) than for contact priors.

### Gaps
- VirtualHome Unity-asset licence and AI2-THOR humanoid support were not confirmed.
- No embodied dataset with **day-long, multi-person household routines** (as opposed to short tasks) was found.

---

## 5. Licence resolution for items flagged unverified in the earlier round

### Takeaway
Most open gaps are now closed. **CoMix (UK, BE, 2.0, last round) = CC BY 4.0**; **Zimbabwe contact survey = CC BY-NC-ND 4.0** (more restrictive than expected); **Prem 2021 matrices: Zenodo "other-open", GitHub GPL-3**; **MIT LL PACT datasets = CC BY 4.0 but MITRE Range-Angle (TC4TL training data) = CC BY-NC 4.0**; **Copenhagen Networks Study = MIT licence**; **Malawi village = CC BY-NC-SA**; **Meta HRSL = CC BY 4.0**; **OSM = ODbL** confirmed; **Google Places: lat/lng cacheable only 30 days, and Maps Platform Terms prohibit use for HIPAA-regulated health information and "High Risk Activities"**.

### Cited Findings

| Item | Resolved licence / terms | Detail | Source | Verif. |
|---|---|---|---|---|
| CoMix UK | **CC BY 4.0** | Published 2024-09-05; 214 MB | [Zenodo 13684044](https://zenodo.org/records/13684044) | V |
| CoMix Belgium | **CC BY 4.0** | 34 MB | [Zenodo 10549953](https://zenodo.org/records/10549953) | V |
| CoMix 2.0 | **CC BY 4.0** | 2022-08-22; 85 MB | [Zenodo 7331926](https://zenodo.org/records/7331926) | V |
| CoMix last round (BE, CH, NL, UK) | **CC BY 4.0** | 2024-05-08; 12 MB | [Zenodo 11154066](https://zenodo.org/records/11154066) | V |
| Zimbabwe social contact data (Manicaland) | **CC BY-NC-ND 4.0** (non-commercial, **no derivatives**) | 3 MB | [Zenodo 3886638](https://zenodo.org/records/3886638) | V |
| Older adults 70+ NL 2021 | **CC BY 4.0** | 2 MB | [Zenodo 7751724](https://zenodo.org/records/7751724) | V |
| EPICURUS (older individuals) | **CC BY 4.0** | Published 2026-05-18; 10 MB | [Zenodo 20271335](https://zenodo.org/records/20271335) | V |
| Household contact survey Belgium | **CC BY 4.0** | 2 MB | [Zenodo 7965594](https://zenodo.org/records/7965594) | V |
| Zambia & South Africa (CODA) | **CC BY 4.0** | 3 MB | [Zenodo 3874675](https://zenodo.org/records/3874675) | V |
| SocioPatterns Zenodo bundle | **CC BY-NC 4.0** (re-confirmed) | 66 MB | [Zenodo 1117884](https://zenodo.org/records/1117884) | V |
| Prem et al. 2021 synthetic contact matrices | Zenodo v2.0 (2023-09-27): licence id **"other-open"**; GitHub repo **GPL-3.0** | Archive `synthetic-contact-matrices-v2.0.zip` | [Zenodo 4889499](https://zenodo.org/records/4889499); [GitHub LICENSE](https://github.com/kieshaprem/synthetic-contact-matrices) | V |
| Prem et al. 2017 matrices | **Not verified** this session | — | — | U |
| MIT PACT: BLE-RSSI Various Static Configurations, H0H1, KRI Anechoic Chamber, MIT Matrix, Range_Angle | **CC BY 4.0** (©2020 MIT) + "Distribution Statement A: approved for public release, distribution unlimited" | — | [mitll/MIT-Matrix-Data](https://github.com/mitll/MIT-Matrix-Data); [mitll/H0H1](https://github.com/mitll/H0H1); [PACT datasets page](https://mitll.github.io/PACT/datasets.html) | V |
| MIT PACT: Exposure-Notification Beacons | Distribution Statement A only (no LICENSE.md found) | — | [mitll/PACT-Exposure-Notification-Beacons](https://github.com/mitll/PACT-Exposure-Notification-Beacons) | P |
| MITRE Range-Angle Structured / Unstructured | **CC BY-NC 4.0** (©2020 MITRE, PR_20-01034-1) | Structured set = NIST TC4TL training data | [MITRE-Range-Angle-Structured](https://github.com/mkrangle/MITRE-Range-Angle-Structured); [Unstructured](https://github.com/mkrangle/MITRE-Range-Angle-Unstructured) | V |
| Copenhagen Networks Study interaction data | **MIT licence** (Figshare) | Files: `bt_symmetric.csv` (98 MB), calls, sms, fb_friends, genders, notebook | [Figshare 7267433](https://figshare.com/articles/dataset/The_Copenhagen_Networks_Study_interaction_data/7267433) (via Figshare API) | V |
| Rural Malawi (Ozella 2021) | **CC BY-NC-SA** | See Table 2a | [sociopatterns.org](https://sociopatterns.org/datasets/contact-patterns-in-a-village-in-rural-malawi/) | V |
| Meta / Data for Good High Resolution Settlement Layer | **CC BY 4.0**; 1-arcsecond blocks; updated quarterly; managed by Meta; "a selection of countries" | No deprecation notice | [AWS Open Data registry](https://registry.opendata.aws/dataforgood-fb-hrsl/) | V |
| OpenStreetMap | **ODbL** (OSM Foundation); documentation **CC BY-SA 2.0**; must credit OpenStreetMap and state ODbL; derived databases distributed only under the same licence | — | [openstreetmap.org/copyright](https://www.openstreetmap.org/copyright) | V |
| Google Maps Platform Service Specific Terms (Places API) | §14.3: "Customer may temporarily cache latitude and longitude values from the Places API for up to 30 consecutive calendar days, after which Customer must delete the cached latitude and longitude values." General term 3: `place_id` may be cached. §14.2: no use of Places content with a non-Google map. Page shows "Last modified October 28, 2026" | — | [Maps Service Specific Terms](https://cloud.google.com/maps-platform/terms/maps-service-terms) | V |
| Google Maps Platform Terms of Service | "No Caching" except as permitted; "No Use With Non-Google Maps"; must not use the Services "for High Risk Activities" or "**to transmit, store, or process health information subject to United States HIPAA regulations**"; page shows "Last modified August 26, 2026" | — | [Google Maps Platform Terms](https://cloud.google.com/maps-platform/terms) | V |

### Inferences
- Revised commercial-safe M1 set: Gambia 2022, CODA Zambia/SA, NL 70+, EPICURUS, Belgian household survey, **all CoMix records checked** (CC BY 4.0), SA-S-HTS and Kenya SocioPatterns (CC0). Non-commercial only: POLYMOD, SocioPatterns bundle, Malawi, Lyon hospital, **Zimbabwe (also no derivatives, so even derived matrices cannot be shared)**.
- For BLE distance work, the MIT LL sets (CC BY 4.0) are usable commercially, but the MITRE Range-Angle sets that trained the TC4TL challenge are **non-commercial**.
- Prem 2021 matrices: the code is GPL-3; the Zenodo "other-open" label is ambiguous for the matrix files themselves. Treat redistribution of modified matrices with care until the authors clarify.
- The Google Maps HIPAA/high-risk clauses reinforce the earlier decision to keep IRSA's location context on open data (Overture/OSM/Foursquare), since IRSA processes patient health information.

### Gaps
- Remaining CoMix country records (NL, AT, DK, ES, FR, IT, PT, PL, GR, SI, HR, SK, HU, EE, LT, FI, CH) were not checked individually; the four checked are all CC BY 4.0.
- Prem 2017 (PLOS Comp Biol) supplementary-matrix licence not verified.
- Google's terms are US-centred (HIPAA); whether equivalent clauses cover non-US health data was not assessed.

---

## 6. Studies measuring actual caregiver–patient distance or time together at home with sensors

### Takeaway
Only one study found measured **continuous in-home distance between an ill person and a family caregiver**: the **UC Berkeley dementia study** (27 dyads, BLE watches + 3 plug-in receivers, second-by-second, 6 months). It reports change in distance and its link to loneliness, not absolute metres or hours, and the data are not public. Oregon's EVALUATE-AD work logs **time together in the same room** from wall PIR sensors (single case published). Cancer-caregiver sensing studies (BESI-C, smartphone GPS) did **not** measure proximity. No radiopharmaceutical-therapy study with sensor-measured caregiver distance was found.

### Cited Findings
- **Chen K-H, …, Levenson RW (UC Berkeley), Innovation in Aging 2022;6(Suppl 1):745**: 27 people with dementia or MCI and co-resident family caregivers; both wore BLE watches (battery up to 4 months); 3 plug-in Bluetooth receivers per home; **distance measured second by second for 6 months**; caregiver–care-recipient distance increased from the first to the last 3 months (t = 20.67, p < 0.001); larger increases correlated with larger increases in caregiver loneliness (r = 0.46, p = 0.03). Absolute distance and time-near values not reported; data availability not stated — [PMC9771049](https://pmc.ncbi.nlm.nih.gov/articles/PMC9771049/). Press coverage notes BLE signal strength allowed room-to-room mapping and that more synchronised patient–caregiver activity went with lower caregiver anxiety — [Healthcare IT News](https://www.healthcareitnews.com/news/unique-wearable-helps-researchers-study-dementia-patients-and-familial-caregivers).
- **EVALUATE-AD / Tele-STAR case study (Dement Geriatr Cogn Disord 2019)**: one caregiver and spouse with Alzheimer's disease; wall-mounted infrared motion sensors give location and **time together in the same room**; door contact sensors give time out of home; Withings watch gives steps and sleep; caregiver reported 106 h/week of caregiving. Hours together per day not reported; no data-availability statement — [PMC8272983](https://pmc.ncbi.nlm.nih.gov/articles/PMC8272983).
- **TotTag (child–caregiver, UWB)**: see Table 2b; no dyad >16% of a 12-h day within ≤3 ft — [PMC8907335](https://pmc.ncbi.nlm.nih.gov/articles/PMC8907335).
- **BESI-C (cancer pain at home)**: in-home sensors plus smartwatches for patient–family-caregiver dyads; participants named physical closeness to others as a factor in pain, but proximity was not a measured output — [protocol PMC6928698](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6928698/). (A search summary reported 5 dyads and 283 pain events; the source page was not opened.)
- **Smartphone GPS in advanced cancer (JMIR Formative 2025)**: 7 family caregivers and 4 patients, 12 weeks, GPS mobility vs quality of life; no dyadic proximity — [JMIR Form Res 2025 e59161](https://formative.jmir.org/2025/1/e59161).
- **Care-home BLE (2013)**: wearable beacons every 10 s, proximity recognised when ≥9 of 10 packets received — [EUDL PervasiveHealth 2013](https://eudl.eu/pdf/10.4108/icst.pervasivehealth.2013.251943). A 2026 JMIR mHealth study combined accelerometers with Bluetooth location in a dementia long-term-care unit (residents, not home caregivers) — [JMIR mHealth 2026 e70188](https://mhealth.jmir.org/2026/1/e70188/PDF).

### Inferences
- The Berkeley design (BLE watches + a few plug-in receivers, room-level location for both people) is a **directly reusable IRSA pilot protocol**: it gives room co-location and coarse distance for patient and companion over weeks, at low burden. Adding UWB (TotTag/SociTrack class) would give sub-metre distance.
- Because no public dataset links measured distance to caregiving at home, IRSA's M5 calibration still rests on **Levart 2025 diaries** and NRC scenario values (earlier file), with SocioPatterns household durations and TotTag's ≤16% bound as cross-checks.

### Gaps
- Full peer-reviewed results (absolute distances, hours within 1 m) from the Berkeley study were not found; only the abstract.
- No sensor-based caregiver–patient distance study in oncology, nuclear medicine or after radionuclide therapy was found.
- Data from all caregiver sensing studies above appear to be unavailable publicly.
