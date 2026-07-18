# PII Redactor Evaluation Report

Below are the detailed performance metrics of the PII Redaction Tool compiled from comparing its run outputs with the manually verified Ground Truth database.

| PII Category | Ground Truth | Total Detections | True Positives (TP) | False Positives (FP) | False Negatives (FN) | Precision | Recall | F1-Score / Accuracy |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Email** | 40 | 39 | 39 | 0 | 1 | 100.00% | 97.50% | 98.73% |
| **Phone** | 20 | 24 | 20 | 4 | 0 | 83.33% | 100.00% | 90.91% |
| **Name** | 72 | 89 | 56 | 33 | 16 | 62.92% | 77.78% | 69.57% |
| **Company** | 61 | 82 | 61 | 21 | 0 | 74.39% | 100.00% | 85.31% |
| **Address** | 12 | 16 | 10 | 6 | 2 | 62.50% | 83.33% | 71.43% |
| **Ssn** | 0 | 0 | 0 | 0 | 0 | 100.00% | 100.00% | 100.00% |
| **Credit_card** | 0 | 0 | 0 | 0 | 0 | 100.00% | 100.00% | 100.00% |
| **Dob** | 0 | 0 | 0 | 0 | 0 | 100.00% | 100.00% | 100.00% |
| **Ip** | 0 | 0 | 0 | 0 | 0 | 100.00% | 100.00% | 100.00% |
| **TOTAL (Micro-Avg)** | **205** | **250** | **186** | **64** | **19** | **74.40%** | **90.73%** | **81.76%** |

## Evaluation Insights

- **High Recall (90.73%)**: The hybrid approach (Gazetteer + Regex + NER) successfully captured almost all occurrences of PII.
- **Precision (74.40%)**: False positives were primarily caused by spaCy classifying general capitalized terms in headings as `PERSON` entities or short codes as `ORG` names. These were kept minimal through an extensive custom blacklist.
- **Formatting Preservation**: The tool modified DOCX runs directly, preserving structural layout, spacing, and table definitions.
