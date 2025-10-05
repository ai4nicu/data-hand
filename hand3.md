# Handbook Part 3: Data Lifecycle Management for Neonatal EEG

Contributions welcome! Please open a PR or issue on [GitHub](https://github.com/ai4nicu/data-hand)
or contact the authors.

---

## 1. Purpose & Principles

This document defines a practical, evidence‑informed framework to manage neonatal EEG data across
its entire lifecycle. It aims to:

- Protect patients and staff through privacy‑by‑design and security‑by‑default.
- Enable reproducible research.
- Align with FAIR principles (Findable, Accessible, Interoperable, Reusable) and relevant standards (e.g., EEG‑BIDS, DICOM Neurophysiology) without mandating a single toolchain.
- Support operational constraints of a NICU (limited time, shifting staff, vendor ecosystems).

**Guiding principles**
- Minimum necessary data; role‑based access (RBAC); auditability.
- Standardize early; document everything; automate where feasible.
- Reproducibility for both science and clinical audit.

See [Section 14](#glossary) and [Section 19](#acronyms) for a glossary of terms and acronyms.

---

## 2. Stakeholders & Roles

| Role                                       | Primary Responsibilities                                               |
|--------------------------------------------|------------------------------------------------------------------------|
| Principal Investigator / Clinical Lead     | Oversight, purpose & lawful basis, sign‑off                            |
| Data Protection Officer (DPO) / Governance | Policy, DPIA, data processing agreements, retention & deletion control |
| Clinical Engineering / Neurophysiology     | Device configuration, calibration, electrode standards, acquisition QC |
| IT / Security                              | Identity, access, networking, encryption, backup/DR, monitoring        |
| Data Steward / BIDS Curator                | Naming schemes, metadata quality, dataset integrity                    |
| Researchers / ML Engineers                 | Pipelines, documentation, provenance, model governance                 |
| Data Access Committee (DAC)                | Review & audit data requests, DUAs, export controls                    |

---

## 3. Lifecycle Overview (High‑Level)

1. **Plan & Approve** — Purpose, protocol, consent/assent, DPIA, governance.
2. **Acquire** — EEG (± video, ECG, respiration, SpO₂), device metadata, operator notes.
3. **Ingest** — Secure transfer, integrity checks (hashes), accession identifiers.
4. **Standardize** — Convert vendor formats to **EEG‑BIDS** (GDF/BrainVision/parquet) or **DICOM
   Neurophysiology** for PACS/VNA workflows; harmonize events/ontologies.
5. **Store & Protect** — Tiered storage (hot/warm/cold), encryption, RBAC, audit logs, backup/DR.
6. **Quality Control** — Automated & manual QC; completeness & conformance checks; SNR/noise metrics.
7. **Process & Derive** — Pipelines in containers; features/derivatives (Parquet/Arrow) with full provenance.
8. **Share & Access** — Controlled access via DAC and DUAs; de‑identification; secure workspaces.
9. **Operationalize** — For cotside algorithms: deployment, monitoring, incident & model‑drift management.
10. **Archive & Delete** — Retention to policy; archival formats; verifiable deletion.

---

## 4. Acquisition & Ingest

### 4.1 Pre‑Acquisition
- **Protocol & Consent:** Approved purpose, consent/assent templates, and exclusion rules.
- **Device Configuration:** Sampling rate, filters (type/order, −3 dB points), reference/montage, channel labels consistent with a site‑approved map.
- **Electrodes & Impedances:** Document target and measured impedances; date/time; operator ID.
- **Time Sync:** NTP/PTP and clock drift checks for EEG and ancillary devices (video, monitors).

### 4.2 Signals & Modalities
- **Core:** EEG (≥12 cerebral channels recommended) + ECG, respiration; consider EOG, submental EMG when indicated.
- **Optional:** Video EEG—requires additional privacy controls (faces, staff, audio).

### 4.3 Data Formats at Source
- **Vendor Native:** Retain verbatim for medico‑legal and fidelity.
- **Immediate Standardization:** If video-EEG, export/ingest to **DICOM Neurophysiology** for PACS/VNA; otherwise, export to **EDF/EDF+/GDF/parquet** or **BrainVision** for EEG‑BIDS.

### 4.4 Ingest Workflow (Checklist)
- Secure transfer (SFTP/VPN/secure share); checksum (**sha256**) at source and target; compare.
- Assign **accession identifiers** (subject/session/acq/run) using a site prefix (e.g., `NICU01`).
- Record ingest log (who/when/how much, hash, device IDs, operator ID).
- Quarantine raw drops; only release to curated store after QC and metadata validation.

---

## 5. Standardization: Structures & Schemas

### 5.1 EEG‑BIDS (Recommended Organizational Standard)
- **Raw waveform container:** **EDF/EDF+** or **BrainVision** in `sub-*/ses-*/eeg/`.
- **Sidecars & Tables:** `*_eeg.json` (device & acquisition params), `channels.tsv`, `electrodes.tsv`, `coordsystem.json`, `events.tsv` (with harmonized neonatal terms), `scans.tsv`, `participants.tsv`.
- **Annotations:** Prefer machine‑readable, columnar events with onset/duration and coded labels.
- **Derivatives:** Store features, spectrograms, labels in `derivatives/` with a `dataset_description.json` per derivative pipeline and full provenance links.

### 5.2 DICOM Neurophysiology (Clinical Enterprise Path)
- Use when integrating with **PACS** (Picture Archiving and Communication System)/**VNA** (Vendor Neutral Archive). Enables enterprise archival, retention, audit, and linkage to the EHR.

### 5.3 Other Ecosystem Formats (For Awareness & Interop)
- **MEF3:** High‑performance epilepsy research format with robust indexing.
- **NIX (G‑Node):** HDF5‑based; general neurophysiology data + metadata.
- **NWB (Neurodata Without Borders):** Widely used in systems neuroscience; good for complex experiments.
- **HDF5 / Zarr:** Scalable chunked arrays (useful for derived tensors).
- These can complement BIDS at the **derivatives** layer; prefer BIDS for the organizational scaffold.

### 5.4 Controlled Vocabularies & Ontologies
- Align neonatal events and interpretations to **ACNS neonatal terminology** and **HED (Hierarchical
  Event Descriptors)** where applicable. Maintain a site‑level **event dictionary** mapping local
  terms → standard codes.

---

## 6. Metadata: Minimum & Recommended

**Minimum** (per session):
- Subject pseudonym (e.g., `sub-001`), sex, GA/PMA, birthweight range bucket (or omit if sensitive), study/site identifiers.
- Device vendor/model, amplifier input range, sampling rate, filter settings, reference, montage, channel list, electrode type.
- Start/stop timestamps (date‑shifted if required), operator ID (pseudonymized), room ID (if used).
- Consent version & scope (e.g., research reuse allowed/not allowed; video consent scope).

**Recommended additions:**
- Impedance measurements; electrode locations; events (HED‑coded); clinical context tags (e.g., HIE grade, medications, hypothermia); adverse events; artefact notes.
- Hashes for every file; parent hash for dataset snapshot; storage tier; DUAs linked.

---

## 7. Privacy, De‑Identification, & Governance

- **Pseudonymization:** Replace direct identifiers; maintain a re‑identification key in a separate, access‑controlled vault; log all key access.
- **Date Handling:** Apply **constant per‑subject date/time shift** for de‑identified exports; preserve intervals; record offset securely.
- **Headers & Video:** Scrub PHI from waveform headers and filenames; for video, consider face blurring and audio stripping for shares.
- **Data Use Agreements (DUAs):** Scope, permitted users/uses, time limits, publication rules, breach procedures.
- **Access Control:** RBAC with least privilege; two‑person rule for de‑identification key; periodic access reviews.
- **Auditability:** Immutable logs of data access, exports, and transformations.

> **Note:** Ensure alignment with local/national regulation (e.g., GDPR) and institutional
> policy. Retention/deletion, cross‑border transfers, and secondary use require explicit governance.

---

## 8. Storage, Security & Resilience

### 8.1 Tiered Storage
- **Hot (working):** Active curation/analysis; SSD; frequent snapshots.
- **Warm (curated):** Master BIDS dataset and validated derivatives.
- **Cold (archive):** Long‑term store (object storage/tape) with integrity checks (fixity).

### 8.2 Backup & Disaster Recovery
- **3‑2‑1 rule:** 3 copies, on 2 media, 1 offsite. Periodically test restores.
- End‑to‑end checksums; integrity monitoring (fixity audits with hash manifests).

### 8.3 Security Controls
- Encryption in transit (TLS) and at rest (KMS/HSM‑backed keys). 
- Network segmentation; bastion access; MFA for privileged roles.
- Secrets management (vault); short‑lived credentials; no hard‑coded secrets.
- Continuous monitoring/alerting; vulnerability management for endpoints and servers.

---

## 9. Quality Control (QC)

### 9.1 Conformance & Completeness
- BIDS validator pass; required tables present; metadata keys present; file hashes recorded.

### 9.2 Signal Quality Metrics (examples)
- **Impedance** summary (per channel, thresholds, outliers).
- **Dropouts/flatlines** (% time, per channel).
- **Line‑noise SNR** and harmonics; **broadband noise** estimate.
- **Amplitude distributions** (per channel) and inter‑channel correlation matrix.
- **Spectral slopes** (1/f exponent) and bandpower sanity checks.

### 9.3 Manual Review
- Randomized spot‑checks of 30–60 s windows; confirmation of montage & filters.
- Event label sanity; check for time base drift; compare ECG and respiration traces.

### 9.4 QC Artifacts & Reporting
- Store QC as `derivatives/qc/` with machine‑readable summaries (JSON/Parquet) and plots (PNG/PDF). Link each QC artifact to file hashes and dataset versions.

---

## 10. Processing & Derivatives

### 10.1 Pipelines & Reproducibility
- Use **containers** (e.g., Docker) or reproducible Python environments (e.g., `uv`) with pinned versions.
- Orchestrate with **Snakemake/Nextflow/CWL/DVC**; capture parameters, code commit, container digest, input/output hashes.
- Emit a **provenance record** (e.g., RO‑Crate or simple JSON) per run.

### 10.2 Derivative Types (examples)
- Filtered/re‑referenced EEG; artefact masks; spectrograms; time–frequency coefficients; channel maps.
- Features (amplitude statistics, bandpowers, fractal/entropy metrics, burst suppression metrics, IBI/SDA metrics).
- Labels (expert annotations, consensus, adjudication histories) with versioning.

### 10.3 File Formats
- **Columnar** for features/labels: **Parquet/Arrow** (fast IO, schema, compression).
- **HDF5/Zarr** for large tensors; **PNG/TIFF** for images; **CSV/TSV** only for small, human‑readable tables.

---

## 11. Access, Sharing & Safe Analytics

- **Data Access Committee (DAC):** Intake form, review cadence, conflict checks.
- **Secure Workspaces:** VDI or managed notebooks; no raw data download by default.
- **Export Controls:** Whitelisted patterns; data escrow; delayed publication if required.
- **External Sharing:** Share **de‑identified BIDS** subsets; include clear data dictionary and event ontology mapping; attach DUA & citation instructions.
- **Federated/Secure Compute:** Consider federated learning or model‑to‑data approaches when transfer is constrained.

---

## 12. AI Model Governance

- **Data Splits:** Subject‑level splits; nested CV; strict separation of development/validation/test cohorts; track leakage risks.
- **Registries:** Register datasets, models, metrics, and training runs (hashes, configs, seeds) in a central catalogue.
- **Bias & Fairness:** Monitor performance across strata (GA, PMA, clinical subgroups); document known limitations.
- **Traceability:** Each model score links to code+data+config digests.
- **Post‑Deployment:** Monitor data drift, performance, false alarms; maintain rollback plan.

---

## 13. Archival, Retention & Deletion

- **Archival Package:** Frozen copy of BIDS + hashes + provenance + DUA + readme for future readers.
- **Retention:** Follow institutional/regulatory rules for pediatric clinical data; document durations per data class (raw EEG, video, derivatives, audit logs).
- **Deletion:** Verifiable deletion procedures; document what is kept vs destroyed (e.g., maintain hashes for chain‑of‑custody).

---

## 14. Risks & Mitigations (Examples)

| Risk                                      | Impact                        | Mitigation                                                |
|-------------------------------------------|-------------------------------|-----------------------------------------------------------|
| Re‑identification via timestamps or video | Privacy breach                | Date‑shift, face/audio suppression, tight RBAC, approvals |
| Inconsistent channel naming/montage       | Misinterpretation; invalid ML | BIDS `channels.tsv` validator; site map; automated checks |
| Clock drift between devices               | Misaligned events             | NTP/PTP, synchronization logs, drift tests                |
| Data corruption over time                 | Loss of research value        | Fixity audits, multiple copies, routine restore tests     |
| Pipeline non‑reproducibility              | Irreproducible results        | Containers, pinned deps, full provenance                  |

---

## 15. Glossary

- **BIDS (EEG‑BIDS):** Brain Imaging Data Structure—organizational standard for EEG datasets.
- **DICOM Neurophysiology:** DICOM objects for waveforms/annotations enabling PACS/VNA storage.
- **PACS / VNA:** Picture Archiving and Communication System / Vendor Neutral Archive.
- **HED:** Hierarchical Event Descriptors—ontology for event annotations.
- **PHI/PII:** Protected Health/Personal Information.
- **RBAC:** Role‑Based Access Control.
- **FAIR:** Findable, Accessible, Interoperable, Reusable.

---

## 16. Templates & Examples

### 16.1 Example BIDS Layout (EEG‑only)
```
project-root/
  dataset_description.json
  participants.tsv
  CHANGES
  README.md
  sub-001/
    ses-001/
      eeg/
        sub-001_ses-001_task-rest_eeg.edf
        sub-001_ses-001_task-rest_eeg.json
        sub-001_ses-001_task-rest_channels.tsv
        sub-001_ses-001_task-rest_events.tsv
        sub-001_ses-001_task-rest_electrodes.tsv
        sub-001_ses-001_task-rest_coordsystem.json
  derivatives/
    qc/
      sub-001_ses-001_qc.json
      sub-001_ses-001_qc.png
    features/
      sub-001_ses-001_features.parquet
```

### 16.2 `participants.tsv` (example columns)

| participant_id | sex | GA_weeks | PMA_weeks | birthweight_g | diagnosis    | hypothermia |
|----------------|----:|---------:|----------:|--------------:|--------------|-------------|
| sub-001        |   F |       38 |        39 |          3150 | HIE-moderate | yes         |

> Store sensitive clinical variables in a controlled `phenotype/` folder if policy requires tighter controls.

### 16.3 Events Dictionary (excerpt)

| code          | label                  | description                                         | ontology   |
|---------------|------------------------|-----------------------------------------------------|------------|
| NEONATE/BURST | Burst                  | High‑amplitude burst in burst‑suppression           | HED tag(s) |
| NEONATE/SDA   | Discontinuous activity | Low‑amplitude period between bursts                 | HED tag(s) |
| SEIZURE/ONSET | Seizure onset          | First electrographic change consistent with seizure | HED tag(s) |

### 16.4 QC Checklist (operational)
- [ ] Device & channel map verified
- [ ] Sampling rate & filters recorded
- [ ] Impedances within target range
- [ ] Hashes computed & logged
- [ ] BIDS validation passed
- [ ] Events/labels conform to dictionary
- [ ] QC metrics reviewed & signed off

---

## 17. Operational Playbooks (Short)

- **New Dataset Intake:** Ingest → Hash → Quarantine → Curate to BIDS → QC → Release.
- **External Share:** Extract subset → De‑identify → Validate → DAC approval → Package (BIDS + dictionary + DUA) → Secure delivery.
- **Incident (Suspected Breach):** Contain → Notify per policy → Forensics → Report → Post‑mortem → Remediation.

---

## 18. Open Questions

- Finalize the **controlled vocabulary** for neonatal events (map local → ACNS/HED).
- Decide the **enterprise default** for video EEG (DICOM to PACS/VNA vs research store).
- Choose a **pipeline orchestrator** (Snakemake/Nextflow/DVC) and a **registry** for datasets/models.
- Define **retention durations** per data class with governance.
- Pilot **federated analysis** for cross‑site studies.


## 19. Acronyms

| Acronym  | Meaning                                                         |
| -------- | --------------------------------------------------------------- |
| ACNS     | American Clinical Neurophysiology Society                       |
| BIDS     | Brain Imaging Data Structure (organizational/metadata standard) |
| CWL      | Common Workflow Language                                        |
| DAC      | Data Access Committee                                           |
| DICOM    | Digital Imaging and Communications in Medicine                  |
| DPO      | Data Protection Officer                                         |
| DPIA     | Data Protection Impact Assessment                               |
| DR       | Disaster Recovery                                               |
| DUA      | Data Use Agreement                                              |
| DVC      | Data Version Control (reproducible data pipelines)              |
| ECG      | Electrocardiogram                                               |
| EDF      | European Data Format (for biosignals; EDF/EDF+)                 |
| EEG      | Electroencephalography                                          |
| EEG-BIDS | BIDS specification/extension for EEG data                       |
| EHR      | Electronic Health Record                                        |
| EMG      | Electromyography                                                |
| EOG      | Electro-oculogram                                               |
| FAIR     | Findable, Accessible, Interoperable, Reusable                   |
| GA       | Gestational Age                                                 |
| GDPR     | General Data Protection Regulation                              |
| HDF5     | Hierarchical Data Format version 5                              |
| HED      | Hierarchical Event Descriptors (event ontology)                 |
| HIE      | Hypoxic–Ischemic Encephalopathy                                 |
| HSM      | Hardware Security Module                                        |
| IBI      | Inter-Burst Interval                                            |
| KMS      | Key Management Service (encryption key management)              |
| MEF3     | Multiscale Electrophysiology Format, version 3                  |
| MFA      | Multi-Factor Authentication                                     |
| NICU     | Neonatal Intensive Care Unit                                    |
| NIX      | Neurophysiology data format from G-Node                         |
| NWB      | Neurodata Without Borders                                       |
| NTP      | Network Time Protocol                                           |
| PACS     | Picture Archiving and Communication System                      |
| PHI      | Protected Health Information                                    |
| PII      | Personally Identifiable Information                             |
| PMA      | Post-Menstrual Age                                              |
| PTP      | Precision Time Protocol (IEEE 1588)                             |
| QC       | Quality Control                                                 |
| RBAC     | Role-Based Access Control                                       |
| RO-Crate | Research Object Crate (provenance/packaging standard)           |
| SDA      | Segments of Discontinuous Activity (discontinuous neonatal EEG) |
| SFTP     | SSH File Transfer Protocol                                      |
| SHA-256  | Secure Hash Algorithm, 256-bit                                  |
| SNR      | Signal-to-Noise Ratio                                           |
| SpO₂     | Peripheral capillary oxygen saturation                          |
| TLS      | Transport Layer Security                                        |
| VDI      | Virtual Desktop Infrastructure                                  |
| VNA      | Vendor Neutral Archive                                          |
| VPN      | Virtual Private Network                                         |
| Zarr     | Chunked, compressed N-dimensional array store                   |

