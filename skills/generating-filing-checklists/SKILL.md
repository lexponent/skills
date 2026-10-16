---
name: generating-filing-checklists
description: Generates a comprehensive checklist of formatting and content requirements for a court filing. Use when the user asks for requirements, a checklist, rules, or "what I need" for filing a specific document — e.g., "formatting requirements for an MSJ", "checklist for a motion to compel", "what do I need in my opposition brief".
---

# Formatting and Content Checklist

Produce a checklist of every requirement a filing of this type could need to satisfy — what must appear in the document's four corners, what must be attached, and how it must be laid out. Every item must be verifiable by reading the filing itself.

## Lexponent tools

The tools this skill names (`list_rulesets`, `list_documents`, `search_rules`, `grep_rules`, `find_rule_by_title`, `cite`) come from the Lexponent connector. Hosts show them with a prefix, for example `mcp__lexponent__search_rules` or `mcp__claude_ai_Lexponent__search_rules`. Before concluding that a Lexponent tool is unavailable, look for a tool whose name contains "lexponent" in any letter case and ends with the tool's name.

## Determine scope

Before retrieving any rules, determine the documents in scope.

1. Determine the sovereign of the matter: `fed` for federal court, or the two-letter state code (e.g. `ca`, `ny`, `tx`). If it isn't stated or clear from the matter, ask the user.
2. Call `list_rulesets` with `sovereign`. It returns every ruleset in that sovereign, each with a `label`, `description`, `tier` (`code`, `reg`, `high`, `appellate`, `trial`) and `division` (e.g. "Ninth Circuit"). Match the user's court to the court-tier rulesets by label. The `code` rulesets apply across the sovereign (rules of procedure, evidence code, substantive codes); include the ones relevant to the question.
3. Call `list_documents` with `rulesetIds: [...]` for the matched rulesets. The response includes each document's `id` (use this in subsequent calls) and `citationFormat` (use this when composing citation labels).

Use the returned `id`s for all later `search_rules`, `grep_rules`, `find_rule_by_title`, and `cite` calls.

## Output format

Mandatory structure:

```
# Formatting and Content Checklist
## [Filing type]
**Documents:** [list of rule documents the checklist is based on]

## Formatting
### [Sub-category]
- [ ] [Requirement]. [(Rule citation)](url)
- [ ] [Condition: triggering choice] [Conditional requirement]. [(Rule citation)](url)

## Content
### [Sub-category]
- [ ] [Requirement]. [(Rule citation)](url)
- [ ] [Condition: triggering choice] [Conditional requirement]. [(Rule citation)](url)
```

The top-level structure is mandatory: the title, the filing-type heading, the **Documents** line, and the two sections **## Formatting** and **## Content**. Sub-categories under each are filing-type-specific — they emerge from the rules retrieved, not a fixed taxonomy.

Example:

```
# Formatting and Content Checklist
## Motion to Strike or Tax Costs
**Documents:** California Rules of Court (Titles 1, 2, 3) · LASC Local Rules (Chapters 1–3) · California Code of Civil Procedure

## Formatting

### Margins
- [ ] Left margin is at least 1 inch. [(Cal. R. Ct. 2.107)](https://lexponent-rule-documents.s3.us-east-1.amazonaws.com/ca-crc-title-2.pdf#page=8)

### Memorandum Page Limits
- [ ] Opening memorandum does not exceed 15 pages. [(Cal. R. Ct. 3.1113(d))](https://lexponent-rule-documents.s3.us-east-1.amazonaws.com/ca-crc-title-3.pdf#page=171)
- [ ] [Condition: memorandum exceeds 10 pages] Memorandum includes a table of contents and table of authorities. [(Cal. R. Ct. 3.1113(f))](https://lexponent-rule-documents.s3.us-east-1.amazonaws.com/ca-crc-title-3.pdf#page=171)

## Content

### Service and Filing Deadlines
- [ ] Moving and supporting papers are served and filed at least 16 court days before the hearing. [(Code Civ. Proc., § 1005(b))](https://lexponent-rule-documents.s3.us-east-1.amazonaws.com/ca-code-ccp.pdf#page=804)
- [ ] [Condition: service by mail within California] 16-court-day period is increased by 5 calendar days. [(Code Civ. Proc., § 1005(b))](https://lexponent-rule-documents.s3.us-east-1.amazonaws.com/ca-code-ccp.pdf#page=804)
```

The **Documents** header lists the rule documents the checklist is based on (these define the jurisdiction). Sub-categories emerge from the rules, not a fixed taxonomy. If a rule offers a choice (e.g., "Times New Roman or Arial"), include both. Conditional rules apply to both Formatting and Content; flag with `[Condition: triggering choice]` at the start of the item — never omit on the assumption the choice doesn't apply. Use the `cite` MCP tool to produce verified citation links.

## What's in scope

Any requirement whose compliance produces something visible in the filing. The dimensions you'll encounter on any filing type:

- **Layout** — physical and visual properties of every page
- **Constituent documents** — what documents make up the filing
- **Content elements** — substantive recitals, citations, references, attachments within each document
- **Procedural artifacts** — anything in the filing that proves a required prerequisite procedure was followed

Every dimension applies to every filing type; specific items in each dimension vary — let the rules tell you what they are.

If complying with a rule requires something visible in the filing — an attached declaration, a recital, a particular document — the rule is in scope, even if the rule's text describes a hearing, a service procedure, or another off-the-page activity. Check what compliance produces, not what the rule's surface text describes.

Out of scope: filer actions that produce no artifact at all (paying a fee, calling the clerk, the act of serving).

## Approach

Identify the filing class from the filing type. The keywords below are common examples, not exhaustive — use semantic understanding of the filing description, not just literal string match.

- "complaint", "cross-complaint", "answer", "amended complaint" / "amendment to complaint", "statement of damages", "petition" (trial-court level, e.g., petition for administrative writ to a trial court) → **pleading**
- "interrogatories", "request for admission", "request for production", "demand for inspection", "responses to ..." → **discovery-request**
- "request for entry of default", "default judgment", "prove-up" → **default-package**
- "motion ...", "demurrer" (or jurisdiction's equivalent failure-to-state motion), "ex parte application" / "ex parte", "motion in limine", "opposition" / "reply" / "supplemental brief" (these are constituent papers within a motion event) → **motion**
- appellate-level filings ("brief on appeal", "opening brief", "respondent's brief", "petition for writ to an appellate court") → confirm the in-scope corpus covers appellate practice; if it doesn't, flag and ask the user
- default → **motion**

Read `references/common.md` (research areas universal across all filings) and `references/<class>.md` (class-specific) before researching. Class files currently available: `motion.md`, `pleading.md`, `default-package.md`, `discovery-request.md`. For other classes, use `common.md` plus general knowledge of the filing. The references list research areas, starter output sections, and a concept grep bank — treat the areas and bank as a floor to cover, not a fixed template; extend them as the rules demand.

Treat the dimensions as a research frame: for each category, query the rule corpus and capture any rules that impose a requirement. A category may yield several rules, one, or none. For each retrieved rule, place each requirement under the category it addresses — a single rule may contribute items to multiple categories. If a requirement doesn't fit any category, the frame is missing a category; extend it. Stop when every category has been checked.

Use the retrieval MCPs as a **discovery sweep**, not spot lookups. Recall — not extraction — is the bottleneck: once a rule comes back from the corpus it almost always lands in the checklist, so the whole job is making sure every requirement actually comes back.

**Work one document at a time.** Treat the in-scope `documentId`s as an explicit list and give each its own pass before moving on. Coverage is then a countable list — every document gets a pass — rather than a fuzzy sweep, and every call stays scoped to one document, so results come back inline instead of overflowing into a side-file you never read.

For each document:

1. **Grep it against the concept bank.** `common.md` ends with a **Concept grep bank** — regex terms for requirements that recur across filings — and some class files add a class-specific bank. Run the bank against this document with `grep_rules` as a few grouped patterns (one per concept area — layout, signatures, service, …); don't glue the whole bank into a single alternation, which matches so many rules the result overflows. Walk `has_more`. Grep returns only matching rules, so it cheaply surfaces the easy-to-miss one-liners (a font-color rule sitting between font-size and margins) and the local-rule echo of a state rule that semantic search skips. The bank is a **floor, not a ceiling**: add terms for anything specific to this filing.
2. **Add `search_rules`** for concepts whose wording the bank might miss, scoped to this one document. Single-document scope keeps the full-text results inline; a multi-document search returns up to ten full rules per document and overflows into a side-file. If a call overflows anyway, narrow and re-run — never fall back to working from memory.
3. **Follow cross-references with `find_rule_by_title`** — only to pull a rule named in something you already retrieved. It is lookup, not discovery; never seed it from rule numbers you happen to remember, or coverage caps at what you already know.
4. **Capture each requirement** under the category it addresses (one rule may feed several) before moving to the next document.

**Optional — parallelize across documents.** The per-document pass above is self-contained, so if a subagent/Task tool is available you may run the documents concurrently instead of in sequence: dispatch one subagent per in-scope document, each performing the same pass and **returning** its requirement list as text (return it — don't rely on file paths). Then merge the lists: drop true duplicates, but keep a state rule and its local-rule echo as separate lines, and preserve every citation verbatim. This is an optimization keyed on whether the tool exists, not a requirement — the sequential pass is the default, and the merge must not drop anything.

## Final action

Output the complete markdown checklist as your response, following the mandatory structure in "Output format" above. The response itself is the canonical artifact. If a writable working directory is available, also save the same markdown to `checklist.md` in it so the user can download the file; if no filesystem is available or the write fails, that is fine — the response remains canonical. Do not truncate or summarize the checklist in the response.
