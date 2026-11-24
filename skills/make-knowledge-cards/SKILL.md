---
name: make-knowledge-cards
description: Converts an article pasted in the conversation or a local Markdown/TXT file into 5-8 knowledge cards, each with a title, core knowledge, a concise explanation, and an example or self-test question. Use when the user asks to turn an article, note, essay, or text file into knowledge cards, flashcards, 知识卡片, or study notes. Does not fetch web pages and does not process PDF, Anki decks, or graphical interfaces.
agent_created: true
---

# Make Knowledge Cards

## Overview

Turn a source text into a small set of high-density knowledge cards. Every statement on a card must be traceable to the source text; never add outside knowledge. The target is 5-8 cards, but the correct count is the number of genuinely distinct, important points the source actually supports. Output fewer cards when the source is thin; never pad to reach a number.

## Scope and Input Handling

Apply this decision table before extracting any point:

| Input | Action |
| --- | --- |
| Article text pasted in the conversation | Process the text, even if a URL appears alongside it; ignore the URL |
| Local `.md` / `.txt` file path | Read the complete file, then process it |
| URL with no text and no local file | Do not fetch. Ask the user to paste the text or save it as a local `.md` / `.txt` file |
| PDF, Anki deck/export, image, or other unsupported file | Do not process. Advise converting it to `.md` / `.txt` first, or pasting the text |
| No text and no file | Ask the user to provide the content |

Never fetch web pages, open PDFs, generate Anki decks, or use GUI operations. Preserve the source wording for key terms, numbers, names, and quoted claims.

## Workflow

1. Read the source completely. Identify the topic, the author's central claim or purpose, and the supporting points.
2. Build a scratch list of candidate knowledge points. Candidate types: concept definition, mechanism, cause-effect relation, distinction, method, key datum, conclusion, and an explicit author preference/evaluation when it carries the central claim.
3. Rank candidates with the deletion test: remove a candidate; if no remaining card's 核心知识 loses its basis and no actionable action disappears, drop the candidate. Descriptive, aesthetic, or offhand observations (e.g. "the UI is clean") are dropped by default unless they carry the central claim. Also favor points the author emphasizes, points needed to understand other points, counterintuitive findings, and actionable methods.
4. Resolve candidates into final points:
   - Expressing the same point → merge into one.
   - A number used only as evidence for a nearby claim → keep it inside that claim's card, never as its own card, unless the source states the number as an independent conclusion. A number presented as a standalone fact or setting (e.g. a configured duration, a defined threshold) may form its own point.
   - A pure causal chain (A so B, A therefore B) → one card by default. Split only when the conclusion adds a new actionable or counterintuitive claim (e.g. "cramming before an exam backfires").
   - A compound assertion → split only if both parts independently pass the deletion test in step 3.
   - Each final point must map to exactly one card.
5. Set the card count from N, the number of final points after step 4:
   - N = 0 → output no cards and append no explanation line; output only "原文无可提取的独立知识点。" in the user's language.
   - 1 ≤ N ≤ 8 → output all N cards.
   - N > 8 → merge the least important related points where possible; if still above 8, output the 8 most important. Never reach the target by inventing, repeating, or hair-splitting.
   - 1 ≤ N < 5 → append the explanation line described in Output.
6. Write each card using the exact card format below, filling all four fields.
7. Put the most load-bearing point in card 1: the point whose removal would leave the largest number of other cards without a basis. For argument texts this is normally the author's thesis; for explanatory or instructional texts, the core mechanism or rule; for notes, the concrete fact or setting on which an evaluation depends. If two points are equally load-bearing, choose the one stated first in the source. Order the rest from foundational to dependent; never place an evidence card before the claim it supports.
8. Run the final quality check and revise before returning.

## Card Format

Use exactly these four fields, in this order:

```
### 卡片 N：<标题>

- **核心知识**：<one sentence stating the point>
- **简明解释**：<1-3 plain-language sentences drawn only from the source>
- **例子或自测问题**：<an example from the source, or a self-test question answerable from the source>
```

Field rules:

- 标题: short and specific; name the single point. Avoid generic titles such as "背景" or "总结".
- 核心知识: one assertion only, not a list. Keep key terms and numbers exactly as in the source. For argument or opinion texts, attribute the claim to the author (e.g. begin with "作者认为" / "作者指出").
- 简明解释: 1-3 sentences using the source's surrounding detail (mechanism, condition, number, comparison). Do not merely restate 核心知识 in different words. If the point has no independent surrounding detail, use a contrast or a direct consequence stated elsewhere in the source; do not merge or drop an otherwise independent point for lack of detail.
- 例子或自测问题: apply this priority in order. (1) An example from the source that is not already used in 简明解释; quote its key terms and numbers verbatim. (2) Otherwise, exactly one self-test question whose answer is fully contained in the source; write the question only, without an answer; a two-part contrast may be one question with two sub-questions, but never split the point into two cards to fill space. (3) Only if the point cannot be turned into an answerable question, write "原文未提供例子". Never invent a scenario, dataset, or fact.

## Hard Rules

- One card, one point. Split compounds only as specified in step 4; merge duplicates.
- No fabrication. Every claim in every field must be supported by the source text. Do not use outside knowledge to complete, verify, supplement, or rebut an explanation.
- For opinion or argument texts, traceability means faithfully paraphrasing the author's claim and attributing it, not asserting that the claim is true.
- No padding. When information is insufficient, output fewer than 5 cards (or no cards) instead of forcing a count.
- No duplication. Do not restate the same point across cards with different wording.
- No external lookup. Do not fetch or consult other material.

## Output

- Output only the cards in Markdown. No preamble, overall heading, closing summary, or divider; separate cards with one blank line.
- Write card content in the source's dominant language; retain foreign proper nouns and quotations verbatim. Keep the four field labels in Chinese even when the body is in another language.
- When 1 ≤ N < 5, the explanation line is the only content allowed outside cards and must be the last line, e.g. 原文信息量有限，仅能提取 N 个独立知识点。
- Write the explanation line and any unsupported-input notice in the language of the user's request.
- For unsupported input, output only the limitation notice and the concrete alternatives from the decision table; do not produce cards.

## Final Quality Check

Verify before returning:

1. Each card has all four fields in the specified format, with no extra surrounding content.
2. Each 核心知识 is a single point traceable to a specific part of the source, and attributed to the author when the source is an opinion piece.
3. No 简明解释 merely paraphrases its own 核心知识.
4. No two cards teach the same thing; no number is isolated from the claim it supports.
5. No field contains a fact, example, or number absent from the source.
6. Card 1 carries the most load-bearing point, and the count reflects real information density rather than a quota.
