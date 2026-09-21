# NotaFlow Scope

Living document: defines what the system extracts, in which format, and how success is measured. Any scope change must be recorded here before it is implemented.

## 1. Problem

People and companies receive Brazilian invoices as PDFs, photos and paper, and type their data manually into spreadsheets and systems. NotaFlow extracts this data automatically, validates the result and routes to human review only the cases where confidence is not high enough.

## 2. Documents in scope

| Type         | Description                                                                        | Input formats                  |
| ------------ | ---------------------------------------------------------------------------------- | ------------------------------ |
| NF-e (DANFE) | Brazilian electronic invoice for goods, represented by its printed version (DANFE) | Native PDF, scanned PDF, photo |
| NFC-e        | Brazilian electronic consumer receipt (supermarkets, stores, pharmacies)           | Photo, PDF                     |

### Input constraints

- Accepted formats: PDF, JPG, PNG and HEIC.
- Maximum size: 10 MB per file.
- Up to 5 pages per document (invoices with many items may span more than one page).
- One document per file.
- Language: Brazilian Portuguese.

## 3. Out of scope (v1)

- NFS-e (service invoices): each municipality uses a different layout.
- CT-e, MDF-e, bank slips (boletos), contracts and handwritten receipts.
- Buyer data (CPF, name, address): not extracted, following data minimization principles.
- Detailed taxes (ICMS, IPI, PIS, COFINS), NCM, CFOP and product codes.
- Integration with accounting systems or ERPs.

These items may be added in future versions and are tracked in the backlog.

## 4. Extracted fields

### 4.1 Document fields

| Field             | Type    | Format                             | Required | Critical |
| ----------------- | ------- | ---------------------------------- | -------- | -------- |
| `document_type`   | enum    | `nfe` or `nfce`                    | Yes      | No       |
| `access_key`      | string  | 44 digits, no spaces               | Yes      | Yes      |
| `number`          | string  | Digits only, no leading zeros      | Yes      | No       |
| `issuer_cnpj`     | string  | 14 digits, no punctuation          | Yes      | Yes      |
| `issuer_name`     | string  | As printed on the document         | Yes      | No       |
| `issue_date`      | date    | `YYYY-MM-DD`                       | Yes      | Yes      |
| `discount_amount` | decimal | 2 decimal places, dot as separator | No       | No       |
| `total_amount`    | decimal | 2 decimal places, dot as separator | Yes      | Yes      |
| `items`           | list    | See section 4.2                    | Yes      | No       |

### 4.2 Item fields

| Field         | Type    | Format                        |
| ------------- | ------- | ----------------------------- |
| `description` | string  | As printed on the document    |
| `quantity`    | decimal | Up to 4 decimal places        |
| `unit`        | string  | As printed (UN, KG, CX, etc.) |
| `unit_price`  | decimal | Up to 4 decimal places        |
| `item_total`  | decimal | 2 decimal places              |

### 4.3 Evidence

For each critical field, the system also returns the excerpt of the document from which the value was read (`<field>_evidence`). Evidence is used during validation and displayed in the review interface.

### 4.4 General rules

- Field not found in the document: return `null`. The system must never make up values.
- Monetary values are always in Brazilian reais (BRL).
- Dates that include a time are reduced to the date.
- Text is kept as it appears on the document, with extra whitespace removed.

## 5. Critical fields

Critical fields are those whose errors cause direct harm to the user: `access_key`, `issuer_cnpj`, `issue_date` and `total_amount`. A document can only be auto-approved if all critical fields have high confidence.

## 6. Success metrics

| Metric                  | Definition                                                          | Initial target                      |
| ----------------------- | ------------------------------------------------------------------- | ----------------------------------- |
| Critical field accuracy | % of correct critical fields in the test set                        | ≥ 98%                               |
| Overall field accuracy  | % of all fields correct                                             | ≥ 95%                               |
| Automation rate         | % of documents approved without human review                        | ≥ 70%                               |
| Silent error rate       | % of auto-approved documents with at least one wrong critical field | ≤ 1%                                |
| Cost                    | LLM cost per 1,000 documents                                        | Defined after the baseline (week 2) |
| Latency                 | Processing time per document, p95                                   | ≤ 60 seconds                        |

The most important metric is the silent error rate: sending more documents to review is preferable to approving a wrong one.

Targets are initial and may be revised after the week 2 experiments, with the rationale recorded in `docs/decisions/`.

## 7. Comparison criteria

Used by the evaluation script to decide whether a field is correct:

- `access_key`, `issuer_cnpj`, `number`: exact match after removing non-digit characters.
- `issue_date`: same date.
- Monetary values: difference of at most BRL 0.01.
- Text (`issuer_name`, `description`): similarity ≥ 90 (rapidfuzz), after normalizing case and whitespace.
- `items`: item-by-item matching by description before comparing fields.

## 8. Privacy

- Documents contain personal data and are handled in compliance with the LGPD (Brazil's General Data Protection Law).
- Only the fields listed in section 4 are extracted.
- Original files are automatically deleted after 30 days.
- Extracted data never appears in logs.
- Third-party documents only enter the dataset with their owner's authorization.
