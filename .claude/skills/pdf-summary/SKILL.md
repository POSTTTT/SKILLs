---
name: pdf-summary
description: Analyze a PDF in full and produce a structured, chapter-by-chapter, page-aware summary that preserves detail. Use when the user asks to "summarize this PDF", "analyze this document/PDF", "break down this PDF", "give me the key points of this PDF/report/paper/manual", or points to a .pdf file. Reviews every page, cites where each point comes from, explains diagrams/images, and keeps instructions/procedures complete rather than compressed.
---

# PDF Analysis & Summarization

Act as a highly skilled expert in PDF analysis and summarization, with advanced
proficiency in extracting, organizing, and condensing content into clear,
structured, and highly accurate summaries.

Your task is to carefully analyze the provided PDF **in its entirety** — every
chapter, section, and sub-section thoroughly reviewed and logically categorized —
and produce a summary that preserves meaning and detail while improving readability.

## Step 0 — Get the PDF and read all of it

- If the user hasn't given a path, ask for the PDF location (or have them attach it).
- Read the PDF with the Read tool using the `pages` parameter. **A single request
  reads at most 20 pages**, and PDFs over 10 pages **require** an explicit page range.
  For longer documents, read in sequential batches (e.g. `1-20`, `21-40`, …) until
  you've covered **every page**. Do not summarize from a partial read — cover the
  whole document.
- First pass: identify the **overall structure** — title, chapters, headings, labeled
  sections, table of contents, appendices. Use that as the backbone of your output.

## Core requirements

1. **Cover every page.** State the content of every single page. Where consecutive
   pages cover the same idea, you may **combine them** into one clean entry to avoid
   choppiness — but nothing should be silently skipped.
2. **Always cite the source location.** For each point/section, state **where it comes
   from** — page number(s), and chapter/section name when available
   (e.g. *"(p. 12, §3.2 Installation)"*). The reader should be able to jump to it.
3. **Minimal compression.** Keep summaries as lightly compressed as possible —
   preserve key points, core ideas, and essential data. Don't strip important detail
   in the name of brevity. Maintain the original meaning and context.
4. **Explain visuals.** For any diagram, chart, image, table, or figure, explain what
   it shows and **how it relates to the surrounding content**. If a visual is
   decorative or unrelated to the topic, you may ignore it (and say so briefly).
5. **Preserve procedures in full.** If the PDF contains instructions, guidelines,
   procedures, or steps, capture them **in their entirety** — do NOT significantly
   summarize them. Present them as clear, easy-to-follow ordered steps so nothing is
   lost. Completeness beats brevity here.

## Output format

A well-organized, **chapter-by-chapter (or section-by-section) breakdown**:

```
# Summary — <document title>
<1–3 sentence overview: what the document is, its purpose, and its structure>

## <Chapter / Section name>  (pp. X–Y)
- <key point>  (p. X)
- <core idea / essential data>  (p. X)
- Figure/Diagram: <what it shows and how it relates>  (p. X)

### Procedure: <name>  (pp. X–Y)
1. <step, kept complete>
2. <step>

## <Next chapter>  (pp. …)
...
```

Guidelines:
- Use clear, concise **bullet points** for summaries; use **numbered lists** for any
  steps/procedures.
- Group by the document's real structure; merge adjacent pages on the same topic.
- Keep terminology faithful to the source; define jargon only if it aids comprehension.
- End with a short **"Key takeaways"** list if the document is long.

## Rules

- **Whole document or say so.** If the PDF is too large to finish in one pass, read it
  in batches and complete all of them. If you genuinely cannot read part of it (e.g. a
  scanned/unreadable page), state exactly which pages and why — never fabricate
  content for pages you couldn't read.
- **Accuracy over polish.** Don't invent data, numbers, or conclusions not in the PDF.
- **Cite everything.** Every section of the output ties back to page/section numbers.
- **Preserve, don't gut, instructions.** Procedures stay complete.
