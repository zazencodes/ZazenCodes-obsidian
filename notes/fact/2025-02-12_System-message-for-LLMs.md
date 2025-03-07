---
date: 2025-02-12
tags:
    - fact
hubs:
    - "[[ai-engineering]]"
urls:
    - https://microsoft.github.io/Workshop-Interact-with-OpenAI-models/Part-2-labs/System-Message/
---

# System message for LLMs

### The Role of the `system` Message

The `system` message in OpenAI’s Chat API:

- **Defines global behavior**: It **persists across interactions**, ensuring consistent responses.
- **Reduces verbosity in user prompts**: The user doesn't need to repeat setup instructions every time.

### Why use it?

It's useful for **multi-turn interactions** or **strict response formatting** requirements.

#### If the Model Retains Context, Why Use a System Message?

##### The System Message Has a Higher Weight in Context Interpretation

Even though **all previous messages** are in the context, OpenAI models **prioritize system messages differently** than user/assistant messages.
- **System messages define model behavior** in a way that persists more strongly across turns.
- **User messages are treated as requests**, not as global instructions.
- The model may **deprioritize** or **reinterpret** an instruction embedded in a user message over multiple turns.

> 📌 **Example Issue: User Message Instructions Fading Over Time**
>
> The AI can **start drifting**. This happens because:
> - The instruction **loses weight over time** (since it's in a past user message, not the system message).
> - The model **shifts toward conversational responses** instead of structured Unreal commands.

##### The System Message Reinforces Role & Format for Every Response

When you use a **system message**, the instruction **remains a priority directive** throughout the conversation.


### When Does a System Message Matter the Most?

| **Scenario** | **Use System Message?** | **Why?** |
|-------------|----------------|--------|
| **Single-turn response** | ❌ No | The user prompt alone is enough. |
| **Multi-turn conversation with structured outputs** | ✅ Yes | Ensures format consistency. |
| **Multi-turn casual Q&A** | ❌ No | Context retention usually works fine. |
| **Multi-turn with role adherence (legal consultant, coding assitant, etc.)** | ✅ Yes | Prevents AI from shifting roles. |

### Should system prompts include few-shot learning examples?

**No.** System messages should define general behavior.

A system message should **set consistent behavior without requiring examples**, e.g.

```json
{"role": "system", "content": "You are a medical diagnosis AI assistant. Your job is to analyze patient symptoms and provide structured medical assessments. Always return a JSON object containing 'symptoms_observed', 'potential_diagnoses', 'recommended_tests', and 'treatment_suggestions'."}
```
If you want to use **few-shot learning**, it's better to include examples in a **user message or fine-tune the model**.

