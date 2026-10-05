# 🔐 PII Redaction Engine

A Python-based hybrid **Personally Identifiable Information (PII) detection and anonymization engine** for sensitive financial and legal documents.

The system combines **Gazetteer matching, Regex rules and spaCy Named Entity Recognition (NER)** to detect PII, resolve overlapping detections, generate consistent fictitious replacements and preserve DOCX structure and inline formatting.

> **Privacy note:** The source document and generated detection artifacts used during the original evaluation are intentionally excluded from the public version of this repository. Use your own test document and ground-truth data.

## ✨ Detection Pipeline

    DOCX
      │
      ├── Gazetteer matching ─┐
      ├── Regex detection ────┼──> Candidate spans
      └── spaCy NER ──────────┘
                  │
                  ▼
           Overlap resolution
                  │
                  ▼
          Consistent fake mapping
                  │
                  ▼
          DOCX run-level replacement
                  │
                  ▼
           Redacted DOCX + audit log

## 🧠 Engineering Highlights

- **Gazetteer matching** for known people, organizations and addresses.
- **Regex detection** for structured PII such as emails and phone numbers.
- **spaCy NER** for contextual PERSON, ORG and GPE entities.
- **Overlap resolution** to prevent duplicate replacements.
- **Consistent mapping** so the same entity receives the same replacement.
- **Formatting preservation** through DOCX run-level editing.
- **Merged-cell protection** to avoid repeated processing of shared Word paragraphs.

## 📊 Evaluation

The original evaluation used a manually verified ground-truth set containing **205 PII instances**.

| Metric | Result |
|---|---:|
| Precision | **74.40%** |
| Recall | **90.73%** |
| F1 Score | **81.76%** |
| Total detections | **250** |
| True positives | **186** |
| False positives | **64** |
| False negatives | **19** |

| Category | Precision | Recall | F1 |
|---|---:|---:|---:|
| Email | 100.00% | 97.50% | 98.73% |
| Phone | 83.33% | 100.00% | 90.91% |
| Name | 62.92% | 77.78% | 69.57% |
| Company | 74.39% | 100.00% | 85.31% |
| Address | 62.50% | 83.33% | 71.43% |

The main remaining challenge is **precision on names and organizations**, where general-purpose NER can misclassify capitalized financial/legal terminology.

## 🗂️ Project Structure

    .
    ├── redact_pii.py
    ├── evaluate_redactor.py
    ├── compile_ground_truth.py
    ├── evaluation_report.md
    ├── ground_truth.json
    └── README.md

Private input documents and generated PII audit logs should remain local and must not be committed to a public repository.

## ⚙️ Setup

Python 3.10+ is recommended.

    python -m venv .venv

Windows:
    .venv\Scripts\activate

macOS/Linux:
    source .venv/bin/activate

    pip install -r requirements.txt
    python -m spacy download en_core_web_sm

## ▶️ Run

Place your own input document in the project directory and configure the input/output paths in redact_pii.py.

    python redact_pii.py
    python evaluate_redactor.py

## 🔬 Trade-offs

- Gazetteers improve recall for known entities but do not generalize to unseen names.
- Regex provides strong precision for structured PII.
- Lightweight spaCy NER improves OOV coverage but introduces false positives in domain-specific documents.
- Address detection remains difficult without a dedicated document-layout/address parser.

## 🔒 Security

Do not commit original documents containing real PII, detection logs containing original values, API keys, credentials or local machine paths.

For a public demo, use synthetic or fully anonymized documents.

## License

No license is currently specified. Add a license before accepting external contributions.
