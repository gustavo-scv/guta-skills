---
name: remove-ai-signs
description: Revise drafts to remove common AI-writing signs while preserving meaning, facts, citations, voice, and user intent. Use when the user asks to humanize, de-AI, remove AI tells, make text sound less AI-generated, clean up chatbot prose, or revise output using Wikipedia-style AI writing sign guidelines.
---

# Remove AI Signs

Revise prose by fixing the underlying writing problems that make it look generated: generic significance claims, superficial analysis, formulaic structure, source theater, formatting artifacts, and unsupported assertions. Do not merely disguise AI output; preserve truth, context, and accountability.

## Operating Rules

- Preserve the author's intended meaning, factual claims, citations, constraints, and level of formality unless the user asks for a stronger edit.
- Prefer deletion over replacement for unsupported praise, broad legacy claims, invented analysis, or filler.
- Replace vague importance with specific evidence: dates, actions, mechanisms, consequences, named sources, or concrete examples.
- Flag claims that need source checking instead of laundering them into smoother prose.
- Do not accuse a writer of using AI. Treat AI signs as revision cues, not proof.
- Do not make text worse by adding typos, awkwardness, slang, random sentence fragments, or artificial imperfection.

## Workflow

1. Identify the target genre.
   - For Wikipedia or encyclopedic text: use neutral, source-bound prose. Remove synthesis, promotional language, and unsupported notability claims.
   - For professional writing: keep the user's voice and purpose, but reduce canned framing and inflated stakes.
   - For technical or product text: keep precision, warnings, and requirements even if they sound formal.

2. Audit for AI-sign clusters.
   - Generic significance: "plays a vital role", "stands as a testament", "underscores its importance", "marks a pivotal moment", "reflects broader trends".
   - Superficial analysis: dangling "-ing" clauses that add vague interpretation, such as "highlighting", "ensuring", "reflecting", "fostering", or "contributing to".
   - Source theater: claims that sources are "independent", "widely covered", "high quality", or "significant" without showing what they actually establish.
   - Over-positive smoothing: replacing specific facts with praise, legacy, resilience, transformation, impact, or "rich cultural heritage".
   - Formulaic structure: "In conclusion", "Overall", repetitive paragraph summaries, balanced but empty transitions, or every paragraph ending with a lesson.
   - Stiff diction: "utilized", "authored", "relocated", "attempted", "facilitated", "showcased", "robust", "dynamic", "notable", when plainer words fit.
   - Formatting artifacts: Markdown bold in plain prose, fake section headings, placeholder citations, malformed links, excessive bullet lists, or copied chatbot refusal language.

3. Rewrite with evidence and restraint.
   - Cut unsupported significance sentences.
   - Convert vague analysis into attributable facts, or remove it.
   - Replace stock transitions with the real relationship between ideas.
   - Use ordinary verbs and nouns.
   - Shorten paragraphs that restate the same point.
   - Keep necessary caveats, but remove didactic openers like "it is important to note" when the sentence can state the caveat directly.

4. Verify the result.
   - Every remaining evaluative claim is attributed, sourced, or clearly framed as the author's own stance.
   - No citations are invented, overclaimed, or made to support analysis they do not contain.
   - The text has concrete details where the original had broad claims.
   - The prose fits the target genre instead of sounding like generic essay or article filler.
   - Formatting is valid for the destination: Markdown, wikitext, plain text, email, or docs.

## Rewrite Patterns

Use these transformations as defaults:

- "X plays a crucial role in Y" -> State the actual function X performs in Y.
- "This highlights the importance of X" -> Delete, unless a source explicitly makes that interpretation.
- "X has been covered by prominent media outlets" -> Name the specific coverage only when relevant, and state what it verifies.
- "It is important to note that rules vary" -> "Rules vary by jurisdiction."
- "The project fostered innovation and collaboration" -> Describe who collaborated, what changed, and what resulted.
- "In conclusion" section -> Remove unless the genre requires a conclusion; otherwise end on the final substantive point.

## Output Style

When the user asks for a rewrite, provide the revised text first. Add brief notes only for material cuts, unresolved factual risks, citation problems, or places where source checking is needed.

When the user asks for an audit, list the strongest issues first and quote only short excerpts needed to identify them. Explain the fix in practical terms.

When the user asks for both, return:

1. Revised text
2. Notes on removed AI-sign patterns
3. Source or fact-check risks, if any

## False-Positive Guardrails

Do not treat these as problems by themselves: perfect grammar, formal tone, transition words, correct markup, ordinary citations, or a mix of casual and formal register. Revise only when a pattern weakens accuracy, specificity, neutrality, or readability.
