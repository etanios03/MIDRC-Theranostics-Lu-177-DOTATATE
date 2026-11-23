# MIDRC-Theranostics-Lu-177-DOTATATE

Extends the MIDRC Gen3 data model for MIDRC-Theranostics, with Lu-177 DOTATATE as an example use case

The model has **36 tables and 83 links**:
- **27 MIDRC tables, 5 modified** 
- **9 new Lu-177 DOTATATE tables**
- **12 new link connections**

---

## 1. Additions to existing MIDRC tables

| MIDRC table | Added property | Type | Description |
|---|---|---|---|
| `case` | `case_id` | string (required) | A de-identified identifier for the subject |
| `imaging_study` | `loinc_modality` | string | The LOINC method or imaging modality associated with the assigned LOINC code |
| `imaging_study` | `modality` | enum: `PET`, `CT`, `SPECT`, `MRI` | Modality of the imaging study, from DICOM tag (0008,0060) |
| `imaging_study` | `SUV_max` | number | Highest tracer concentration in a region of tissue |
| `imaging_study` | `mean_SUV` | number | Average radiopharmaceutical concentration within a region |
| `imaging_study` | `tumor_volume` | number | Volume of tumor |
| `imaging_study` | `scan_timepoint` | integer | Days from treatment cycle start at which the study was acquired (for linking scans taken at several timepoints) |
| `pt_series_file` | `injected_activity` | number | Injected activity |
| `pt_series_file` | `injection_time` | datetime | Time of injection |
| `pt_series_file` | `imaging_time` | datetime | Time of imaging |
| `pt_series_file` | `image_acquisition_params` | array | Parameters used for image acquisition |
| `pt_series_file` | `image_reconstruction_params` | array | Parameters used for image reconstruction |
| `pt_series_file` | `SSTR2_SUV_max` | number | SUVmax in SSTR2-expressing tissue on SSTR PET; supports eligibility assessment for Lu-177 DOTATATE |
| `pt_series_file` | `SSTR2_SUV_mean` | number | SUVmean over the SSTR2-expressing region of interest on SSTR PET |
| `ct_series_file` | `injected_activity`, `injection_time`, `imaging_time`, `image_acquisition_params`, `image_reconstruction_params` | number / datetime / datetime / array / array | Same acquisition fields as above |
| `nm_series_file` | `injected_activity`, `injection_time`, `imaging_time`, `image_acquisition_params`, `image_reconstruction_params` | number / datetime / datetime / array / array | Same acquisition fields as above; holds planar NM acquisitions |

---

## 2. New tables

| New table | Category | Parent table(s) | Purpose | Properties |
|---|---|---|---|---|
| `surgical_information` | medical_history | `case` | Surgery performed before or after theranostic therapy | `date_of_surgery`, `primary_tumor_removal`, `time_between_RPT_surgery`, `pre_tumor_volume`, `post_tumor_volume`, `debulking_threshold` |
| `prior_treatment` | medical_history | `case` | Other therapies before or alongside theranostic therapy (hormone, chemotherapy, somatostatin analog, etc.) | `therapy_type`, `start_therapy_date`, `end_therapy_date`, `time_between_RPT_other`, `pre_tumor_volume`, `post_tumor_volume`, `response_to_therapy`, `response_criteria` |
| `lesion` | clinical | `case` | Tumor information before RPT | `tumor_site`, `AJCC_stage`, `tumor_grade`, `extra_abdominal_mets`, `ki_67` |
| `treatment_course` | clinical | `case` | An overall course of radiopharmaceutical treatment | `radiopharmaceutical`, `therapy_type`, `number_of_cycles`, `start_date` |
| `therapy_cycle` | clinical | `treatment_course` | A single RPT cycle and the activity delivered | `cycle_number` (required), `injected_activity`, `therapeutic_target` |
| `laboratory_panel` | clinical | `therapy_cycle` | Lab results for a cycle (renal function, blood work) | `loinc_code`, `lab_value`, `lab_unit`, `collection_timepoint` |
| `dosimetry` | clinical | `therapy_cycle`, `imaging_study` | Absorbed dose for a target organ or lesion in a cycle | `target`, `target_id`, `mean`, `SD`, `D50`, `D90`, `lesion_volume`, `cycle_num`, `dosimetry_method`, `software_used`, `time_activity_curve_fit` |
| `spect_image_series` | data_file | `imaging_study` | SPECT imaging files from one acquisition | `injected_activity`, `administration_time`, `imaging_time`, `image_acquisition_params`, `image_reconstruction_params`, `collimator`, `image_units`, `calibration_factor` |
| `clinical_outcome` | clinical | `therapy_cycle`, `dosimetry` | Treatment response, survival and toxicity | `treatment_response`, `progression_free_survival`, `overall_survival`, `toxicity_grade`, `toxicity_organ`, `days_to_progression` |


---

## 3. New link connections

All new links are `one_to_many` and optional 

| Parent | → | Child | Link name |
|---|---|---|---|
| `case` | → | `surgical_information` | `cases` |
| `case` | → | `prior_treatment` | `cases` | 
| `case` | → | `lesion` | `cases` | 
| `case` | → | `treatment_course` | `cases` | 
| `treatment_course` | → | `therapy_cycle` | `treatment_courses` |
| `therapy_cycle` | → | `laboratory_panel` | `therapy_cycles` |
| `therapy_cycle` | → | `dosimetry` | `therapy_cycles`  |
| `therapy_cycle` | → | `imaging_study` | `therapy_cycles`  |
| `therapy_cycle` | → | `clinical_outcome` | `therapy_cycles`  |
| `imaging_study` | → | `dosimetry` | `imaging_studies`  |
| `imaging_study` | → | `spect_image_series` | `imaging_study_has_spect_image_series` |
| `dosimetry` | → | `clinical_outcome` | `dosimetries`  |


### Treatment hierarchy

```
case
├── surgical_information
├── prior_treatment
├── lesion
├── condition 
├── imaging_study 
└── treatment_course
    └── therapy_cycle
        ├── laboratory_panel
        ├── clinical_outcome
        ├── dosimetry ──► clinical_outcome
        └── imaging_study
            ├── dosimetry
            ├── spect_image_series
            └── pt / ct / nm_series_file  (MIDRC, with added theranostics fields)
```

---

## 4. Visualization

[Gen3schemadev package](https://github.com/AustralianBioCommons/gen3schemadev)

`gen3schemadev visualise -i lu_177_dotatate_midrc_bundled.json`


## 5. Local Gen3 Instance Implementation 

`gen3_instance_setup/gen3_setup_wsl.sh`
