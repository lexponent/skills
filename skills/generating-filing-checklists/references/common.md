# Common research areas — applies to every filing

Topics typically covered by court rules across every filing type. Query the corpus for each before turning to class-specific material.

## Research areas

- Page format: paper size and quality, font (size, style, color), margins, line spacing and numbering
- Caption and first-page layout
- Pagination
- Footers
- Citation format (case citation requirements; the citation manual or style the jurisdiction prescribes, where specified)
- Privacy and redaction (SSNs, financial accounts, minors)
- Service deadlines and proof of service (extensions by service method, proof-of-service contents)
- Electronic filing technical requirements (PDF/text-searchable, bookmarks, signatures, court-day cutoffs)
- Lodged materials (separately submitted materials not attached to filings)

## Common output sections

These appear in nearly every checklist; sub-categories emerge from the rules.

Formatting:
- Paper / Physical Format
- Font
- Margins
- Spacing and Line Numbering
- First Page / Caption
- Footer
- Pagination
- Citation Format
- Privacy / Redaction
- Electronic Filing

Service and Filing Deadlines

Proof of Service

These are starter sections — extend or replace as the rules demand for the specific filing.

## Concept grep bank

Run every line below against **every** in-scope document with `grep_rules` (case-insensitive POSIX regex; batch within a line, one call per documentId, walk `has_more`). These are the universal concepts; the class file adds more. This is a floor — add terms for anything specific to the filing. Breadth is free: grep returns only matching rules, and this is the step that catches the one-line and cross-document requirements semantic search misses.

- Paper/layout: `paper|opaque|unglazed|inch|margin|spac|line.?number|paginat|footer|caption|bind|fasten|staple|rider|legib`
- Font: `font|typeface|type size|point|black|blue`
- Signature: `signature|sign|perjury|verif`
- Privacy: `redact|confidential|social security|financial account|privacy|under seal`
- Service & filing time: `serv(e|ed|ice)|proof of service|by mail|overnight|express mail|facsimile|electronic serv|court day|days before|deadline|extend`
- Citation/authority: `citation|style manual|bluebook|judicial notice|official report`
- Electronic filing: `electronic|e-fil|PDF|searchable|bookmark|envelope|signature`
