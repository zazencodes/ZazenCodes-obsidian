---
date: 2024-12-05
tags:
  - video
hubs:
  - ai-engineering
urls:
  - https://youtu.be/OrtBEBLMXdM?si=kM0BqjC5RQPPnnHu
---

# Iterating on LLM apps at scale Discord

Ian Webster
- Let dev platform and dev rel team at discord
- Working on LLMs for last year

## Evals
- Enterprises won't deploy LLMs until they can **measure** and **mitigate** security, legal and safety risk
- Keep it simple
	- Treat them as unit tests
	- Basic guardrails and success metrics
	- Break down RAG evals into small pieces
- e.g. goal was to train a casual chat personality
	- solution: check if output started with a lower-case letter
		- runs really quick, is deterministic
- e.g. RAG with web search
	- test set 1 - when does RAG trigger a web search?
	- test set 2 - how does it perform on local files?
	- again, runs quick, is deterministic

- Build an eval culture
	- They are just tests
	- They should run locally
	- They should not depend on a third-party or cloud
	- You shouldn't have trouble understanding your evals
	- It should be easy to run dozens of evals per day
	- Every PR should have an eval (past the link to the eval in the PR, or use CI/CD)

## Prompting
- Specificity vs detail tradeoff for prompts
	- Keep prompts simple
	- You can't prompt out all the failures from the evals - resist the urge to pile on too much into the prompt
	- Prompts are a form of vendor lock-in. GPT prompts work different than Claude prompts, so keeping prompts simple helps avoid this


