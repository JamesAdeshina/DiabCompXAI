# Modalities Summary for Modeling Diabetic Complications

This document outlines the planned input modalities for the project:  
**"Early Prediction of Diabetic Complications Using Multi-Modal Deep Learning"** using the **MIMIC-IV** dataset.

---

## 1. Structured Data

| Feature | Description | Notes |
|--------|-------------|-------|
| Age | Age at first diabetes-related admission | Already extracted |
| Gender | Biological sex (M/F/O) | Check for imbalance |
| Ethnicity | Race/ethnicity group | Optional one-hot encoding |
| Admission type | Elective, emergency, urgent, etc. | Useful for stratifying severity |
| Insurance type | Optional socioeconomic proxy | May help model health access |

✅ **Status**: Extracted and available

---

## 2. Laboratory Test Results

| Lab Test | Description | Notes |
|----------|-------------|-------|
| Glucose | Point-in-time blood glucose | Core diabetes marker |
| Hemoglobin A1C | Average blood sugar (2–3 months) | May have missingness |
| Creatinine | Kidney function marker | Strong nephropathy predictor |
| Albumin | Protein level — may indicate malnutrition | Optional |
| ACR (Albumin:Creatinine Ratio) | Microalbuminuria — early nephropathy | Key for kidney risk |
| BUN | Additional kidney metric | Correlated with creatinine |
| LDL / HDL | Lipid profile for cardiovascular risk | May be incomplete |
| Triglycerides | Fat metabolism marker | Useful for CVD risk |
| CRP | Inflammation marker | Optional — check coverage |

✅ **Status**: Awaiting coverage check  
⚠️ **Action**: Assess missingness and distribution in `03_lab_coverage/`

---

## 3. Diagnoses (ICD Codes)

| Feature | Description | Notes |
|--------|-------------|-------|
| Diabetes confirmation | ICD-9/10 codes for T1DM, T2DM | Used to define diabetic cohort |
| Complications | Retinopathy, nephropathy, etc. | Used to generate binary/multi-label outcome flags |
| Comorbidities | Charlson or Elixhauser index | Can be numeric count or embedded vectors |

✅ **Status**: Codes defined, feature extraction in progress  
⚠️ **Challenge**: Align timing of complications with prediction window

---

## 4. Medications

| Feature | Description | Notes |
|--------|-------------|-------|
| Insulin | Indicates severe diabetes | Use dosage/frequency if available |
| Metformin | First-line medication | Binary (yes/no) or time-dependent |
| Statins | Used for lipid control | Proxy for cardiovascular prevention |
| ACE inhibitors / ARBs | Kidney protection | Useful for nephropathy modeling |

✅ **Status**: Will be extracted from prescriptions or inputevents  
⚠️ **Action**: Filter meds to diabetes and complication-related classes

---

## 5. Time Series (Vitals & Labs Over Time)

| Feature | Description | Notes |
|--------|-------------|-------|
| Heart rate | From `chartevents` | Can indicate infection or sepsis |
| Blood pressure | Systolic/diastolic trends | Cardiovascular and kidney impact |
| Respiratory rate | Optional | Useful in ICU settings |
| Lab trajectories | Glucose, creatinine, etc. over time | Enables temporal modeling (optional) |

✅ **Status**: Optional — if computational resources allow  
⚠️ **Action**: Aggregate by time-window or use with recurrent models

---

## Summary Table

| Modality | Source Table | Status | Priority |
|----------|--------------|--------|----------|
| Structured | admissions, patients | Ready | ✅ High |
| Labs | labevents | Pending QC | ✅ High |
| Diagnoses | diagnoses_icd | In progress | ✅ High |
| Medications | prescriptions | In progress | ✅ High |
| Time Series | chartevents, labevents | Optional | ⚠️ Medium

---




## 📊 Prediction Framework

### 🎯 Targets
Binary labels for whether a patient developed the following diabetic complications (1 = Yes, 0 = No):

- Retinopathy  
- Nephropathy  
- Neuropathy  
- Cardiovascular Disease  
- Foot Ulcer  

These are derived based on diagnosis dates anchored on the first diabetes diagnosis (N = 10,807).

---

### 🧠 Input Variables (Features)

#### 👤 Demographics
- `age_at_diag` – Age at first diabetes diagnosis (calculated from `dob` and `diag_date`)
- `gender`
- `ethnicity` *(if available in your dataset)*

#### 🔬 Lab Measurements (aggregated per patient over time windows)
- `glucose`
- `a1c`
- `creatinine`
- `albumin`
- `albumin_creatinine_ratio`
- `ldl`
- `hdl`
- `triglycerides`

Aggregation: Use statistical summaries (e.g., `mean`, `max`) across the valid post-diagnosis time window for each complication (e.g., 30–730 days).

#### 🏥 Admissions & Diagnoses
- `number_of_admissions` (prior to complication)
- `time_since_last_admit`
- `diagnosis_count_pre_event` *(optional)*

---

*These features will be used to train TabNet and MLP models for early prediction of diabetic complications.*

