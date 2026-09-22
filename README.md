# AgriTrust-ACFL — Code, Execution Record and Results

Reproducibility package for the manuscript **"AgriTrust-ACFL: Adaptive Client-Trust Federated Learning with Staleness-Aware Buffered Aggregation for Byzantine-Resilient Agricultural IoT Intrusion Detection"** (R. Augustian Isaac, submitted to the *Journal of Information Security and Applications*).

AgriTrust-ACFL is a trust-calibrated federated intrusion-detection aggregator for agricultural IoT. Each round it scores every farm gateway's own model update (directional consensus, norm plausibility, near-duplicate similarity, communication reliability and trust memory), down-weights stale updates, clips extreme updates, and quarantines clients below a drift-normalised, validation-calibrated trust threshold, with bounded exclusion so that no honest client is locked out.

The evidence in this package supports the following claims. It does **not** show that AgriTrust-ACFL is the most accurate method: Trimmed Mean, Coordinate Median and Multi-Krum lead on Macro-F1 in most scenarios (see *Limitations*).
- Attack detection with low honest false quarantine.
- Fairness across farms.
- Robustness under intermittent connectivity, competitive with classical robust aggregators.

---

## 1. Contents

| Item | Description |
|---|---|
| `AgriTrust_ACFL_Q1_v2_7_2_Colab.ipynb` | **Main notebook with all outputs.** Final execution record: every cell with its output, plus a results appendix displaying every table and figure inline. |
| `AgriTrust_ACFL_Q1_v2_7_Colab.ipynb` | Supplementary execution log of the original long FULL_REAL computation (progress lines of the multi-hour run). |
| `tables/` | 47 CSV tables (Table 00–23, 99), the source of every number in the manuscript. The manuscript cites them as supplementary tables: [Table S06] is `tables/Table_06_…csv`, [Table S17b] is `tables/Table_17b_…csv`, and so on. |
| `figures/` | 28 figures from the notebook, each as PNG and PDF. |
| `MANUSCRIPT_EVIDENCE_PACK.md`, `MANUSCRIPT_RESULT_SUMMARY.txt` | Evidence map from manuscript sections to tables and figures, and a summary of the headline results. |
| `checkpoints/e0cc45b388bf/` | Stage checkpoints (a few MB). They let anyone reload all results in minutes, without the ~9 h recomputation. |

**Note on checkpoints.** `unsw_federated.pkl` was computed on UNSW-NB15 files whose train/test names were reversed in the Kaggle mirror (Section 3.2). It has been **removed** from this deposit; the valid UNSW results are in `unsw_federated_official_orientation.pkl`.

---

## 2. Computing environment

| Item | Value |
|---|---|
| Platform | Google Colab (Pro), Google Drive for outputs and checkpoints |
| Accelerator used for all computation | NVIDIA Tesla T4 (14.6 GB) |
| Software (computation session) | Python 3.13.15; PyTorch 2.11.0+cu128; pandas, NumPy, scikit-learn, SciPy, matplotlib (Colab defaults, no extra installs) |
| Measured speed | 2.69 s per federated round (CIC-IoT-2023, 10 clients); about 8–12 s per round on UNSW-NB15 |
| Total compute | About 9 h for FULL_REAL, spread across stage checkpoints (Section 5) |

The final presentation run (`v2_7_2`) happened to execute on a **CPU** runtime (the notebook prints a CPU warning). This does not affect any result: every stage was loaded from checkpoints computed on the T4, and 0 federated runs were recomputed.

---

## 3. Data

The raw datasets are **not redistributed** in this package. Obtain them from the sources below. The notebook can also download them automatically through the Kaggle mirrors listed.

### 3.1 CIC-IoT-2023 (primary dataset)
- **Official source:** Canadian Institute for Cybersecurity, University of New Brunswick. https://www.unb.ca/cic/datasets/iotdataset-2023.html
- **Reference:** Neto E.C.P. et al., "CICIoT2023: A Real-Time Dataset and Benchmark for Large-Scale Attacks in IoT Environment", *Sensors* 23(13):5941, 2023.
- **Copy used in this study:** Kaggle convenience mirror `madhavmalhotra/unb-cic-iot-dataset`, 169 CSV part-files, 47 columns each.
- **Verification:** `Table_00b_Primary_File_Manifest.csv` lists each file's size, rows read and a SHA-256 fingerprint of its first MiB, so another copy can be checked against the one used here.

**Sampling and preprocessing** (all recorded in Tables 00, 00b, 00c, 01, 01b):
1. **Row sampling:** the first 1,066 rows of every file (ceil(180,000 / 169)), capped at 180,000 rows. All 169 files contributed.
2. **Label harmonisation** into 6 classes, listed in Table 01b. The rare "Other" group (128 rows, 0.07%) is excluded from the task and documented.
3. **Class profile:** DDoS 72.9%, DoS 17.3%, Mirai 5.7%, Benign 2.3%, Spoofing 1.1%, Recon 0.8%.
4. **Leakage control:**
   - Exact duplicates are removed *before* splitting (0 found).
   - 0 test rows share a feature vector with a training row.
   - Feature selection, imputation and scaling are fitted on training rows only.
5. **Stratified split:** train 105,224, validation 11,692, calibration 26,981, test 35,975. The model uses 39 features.

### 3.2 UNSW-NB15 (external replication)
- **Official source:** UNSW Canberra Cyber, https://research.unsw.edu.au/projects/unsw-nb15-dataset (partitioned `UNSW_NB15_training-set.csv` / `UNSW_NB15_testing-set.csv`).
- **Reference:** Moustafa N., Slay J., "UNSW-NB15: a comprehensive data set for network intrusion detection systems", MilCIS 2015.
- **Copy used in this study:** Kaggle mirror `alextamboli/unsw-nb15`.
- **Orientation fix:** this mirror ships the two partition files with their **names reversed**. The notebook detects this from the official sizes and swaps them back, so the run uses 175,341 training and 82,332 test records. This is recorded as `UNSWPartitionsSwapped = True` in Table 21.
- UNSW-NB15 is preprocessed separately (10 classes, 39 numeric features) and is **never pooled** with CIC-IoT-2023.

---

## 4. How to run

### 4.1 Setup (once)
1. Upload the notebook to Colab. Choose **Runtime → Change runtime type → T4 GPU**.
2. Create a Kaggle API token (kaggle.com → Settings → API → *Create New Token*). The notebook's Step 0a-2 cell asks for `kaggle.json` on first use and can keep a copy in your Drive.
3. Alternatively, place the official files in Google Drive:
   - `MyDrive/AgriTrust_Data/CICIoT2023/` (CSVs or zip)
   - `MyDrive/AgriTrust_Data/UNSW_NB15/` (the two partition CSVs)

### 4.2 Run modes (Step 0a, the only cell to edit)
| `RUN_MODE` | Data | Purpose | Time on a T4 |
|---|---|---|---|
| `SMOKE` | synthetic | checks that the code runs | minutes |
| `QUICK_REAL` | 60,000 real rows, 1 seed | diagnostic run; must pass Table 08c | ~2 h |
| `FULL_REAL` | 180,000 real rows, 10 seeds, UNSW-NB15 | publication run | ~9 h (several sessions possible) |

Each mode writes to its own folder, `MyDrive/AgriTrust_Results_<MODE>/`.

### 4.3 Resume after a disconnect
Every heavy stage is saved to `checkpoints/<RESUME_KEY>/`. If Colab disconnects: reconnect, keep the same settings, and choose **Run all**. Finished stages load in seconds, and only the interrupted stage restarts.

The resume key is fixed before any in-run setting change and includes a data fingerprint, so a changed configuration or dataset never reuses old checkpoints. For this study the key is **`e0cc45b388bf`**.

### 4.4 Reproduce the results without recomputation
Copy the `tables/`, `figures/` and `checkpoints/` folders (with `checkpoints/e0cc45b388bf/`) into `MyDrive/AgriTrust_Results_FULL_REAL/`, keep `RUN_MODE = 'FULL_REAL'`, and choose **Run all**. All stages load from the checkpoints, and the results appendix re-displays every table and figure.

---

## 5. Experimental protocol (FULL_REAL)

### 5.1 Federation and model
| Setting | Value |
|---|---|
| Clients | 10 simulated farm gateways |
| Data split | Label-skewed Dirichlet split (α = 0.25) with a **25% cap on any client's share of the training rows** |
| Dominant-client scenario | Uncapped split, largest client 43%; reported separately in Table 13b |
| Connectivity | Per-client availability and staleness profiles (Table 03); buffered, staleness-aware aggregation with λ = 0.45 fixed by design |
| Model | Identical MLP for every method: hidden size 128, 22,662 parameters, AdamW (lr 1e-3) |
| Training schedule | 40 rounds, 2 local epochs, batch 512 |

### 5.2 Trust mechanism
- **Calibration:** τ is calibrated on 10 benign warm-up rounds (2 burn-in rounds excluded, 64 client-rounds used): τ_cal = 0.608.
- **Rule selection:** the trust-memory rule was selected on **validation**, subject to a clean false-quarantine limit of 10%. The selected rule was **symmetric**.

### 5.3 Methods and experiments
- **Baselines:** FedAvg, FedProx, Async-Stale, Coordinate Median, Trimmed Mean, Multi-Krum, RawTrust (trust weighting without clipping or gating).
- **Attacks** at 10/20/30/40% malicious clients:
  - sign-flip, model replacement, Gaussian, collusive scaling
  - ALIE (Baruch et al., NeurIPS 2019), with a ramped z
  - jittered ALIE, an adaptive evasion test
- **Attack protocol:** attackers are persistent. Each method attacks from its own clean warm-up checkpoint (12 rounds) with its trust state carried over, and attack damage is paired with a no-attack continuation.
- **Seeds:** `[7, 19, 41, 73, 101, 131, 157, 181, 211, 239]`, 10 independent seeds for the clean and poisoning headline comparisons (4 attack scenarios at 30%).
- **Statistics:** Friedman omnibus test, then two-sided Wilcoxon signed-rank tests of AgriTrust-ACFL against each baseline, with Holm correction and paired effect sizes (Tables 17, 17b, 17c, 18).
- **Additional experiments:**
  - benign noise/staleness false-quarantine audit
  - dropout × staleness connectivity grid
  - per-client fairness
  - 9-row cumulative component ablation over 3 seeds (Table 16c)
  - τ/λ sensitivity on validation
  - round-1 (cold-start) attack
  - UNSW-NB15 federated replication (single seed, descriptive)

### 5.4 Stage compute times (from the checkpoints)
| Stage | Minutes | Stage | Minutes |
|---|---|---|---|
| Trust setup | 8.4 | Connectivity grid | 30.3 |
| Main runs | 14.0 | Dominant-client scenario | 15.2 |
| Warm starts | 3.5 | Ablation (3 seeds) | 47.6 |
| Poisoning grid | 70.7 | Sensitivity | 13.2 |
| Cold-start attack | 3.8 | UNSW-NB15 (official orientation) | 74.0 |
| Benign audit | 14.0 | Multi-seed (10 seeds) | ≈ 4.7 h of training (Tables 17, 17c) |

---

## 6. Integrity safeguards
- **Claim-eligibility gate** (Table 20). A claim may enter the manuscript only if its supporting experiment ran on real data with the required seeds.
- **Diagnostic acceptance** (Table 08c) and **data integrity** (Table 00c) checks.
- **Run-configuration registry** (Table 05d). It shows that the main, connectivity, poisoning and multi-seed experiments used one identical AgriTrust configuration.
- **No tuning on test data.** All method decisions were made on validation or training-side evidence during QUICK_REAL, before FULL_REAL, and are logged in the notebook changelog (first cell, v2.1–v2.7.2).
- Synthetic smoke output is never used as evidence.
- Gaussian update noise is a robustness proxy, **not** a differential-privacy guarantee.
- The server scores individual updates, so the method is **not compatible with secure aggregation** as implemented.

## 7. Limitations (reported in the manuscript)
- **Accuracy:** AgriTrust-ACFL is competitive, but it does not lead on Macro-F1. Trimmed Mean is significantly better on clean data (Holm-corrected p = 0.027).
- **Cold start:** attackers present from round 1 defeat AgriTrust-ACFL (Table 09c). Multi-Krum is the only robust baseline under that condition.
- **ALIE:**
  - ALIE causes no measurable damage on this data (Table 09).
  - A single ALIE attacker cannot be caught by the similarity signal.
  - Jittered ALIE evades detection.
- **Seed variability:** clean-data results vary more across seeds for the asynchronous methods (AgriTrust-ACFL SD 0.06), a consequence of random intermittent participation.
- **Data copies:** both datasets were obtained through Kaggle mirrors, verified by the file manifest and official row counts.
- **Scope of the UNSW-NB15 replication:** it is single-seed and descriptive only.
- **Aggregation style:** the server is staleness-aware and buffered (semi-asynchronous), not an event-driven asynchronous server.

---

## 8. Citation, licence, contact
- **Manuscript:** R. Augustian Isaac, "AgriTrust-ACFL: Adaptive Client-Trust Federated Learning with Staleness-Aware Buffered Aggregation for Byzantine-Resilient Agricultural IoT Intrusion Detection", submitted to the *Journal of Information Security and Applications* (Elsevier), 2026. The full reference and DOI will be added on publication.
- **Repository:** https://github.com/IsaacJournals/AgriTrust-ACFL
- **Licence:** MIT License (see `LICENSE`) for the code and the results in this repository. The CIC-IoT-2023 and UNSW-NB15 datasets are not redistributed and remain under their original providers' terms.
- **Contact:** Dr. R. Augustian Isaac, Department of Artificial Intelligence and Machine Learning, Saveetha Engineering College, Chennai, India. E-mail: augustianisaacr@saveetha.ac.in
