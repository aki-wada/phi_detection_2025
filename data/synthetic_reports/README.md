# Synthetic Radiology Reports Dataset for PHI Detection

## Overview
Synthetic Japanese radiology reports for PHI detection model development. Contains no real patient information.

## Dataset
- **Total Reports**: 160 (100 training + 60 test)
- **Total PHI Instances**: 874
- **Language**: Japanese

## Files
- `Synthetic_reports_dataset_100.csv`: Training set (100 reports)
- `Synthetic_reports_dataset_60.csv`: Test set (60 reports)

## Schema
```csv
report_text: Full report text
phi_items: PHI annotations in JSON format


PHI Entity Types
The following PHI entity types are included:

PATIENT_NAME: Patient names (including kanji and hiragana)
PATIENT_ID: Patient identifiers
DATE_OF_BIRTH: Date of birth
AGE: Age information
GENDER: Gender/Sex
ADDRESS: Physical addresses (postal codes, building names, room numbers)
PHONE_NUMBER: Phone numbers (landline and mobile)
EMERGENCY_CONTACT: Emergency contact information (names, relationships, phone numbers)
DOCTOR_NAME: Physician names
FACILITY_NAME: Healthcare facility names
FACILITY_ADDRESS: Facility addresses
FACILITY_PHONE: Facility phone numbers
EXAMINATION_DATE: Examination dates and times
DEVICE_INFO: Medical device information (serial numbers, etc.)

Dataset Characteristics

Language: Japanese
Average Report Length: ~2,000-3,000 characters
PHI Density: Average of 5.5 PHI instances per report
Data Diversity:

Multiple healthcare facility types (university hospitals, clinics, health screening centers)
Various imaging modalities (MRI, CT, plain radiography, ultrasound)
Wide age range coverage (pediatric to geriatric)
