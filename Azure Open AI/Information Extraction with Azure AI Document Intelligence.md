# Azure AI Document Intelligence

## What it is

An Azure AI service for building automated data extraction pipelines from documents. Instead of manually parsing PDFs, images, forms, or receipts, you point it at a document and it returns structured data — field names, values, tables, signatures, and more.

It uses **OCR (Optical Character Recognition)** as its foundation — it reads text from scanned images and PDFs — but goes beyond raw OCR by understanding document layout and extracting meaning from that structure. For example, it does not just read "Total: $42.50" as a string; it identifies that "Total" is a field label and "$42.50" is its value.

---

## Core Capabilities

|Capability|What it does|
|---|---|
|OCR|Reads text from images, scanned PDFs, and handwritten documents|
|Layout analysis|Identifies tables, paragraphs, headings, checkboxes, and selection marks|
|Field extraction|Maps detected text to named fields (e.g. invoice number, date, total)|
|Table extraction|Pulls tabular data out of documents into structured rows and columns|
|Signature detection|Detects the presence of a signature on a document|
|Confidence scores|Returns a confidence value per extracted field so you know how reliable the result is|

---

## Pre-built Models

Azure AI Document Intelligence ships with pre-trained models for common document types. You do not need to train anything — just send a document and get structured output back.

|Model|What it extracts|
|---|---|
|**Receipt**|Merchant name, date, line items, subtotal, tax, total|
|**Invoice**|Vendor, customer, invoice number, line items, amounts, due date|
|**ID Document**|Name, DOB, address, document number from passports and driver's licenses|
|**Business Card**|Name, title, company, phone, email, address|
|**W-2**|Employer, employee, wages, tax withholdings|
|**Health Insurance Card**|Member name, plan, group number, payer|
|**Contract**|Parties, dates, clauses (preview)|
|**Layout**|Generic — extracts all text, tables, and structure without a domain-specific model|

Use a pre-built model whenever your document type is on this list. Only build a custom model when your documents have a structure that none of the pre-built models cover.

---

## Azure AI Document Intelligence Studio

A no-code web interface at **documentintelligence.ai.azure.com** for:

- Testing pre-built models against your own documents without writing any code
- Labeling documents to train a custom model
- Reviewing extraction results field by field
- Incorporating human feedback and corrections into custom model training (see below)

Good for validating whether a pre-built model works for your use case before building anything.

---

## How it works — high level

```
Document (PDF / image / scan)
        ↓
Azure AI Document Intelligence API
        ↓
OCR — reads all text and detects layout
        ↓
Model applies field mapping to extracted text
        ↓
Structured JSON output — fields, values, confidence scores, tables
        ↓
Your app processes the structured data
```

---

## Output Format

Results come back as JSON. Each extracted field includes the value and a confidence score between 0 and 1.

```json
{
  "fields": {
    "MerchantName": {
      "value": "Starbucks",
      "confidence": 0.98
    },
    "TransactionDate": {
      "value": "2024-03-15",
      "confidence": 0.95
    },
    "Total": {
      "value": 7.50,
      "confidence": 0.99
    }
  }
}
```

Use confidence scores to decide whether to auto-accept a result or route it for human review. A common threshold is 0.80 — anything below that gets flagged.

---

## Custom Models

When pre-built models do not cover your document type, you can train your own.

**Step 1 — Prepare sample documents** Collect at least 5 representative examples of the document (more is better — aim for 20+). Documents should cover the realistic variation you expect (different fonts, layouts, handwritten vs printed).

**Step 2 — Label in Document Intelligence Studio** Upload your samples to the Studio, draw bounding boxes around fields, and assign field names. This tells the model what to look for and where.

**Step 3 — Train** Trigger training from the Studio or via API. Azure trains a model on your labeled samples and returns a model ID.

**Step 4 — Evaluate** Test the model against documents it has not seen. Review confidence scores and extraction accuracy.

**Step 5 — Incorporate human feedback** When your deployed model gets something wrong in production, you can send the corrected labels back to the Studio and retrain. This is the feedback loop that improves accuracy over time — the model learns from its mistakes on real documents.

---

## Receipt Model — Specific Fields

Since the receipt model was called out specifically:

|Field|Example value|
|---|---|
|MerchantName|Whole Foods Market|
|MerchantAddress|123 Main St, Austin TX|
|TransactionDate|2024-03-15|
|TransactionTime|14:32|
|Items|[{ description, quantity, price }]|
|Subtotal|38.20|
|Tax|3.15|
|Tip|5.00|
|Total|46.35|
|PaymentMethod|Visa|

---

## Storage Browser Integration

Azure AI Document Intelligence Studio has a **Storage Browser** that connects directly to an Azure Blob Storage container. Instead of uploading documents one at a time through the UI, you point it at a blob container and it reads your documents from there. Useful when you have large batches of documents to label or process.

Setup: in Document Intelligence Studio → connect to your Azure Storage Account → select the container your documents are in.

---

## On-Premises Deployment with Docker

If your data cannot leave your network due to compliance or data residency requirements, Azure AI Document Intelligence can be run as a **Docker container on your own infrastructure** — no data leaves your premises.

**How it works**

Microsoft provides official Docker images for Document Intelligence. You pull the container, configure it with your API credentials, and it runs the same models locally.

```bash
# Pull the container image (example for layout model)
docker pull mcr.microsoft.com/azure-cognitive-services/form-recognizer/layout

# Run the container
docker run --rm -it -p 5000:5000 \
  -e Eula=accept \
  -e Billing=https://your-resource.cognitiveservices.azure.com/ \
  -e ApiKey=your-api-key \
  mcr.microsoft.com/azure-cognitive-services/form-recognizer/layout
```

Your app then calls `http://localhost:5000` instead of the Azure endpoint — same API, same response format, running locally.

**When to use the container**

- Documents contain PII or sensitive data that must not leave your network
- Air-gapped environments with no internet access
- Compliance requirements that prohibit sending data to cloud APIs
- Low-latency requirements where cloud round-trips are too slow

**Limitations**

- You are responsible for scaling and infrastructure
- Not all models are available as containers — check the Microsoft docs for the current supported list
- Still requires a valid Azure billing endpoint for licensing (the container phones home for billing only, not for document processing)

---

## Choosing the Right Approach

|Scenario|What to use|
|---|---|
|Common document type (receipt, invoice, ID)|Pre-built model — no training needed|
|Custom form or proprietary document layout|Custom model trained in Studio|
|Need to validate before building|Document Intelligence Studio (no code)|
|Data cannot leave your network|Docker container on-premises|
|Large batch processing|Connect Studio to Azure Blob Storage|
|Results need human review|Use confidence scores to route low-confidence results to a review queue|

---

## Related Notes

[[Retrieval Augmented Generation (RAG) with Azure AI Search]]
[[Building Applications with OpenAI]]
[[AI Plan for JumpStart]]