# Dataset

Raw documents and labels are not versioned because they contain personal data.

## Structure

- `raw/`: original documents (PDF, images) and their NF-e/NFC-e XML files
- `labels/`: ground truth JSON files, one per document
- `manifest.csv`: document catalog (created in task T04)

## Sources

- Personal purchase invoices received by email (NF-e and NFC-e with XML)
- Photos of personal consumer receipts (NFC-e), validated through the SEFAZ QR code lookup
- Third-party documents, only with the owner's authorization

## Current status

- 9 documents (6 NF-e, 3 NFC-e), from 5 different issuer systems
- Collection is ongoing; target: 150 documents
