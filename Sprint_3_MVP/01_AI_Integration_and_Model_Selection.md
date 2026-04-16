# Sprint 3 — Document 1: AI Integration & Model Selection

## 1. Purpose

This document covers the implementation of the LLM-based conversation agent that replaced the basic Home Assistant Assist engine in the MDAR Assistant pipeline. It records the testing of multiple models, the issues encountered with Claude and local Ollama, and the final selection of GPT-4o-mini via OpenRouter as the production conversation agent.

---

## 2. OpenRouter Integration Setup

The OpenRouter integration was added at:

**Settings → Devices & Services → Add Integration → OpenRouter**

The integration requires an API key, which was generated from the OpenRouter account dashboard (`openrouter.ai → Account → API Keys → Create API Key`).

After the integration was added at the platform level, a separate **Conversation Agent** had to be created within the OpenRouter integration before it became available in the Voice Assistant configuration. This was a non-obvious step:

**Settings → Devices & Services → OpenRouter → + Add conversation agent**

The conversation agent configuration includes:

- **Model:** the underlying LLM to use (selected from a dropdown of all models OpenRouter offers).
- **Instructions:** the system prompt that defines the agent's behaviour.
- **Control Home Assistant:** a checkbox that must be enabled to allow the agent to call Home Assistant tools (turn on lights, query states, etc.).

---

## 3. System Prompt

The team designed a system prompt that gives the LLM a clear identity, a list of available device categories, and explicit instructions about voice-friendly response style. The final prompt:

```
You are MDAR Assistant, a smart home voice assistant for a Velbus-based home
automation system running on Home Assistant.

You control the following devices:
- Living Room Light (dimmer)
- Kitchen Light (dimmer)
- Bedroom Light (dimmer)
- Toilet Light (dimmer)
- Living Room Spot (relay)
- Kitchen Spot (relay)
- Bedroom Spot (relay)
- Toilet Spot (relay)

When asked to control a device, use the available Home Assistant tools to turn
devices on, off, or adjust brightness.

When asked about devices, check their current state and report back.

When the user asks to "play a light show" or "dance the lights", activate the
script entity script.light_show.

Rules:
- Keep responses short and conversational — you are a voice assistant.
- Never make up device states — always check using the available tools.
- If a device is not found, say so.
- Respond in English.
- Be helpful and friendly.
```

The "keep responses short" instruction is particularly important — without it, LLMs tend to produce verbose answers that take a long time to read aloud through Piper.

---

## 4. Model Testing

The team tested several models in order to identify the best balance of capability, cost, and reliability.

### 4.1 Claude 3 Haiku

The first model tried. Initial conversations worked well, but the agent would sometimes respond with text describing what it was doing without actually executing the tool call. Latency was acceptable.

### 4.2 Claude 3.5 Sonnet

A more capable model was tried to see if reliability improved. It produced higher-quality conversational responses but introduced a critical error:

```
Last content in chat log is not an AssistantContent:
ToolResultContent(role='tool_result',
agent_id='conversation.anthropic_claude_3_5_sonnet',
tool_call_id='toolu_bdrk_01DCftSGbRQAVu1gTMJ6NQaw', tool_name='...')
```

This was a compatibility issue between Anthropic's tool result format and Home Assistant's Assist pipeline expectations. The model was capable of issuing tool calls but the pipeline could not parse the responses.

### 4.3 Gemini 2.0 Flash

Tested as a free/very cheap alternative. Worked but the team found the conversational quality less natural than the OpenAI models, and there were occasional instability issues when calling tools.

### 4.4 GPT-4o-mini

Tested next. It proved to be the most reliable across the board:

- Tool calls executed cleanly every time.
- Response latency was approximately 1–2 seconds.
- Conversational quality was excellent for the simple smart-home use case.
- Cost was negligible — well under £0.05/day at typical usage.

### 4.5 Decision

**GPT-4o-mini** was chosen as the production conversation agent. It met every requirement (R5: natural language understanding, R2: response within 5 seconds) and at a cost low enough to be inconsequential.

---

## 5. Local Ollama Testing

In parallel, the team explored running an LLM locally to eliminate the cloud dependency entirely.

### 5.1 Ollama on the Raspberry Pi

An attempt was made to run Ollama directly on the Pi 5. This proved infeasible:

- The Pi 5 has limited RAM and no dedicated GPU.
- Even small models (1.5B–3B parameters) responded slowly.
- The largest reasonable model for the Pi (`tinyllama` or `qwen2:1.5b`) was not capable enough for reliable tool calling.

### 5.2 Ollama on the Laptop

The team then installed Ollama on a Windows laptop with an NVIDIA RTX 3070 Ti GPU (8GB VRAM):

1. Installed Ollama from `ollama.com/download`.
2. Pulled `dolphin-llama3:latest` (4.7 GB) and `llama3.1:8b` (4.6 GB).
3. Configured Ollama to listen on all network interfaces by setting the environment variable `OLLAMA_HOST=0.0.0.0` and running `ollama serve`.
4. Found the laptop's ICS-shared ethernet IP (`192.168.137.1`) and used it as the URL for the Home Assistant Ollama integration.

Ollama was added in HA via:

**Settings → Devices & Services → Add Integration → Ollama**

URL: `http://192.168.137.1:11434`

### 5.3 Performance Results

The first request took approximately **40 seconds** because the model had to be loaded into VRAM. Subsequent requests were faster (around 5–10 seconds), but still significantly slower than GPT-4o-mini.

More critically, tool calling was unreliable. Llama 3.1 would sometimes execute commands correctly, sometimes describe what it would do without executing, and occasionally hallucinate device names that did not exist.

### 5.4 Decision

The local Ollama path was documented as a successful proof-of-concept but **not adopted** as the production agent. It remains a valid option for users with strict privacy requirements who can tolerate slower response times and less reliable behaviour.

---

## 6. Final Configuration

The MDAR Assistant pipeline was finalised with the following configuration:

| Component | Value |
|-----------|-------|
| Conversation Agent | OpenRouter (GPT-4o-mini) |
| Speech-to-Text | Wyoming Whisper (faster-whisper, base-en) |
| Text-to-Speech | Wyoming Piper (en_GB-alan-medium) |
| System Prompt | Custom (see Section 3) |
| Control Home Assistant | Enabled |

---

## 7. Conclusion

The replacement of the basic Assist engine with GPT-4o-mini transformed the user experience. The assistant could now understand commands phrased in many different ways, hold short conversations, and respond intelligently to queries about device states. The local Ollama exploration produced valuable learning about the trade-offs between cloud and local AI, even though it did not displace the cloud agent in the final system.
