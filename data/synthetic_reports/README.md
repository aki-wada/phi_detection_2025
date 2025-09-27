# Synthetic Radiology Reports Dataset for PHI Detection

## Overview
This directory contains a synthetic radiology report dataset created for developing and evaluating PHI (Protected Health Information) detection systems. All data is artificially generated and contains no real patient information.

## Dataset Composition
- **Total Reports**: 160
- **Total PHI Instances**: 874
- **Report Types**: Radiology examination reports (MRI, CT, X-ray, etc.)
- **Language**: Japanese medical reports

## File Descriptions

### 📄 Synthetic_reports_dataset_100.csv
- **Number of Reports**: 100
- **Purpose**: Training/Development dataset
- **Schema**:
  - `report_text`: Full radiology report text (Japanese)
  - `phi_items`: PHI annotation information (JSON format)

### 📄 Synthetic_reports_dataset_60.csv
- **Number of Reports**: 60
- **Purpose**: Test/Evaluation dataset
- **Schema**:
  - `report_text`: Full radiology report text (Japanese)
  - `FinalPhase`: Final phase PHI extraction results
  - `Completion result`: Model output results

## PHI Annotation Format
```json
{
  "report_id": "rep-mri-20240806-7a8b9c2d",
  "phi_annotations": [
    {
      "entity_type": "PATIENT_NAME",
      "text": "佐藤明子",
      "start": 45,
      "end": 49
    },
    {
      "entity_type": "PATIENT_ID", 
      "text": "MR2024-78234",
      "start": 72,
      "end": 84
    }
  ]
}
