# Handbook Part 1: acquisition of continuous neonatal EEG
*Summary of technical recommendations for acquiring continuous EEG (cEEG) for neonates. This is not
a SOP. Always follow local clinical policy and formal guidelines.*

> **⚠️ Disclaimer ⚠️**  
> This handbook provides a summary of guidelines and is not **advisory**. If anything here conflicts
> with your hospital’s policies or with ACNS/IFCN/ILAE guidance, **follow the policy/guideline** and
> the attending neurophysiologist’s instructions. See References [below](#references--key-guidance-selected) for key guidelines.


Contributions welcome! Please open a PR or issue on [GitHub](https://github.com/ai4nicu/data-hand)
or contact the authors.

---

## 1) Scope & prerequisites
- **Population:** Preterm and term neonates (typically < 48 weeks postmenstrual age) monitored continuously in the NICU.  
- **Team:** Qualified neurodiagnostic technologist sets up and maintains the study; interpreting physician provides oversight; bedside nursing annotates events.  
- **Synchronized video** is recommended.

---

## 2) Equipment & core digital settings
**EEG system**
- Digital amplifier meeting minimum technical requirements for clinical EEG.  
- **Sampling rate:** ≥ **200 Hz** (≥ **512 Hz** preferred).  
- **Bit depth:** ≥ **16-bit** A/D resolution (24-bit common with modern amplifiers).  
- **Acquisition filters:** Use conservative defaults to preserve diagnostic bandwidth; for example a
  high-pass with a cut-off of 0.08~Hz in addition to an anti-aliasing filter. 
  
**Amplifier specifications (input impedance & CMRR)**
- **Input impedance:** Modern clinical EEG amplifiers typically provide **≥ 1 GΩ per input**. High
  input impedance helps minimize signal loss across electrode–skin impedances and reduces
  sensitivity to small impedance mismatches, which is important for low-amplitude neonatal signals.
- **Recommendation:** Use an amplifier with **≥ 100 MΩ** input impedance (preferably **≥ 1
  GΩ**). Keep electrode impedances **low and balanced** to preserve signal quality.
- **Common-mode rejection ratio (CMRR):** Aim for **≥ 100 dB at power-line frequency (50/60 Hz)**;
  **≥ 110 dB** is preferable. Effective CMRR in practice depends on **balanced electrode
  impedances**, good lead routing, and proper grounding per manufacturer instructions.
- **Practical note:** High input impedance and high CMRR work together—**balanced impedances** and
  careful cable management are essential to achieve the amplifier’s specified performance. Follow
  your device manual and local biomedical engineering policy.
  

**Video & storage**
- Continuous, synchronized **video** recording; maintain framing and verify integrity during the study.  
- If trends can be regenerated from raw EEG, separate storage of trend files is optional—**follow
  local retention policy**.
- Storage EEG recordings in a format that preserves raw data and metadata, e.g. manufacturer’s
  native format.  Avoid exporting to lossy formats (e.g. EDF or EDF+) unless necessary.

---

## 3) Electrodes, impedances & safety
**Type & application**
- Use **Ag/AgCl or gold disk/cup** electrodes; central fill-hole cups help re-gel during long recordings.  
- **Avoid needle electrodes** for routine neonatal EEG; if used in an emergency where cup/disk
  electrodes cannot be applied promptly, use them temporarily and replace with surface electrodes as
  soon as feasible per local policy and ACNS guidance.
- Fixation via paste or collodion per lab policy. Keep impedances **low and balanced** across
  channels; re-check periodically. With modern EEG amplifiers, **< 10 kΩ** is usually sufficient but
  a balanced impedance across channels is more important than absolute values.

**Skin care**
- Neonatal skin is fragile: pad leads, avoid wire coils/pressure points, and coordinate with nursing
  to protect compromised skin.

---

## 4) Neonatal channel set & montages
**Minimum channels**
- At least **12 cerebral channels** **plus** polygraph channels (ECG, respiration; consider EOG and
  submental EMG) are recommended by ACNS. 

**Layout (reduced neonatal array)**
- Include **midline** (e.g., **Cz**, often Fz/Pz) and bilateral frontal–central–temporal–occipital coverage scaled to head size.  
- Use neonatal **bipolar chains** and/or mixed referential/bipolar montages; include **Cz** to
  improve detection of rolandic sharps and some seizures.

**When to extend coverage**
- If head size permits or focality is suspected, extend toward IFCN’s **10–10 “basic array”**
  (including the inferior temporal chain). Most neonates still require a reduced array—use clinical
  judgment.

---

## 5) Polygraphy (recommended)
Add non-EEG channels to aid **state classification** and artifact recognition:  
- **ECG (routine)** and **respiration** (1–3 channels depending on support), plus **EOG**
  (horizontal/vertical) and **submental EMG** as available.

---

## 6) Default display settings (typical neonatal)
- **Sensitivity:** start at **~ 7 µV/mm**; adjust as needed (briefly increase sensitivity to inspect low-voltage fast activity).  
- **Low-frequency stop filter (LFF):** **0.3–0.6 Hz** (frequency cutoff at −3 dB).  
- **High-frequency stop filter (HFF):** **~ 70 Hz** for neonatal EEG.  
- **Polygraph defaults:**  
  - EOG: similar sensitivity, LFF 0.3–0.6 Hz, HFF ~ 70 Hz  
  - EMG: LFF ~ 5 Hz, HFF ~ 70 Hz  
  - Resp: LFF 0.3–0.6 Hz

> **Note:** Report the **filter type, order** and **−3 dB (or −6 dB) frequencies** explicitly.

---

## 7) Patient preparation & hookup workflow
- **Coordinate** with nursing; plan around a **feed** (apply electrodes first, feed before recording) to promote natural sleep.  
- **Sleep state:** EEG recordings are typically best during **natural sleep**. If sedation is
  present for other clinical care, **document** agents/doses/times and note potential EEG effects
  (per local policy and guidelines).
- **Cable management:** leave slack; route away from ventilator tubing/lines; avoid loops; secure leads outside incubator hinge paths.  
- **Safety checks:** confirm alarms; ensure emergency access to the infant; clarify MRI/CT compatibility plan if imaging is likely.

---

## 8) Recording duration & state sampling
- **Background/state characterization:** aim for **≥ 60 minutes** to capture **wakefulness, active sleep, and quiet sleep** (20–30 min is usually insufficient).  
- **High-risk seizure screening (cEEG):** **~ 24 hours**; if electrographic seizures are detected, **continue until ≥ 24 h seizure-free**, unless directed otherwise by the interpreting neurologist.  
- **Avoid repetitive photic stimulation** in neonates.

---

## 9) Video, event annotation & bedside observation
- Keep **synchronized video** running and the patient centered in view; adjust camera as needed.  
- **Annotations:** bedside staff mark clinical events (care procedures, medication, ventilator
changes, suspected seizures etc.) using system event markers and a brief log.  
- **Observer role:** A trained bedside observer can watch video/trends and escalate concerns; this **does not** replace qualified EEG interpretation.

---

## 10) Artifact avoidance & NICU-specific tips
- **Respiratory/CPAP/ECMO artifacts:** use respiration channels and ECG to differentiate cerebral from physiologic signals; optimize lead routing/fixation to reduce vibration pickup.  
- **Myogenic/movement:** note swaddling comfort; pacifier sucking patterns are visible on video; consider brief pauses in non-essential care during key epochs.  
- **Electrical noise:** verify incubator/heater grounding per biomedical policy; avoid electrode
  wire coils (also reduces RF risk near MRI).

---

## 11) Complementary aEEG (when present)
- aEEG trends can aid background surveillance and flag suspect periods but are **less sensitive**
  than conventional cEEG for neonatal seizure detection. Use aEEG **as an adjunct**, not a
  replacement (e.g., trend C3/P3 & C4/P4 alongside full-array EEG).

---

## 12) Ongoing maintenance, review cadence & reporting
- **Maintenance:** re-gel/refix electrodes proactively; re-check impedances and camera view each shift or when artifacts increase.  
- **Review cadence:** continuous acquisition with **intermittent technologist checks** and **intermittent physician interpretation** at a frequency defined by your service protocol and clinical status.  
- **Transparent reporting:** include **sampling rate, bit depth, LFF/HFF with −3 dB points,
  amplifier type, montage(s), electrode set & locations, impedance range, polygraph channels, states
  captured,** and any quantitative/trend methods used.

---

## References & key guidance (selected)
- **American Clinical Neurophysiology Society (ACNS)**
  - Guideline 5: *Minimum Technical Standards for Pediatric Electroencephalography*. *J Clin Neurophysiol* (2016). [doi:10.1097/WNP.0000000000000321](https://doi.org/10.1097/WNP.0000000000000321)  
  - Shellhaas RA, Chang T, Tsuchida TN, et al. *The American Clinical Neurophysiology Society’s Guideline on Continuous Electroencephalography Monitoring in Neonates*. *J Clin Neurophysiol* (2011). [doi:10.1097/WNP.0b013e31823e96d7](https://doi.org/10.1097/WNP.0b013e31823e96d7)  
  - Guideline 1: *Minimum Technical Requirements for Performing Clinical Electroencephalography*. *J Clin Neurophysiol* (2016). [doi:10.1097/WNP.0000000000000308](https://doi.org/10.1097/WNP.0000000000000308)  
  - Guideline 4: *Recording Clinical EEG on Digital Media*. *Neurodiagnostic Journal*
    (2016). [doi:10.1080/21646821.2016.1245563](https://doi.org/10.1080/21646821.2016.1245563)

- **International Federation of Clinical Neurophysiology (IFCN)**
  - Seeck M, Koessler L, Bast T, et al. *The standardized EEG electrode array of the IFCN*. *Clin
    Neurophysiol*
    (2017). [doi:10.1016/j.clinph.2017.06.254](https://doi.org/10.1016/j.clinph.2017.06.254)
	
- **International League Against Epilepsy (ILAE) — neonate-focused**
  - Pressler RM, Cilio MR, Mizrahi EM, et al. *The ILAE classification of seizures and the epilepsies: Modification for seizures in the neonate. Position paper by the ILAE Task Force on Neonatal Seizures.* *Epilepsia.* 2021;62(3):615–628. [doi:10.1111/epi.16815](https://doi.org/10.1111/epi.16815)  
  - Pressler RM, Abend NS, Auvin S, et al. *Treatment of seizures in the neonate: Guidelines and consensus-based recommendations—Special report from the ILAE Task Force on Neonatal Seizures.* *Epilepsia.* 2023;64(10):2550–2570. [doi:10.1111/epi.17745](https://doi.org/10.1111/epi.17745)
	
- **IFCN & ILAE (joint technical standards)**
  - Peltola ME, Beniczky S, Crawford P, et al. *Routine and sleep EEG: Minimum recording standards of the International Federation of Clinical Neurophysiology and the International League Against Epilepsy.* *Epilepsia.* 2023;64(3):509–527. [doi:10.1111/epi.17448](https://doi.org/10.1111/epi.17448)

- **Science-focused reporting standard (all ages)**
  - Keil A, Debener S, Gratton G, et al. *Committee report: Publication guidelines and
    recommendations for studies using EEG/MEG*. *Psychophysiology*
    (2014). [doi:10.1111/psyp.12147](https://doi.org/10.1111/psyp.12147)




