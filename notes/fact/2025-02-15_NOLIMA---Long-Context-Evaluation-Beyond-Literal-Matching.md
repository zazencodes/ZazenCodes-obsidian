---
date: 2025-02-15
tags:
    - fact
hubs:
    - "[[ai-engineering]]"
urls:
    - https://arxiv.org/pdf/2502.05167
    - https://www.linkedin.com/posts/nicolay-gerold_rag-is-dead-long-live-rag-llms-suck-at-activity-7295766165363077120-36wf?utm_source=share&utm_medium=member_desktop&rcm=ACoAAB5u5_4BSk4o8UhztCxqBxZNhucNBXH1eDQ
---

# NOLIMA - Long Context Evaluation Beyond Literal Matching

## Key Takeaways
- NOLIMA is a new benchmark that tests LLMs' true understanding of long texts, not just word matching
- Performance degrades significantly with longer contexts, even in top models
- "Lost-in-the-middle" effect: facts in middle of long texts are harder to retrieve
- RAG and text chunking remain crucial for optimal performance

## The Problem with Current Testing
People claim long-context is solved based on simple "needle in the haystack" tests. These tests are flawed because they rely on direct word matching:
- Question: "Who was the first person to land on Mars?"
- Text: "...and after years of preparation, Sarah Chen became the first person to land on Mars..."

## How NOLIMA Works
NOLIMA tests true comprehension by:
- Inserting one key fact ("needle") into irrelevant text
- Questions require understanding, not just matching
- Uses different but related words between question and answer

Example:
- Question: "Which character has been to Dresden?"
- Hidden fact: "Yuki lives next to the Semper Opera House" (opera house in Dresden)

## Performance Results
Even top AI models struggle with longer texts:
- o3-mini degrades to 18.9 at 32K context window
- Deepseek R1 to 20.7 at 32K
- Most models struggle past 4k
- Performance dips when key information appears in middle of text

## Implications
1. RAG is still really important even with growing context windows
2. Breaking text into smaller chunks still helps
3. We need better ways to test how models handle long texts
4. Models need to improve at reasoning, not just matching

