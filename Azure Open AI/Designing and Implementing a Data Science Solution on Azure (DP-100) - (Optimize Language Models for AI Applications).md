# Azure OpenAI

## What it is

Microsoft's enterprise-grade hosting of OpenAI's models inside the Azure ecosystem. You get access to the same models as OpenAI (GPT-4o, embeddings, DALL-E, Whisper) but deployed inside Azure's infrastructure — meaning your data stays within your chosen region, subject to Microsoft's compliance standards, and integrated with the rest of your Azure services.

The key distinction from using OpenAI directly is **where your data goes and who controls the environment**. With Azure OpenAI, Microsoft guarantees the data does not leave your configured region, does not train future models, and is subject to enterprise SLAs.

---

## Features

### Azure Integration

Connects natively with other Azure services — Azure Machine Learning for MLOps workflows, Azure Functions for event-driven AI triggers, Azure AI Search for retrieval pipelines, Azure Blob Storage for document ingestion, and more. You are not building in isolation; it slots into your existing Azure infrastructure.

### Enterprise Focus

- **Private networking** — deploy behind a Virtual Network (VNet) so the endpoint is never exposed to the public internet
- **SLAs** — guaranteed uptime commitments with financial backing
- **Data compliance** — meets standards including GDPR, HIPAA, ISO 27001, SOC 2
- **Data confidentiality** — your prompts and completions are not used to train OpenAI models

### Deployment Options

- Deploy the same model to multiple Azure regions to reduce latency for users in different geographies
- Create multiple deployments of different models within the same resource — for example one deployment of `gpt-4o` for complex tasks and one of `gpt-4o-mini` for lightweight operations, each with its own throughput allocation

### Pricing

Usage-based — you pay per token consumed (input + output), per image generated, or per minute of audio transcribed. No upfront commitment required unless you want reserved capacity (Provisioned Throughput Units) for guaranteed throughput.

### Customization

Models can be fine-tuned on your own data to shift behavior toward your domain. Covered in detail in the Fine Tuning section below.

---

## Types of Models

|Type|What it does|Examples|
|---|---|---|
|**Language**|Text generation, chat, summarization, code, reasoning|GPT-4o, GPT-4o-mini|
|**Embedding**|Converts text to float vectors for semantic search and similarity|text-embedding-3-small, text-embedding-ada-002|
|**Image**|Generates or analyzes images from text prompts|DALL-E 3, GPT-4o vision|
|**Speech**|Transcribes audio to text or converts text to speech|Whisper, TTS|

---

## How Chat Completions Work

### Roles

Every message sent to a chat model has a role that tells the model who is speaking:

|Role|Purpose|
|---|---|
|**system**|Sets the model's behavior, persona, and constraints for the entire conversation. Sent once at the start.|
|**user**|The human's input — a question, instruction, or prompt.|
|**assistant**|The model's previous responses. Included in history to give the model context.|

```json
[
  { "role": "system",    "content": "You are a helpful HR assistant." },
  { "role": "user",      "content": "Who are the top candidates for this role?" },
  { "role": "assistant", "content": "Based on the skills provided..." },
  { "role": "user",      "content": "Can you rank them by years of experience?" }
]
```

The `Passed Messages Included` parameter controls how many prior turns of conversation history are sent with each request. More history = more context for the model but more tokens consumed per call.

### Parameters

| Parameter                     | What it controls                                                                                                                                       |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Temperature**               | Randomness of output. Higher (e.g. 0.9) = more creative and unpredictable. Lower (e.g. 0.2) = more focused, consistent, and deterministic.             |
| **Top P**                     | Alternative to temperature. Limits the model to the top P% of probable next tokens. Do not use both Temperature and Top P at the same time — pick one. |
| **Max Response (max_tokens)** | Hard cap on how many tokens the model can generate in a single response. Prevents runaway long outputs and controls cost.                              |
| **Stop Sequence**             | One or more strings that tell the model to stop generating when encountered. Useful for structured outputs where you know what the response ends with. |

**Without Top P:** the model considers all possible next tokens weighted by probability. This is the default behavior and is fine for most use cases.

### Sentiment Analysis with Examples

You can steer the model toward a specific output format by including few-shot examples directly in the system message or as prior user/assistant turns. This is especially useful for classification tasks like sentiment analysis:

```json
[
  { "role": "system", "content": "Classify the sentiment of the input as Positive, Negative, or Neutral. Reply with only the label." },
  { "role": "user",      "content": "The onboarding experience was smooth and well organized." },
  { "role": "assistant", "content": "Positive" },
  { "role": "user",      "content": "I had no issues at all." },
  { "role": "assistant", "content": "Positive" },
  { "role": "user",      "content": "The system crashed twice during setup." }
]
```

The model learns the expected format from the examples and applies it to the final input.

---

## AI Playground vs Development Environment

Changes made in the Azure AI Foundry Playground (ai.azure.com) **do not automatically carry over to your application**. The playground is a testing sandbox only.

To use a configuration you tested in the playground in your app, you need to export it to code — the playground has a "View code" button that generates the equivalent API call in Python, C#, or REST. Copy that into your application and adjust as needed.

Think of the playground as a place to iterate on prompts and parameters quickly before committing them to code.

---

## OpenAI Model Evaluation

Before relying on a model for a production feature, evaluate it against a representative set of test cases. Azure AI Foundry has a built-in evaluation tool that lets you:

- Define a dataset of inputs and expected outputs
- Run the model against each input
- Score results by accuracy, groundedness, relevance, or custom metrics

This is how you validate that a model change (prompt update, parameter tweak, or model version upgrade) does not regress existing behavior before deploying it.

---

## RAG — Retrieval Augmented Generation

### What it is

A pattern for giving a language model access to your own data without fine-tuning. Instead of sending the model a huge document and hoping it finds the relevant part, you:

1. Pre-process your data into chunks
2. Store those chunks as embeddings in a vector store
3. At query time, embed the user's question and find the closest matching chunks using cosine similarity
4. Send only the relevant chunks alongside the user's question to the model

The model answers using the retrieved context rather than relying on what it learned during training. This is how you build Q&A over your own documents, knowledge bases, or databases.

### Key concepts

**Chunking** — splitting source documents into smaller pieces before embedding. Too large and the embedding loses specificity. Too small and there is not enough context. A common starting point is 500–1000 tokens per chunk with some overlap between chunks.

**Vector** — a set of numbers (float array) that represents the meaning of a piece of text in a high-dimensional space. Texts with similar meanings produce vectors that are close together. Also called an embedding.

**Cosine similarity** — the formula used to measure how close two vectors are. Returns a value between 0 and 1. Higher = more similar meaning. This is the core of semantic search.

**Semantic search** — finding documents by meaning rather than exact keyword match. "Software engineer with React experience" and "frontend developer skilled in React" would have high cosine similarity even though they share few exact words.

### RAG with Python — key libraries

```bash
pip install langchain langchain-community faiss-cpu openai
```

|Library|Purpose|
|---|---|
|`langchain`|Framework for chaining LLM calls, retrievers, and prompts into pipelines|
|`langchain-community`|Community integrations — document loaders, vector stores, tools|
|`faiss-cpu`|Facebook's vector similarity search library — stores and queries embeddings locally|

RAG data must be in the format of **Documents** — LangChain's wrapper that pairs text content with metadata (source file, page number, etc.).

```python
from langchain.schema import Document

docs = [
    Document(page_content="React is a JavaScript library for building UIs.", metadata={"source": "react-docs"}),
    Document(page_content="TypeScript adds static typing to JavaScript.", metadata={"source": "ts-docs"})
]
```

### RAG without code — Azure AI Foundry

Azure AI Foundry lets you set up a RAG pipeline through the UI without writing any code:

- Upload your documents to Azure Blob Storage
- Connect them to an Azure AI Search index (handles chunking and embedding automatically)
- Connect the index to a chat deployment
- Test in the playground

Good for prototyping or low-code scenarios. For production use, build the pipeline in code for full control.

---

## Fine Tuning

### What it is

Fine tuning takes a pre-trained model and continues training it on your own labeled data, modifying the model's internal weights to shift its behavior toward your domain. The model learns patterns from your examples that it would otherwise need to be prompted for every time.

### Fine tuning vs RAG

||RAG|Fine Tuning|
|---|---|---|
|How it works|Retrieves external context at query time|Changes the model's internal weights|
|Model changed|No — model stays the same|Yes — produces a new model version|
|Best for|Giving the model access to current or large bodies of knowledge|Changing the model's style, tone, format, or domain-specific behavior|
|Data needed|Documents, chunks, embeddings|Labeled input/output pairs|
|Cost|Per query (retrieval + completion)|Upfront training cost + per-token inference|
|When to use|Your data changes frequently or is large|The model consistently does something wrong that prompting alone cannot fix|

### Methods of fine tuning

|Method|How it works|
|---|---|
|**Supervised**|Train on labeled input/output pairs — you show the model the right answer|
|**DPO (Direct Preference Optimization)**|Train on pairs of responses where one is preferred over the other — teaches the model what good looks like relative to bad|
|**Reinforcement**|Model is rewarded or penalized based on a scoring function — useful when the right answer is hard to define explicitly|

---

## Prompt Flow

### What it is

A visual development tool in Azure AI Foundry for wiring together nodes into end-to-end AI workflows. Instead of writing orchestration code, you connect inputs, model calls, logic, and outputs visually. The finished flow deploys as a callable REST API endpoint.

Use cases include RAG pipelines, research assistants, document processing workflows, multi-step classification, and anything that chains multiple AI or logic steps together.

### Setup

1. Create a **Hub** in Azure AI Foundry — this is the workspace container
2. Inside the hub, create a **Project**
3. Inside the project, create a **Prompt Flow**

### Flow types

|Type|Purpose|
|---|---|
|**Standard Flow**|General-purpose — build, test, and deploy any LLM application|
|**Chat Flow**|Optimized for conversational applications with built-in chat history handling|
|**Evaluation Flow**|Runs a dataset through a flow and scores outputs — used for model evaluation|

### Example — Sentiment Analysis Flow (Standard Flow)

A two-node flow that takes a user's text input and returns a sentiment label:

**Node 1 — Authenticate** Connects to your Azure OpenAI deployment using your configured credentials. Outputs an authenticated client that subsequent nodes use.

**Node 2 — Sentiment Analysis** Takes Node 1's authenticated client and the user's input text. Sends a chat completion request with a system message defining the sentiment classification task (with few-shot examples if needed). Returns the sentiment label — Positive, Negative, or Neutral.

Once tested, deploy the flow as a REST endpoint. Your application calls that endpoint with a text input and receives the sentiment label back — no model code in your app, just an API call.

---

## Related Notes

- [[AI Recommended Talents Pipeline V2]]
- [[Azure AI Search]]
- [[Azure AI Document Intelligence]]
- [[Building Apps with OpenAI]]
- [[AI Interoperability]]