# Handbook Part 2: Exploration of guidelines for storing neonatal EEG

Contributions welcome! Please open a PR or issue on [GitHub](https://github.com/ai4nicu/data-hand)
or contact the authors.


## Summary
- **Clinic / Archive / Compliance**: Capture in vendor format; archive as **DICOM Neurophysiology**
  when video matters; preserve audit trails and protected health information (PHI) handling.
- **Research / Curation**: Mirror recordings in **BIDS** with **EDF/BDF/BrainVision (or GDF/NWB)**;
  use sidecars for rich, controlled metadata and events.
- **Machine learning (ML)**: Keep raw data immutable; write to **parquet** files for
  features/labels/masks and raw EEG as BIDS Derivatives; enforce HED/HED-SCORE (HED: hierarchical
  event descriptors; SCORE: standardised computer-based organised reporting of EEG) event terms and
  full pipeline provenance.

---

## 0) FAIR Goals
**FAIR** means **Findable, Accessible, Interoperable, and Reusable**. For neonatal EEG intended for
machine learning (ML) development, FAIR should be the *design objective*:

- **Findable**: Stable identifiers (such as digital object identifiers, DOI), searchable metadata
  (such as patient ID, gestational age, postmenstrual age, amplifier, reference electrode, filters etc.),
  and an indexable directory layout. 
- **Accessible**: Authenticated, documented access paths (e.g., data use agreements), durable URLs,
  and versioned snapshots (e.g., Datalad, DVC, Git-annex). Provide README-level instructions and
  contact.
- **Interoperable**: Community schemas and controlled vocabularies rather than free-text. For
  example, **controlled event terms** (e.g., HED/HED-SCORE), explicit units (µV), and power line
  frequency (e.g. 50 or 60 Hz).
- **Reusable**: Sufficient context to *reproduce* analyses: raw signals stored in lossless format, complete
  acquisition metadata, **license**, **consent/use constraints**, and **pipeline provenance**
  (software versions, parameters) for every derivative.

> FAIR can prevent label drift, PHI leaks, and non-reproducible ML. The rest of this report shows
> how BIDS (plus suitable file formats) is a candidate for the foundation of FAIR neonatal EEG
> datasets.

---

## 1) Brain Imaging Data Structure — BIDS

### 1.1 BIDS (EEG-BIDS)
BIDS is a community standard that organises datasets via a **directory layout** and
**machine-readable files** known as _sidecars_ (`*_eeg.json`, `channels.tsv`, `events.tsv`,
`participants.tsv`, etc.). For EEG, the **on-disk signal file** is kept in a recommended
legacy/standard format (EDF/BrainVision). BIDS also defines **derivatives** for processed data and
**validators** to enforce consistency.

**Key properties**
- Human- and machine-readable metadata.
- Extensible but constrained schema; encourages controlled vocabularies.
- Clear separation of **raw** vs **derivatives**; promotes reproducibility.

**Terminology — “sidecar” files.** In BIDS, “sidecars” are the JSON/TSV files that accompany each
raw recording (e.g., `*_eeg.json`, `channels.tsv`, `events.tsv`, `participants.tsv`). They carry
machine-readable metadata and annotations that waveform files (e.g. EDF/BrainVision/GDF) cannot
express by themselves.

### 1.2 EDF/EDF+
EDF is a simple, open, widely supported waveform container using per-channel scaling of **16-bit
integers** with an ASCII header. **EDF+** adds an **annotation channel** and supports
**discontinuous** recordings. **BDF/BDF+** is a 24-bit variant (often BioSemi). EDF is a
*container*, not a dataset standard.

**Key properties**
- Ubiquitous support across viewers and toolkits.
- Minimal metadata; annotations are free-text in a special channel.
- No native compression, no standardised video linkage.
- Lossy format, enforces a minimum/maximum to allow for storage as 16-bit integers

---

## 2) Limitations and suitability for neonatal EEG

### 2.1 BIDS 
- **Origins & ecosystem:** Many exemplars come from ERP/experimental neuroscience; clinical/neonatal
  contexts need **richer vocabularies** (e.g., background grades, IBI or seizures labels,
  medications and hypothermia information). BIDS **allows** these via extra columns but **doesn’t
  mandate** a clinical ontology—projects must adopt and enforce one.
- **No standardised video:** BIDS does not standardise synchronised video-EEG. When video is
  clinically required, pair with **DICOM Neurophysiology** for archiving and linkage.
- **Learning curve:** Requires discipline to create complete JSON/TSV sidecars; conversion and
  validation steps are required in data processing pipeline.

### 2.2 EDF/EDF+
- **Annotations are weakly standardised:** Free-text markers lead to heterogeneity (e.g., “sz?”,
  “seizure”, “ictal event”). Hard for ML pipelines without a controlled vocabulary.
- **Limited numeric fidelity:** 16-bit integer storage adds quantisation and enforces a range of
  allowable values; BDF improves to 24-bit but is not universal.
- **Header/PHI quirks:** ASCII patient/record fields may contain PHI; date fields are rigid/quirky
  and complicate anonymization; widespread practice is **date-shifting**.
- **No compression, capacity for video, and minimal extensibility:** Increases storage and I/O
  burden at scale; richer device metadata must live outside the file.

---

## 3) Alternatives to BIDS and EDF

### 3.1 Keep EDF (or BrainVision) for **raw**, but wrap with BIDS
- **Why:** Widely supported; BIDS sidecars carry the rich metadata and event structure EDF
  lacks. Validators and exporters exist (e.g., MNE-BIDS).
- **Prefer BrainVision or BDF when you can:** BrainVision separates **header (.vhdr)** and **markers
  (.vmrk)** from the binary, which yields cleaner events; **BDF/BDF+** offers **24-bit**
  headroom. (Note: BrainVision is openly documented but vendor-controlled.)
- **GDF** (General Data Format) improves on EDF with an event table, richer units, and time-zones;
  adoption is narrower but technically strong.

### 3.2 Clinical archiving & synchronised video
- Use **DICOM Neurophysiology** for hospital workflows: long-term storage, **waveform + video**, audit trails, and enterprise interoperability. Maintain a BIDS mirror for research.

### 3.3 Single-file, rich schema for research access
- **NWB (Neurodata Without Borders):** HDF5-based, typed schema; good for one self-describing file
  per session with rich metadata and APIs (PyNWB/MatNWB). Increasing EEG support.
- **NIX (+ odML):** HDF5-backed model with strong metadata structures; interacts well via Neo
  ecosystem.
- **MEF3:** Focused on large-scale EEG datasets with lossless compression, block indexing, and
  optional encryption (HIPAA-friendly). Great for fast random access and privacy-aware sharing.
- **XDF/OpenXDF:** Multi-stream acquisition (via Lab Streaming Layer) with synchronised streams;
  great for capture, typically converted to BIDS/NWB for curation/sharing.

### 3.4 Parquet (Arrow) — for **derivatives and ML datasets**
- **Parquet (Arrow):** Columnar, typed, compressed tables with predicate pushdown—ideal for
  **epoch-level features, labels, and masks** and **raw EEG**; language-agnostic (Python/R/Julia/Go/Java/Rust).
- **Integration:** Store under **`derivatives/`** in BIDS with provenance (`pipeline_description.json`, parameter JSONs) to remain FAIR.

### 3.5 Alternatives to BIDS — Expanded Profiles

**DICOM Neurophysiology (WG-32)**  
*What it is*: Clinical standard for neurophysiology waveforms in hospital ecosystems (PACS/VNA), with provisions to link **waveform + synchronised video** and audit trails.  
*Strengths*: Enterprise interoperability; long-term archival; mature PHI handling; standardised time bases and device descriptors; ideal for **video-EEG**.  
*Limitations*: Heavyweight; fewer open-source analytics tools; not designed as a research curation layout.  
*Use it when*: You need clinical video EEG workflows or hospital archiving; keep a **BIDS mirror**
for research/ML.

**NWB (Neurodata Without Borders)**  
*What it is*: Typed, self-describing **HDF5-based** data language for neurophysiology with strong APIs (PyNWB/MatNWB).  
*Strengths*: One file per session with rich schema; good random access; can embed processed data; growing EEG support.  
*Limitations*: Heavier dependency stack; fewer clinical viewers; requires schema literacy.  
*Use it when*: You want a **single, richly annotated container** for analysis/sharing; acceptable in BIDS via EEG-BIDS in some contexts.

**NIX (+ odML)**  
*What it is*: Open HDF5-backed model with first-class metadata via **odML**; widely used through the Neo ecosystem.  
*Strengths*: Strong metadata; cross-language bindings; flexible object model.  
*Limitations*: Smaller user base than BIDS/NWB; fewer off-the-shelf ML toolchains.  
*Use it when*: You want an **open, schema-driven** container and tight metadata control.

**MEF3 (Multiscale Electrophysiology Format)**  
*What it is*: Open spec (Mayo Clinic) focused on **lossless compression**, **block indexing**, and **optional encryption**.  
*Strengths*: Fast random access to huge EEG datasets; built-in privacy controls (encryption); robust for clinical epilepsy datasets.  
*Limitations*: Smaller community; fewer general viewers; export/conversion steps needed for BIDS.  
*Use it when*: You need **big-data performance** and **secure sharing**.

**GDF (General Data Format for Biomedical Signals)**  
*What it is*: Open successor to EDF with **event tables, units, and time-zones**; part of the BioSig stack.  
*Strengths*: Technically stronger raw container than EDF; explicit events/units; open tooling.  
*Limitations*: Less ubiquitous in clinics; viewers less common than for EDF.  
*Use it when*: You control exports and want a **cleaner raw signal format** while still converting to BIDS.

**XDF / OpenXDF (via Lab Streaming Layer)**  
*What it is*: Lab capture format for synchronised multi-stream data (EEG + AUX) with per-stream headers and time-sync.  
*Strengths*: Excellent for **acquisition/fusion**; supports many simultaneous signals.  
*Limitations*: Not an archival/curation standard; typically converted to BIDS/NWB.  
*Use it when*: You need **synchronisation at acquisition**; export to BIDS for sharing/ML.

---


## 4) Requirements for Robust ML Development (Neonatal EEG)

1. **Signal fidelity & sampling**
   - **Sampling rate**: ≥200 Hz (prefer **≥512 Hz**) for neonatal EEG to preserve spike morphology,
     line-noise separation, and anti-alias headroom.
   - **DC-coupled amplifiers**: Some modern EEG systems are **DC-compliant** (no embedded high-pass
     filters), so very slow drifts and offsets (near-DC) are recorded. In **EDF/EDF+**, each channel
     maps integer counts to physical units via **`physical_min`/`physical_max`** (e.g., ±5000
     µV). With DC offsets, you may need **very wide physical ranges** to avoid clipping, reducing
     effective resolution. Consider **BDF (24-bit)** for headroom, or containers that support higher
     precision/float encodings (e.g., BrainVision with `BinaryFormat=IEEE_FLOAT_32`,
     GDF). Regardless, **do not high-pass the raw** to “fix” drift; instead, document
     `HardwareFilters` (HPF = DC/0 Hz) in `*_eeg.json`, and apply detrending/high-pass only in
     **derivatives** (with full provenance).
   - **Preserve very low frequencies**: Maintain a processing path with high-pass filers,
     e.g. **0.3–0.6 Hz**; document the *effective* passband (including the anti-aliasing filter) per
     channel in `channels.tsv`.

2. **Metadata completeness**
   - In `*_eeg.json` & `channels.tsv` record: sampling rate, reference, filter settings, impedance,
     units, power-line frequency (50 Hz or 60 Hz), amplifier, and cap/electrode information.
   - In `participants.tsv/json` capture as much clinical information as possible; for example,
     gestational and postmenstrual age at recording, birthweight, diagnosis, therapeutic
     hypothermia, sedatives/anticonvulsants, temperature, clinical state.

3. **Labels & events**
   - `events.tsv`: machine-actionable **onset/duration**; controlled `trial_type`; columns for
     **rater_id**, **confidence**, **adjudication_status**, **evidence**.
   - Use consistent rules for annotations (e.g., seizures, IBI, bursts) and surrounding windows.

4. **De-identification**
   - **Date-shift** per subject (constant offset); scrub EDF/BrainVision headers of PHI; maintain key files separately.

5. **Splits, provenance, reproducibility**
   - Record **subject-wise** train/val/test splits to avoid leakage.
   - Version code, containers, and parameters; write **derivatives** metadata (`pipeline_description.json`, software versions).

6. **Storage formats for ML scale-out**
   - **Parquet** tables for EEG and features/labels/masks (fast, efficient storage; easy cross-language).

7. **Governance & QC**
   - Automated validators (BIDS), schema checks for labels, impedance/line-noise QC plots, and data drift monitoring for updates.

---

## 6) Recommended Stack & Conversion Pipeline

### 6.1 Acquisition & Archiving
- **Clinic first**: Capture vendor format; export/ingest to **DICOM Neurophysiology** if video-EEG
  is involved for clinical use in PACS and long-term storage in the vendor neutral archive (VNA).
- **Research mirror**: Export raw signals as **EDF/BDF** (or **BrainVision/GDF/NWB**) to a secure
  research store; treat these as **immutable source**.

### 6.2 Curation to BIDS (research layer)
1) **Layout**: Create BIDS folders (`sub-*/ses-*/eeg/`).  
2) **Convert**: Use your toolkit (e.g., MNE-BIDS) to write `*_eeg.json`, `channels.tsv`, `electrodes.tsv`, `coordsystem.json`, and **`events.tsv`** with **controlled terms**.  
3) **Anonymize**: Apply **date-shift** per subject; scrub headers (names/IDs) in EDF/BrainVision.  
4) **Validate**: Run the BIDS Validator and fix errors/warnings.  
5) **Snapshot**: Version with Git/Datalad; record a release tag.

*Conceptual Python snippet (MNE-BIDS)*
```python
from mne_bids import write_raw_bids, BIDSPath
raw = ...  # read vendor or EDF/BDF
bids_path = BIDSPath(subject="001", session="001", task="rest", root="/data/bids")
write_raw_bids(raw, bids_path, overwrite=False, anonymize={'daysback': 40000})
```


### 6.3 Derivatives for ML
- **Waveform outputs** (optional): Keep filtered/resampled versions as **Parquet/BrainVision/GDF** under
  `derivatives/<pipeline>/eeg/` with `pipeline_description.json` and full parameters.
- **Tabular ML datasets**: Store **features/labels/masks** in **Parquet** partitioned by
  `sub=/ses=/run=` for selective reads. Include a **schema JSON** and an **index** mapping epochs to
  raw BIDS entities and onsets.
- **Provenance**: Record software versions (e.g., `mne`, `scipy`, your package), parameter hashes,
  container digests; cite them in `pipeline_description.json`.

### 6.4 Governance & QC
- **Automated checks**: impedance thresholds, flat/noisy channel flags, line-noise metrics, and data drift reports per release.
- **Access control**: Separate PHI linkage keys; publish only de-identified BIDS.

---

## 7) Draft Schemas & Templates 

### 7.1 `events.tsv` (neonatal-friendly columns)
**Columns**: `onset` (s), `duration` (s), `trial_type` (controlled terms), `rater_id`, `confidence` (0–1), `adjudication_status` (e.g., single, consensus), `channels` (optional list), `evidence`, `notes` (sparse).

**Example**

| onset | duration | trial_type | rater_id | confidence | adjudication_status | evidence     |
|-------|----------|------------|----------|------------|---------------------|--------------|
| 12.30 | 58.0     | seizure    | R1       | 0.92       | consensus           | clinical+EEG |
| 71.00 | 15.43    | burst      | R2       | 0.80       | single              | EEG          |
| 88.50 | 22.04    | IBI        | R2       | 0.75       | single              | EEG          |




**Suggested `trial_type` set (editable)**: `seizure`, `preictal`, `postictal`, `burst`, `IBI`, `PRS` (positive rolandic sharp), `artifact_movement`, `artifact_electrode`, `aw_state` (awake/drowsy/sleep), `hypoxic_event`.

### 7.2 `participants.tsv` (extra neonatal context)
**Columns**: `participant_id`, `sex`, `GA_weeks`, `PMA_weeks`, `birthweight_g`, `diagnosis`, `hypothermia` (yes/no), `sedatives` (list), `anticonvulsants` (list), `apgar_5min`, `MRI_findings`, `outcome_followup` (if applicable; controlled access).

**Example**

| participant_id | sex | GA_weeks | PMA_weeks | birthweight_g | diagnosis    | hypothermia |
|----------------|-----|----------|-----------|---------------|--------------|-------------|
| sub-001        | F   | 38       | 39        | 3150          | HIE-moderate | yes         |


### 7.3 `*_eeg.json` (minimal template)
```json
{
  "TaskName": "rest",
  "PowerLineFrequency": 50,
  "SamplingFrequency": 512,
  "EEGReference": "Cz",
  "HardwareFilters": {
    "Highpass": "DC (0 Hz)",
    "Lowpass": "70 Hz",
    "Notch": "none"
  },
  "Manufacturer": "<amplifier>",
  "CapManufacturer": "<cap>",
  "RecordingDuration": 3600,
  "RecordingType": "continuous"
}
```

### 7.4 `channels.tsv` (key fields)
`name`, `type` (EEG, ECG, EMG, EOG, RESP), `units` (uV), `sampling_frequency`, `low_cutoff`, `high_cutoff`, `notch`, `status` (good/bad), `status_description`.

### 7.5 Parquet schema for features/labels (example)
| Column                         | Type         | Notes                           |
|--------------------------------|--------------|---------------------------------|
| sub                            | string       | BIDS subject                    |
| ses                            | string       | BIDS session                    |
| run                            | string       | BIDS run                        |
| epoch_start_sec                | float        | Start time within recording     |
| epoch_len_sec                  | float        | Epoch length                    |
| fs_hz                          | float        | Sampling rate used for features |
| ref                            | string       | Reference used                  |
| hpf_hz, lpf_hz, notch_hz       | float        | Effective filters               |
| bad_chan_mask                  | binary/bytes | Bitmask or JSON list            |
| y_seizure, y_IBI, ...          | int/bool     | Labels per epoch                |
| rater_set                      | string/json  | Raters contributing             |
| pipeline, version, params_hash | string       | Provenance                      |

---

## 8) Open Questions 
- Finalise the **controlled vocabulary** for neonatal events (align with ACNS neonatal terminology + HED/HED-SCORE terms where applicable).
- Decide **preferred raw** format when export is under our control: BrainVision (markers), BDF (24-bit), GDF, or NWB.
- Confirm anonymization policy (date-shift magnitude, PHI scrubbing rules, linkage key handling).
- Choose **derivatives** storage: Parquet (recommended) vs embedding in NWB/NIX.
- Define **QC thresholds** (impedance, line noise, flat channels) and automated flagging.

---

## References (non-exhaustive)
- **EEG-BIDS specification (current):** https://bids-specification.readthedocs.io/en/stable/modality-specific-files/electroencephalography.html  
  Pernet CR, et al. *EEG-BIDS, an extension to the brain imaging data structure for EEG.* *Sci Data* 2019. https://doi.org/10.1038/s41597-019-0104-8
- **BIDS Validator:** Web app https://bids-standard.github.io/bids-validator/ · GitHub https://github.com/bids-standard/bids-validator
- **EDF/EDF+ specifications:** EDF (1992) https://www.edfplus.info/specs/edf.html · EDF+ (2003) https://www.edfplus.info/specs/edfplus.html
- **BrainVision Core Data Format 1.0:** PDF https://www.fieldtriptoolbox.org/assets/pdf/BrainVisionCoreFileFormat_1.0_2018-08-02.pdf · Vendor page https://www.brainproducts.com/support-resources/brainvision-core-data-format-1-0/
- **DICOM Neurophysiology (WG-32):** Working Group page https://www.dicomstandard.org/activity/wgs/wg-32 · Neurophysiology Waveforms supplement https://www.dicomstandard.org/news/supplements/view/neurophysiology-waveforms · Halford et al., 2021 overview https://www.sciencedirect.com/science/article/abs/pii/S1388245721000523
- **NWB (Neurodata Without Borders):** https://nwb.org/ · Rübel O, et al. *The NWB ecosystem for neurophysiological data science.* *eLife* 2022;11:e78362. https://doi.org/10.7554/eLife.78362
- **NIX (INCF):** https://www.incf.org/sbp/nix · Martone M, et al. *NIX – Neuroscience information exchange format.* *F1000Research* 2020. https://f1000research.com/documents/9-358
- **odML (metadata):** odMLtables paper (Sprenger J, 2019) https://pmc.ncbi.nlm.nih.gov/articles/PMC6776611/ · Docs https://g-node.github.io/python-odml/
- **Neo (object model & I/O):** Homepage https://neuralensemble.org/neo/ · Garcia S, et al. *Neo: an object model for handling electrophysiology data.* *Front Neuroinform* 2014. https://pmc.ncbi.nlm.nih.gov/articles/PMC3930095/
- **MEF3 (Multiscale Electrophysiology Format):** Spec (PDF) https://osf.io/e3sf9/download · iEEG-Portal overview https://main.ieeg.org/?q=node%2F28
- **GDF (General Data Format):** Spec (arXiv) https://arxiv.org/abs/cs/0608052 · BioSig docs https://biosig.sourceforge.net/documentation.html
- **HED & HED-SCORE (controlled vocabularies):** HED overview paper (2016) https://www.frontiersin.org/articles/10.3389/fninf.2016.00042/full · HED site https://www.hedtags.org/ · HED-SCORE schema https://www.hedtags.org/display_hed.html?schema=score · (Update as needed with latest schema publications)
- **MNE-BIDS:** Docs https://mne.tools/mne-bids/stable/ · Anonymization guide https://mne.tools/mne-bids/stable/auto_examples/anonymize_dataset.html · Days-back utility https://mne.tools/mne-bids/stable/generated/mne_bids.get_anonymization_daysback.html · GitHub https://github.com/mne-tools/mne-bids

