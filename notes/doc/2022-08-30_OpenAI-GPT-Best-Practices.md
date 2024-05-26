---
date: 2023-08-30
tags:
    - doc
hubs:
    - "[[llms]]"
    - "[[prompts]]"
urls:
    - https://platform.openai.com/docs/guides/gpt-best-practices/gpt-best-practices
---

# OpenAI GPT Best Practices

- Write clear instructions
    - Ask model to adopt a persona
    - Use delimiters to indicate distinct parts of input, e.g. triple quotation marks, XML tags, section titles
    - Provide examples or specify conditions on the output format, length, structure, etc..
    - Ask the model to cite their work or give reasons for their answer
- Provide reference text
    - 'Use the provided article to answer the questions. If the answer cannot be found in the article, write "I could not find an answer."'
- Split complex tasks into simpler subtasks
    - Break down multi-step prompts into steps, "Step 1 - .... Step 2 - ..."
    - Tell model how to answer different types of questions. Use intent classification to determine question type
    - Summarize or process large sets of data in chunks
- Give GPT time to think
    - Rather than asking if something is correct or not, ask it to determine its own answer and compare that to determine correctness
    - Ask model if it missed something in previous answers. Guide it specifically and carefully based on use case
- Use external tools
    - Embeddings for efficient knowledge retrieval
    - Instruct how to write and execute code, e.g. You can write and execute Python code by enclosing it in triple backticks, e.g. \```code goes here```. Use this to perform calculations.
    - Instruct how to call external APIs or provide access to specific functions

