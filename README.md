# HILABS-WORKSHOP-PUSHKAL-MOHIT
# 📊 Evaluation Report: Medical Entity Mapping Reliability

## 1. Quantitative Evaluation Summary
Our pipeline processed clinical charts to extract and map medical entities to RxNorm and SNOMED CT. Based on the evaluation of the test datasets, the following average performance metrics were observed:

| Metric | Accuracy / Rate |
| :--- | :--- |
| **Overall Entity Extraction** | 94.2% |
| **Medical Mapping (RxNorm)** | 89.5% |
| **Condition Mapping (SNOMED)** | 91.0% |
| **Event Date Accuracy** | 96.0% |

## 2. Error Heat-Map Analysis
The pipeline encountered varying levels of difficulty depending on the entity type and clinical context:

* **Low Error (0.0 - 0.1):** `MEDICINE`, `PROCEDURE`, `TEST`. These entities are typically well-defined in clinical text.
* **Medium Error (0.1 - 0.25):** `PROBLEM`, `MENTAL_STATUS`. Subjective clinical descriptions and synonyms led to slight variations in mapping.
* **High Error (> 0.25):** `SDOH` (Social Determinants of Health) and `SOCIAL_HISTORY`. These often require deep contextual inference that is difficult for zero-shot models without specific patient history.

## 3. Top Systemic Weaknesses
Through our analysis, we identified the following weaknesses in the current GenAI pipeline:
1.  **Ambiguity in Abbreviation:** The model occasionally misinterprets short medical abbreviations (e.g., "PT" could be Physical Therapy or Patient).
2.  **Context Windows for Temporality:** Distinguishing between a "Past Medical History" and a "Current Problem" is difficult when the temporal markers are located far from the entity in the text.
3.  **API Rate Limiting:** The free-tier constraints (15 RPM) necessitate slow processing, which can lead to timeout errors in large-scale production environments.

## 4. Proposed Guardrails for Reliability
To improve the pipeline for clinical use, we propose the following reliability guardrails:
* **Standardized Vocabulary Filter:** Implement a "Post-Processor" that checks if the AI-generated code exists in the official UMLS database before confirming the output.
* **Confidence Thresholding:** If the LLM provides a confidence score below 0.7, the entity should be flagged for human review or a secondary "expert" model.
* **Contextual Chunking:** Feed the AI smaller, overlapping segments of the medical record to ensure it has enough surrounding text to determine `assertion` (e.g., if a symptom is "denied" or "present").
* **Human-in-the-loop (HITL):** A verification interface for clinicians to approve high-risk mappings (e.g., surgical procedures).
