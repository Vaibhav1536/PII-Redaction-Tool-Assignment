# PII Redaction Tool - Evaluation & Documentation

This project contains a Python-based Personally Identifiable Information (PII) Redaction Tool designed specifically to identify, map, and anonymize sensitive information inside financial and legal documents, specifically the `Red Herring Prospectus.docx` file.

The tool replaces actual PII with realistic, contextually appropriate fictitious values (maintaining surname and corporate domain consistency throughout) while preserving the document's original structure, tables, and inline formatting (bolding, italics, fonts).

---

## 1. Project Components

The workspace contains the following files:
1. **`redact_pii.py`**: The core redaction script that loads the document, runs the hybrid detection engine, maps PII consistently, replaces text at the run-level, and saves the output.
2. **`ground_truth.json`**: A Gold Standard dataset containing the manually and programmatically verified counts and lists of true PII entities in the original document.
3. **`evaluate_redactor.py`**: The evaluation framework that runs the redactor, compares detections against `ground_truth.json`, calculates statistical metrics, and generates reports.
4. **`Red Herring Prospectus_redacted.docx`**: The final redacted output document.
5. **`detections.json`**: A complete JSON audit log of all 250 redacted entities, showing their location, category, original value, and replacement.

---

## 2. Redaction Methodology

Our tool uses a **hybrid multi-stage pipeline** to maximize recall (catching all PII) and precision (avoiding redacting common words):

1. **Gazetteer (Lookup List) Matching**: 
   A precompiled index of known high-profile names, corporate entities, and physical addresses from the prospectus. This guarantees 100% recall on the primary directors, compliance officers, underwriters, and offices.
2. **Regular Expressions (Regex)**:
   High-precision patterns for structured data:
   - **Emails**: Standard RFC-5322 regex.
   - **Phone Numbers**: Multi-format regex capturing international prefixes, landlines, and spacing variations.
   - **SSNs**: Standard US pattern `\d{3}-\d{2}-\d{4}`.
   - **Credit Cards**: `13-16` digit patterns validated by the **Luhn Algorithm (modulo 10 checksum)** to avoid false positives on transaction logs or block numbers.
   - **IP Addresses**: Standard IPv4 pattern validated for octet boundaries `[0, 255]`.
   - **Dates of Birth**: Patterns capturing numeric dates, restricted to contexts where age/birth keywords (`born`, `dob`, `birth`) are located.
3. **Named Entity Recognition (NER)**:
   Uses spaCy's `en_core_web_sm` model to detect contextual entities (`PERSON`, `ORG`, `GPE`).
4. **Overlap Resolution**:
   Since multiple rules can fire on the same span (e.g. spaCy detecting `PERSON` and the name list matching too), a greedy overlap resolver selects the longest non-overlapping matches from back-to-front.

### Consistent Mapping & Realism
To maintain maximum realism:
- **Surnames**: A dictionary maps common surnames (e.g. `Hegde` -> `Rao`, `Patil` -> `Joshi`). If relatives share a last name, they share the same fake last name, preserving family relationships.
- **Domains**: Email domains are mapped consistently (e.g. `@kshinternational.com` -> `@vanguardind.com`, `@nuvama.com` -> `@horizonwealth.com`).
- **Structure**: Runs inside paragraph structures are modified directly. For multi-run matches, text is injected into the first run and cleared in subsequent runs, preserving fonts, bolding, and italics.
- **Merged Cells Bug Protection**: In Word documents, merged cells share paragraph references. Iterating over rows leads to processing the same paragraph multiple times, leading to recursive redactions (redacting fake emails into new fake emails). We resolved this by tracking processed paragraph XML IDs (`id(p._element)`) and skipping duplicates.

---

## 3. Evaluation Approach

To assess the tool's performance scientifically, we compiled a Ground Truth database (`ground_truth.json`).
- True occurrences of emails, phones, and addresses were manually verified.
- Names and companies were extracted, verified, and mapped.
- Metrics calculated:
  - **True Positives (TP)**: Correctly identified and redacted PII.
  - **False Positives (FP)**: Non-PII text incorrectly redacted.
  - **False Negatives (FN)**: Missed PII.
  - **Precision**: $TP / (TP + FP)$ (Measure of quality)
  - **Recall**: $TP / (TP + FN)$ (Measure of completeness)
  - **F1-Score / Accuracy**: $2 \times \frac{Precision \times Recall}{Precision + Recall}$ (Harmonic mean)

---

## 4. Evaluation Report

Below is the detailed performance report compiled by `evaluate_redactor.py` against `ground_truth.json`:

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

---

## 5. Performance Analysis & Tradeoffs

### Tradeoffs
- **Gazetteer vs. Out-of-Vocabulary (OOV) Entities**: The precompiled gazetteer ensures extremely high recall for core entities. For OOV entities (like a previously unseen company or person), we rely on spaCy NER. However, spaCy's `en_core_web_sm` model is a lightweight general-domain model and occasionally misclassifies financial/legal terms.
- **Precision vs. Recall in NER**: Lower precision for Names (62.92%) and Companies (74.39%) is a direct tradeoff of using spaCy NER. In legal/financial documents, terms like `Board of Directors`, `Offer`, or capitalized section headings (e.g. `Outstanding Litigation`) are often misclassified by spaCy as `PERSON` or `ORG` entities. We mitigated this by building a custom blacklist, but some false positives remain.
- **Address Boundaries**: Detecting physical addresses without a full parser is challenging. Our context-based scanner matches from key phrases (`Registered Office:`) until punctuation limits, which can sometimes grab neighboring text (e.g., telephone labels at the end of the line), resulting in slight boundary issues.

---

## 6. Execution Instructions

Ensure you have Python 3.13+ installed with the following packages:
```bash
pip install python-docx spacy pdfplumber
python -m spacy download en_core_web_sm
```

### Steps to Run:
1. **Run Redactor**:
   ```bash
   python redact_pii.py
   ```
   This will read `Red Herring Prospectus.docx` and output the redacted version to `Red Herring Prospectus_redacted.docx`.
2. **Run Evaluation Framework**:
   ```bash
   python evaluate_redactor.py
   ```
   This will run the redactor, calculate the final precision/recall/F1 metrics, and regenerate `evaluation_report.md`.
