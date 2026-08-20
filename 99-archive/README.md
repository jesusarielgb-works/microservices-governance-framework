# 99 — Archive

> **What is this?** Documents that are no longer active but are kept for historical context.
> Nothing is ever deleted — it is archived.

## Why keep obsolete documents

- Rejected ADRs explain why a certain decision was NOT made (avoids repeating discussions)
- Deprecated documents show the evolution of the system
- The decision history helps understand the present

## Structure

```
99-archive/
├── deprecated/         ← Documents replaced by others (move here with a note)
└── old-decisions/      ← Proposals and decisions that did not move forward
```

## How to archive a document

1. Add at the beginning of the file:
   ```markdown
   > ⚠️ ARCHIVED — [Date]
   > Replaced by: [link to the new document]
   > Reason: [brief explanation]
   ```
2. Move the file to the corresponding folder
3. Update any links that pointed to the original file
