---
date: 2025-02-03
tags:
    - podcast
hubs:
    - "[[llms]]"
urls:
    - https://www.latent.space/p/cosine
---

# Is finetuning GPT 4o worth it? With Alistair Pullen of Cosine (Genie)


The first fine-tuning attempts were on open source code that included a lot of updates to readme files and documentation and as a result, the model was really good at doing that stuff, but it became really bad at the other things so in order to get it performing better for their use cases they needed to clean the data before fine-tuning


When building an AI system to solve a task, it should be able to approach the task the same way that a human would so for the example of a software engineer, you would want to be able to do things like search the code base for existing implementations and keywords, then find the definitions of the relevant functions using LSP servers and also be able to look up information online


For finding relevant code to a problem (semantic code search - searching for functionality) was not possible with RAG (chunk with overlap, embed) using human language queries. What worked much better was to generate a code snippet using AI that implements the desired functionality and then do the semantic search against the code base with that implemented function.


We trained genie to write patched and diffs because it’s more token efficient. The team did a lot of fine tuning for genie to work this way.
