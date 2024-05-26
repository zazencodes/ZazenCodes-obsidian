---
date: 2023-11-21
tags:
    - video
hubs:
    - "[[llms]]"
    - "[[prompts]]"
urls:
    - https://www.youtube.com/watch?v=eYtYIw0D1wI
---

# Superwise Unraveling prompt engineering

## Mitigating bias and poor performance
- Showing an LMM is always better than telling
    - Few shot >> zero shot
- Run prompt multiple times and take majority vote as the answer for the user
- Token likelihood can provide information about answer confidence

## General principles for quality LMM outputs
- Modular prompt templates (not monolithic)
    - e.g. first classify the input, then select separate prompt to take another pass over the input depending on the case
- Including examples, even if there are spelling or formatting errors, e.g. (note last line)
```
I want you to act as an airline assistant with expertise in customer support. You will field customer requests and determine if a human support team member needs to be contacted ("Yes"), or if an automated support pipeline can fulfill the request ("No").

Examples:
I want to cancel my flight // No
I need an attendant to help me reach my security checkpoint from the gate // Yes
Help me locate the nearest parking space with available spaces // No
Yes // Personal itemz missing
```
- Rather than passing a schema, specify an example of the JSON output desired
```
NO:

DEP: Departure flight code
ARR: Arrival flight code
DEP_TS: Departure time
...

YES:
{
"DEP": "YYZ",
"ARR": "YVR",
"DEP_TS": "2023-09-08 12:09:05PM"
}

```
- Telling it what it should do is much more effective than what it shouldn't do
    - e.g. "Don't use more than 12 steps " : it will always use exactly 12 steps. Better to say: keep it concise
- Include an option to "opt out"
    - e.g. 'If you are unsure of the proper label to provide, then label the request with "Human Support'

## Chain techniques for quality LMM outputs
- Use "Chain of thought" to (increase accuracy)
    - e.g. "Let's think step by step"
- Extract knowledge before providing answer (to increase answer quality)
    - e.g. "Output your knowledge related to the question." Then, "Provide your answer and explain why".
 - "Chain of density" summary
     - a) Identify 1-3 informative entities in article
     - b) Make summary that includes those entities
     - c) Identify 1-3 new informative entities in article
     - d) Make denser summary that also includes those entities
     - e) Repeat steps c and d 2-5 times
- "Chain of verification" to make the LMM think like a person would
    - a) Draft initial response
    - b) Plan verification questions
    - c) Answer the verification questions
    - d) Generate a final answer


