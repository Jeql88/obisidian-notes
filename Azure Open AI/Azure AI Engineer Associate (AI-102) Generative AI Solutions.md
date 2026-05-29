# AI-102 — Generative AI Solutions

## Large Language Models (LLMs)

### What they are

A trained model that predicts and generates data based on patterns learned during training. LLMs specifically focus on text generation and are a type of generative AI — the broader category of models that can generate text, images, video, audio, and more.

The key distinction: LLMs do not "understand" what you type. They predict the most probable next token based on everything that came before it. Every response is a statistical prediction, not comprehension.

---

### How they work — the basics

**Deep learning** is the foundation. Complex tasks require multiple layers of neural networks — this multi-layer structure is called deep learning, which is a subset of machine learning. LLMs are built on deep learning.

**Transformer architecture** is what most modern LLMs are built on — introduced in the landmark paper _"Attention Is All You Need"_. The key innovation is that transformers modify vectors so they represent actual semantic meaning in context. The same word can mean different things in different sentences — the transformer architecture captures that difference. For example "bank" in "river bank" vs "bank account" gets represented differently even though it is the same word.

**Tokens** are the unit the model actually works with — not words or characters but chunks of text (roughly 3–4 characters on average). When you send a prompt, it is converted into tokens. The model predicts the next token, then the next, until it produces a complete response.

---

### Training

**GPT-3 as a reference point:** 96 hidden layers, a massive number of neurons, trained on enormous amounts of text data. During training, the model's parameters (weights) are tuned so it can predict the next token accurately across its training data.

This training process involves:

- Multi-dimensional mathematics at enormous scale
- Extremely high compute cost — training large models costs millions of dollars
- Potential for **bias** — because the model learns from its training data, any bias present in that data can be reflected in the model's outputs

---

### Key use cases

Agents and chatbots, virtual assistants, language translation, fraud detection, content generation, code generation — essentially any task that involves understanding or generating language. The use case surface is vast.

---

## Tokens and Prompts

The model does not read your prompt as text. It converts your input into a sequence of tokens and predicts the next token repeatedly until a stopping condition is met (max tokens reached, stop sequence encountered, or natural end of response).

**Why this matters practically:**

- Every token costs money — both input (your prompt) and output (the response) are billed per token
- Longer prompts = more tokens = higher cost per call
- There is a context window limit — the maximum number of tokens the model can hold at once (input + output combined). Exceed it and earlier content gets dropped

---

## Generative AI Key Concepts

**Hallucinations** — the model generates confident-sounding text that is factually incorrect. Happens because the model is predicting probable tokens, not retrieving verified facts. Strategies to reduce hallucinations include grounding the model with retrieved context (RAG), asking for citations, and using lower temperature settings.

**Content filters** — Azure OpenAI applies built-in safety filters that detect and block harmful content:

- Violent, hate, sexual content
- Direct attacks — a user explicitly trying to get harmful output
- Indirect attacks — harmful instructions embedded inside documents or data the model is processing
- Jailbreak attacks — attempts to bypass the model's safety guidelines through prompt manipulation

**Bias** — models can reflect biases present in their training data. Important to evaluate model outputs for fairness, especially in high-stakes applications.

---

## Microsoft Generative AI Services

|Service|Role|
|---|---|
|**Azure OpenAI**|Hosts OpenAI models (GPT-4o, embeddings, DALL-E, Whisper) in Azure infrastructure|
|**Microsoft Copilots**|Pre-built AI assistants embedded in Microsoft products (Teams, Word, GitHub, etc.)|
|**LangChain**|Open-source orchestration framework — chains together LLM calls, retrievers, tools, and logic into pipelines|

**Orchestrators** like LangChain sit between your application and the model — they manage the flow of data, decide what to retrieve, what to send to the model, and what to do with the response.

---

## Creating an Azure OpenAI Resource

### Key decisions at creation time

- **Region** — deploy to the region closest to where your API is deployed. Pricing varies by region and model availability varies by region. Always check model availability for your target region before committing.
- **Pricing tier** — determines throughput and features available

### Deployment types

|Type|What it is|
|---|---|
|**Global Standard**|Microsoft routes to nearest data center; best availability and lowest cost; no data residency guarantee|
|**Standard**|Pinned to your chosen region; data stays there; required for some features|
|**Global Provisioned Managed**|Reserved throughput capacity, globally routed; for high-volume predictable workloads|
|**Provisioned Managed**|Reserved throughput capacity, regionally pinned; highest cost, most control|

For most use cases, start with **Standard** in your target region.

---

## API Endpoints

|Endpoint|Purpose|
|---|---|
|`/chat/completions`|Chat and instruction-following — the primary endpoint for GPT models|
|`/embeddings`|Generate vector embeddings from text|
|`/images/generations`|Generate images via DALL-E|
|`/completions`|Legacy text completion — older models only, avoid for new builds|

---

## Authentication Options

|Method|When to use|
|---|---|
|**API Key**|Simple and quick; store in Key Vault, never in code or config files|
|**Entra ID (Azure AD)**|Preferred for production — no static keys to rotate, uses managed identity, supports role-based access control|

**Always use least privilege** — assign only the roles a service actually needs. Do not give a read-only service a contributor role.

**Key roles:**

- **Cognitive Services OpenAI User** — can call the API (inference only)
- **Cognitive Services OpenAI Contributor** — can manage deployments and call the API

Use Entra ID with managed identity wherever possible. It removes the key rotation burden and is more secure than static API keys.

---

## Interacting with Azure OpenAI Programmatically

Two options — both call the same underlying API:

**REST** — direct HTTP calls to the endpoint. Language-agnostic. Useful when an SDK is not available for your language or you need precise control over the request.

**SDK** — language-specific wrapper (available for Python, C#, JavaScript, Java, and others). Handles serialization, retries, and authentication boilerplate. Use the SDK unless you have a specific reason not to.

---

## Parameters for Chat Completions

|Parameter|What it controls|Guidance|
|---|---|---|
|`temperature`|Randomness — higher = more creative, lower = more deterministic|Use 0.0–0.3 for extraction/classification tasks; 0.7–1.0 for creative tasks|
|`top_p`|Probability mass cutoff for token selection|Do not use both temperature and top_p — pick one|
|`max_tokens`|Maximum output tokens|Always set this — never leave it unbounded|
|`frequency_penalty`|Penalizes tokens that have already appeared frequently in the output|Reduces repetition in long responses|
|`presence_penalty`|Penalizes tokens that have appeared at all in the output|Encourages the model to introduce new topics|
|`role`|Sets who is speaking in a message (system / user / assistant)|System role sets behavior; user is the human; assistant is prior model output|

---

## Prompt Engineering

The quality of output is directly tied to the quality of input. A vague prompt produces vague output. A well-structured prompt produces consistent, usable output.

### Core principles

1. Be specific and clear — state exactly what you want
2. Avoid ambiguity — if a word or instruction could be interpreted two ways, rewrite it
3. Repeat key constraints after including additional context — the model may lose track of earlier instructions when given long content
4. Specify the exact output format — JSON, bullet list, single word, numbered steps
5. Assign a persona — "You are a senior recruiter reviewing candidates" changes behavior significantly
6. Break complex tasks into steps — instead of one large request, give the model a sequence of steps to follow

### Prompting strategies by example count

|Strategy|What it is|Best for|
|---|---|---|
|**Zero-shot**|No examples — just the instruction|Simple, well-defined tasks|
|**One-shot**|One example of the desired input/output|When format matters|
|**Few-shot**|Multiple examples|When consistency and format are critical|

### Reducing hallucinations

- Provide retrieved context alongside the question (RAG) so the model answers from your data, not from training memory
- Ask for citations — "cite the source for each claim" forces the model to ground its output
- Lower temperature — reduces creative leaps that produce plausible-sounding but incorrect content

### Using external tools and functions

You can describe external functions to the model and it will tell you when and how to call them. The model does not call the function itself — it returns a structured response saying "call this function with these arguments" and your code executes it. This is the foundation of AI agents.

You can also tell the model it can write code which your application then executes on its behalf — another agent pattern.

---

## RAG — Retrieval Augmented Generation

### Why it exists

LLMs only know what they were trained on. They have a knowledge cutoff, no access to your private data, and no awareness of recent events. RAG solves this by retrieving relevant external data at query time and including it in the prompt as context.

> "I only know what I've been taught" — RAG is how you teach it what it needs for each specific query.

### How it works

```
User query
    ↓
Embed the query → float vector
    ↓
Search vector store for closest matching chunks (cosine similarity)
    ↓
Retrieve top K relevant chunks
    ↓
Attach chunks to the prompt as context
    ↓
Model answers using the retrieved context
    ↓
Response grounded in your data, not just training memory
```

### Key concepts

**Chunking** — source documents are split into smaller pieces before embedding. The chunk size affects retrieval quality — too large and the embedding loses specificity, too small and there is not enough context. Chunks overlap slightly so context is not lost at boundaries.

**Vectors and embeddings** — an embedding function (a specific type of neural network) converts text into a multi-dimensional float array that captures semantic meaning. Vectors focus on meaning, not the exact words. "Software engineer" and "developer" will have similar vectors even though they share no words.

**Vector proximity** — semantic search works by finding the vectors in your store that are closest to the query vector. Closest = most semantically similar = most relevant.

**Vector indexing** — many databases support storing and querying vectors natively. Azure AI Search, pgvector (PostgreSQL), Redis, and others all support this.

### RAG additional considerations

Because retrieved chunks are appended to the prompt, RAG increases cost — every chunk added is more tokens billed. This makes three things critical:

- **Relevance** — only retrieve chunks that are genuinely useful. Irrelevant chunks waste tokens and can confuse the model
- **Token count** — monitor how much context you are attaching per query; set a budget and trim if needed
- **Security** — the model will use whatever context you give it. Do not retrieve and pass data the user is not authorized to see

---

## Azure AI Search for RAG

Azure AI Search is the recommended vector store for RAG on Azure. It supports:

- **Full text search** — traditional keyword matching
- **Vector search** — semantic similarity using embeddings
- **Hybrid search** — both simultaneously, results merged via Reciprocal Rank Fusion (RRF). Best of both worlds — catches exact keyword matches and semantic matches in one query

Documents fed into Azure AI Search are split into chunks via a **vector skillset** — a processing pipeline that handles chunking and embedding generation automatically before indexing. Each source document becomes many indexed chunks, each with its own embedding.

---

## Generating Images with DALL-E

Call the `/images/generations` endpoint with a text description. Azure OpenAI hosts DALL-E 3 and returns generated images as URLs or base64-encoded data. Parameters include size, quality, and style.

---

## Related Notes

- [[Azure OpenAI]]
- [[Azure AI Search]]
- [[AI Recommended Talents Pipeline V2]]
- [[Building Apps with OpenAI]]
- [[AI Interoperability]]