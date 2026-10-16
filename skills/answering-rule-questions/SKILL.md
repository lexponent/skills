---
name: answering-rule-questions
description: Answers questions about specific court rules, statutes, or procedures by retrieving and citing rule text. Use when the user asks a specific factual or procedural question that can be answered by citing one or more rules — e.g., "what's the deadline for X", "does the court require Y", "what's the meet-and-confer rule for Z".
---

# Rule Q&A

Answer rule-based questions by retrieving rule text from the in-scope documents and grounding every claim in specific rules.

## Lexponent tools

The tools this skill names (`list_rulesets`, `list_documents`, `search_rules`, `grep_rules`, `find_rule_by_title`, `cite`) come from the Lexponent connector. Hosts show them with a prefix, for example `mcp__lexponent__search_rules` or `mcp__claude_ai_Lexponent__search_rules`. Before concluding that a Lexponent tool is unavailable, look for a tool whose name contains "lexponent" in any letter case and ends with the tool's name.

## Determine scope

Before retrieving any rules, determine the documents in scope.

1. Determine the sovereign of the matter: `fed` for federal court, or the two-letter state code (e.g. `ca`, `ny`, `tx`). If it isn't stated or clear from the matter, ask the user.
2. Call `list_rulesets` with `sovereign`. It returns every ruleset in that sovereign, each with a `label`, `description`, `tier` (`code`, `reg`, `high`, `appellate`, `trial`) and `division` (e.g. "Ninth Circuit"). Match the user's court to the court-tier rulesets by label. The `code` rulesets apply across the sovereign (rules of procedure, evidence code, substantive codes); include the ones relevant to the question.
3. Call `list_documents` with `rulesetIds: [...]` for the matched rulesets. The response includes each document's `id` (use this in subsequent calls) and `citationFormat` (use this when composing citation labels).

Use the returned `id`s for all later `search_rules`, `grep_rules`, `find_rule_by_title`, and `cite` calls.

## Retrieval

1. **Retrieve.** Use `search_rules` for conceptual questions. Start with the user's question in natural-language form; rephrase or run variants on follow-ups to widen recall. Use `grep_rules` for keyword/phrase questions or to find rules using specific terminology. Always combine both. `search_rules` accepts a `documentIds` array — pass all documentIds in one call. `grep_rules` searches one document per call: **grep every in-scope document in the same turn** (one parallel call per documentId). Don't pre-filter to documents "likely to contain the term" — you can't know what a document contains without searching it, and procedural concepts routinely live across multiple in-scope documents (statutes, federal or state rules, local rules, general or standing orders) even when surface phrasing differs. Skipping documents because they seem unrelated is the exact failure mode this pattern exists to prevent.

2. **Follow cross-references with `find_rule_by_title`.** When a returned rule cites another rule by number ("as provided in Local Rule 3.9", "see CCP § 1008"), call `find_rule_by_title` with that identifier to retrieve the canonical text. Use this tool only when you have a specific rule identifier from a prior result — do not guess titles.

Cross-references often span documents (e.g., LA local → California Rules of Court → Code of Civil Procedure). Follow each one.

## Citations

**Every rule referenced in your answer MUST have a verified citation in a trailing `## References` section. The number of links under `## References` must equal the number of distinct rules referenced in the prose.** URLs and page numbers come from the `cite` tool — never type a URL by hand.

For each rule you intend to cite:

1. **Find the documentId.** Pull from the `documentId` field returned by `search_rules` or `find_rule_by_title`.
2. **Construct the `citationLabel`.** Use the `citationFormat` field returned for that document by `list_documents`. The format string contains placeholders like `[Rule Number]` and `[Sub-Section(s)]`; fill them with the rule's actual identifiers. For documents with multiple variants (e.g., a federal circuit doc carrying both FRAP and circuit local rules), pick the variant whose example matches the rule you're citing.
3. **Call `cite({documentId, page, citationLabel})`.** It returns `{citationLink: "[label](url#page=N)"}` — a markdown link grounded by DB lookup. Paste the returned `citationLink` verbatim under `## References`.

If `cite` returns `{error: "unknown-documentId"}`, re-check the `documentId` field on the result row. If you can't resolve the documentId, omit the citation rather than fabricate one; it's better to qualify the answer ("I cannot cite this authoritatively from the documents in scope") than to ship a wrong citation.

## Output format

Answer in prose. Reference rules naturally where applicable in plain text — for example:

> Under Local Rule 3.104(a), the deadline is...
> The Code of Civil Procedure § 659 requires...

These plain-text rule references are encouraged when they aid the answer. **Do not embed `citationLink` markdown links inline in the prose.**

End with a `## References` section. Emit one `citationLink` per line as a markdown list item, in any order:

```
[answer prose with plain-text rule references]

## References

- [citationLink1]
- [citationLink2]
- [citationLink3]
```

If no rules are cited (e.g., no rule directly addresses the question), omit the `## References` section entirely.

## Before responding

Verify before sending your answer:

1. Count the distinct rules referenced in the prose.
2. Count the `citationLink` entries you intend to put under `## References`.
3. The two counts must match. Every rule in the prose has a link under References; every link under References corresponds to a rule the prose actually references.
4. If you called `cite` and got back a `citationLink`, that link MUST appear under `## References` unless you also removed the corresponding rule from the prose.

A response that references rules in prose but ships without a `## References` section is incomplete. Re-check before sending.

## Do not

- Do not rely on general legal knowledge. If it's not in a retrieved rule, it doesn't exist for this question.
- Do not speculate about deadlines, procedures, or requirements that are not explicitly stated in the rules.
- Do not assume one jurisdiction's rule applies to another. Local rules override state rules only where explicitly stated.
- Do not type rule-document URLs by hand; always go through the `cite` tool.
- Do not embed citation markdown links inline in prose; use plain-text rule references in prose and put markdown links only in the trailing `## References` section.
