# ADR-000: Start development with a small initial dataset

## Context

The original plan required 150 labeled documents before development.
Only about 10 were available in the first week.

## Decision

Start development with the available documents as a development set,
while collection continues in parallel. The test split (task T08) is
postponed until at least 50 documents are collected.

## Consequences

- Pipeline development is not blocked by data collection.
- Metrics from this phase are development signals only, not reported results.
- All versions of the same document (PDF, photos, related invoices)
  must stay in the same split to avoid data leakage.
