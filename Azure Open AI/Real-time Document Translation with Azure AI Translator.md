# Azure AI Translator

## What it is

A cloud-based Azure AI service for translating text, documents, and custom content across 100+ languages. It is production-ready out of the box — secure, scalable, and designed for enterprise workloads. Unlike a simple translation API, it supports full document translation (preserving layout and formatting), custom translation models trained on your own terminology, and batch processing for high-volume workloads.

---

## Core Capabilities

|Capability|What it does|
|---|---|
|**Text Translation**|Translates plain text strings in real time — single call, instant response|
|**Document Translation**|Translates full documents (PDF, DOCX, PPTX, etc.) while preserving original formatting and layout|
|**Custom Translator**|Train a custom translation model on your own bilingual data to improve accuracy for domain-specific terminology|
|**Dictionary Lookup**|Returns alternate translations for a word or phrase, including idioms, slang, and context-specific meanings|
|**Transliteration**|Converts text from one script to another without translating (e.g. Arabic text rendered in Latin characters)|
|**Language Detection**|Automatically detects the source language if you do not specify one|

---

## Global vs Regional Resources

When you create an Azure AI Translator resource, you choose between two deployment types:

||Global|Regional|
|---|---|---|
|Endpoint|`api.cognitive.microsofttranslator.com`|`<region>.api.cognitive.microsofttranslator.com`|
|Routing|Microsoft routes to the nearest data center automatically|Traffic stays within the specified region|
|Data residency|Not guaranteed to stay in one region|Guaranteed — data processed only in chosen region|
|Required for batch document translation|No|**Yes**|
|Pricing tier|Free (F0) or Standard (S1) available|Paid tier required|
|Best for|Real-time text translation with no data residency needs|Document translation, compliance requirements, batch workloads|

**Key rule:** If you need asynchronous batch document translation, you must create a **regional** resource on a **paid tier**. The global free tier does not support it.

---

## Finding Your Endpoints

In portal.azure.com → your Translator resource → Keys and Endpoint:

- **Key 1 / Key 2** — API keys for authentication (rotate periodically)
- **Endpoint** — your resource's base URL
- **Location/Region** — needed as a header on every API call (`Ocp-Apim-Subscription-Region`)
- **Text Translation endpoint** — `https://api.cognitive.microsofttranslator.com` (global) or `https://<region>.api.cognitive.microsofttranslator.com` (regional)
- **Document Translation endpoint** — `https://<your-resource-name>.cognitiveservices.azure.com`

---

## Text Translation

### With Python

```python
import requests, uuid, os
from dotenv import load_dotenv

load_dotenv()

endpoint = "https://api.cognitive.microsofttranslator.com"
key      = os.getenv("TRANSLATOR_KEY")
region   = os.getenv("TRANSLATOR_REGION")

def translate_text(text: str, target_languages: list[str], source_language: str = None):
    url = f"{endpoint}/translate"
    params = {
        "api-version": "3.0",
        "to": target_languages          # e.g. ["es", "fr", "ja"]
    }
    if source_language:
        params["from"] = source_language  # omit to auto-detect

    headers = {
        "Ocp-Apim-Subscription-Key":    key,
        "Ocp-Apim-Subscription-Region": region,
        "Content-Type":                 "application/json",
        "X-ClientTraceId":              str(uuid.uuid4())
    }

    body = [{"text": text}]
    response = requests.post(url, params=params, headers=headers, json=body)
    return response.json()

result = translate_text("Hello, how are you?", ["es", "fil", "ja"])
# Returns translations in Spanish, Filipino, and Japanese
```

### With Java

```java
import java.net.http.*;
import java.net.URI;

String endpoint = "https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&to=es&to=fr";
String key      = System.getenv("TRANSLATOR_KEY");
String region   = System.getenv("TRANSLATOR_REGION");

HttpClient client = HttpClient.newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(endpoint))
    .header("Ocp-Apim-Subscription-Key",    key)
    .header("Ocp-Apim-Subscription-Region", region)
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString("[{\"text\": \"Hello, how are you?\"}]"))
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());
```

---

## Dictionary Lookup

Standard translation gives you the most common translation for a word. Dictionary Lookup gives you **all possible translations** including alternate meanings, part-of-speech variants, and example usages in context. Most useful for:

- Idioms and phrases that do not translate literally
- Words with multiple meanings where context matters
- Understanding how a translated word is actually used in sentences

```python
def dictionary_lookup(word: str, source: str, target: str):
    url = f"{endpoint}/dictionary/lookup"
    params = {
        "api-version": "3.0",
        "from": source,   # e.g. "en"
        "to":   target    # e.g. "es"
    }
    headers = {
        "Ocp-Apim-Subscription-Key":    key,
        "Ocp-Apim-Subscription-Region": region,
        "Content-Type":                 "application/json"
    }
    body = [{"text": word}]
    response = requests.post(url, params=params, headers=headers, json=body)
    return response.json()

result = dictionary_lookup("run", "en", "es")
# Returns: correr (to run physically), ejecutar (to run a program),
#          dirigir (to run an organization), etc. with usage examples
```

---

## Asynchronous Batch Document Translation

Translates full documents in bulk — PDFs, Word files, PowerPoints — preserving the original formatting, layout, tables, and images. The text content is translated; the structure stays the same.

### Requirements

- **Regional** Translator resource (not global)
- **Paid tier** (S1 or higher) — free tier does not support this
- **Azure Blob Storage** — source documents go into one container, translated output lands in another
- The Translator resource must be granted access to the storage account

### Grant Translator access to Storage

In portal.azure.com → your Storage Account → Access Control (IAM) → Add role assignment:

- Role: **Storage Blob Data Contributor**
- Assign to: your Translator resource's managed identity

Enable managed identity on the Translator resource first if not already on: Translator resource → Identity → System assigned → On

### How it works

```
Source container (Blob Storage)
    ↓ contains your original documents
Submit translation job via API → specify source container, target container, target language(s)
    ↓ job runs asynchronously — you get a job ID back immediately
Poll job status using job ID
    ↓ when complete
Target container (Blob Storage)
    ↓ contains translated documents with original formatting preserved
```

### Python — submit and poll a batch job

```python
import requests, os, time
from dotenv import load_dotenv

load_dotenv()

# Must be your regional endpoint for document translation
doc_endpoint = f"https://{os.getenv('TRANSLATOR_RESOURCE_NAME')}.cognitiveservices.azure.com"
key          = os.getenv("TRANSLATOR_KEY")

source_url = "https://yourstorage.blob.core.windows.net/source-docs"     # SAS URL or managed identity
target_url = "https://yourstorage.blob.core.windows.net/translated-docs"

def submit_batch_translation(source_url, target_url, target_language="es"):
    url = f"{doc_endpoint}/translator/document/batches?api-version=2024-05-01"
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
    body = {
        "inputs": [{
            "source": { "sourceUrl": source_url },
            "targets": [{ "targetUrl": target_url, "language": target_language }]
        }]
    }
    response = requests.post(url, headers=headers, json=body)
    # Job ID is in the Operation-Location header
    return response.headers.get("Operation-Location")

def poll_job(operation_location):
    headers = { "Ocp-Apim-Subscription-Key": key }
    while True:
        response = requests.get(operation_location, headers=headers).json()
        status = response.get("status")
        print(f"Status: {status}")
        if status in ["Succeeded", "Failed", "Cancelled"]:
            return response
        time.sleep(5)  # wait 5 seconds before checking again
```

---

## Exploring Translation Options

Before committing to a translation approach, Azure AI Foundry and the Azure Portal both offer ways to test:

- **Azure Portal → Translator resource → Try it out** — test text translation with different language pairs and see raw API responses
- **ai.azure.com → Language → Translator** — interactive playground for testing translations, dictionary lookups, and language detection
- Use these to validate language support, check output quality for your specific content, and test edge cases (idioms, technical terms, mixed scripts) before building

---

## Custom Translator

When standard translation quality is not good enough for your domain — medical, legal, financial, or highly technical content — you can train a custom model on your own bilingual data.

**How it works:**

1. Prepare parallel documents — the same content in both the source and target language (your existing translated materials, glossaries, manuals)
2. Upload to the Custom Translator portal (customtranslator.azure.ai)
3. Train a custom model — Microsoft fine-tunes the base translation model on your data
4. Deploy and call using the same Translator API, but pass your `category` ID to route requests to your custom model

```python
params = {
    "api-version": "3.0",
    "to": ["es"],
    "category": "your-custom-category-id"  # routes to your trained model
}
```

The more parallel data you provide, the better the custom model will handle your domain-specific terminology.

---

## Bilingual Chat — Use Case

A common real-world pattern where two parties communicate in different languages through a shared interface — for example, a customer writing in Filipino and a support agent responding in English, with the system translating both directions transparently.

```
Customer types in Filipino
        ↓
Translate to English → shown to support agent
        ↓
Support agent types in English
        ↓
Translate to Filipino → shown to customer
```

**How to build it:**

- Detect the customer's language automatically on first message using the language detection feature
- Store the detected language against the session
- Translate every inbound message to the support language before displaying to the agent
- Translate every outbound agent message to the customer's language before sending

**Other applications of the same pattern:**

- **Content localization** — translate product descriptions, help articles, or UI strings into multiple languages at publish time rather than manually
- **Multilingual document processing** — translate incoming documents from customers or partners before processing them through your existing pipelines

---

## Choosing the Right Approach

|Scenario|What to use|
|---|---|
|Translate short text strings in real time|Text Translation API — global resource, free tier fine|
|Translate full documents preserving layout|Batch Document Translation — regional resource, paid tier required|
|Improve accuracy for domain-specific terms|Custom Translator with your own bilingual training data|
|Handle idioms or ambiguous words|Dictionary Lookup|
|Build a bilingual chat or support tool|Text Translation API with language detection|
|No data residency requirements|Global resource|
|Data must stay in a specific region|Regional resource|

---

## Related Notes

- [[Azure OpenAI]]
- [[Azure AI Document Intelligence]]
- [[Azure AI Search]]
- [[Building Apps with OpenAI]]