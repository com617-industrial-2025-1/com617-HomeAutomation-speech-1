# Bridging Research — Document 1: AI Conversation Agent Research

## 1. Purpose

This document captures the research conducted between Sprint 2 and Sprint 3 to identify a more capable conversation agent for the MDAR Assistant. The basic Home Assistant Assist engine, while functional, only matches predefined sentence templates and cannot understand natural language. To deliver an accessible, conversational experience for the user persona "Arthur", the team needed to integrate a Large Language Model (LLM) as the conversation agent.

This research informed the technology choices made in Sprint 3 and is documented here as part of the Bridging Research phase.

---

## 2. Why Replace the Built-in Assist Engine?

The limitations of the built-in Assist engine, identified at the end of Sprint 2, can be summarised as:

- It cannot understand paraphrased commands ("it's dark in here" instead of "turn on the light").
- It cannot answer general questions ("what devices do you see?").
- It cannot have a conversation or ask follow-up questions.
- It cannot interpret intent — only literal sentence templates.

For Arthur, who has limited mobility and may not always remember the exact command syntax, this rigidity is a significant barrier. An LLM-based conversation agent can interpret a much wider range of inputs and respond conversationally.

---

## 3. Conversation Agent Options Researched

Three categories of conversation agents were considered:

### 3.1 Cloud-Hosted LLM via Direct Provider API

Examples: OpenAI's API directly, Anthropic's Claude API directly, Google's Gemini API directly.

**Pros:** Direct, well-supported, fast.
**Cons:** Locked to a single provider. If the chosen provider has an outage or changes pricing, the system is affected. Requires separate accounts and billing for each provider.

### 3.2 Cloud-Hosted LLM via OpenRouter

OpenRouter is an aggregator service that exposes a single API endpoint compatible with OpenAI's API format, but routes requests to a wide range of models from different providers (OpenAI, Anthropic, Google, Meta, Mistral, and many more).

**Pros:** Single API key gives access to many models. Easy to switch models without changing integration code. Built-in support in Home Assistant from version 2025.8 onwards.
**Cons:** Adds an intermediary layer. Pricing is set by OpenRouter (typically a small markup over the underlying provider).

### 3.3 Local LLM via Ollama

Ollama is a tool for running open-source LLMs locally on consumer hardware. Models like Llama 3.1, Mistral, Gemma, and Qwen can be downloaded and run entirely without internet access.

**Pros:** Fully local, no cloud dependency, no API costs, no privacy concerns.
**Cons:** Heavy hardware requirements. The Raspberry Pi 5 alone cannot reasonably run an 8B-parameter model. A separate computer with a GPU is needed for usable performance.

---

## 4. Comparison of Models

The team identified the following models as candidates:

| Model | Provider | Cost | Speed | Tool Calling | Notes |
|-------|----------|------|-------|--------------|-------|
| GPT-4o | OpenAI (via OpenRouter) | High | Medium | Excellent | Overkill for simple smart home commands |
| GPT-4o-mini | OpenAI (via OpenRouter) | Very low | Fast | Excellent | Best balance of cost, speed, and reliability |
| Claude 3 Haiku | Anthropic (via OpenRouter) | Low | Fast | Good | Older Claude model |
| Claude 3.5 Sonnet | Anthropic (via OpenRouter) | Medium | Medium | Good | Smarter but slower; tool calling has compatibility issues |
| Gemini 2.0 Flash | Google (via OpenRouter) | Free/very low | Very fast | Good | Free tier available; sometimes inconsistent |
| Llama 3.1 8B | Meta (via Ollama) | Free | Slow on Pi, fast on GPU | Unreliable | Local, but tool calling is hit and miss |
| Dolphin Llama 3 | Community fine-tune (via Ollama) | Free | Slow | Unreliable | Local, conversational but weak at tool calling |

**Tool calling** is critical for this project because it is how the LLM actually controls Home Assistant devices. Without reliable tool calling, the LLM can describe what it wants to do but cannot execute it.

---

## 5. The Tool Calling Problem

A significant insight from this research was that **most local open-source LLMs struggle with tool calling**. The pattern of "given a list of available functions, decide which one to call with which arguments" was largely developed and refined by OpenAI for its API and is best supported by OpenAI's GPT-4 family of models.

When the team later tested Claude 3.5 Sonnet via OpenRouter (in Sprint 3), it produced errors of the form:

```
Last content in chat log is not an AssistantContent: ToolResultContent(...)
```

This was traced to a compatibility issue between Anthropic's tool result format and Home Assistant's expectations. The Claude model was technically capable of calling tools but the response format did not align with what Home Assistant could parse.

GPT-4o-mini, by contrast, worked reliably out of the box with Home Assistant's tool calling format.

---

## 6. Cost Analysis

A rough cost analysis was performed assuming an average of 50 voice commands per day:

| Model | Approx. cost per 50 commands/day |
|-------|----------------------------------|
| GPT-4o | ~$0.50–$1.00/day |
| GPT-4o-mini | ~$0.01–$0.03/day |
| Claude 3.5 Sonnet | ~$0.30–$0.60/day |
| Claude 3 Haiku | ~$0.05/day |
| Gemini 2.0 Flash | Free (within free tier) or ~$0.005/day |
| Local Ollama | $0 (after hardware cost) |

GPT-4o-mini stood out as offering excellent capability at a price point that is essentially negligible for personal use. Even at higher usage levels, the monthly cost would remain under £1.

---

## 7. Decision Made for Sprint 3

Based on this research, the team decided on the following strategy:

1. **Primary conversation agent:** GPT-4o-mini via OpenRouter. This provides reliable tool calling, fast response times, and a low operating cost.
2. **Fallback / future improvement:** Local Ollama on the laptop's NVIDIA RTX 3070 Ti GPU. This was tested but found to be too unreliable for tool calling to be the primary agent. It remains a candidate for future work where privacy is paramount.
3. **System prompt design:** A custom system prompt would be written to inform the LLM of the system context (Velbus-based smart home, MDAR Assistant identity) and to encourage short, conversational responses suitable for a voice assistant.

---

## 8. Conclusion

The research confirmed that GPT-4o-mini via OpenRouter is the most pragmatic choice for the Sprint 3 implementation. It satisfies the core requirements (natural language understanding, reliable tool calling, fast response, low cost) without introducing the operational complexity of a self-hosted local LLM.

The local Ollama path remains documented and partially tested, providing a clear migration path if the project's privacy requirements ever require eliminating the cloud dependency entirely.
