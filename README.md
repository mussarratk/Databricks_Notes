
# 🧠 Databricks Certified Generative AI Engineer Associate – Study Notes

Comprehensive notes to prepare for the **Databricks Certified Generative AI Engineer Associate** exam.  
Covers architecture, data preparation, RAG pipelines, governance, deployment, and evaluation — organized for practical study and quick lookup.

---

## 📑 Table of Contents

1. [🧱 Section 1: Design Applications](#-section-1-design-applications)
2. [🧾 Section 2: Data Preparation](#-section-2-data-preparation)
3. [🛠️ Section 3: Application Development](#️-section-3-application-development)
4. [🚀 Section 4: Assembling and Deploying Applications](#-section-4-assembling-and-deploying-applications)
5. [🛡️ Section 5: Governance](#️-section-5-governance)
6. [📈 Section 6: Evaluation and Monitoring](#-section-6-evaluation-and-monitoring)
7. [🧠 Additional Study Notes](#-additional-study-notes)
8. [🧠 Additional GenAI Study Notes (Cleaned & Expanded)](#-additional-genai-study-notes-cleaned--expanded)

---

## 🧱 Section 1: Design Applications
### How to architect GenAI solutions from use-case to execution flow

---

### 1️⃣ Prompt Design Rules

**For structured output →** Use few-shot prompting with clearly labeled examples.  
Example:
```text
Respond in JSON with keys ‘summary’, ‘sentiment’, and ‘entities’.
````

**For classification tasks →** Use zero-shot prompts with explicit classes:

```text
Classify as one of: positive, neutral, negative.
```

**To elicit formatted output →** Include formatting instructions after core context.
Models prioritize recent tokens → put format instructions last.

**Use system prompts to establish tone/persona:**

```text
You are a financial assistant who responds concisely in bullet points.
```

---

### 2️⃣ Task-to-Model Mapping

| Business Need                         | Model Task                           | Preferred Approach                                  |
| ------------------------------------- | ------------------------------------ | --------------------------------------------------- |
| Text summarization                    | Summarization                        | Prompt LLM with extractive/abstractive instructions |
| Structured data extraction (PDF/HTML) | Information Extraction               | Retrieval + field-specific prompt                   |
| Classification                        | Zero-/Few-shot Classification        | Prompt with example-labeled outputs                 |
| Q&A over documents                    | Retrieval-Augmented Generation (RAG) | Embed + retrieve + prompt                           |
| Multi-step reasoning or action        | Agent                                | Tool selection + LLM + memory                       |

🔑 **Default:** Use retrieval + prompting before considering fine-tuning.

---

### 3️⃣ Choosing Chain Components

**Input-based decisions:**

* Raw text → Embed + index in Vector DB
* PDF/HTML → Chunk + tag with metadata
* SQL or structured input → Function calling

**Output-based decisions:**

* Summary → Summarization chain
* Grounded answer → RAG (Retriever + Generator)
* Action (e.g. booking) → Agent with tools

---

### 4️⃣ Translating Business Goals into Pipeline Design

**Mental checklist:**
✅ What is the goal?
✅ What input format is expected?
✅ What output format is required?
✅ Is factual accuracy required?

**Example:**

> "Generate ticket replies based on internal KB."

**Pipeline:**

1. Chunk & embed KB
2. Vector search relevant chunks
3. Prompt LLM with user ticket + top docs
4. Format output in helpdesk response style

---

### 5️⃣ Multi-Stage Reasoning and Tool Use

* **Tools** = APIs or functions (e.g. calculator, search, lookup)
* **Agents** = LLMs that plan tool use across steps
* **Chain-of-thought prompting** → Break complex reasoning into steps

**Example Tool Chain:**

1. Extract keywords from user question
2. Use keywords to search KB
3. Summarize retrieved docs
4. Generate structured answer or invoke function

🔁 RAG = static retrieval → generate
🧠 Agents = dynamic reasoning → tool → loop

---

### 🎯 Key Rules of Thumb

* Use **RAG** before fine-tuning → cheaper, safer, faster
* Use **system prompts** to control tone and intent
* Favor **few-shot examples** for structured output
* Chain components based on **input/output**, not tool familiarity
* Multi-stage reasoning? → Use **agents + tools**

---

## 🧾 Section 2: Data Preparation

### How to structure, clean, chunk, and persist content for retrieval-augmented generation (RAG) pipelines

---

### 1️⃣ Chunking Strategies

* **Default to semantic chunking** → Split on paragraph/topic boundaries

  * Improves relevance and grounding quality
* Avoid mid-sentence cuts
* Use **overlapping sliding windows** for dense information

  * Ensures context continuity

**Adjust chunk size:**

* Small model (4k tokens) → 256–512 token chunks
* Large model (32k tokens) → 1k–2k token chunks

**Always attach metadata:**
`source`, `page_number`, `section_title`, etc.

---

### 2️⃣ Filtering and Cleaning Documents

* Remove ads, headers, footers, disclaimers
* Normalize text → fix line breaks, remove HTML tags
* Avoid irrelevant docs (e.g., full books/manuals)
* Prioritize **quality over quantity**

---

### 3️⃣ Python Packages for Document Parsing

| Format       | Recommended Package        |
| ------------ | -------------------------- |
| PDF          | PyMuPDF (fitz), pdfplumber |
| Word (DOCX)  | python-docx                |
| HTML         | BeautifulSoup              |
| Markdown     | markdown + html2text       |
| Scanned PDFs | pytesseract (OCR required) |

🔑 Use parsers that preserve layout and sectioning.

---

### 4️⃣ Writing Chunks to Delta Lake in Unity Catalog

**Store as structured tables:**

Columns: `chunk_text`, `embedding_vector`, `document_id`, `metadata`

**Process:**

1. Extract → Clean → Chunk → Embed
2. Write to Delta table → `spark.write.format("delta")`
3. Register in Unity Catalog
4. Track lineage via source paths & timestamps

Unity Catalog = governance + metadata lineage.

---

### 5️⃣ Source Document Selection for RAG

Prefer:

* Internal KBs
* FAQs
* Product documentation
* Controlled Wikis

Avoid:

* Raw logs
* Social media
* Unfiltered long docs

---

### 6️⃣ Prompt/Response Pair Selection

* **Classification →** short input, clear labels
* **Summarization →** long input, short output
* **Retrieval QA →** context + question → answer
* Filter out hallucinations, long/ambiguous prompts
* Use for evaluation, fine-tuning, or retrieval simulation

---

### 7️⃣ Retrieval System Evaluation & Re-Ranking

**Metrics:**

* Precision@k
* Recall@k
* NDCG
* MRR

**Re-ranking:**

1. Vector similarity (fast)
2. Cross-encoder (accurate)

---

### 🎯 Key Rules of Thumb

* Semantic chunking + overlaps = best retrieval
* Clean & normalize before embedding
* Store chunks with metadata in Delta Lake
* High-quality docs > quantity
* Evaluate retrievers using precision/recall

---

## 🛠️ Section 3: Application Development

### Building, evaluating, and securing GenAI apps using prompts, agents, tools, and frameworks

---

### 1️⃣ Retrieval-Aware Development

* Create tools (API wrappers, search, lookup)
* Integrate OCR, SQL, REST tools into RAG pipelines
* Adjust chunking for recall and relevance
* Align chunk size with embedding model tokens

---

### 2️⃣ Prompt Engineering in Practice

* Dynamically insert fields (e.g., `{customer_name}`)
* Use templating (`f"Given {product_type}..."`)
* Rewrite prompts for clarity and target audience
* System prompts control style:
  `"You are a friendly assistant."`
* Metaprompts reduce hallucination:
  `"Only use provided context. If unsure, say 'I don’t know'."`

---

### 3️⃣ Model & Embedding Selection

**LLM Selection:**

* Open-weight: Llama 2, MPT → cost control
* Proprietary: GPT-4, Claude → stronger reasoning
* MosaicML → fine-tuning platform

**Key Attributes:**

* Context window size
* Dataset transparency
* Eval metrics (MMLU, HELM, TruthfulQA)

**Embedding Models:**
Choose by query complexity, document size, cost vs performance.

---

### 4️⃣ Frameworks and Chains

Use **LangChain** for:

* Chaining retrieval → transform → generation
* Agents, tools, memory

Use **Agent Frameworks** (LangChain, Semantic Kernel) for:

* Tool selection at runtime
* Multi-step tasks (search → analyze → act)

---

## 🚀 Section 4: Assembling and Deploying Applications

### How to build, register, serve, and secure GenAI applications on Databricks

---

### 1️⃣ Building and Coding Chains

* **LangChain** constructs: `LLMChain`, `RetrievalQA`, `SimpleSequentialChain`
* **Pyfunc model chaining:** wrap preprocessing → LLM → post-processing
* Register via `mlflow.pyfunc.log_model()`

**Example pipeline:**

1. Clean input
2. Invoke retriever/LLM
3. Validate & redact PII

---

### 2️⃣ Registering and Serving Models

* Register in **Unity Catalog** with versioning & schema
* Pipeline for RAG app:

  * Ingest → Chunk → Embed → Index → Chain → Register → Deploy
* Control access via ACLs / API tokens

---

### 3️⃣ Serving Vector Search and LLM APIs

* **Databricks Vector Search** = managed vector index

  * Backed by Delta tables
  * Query with `.query()` or `ai_query()`
* **Mosaic AI Vector Search** = scaled variant

**ai_query() for batch:**

```sql
SELECT ai_query('vector_index_name', question_column) AS answer
```

---

### 4️⃣ Foundation Model Serving

* Serve GPT-4, Claude, LLaMA via MosaicML / Databricks
* `dbx.ChatCompletion.create()` for API calls
* Requires:

  * Model + retriever artifacts
  * Vector search index
  * Cluster or serverless endpoint
  * UC access control

---

### 🎯 Key Rules of Thumb

* Use **LangChain** unless PyFunc is required
* Register all models in **Unity Catalog**
* Use **Vector Search + ai_query()** for low-latency lookups
* Pipeline sequence = `ingest → chunk → embed → index → chain → registry → endpoint`
* Secure endpoints with ACLs/tokens

---

## 🛡️ Section 5: Governance

### Enforcing safety, compliance, and content integrity in Generative AI applications

---

### 1️⃣ Masking Techniques for Guardrails

**Why:**

* Protect PII
* Improve retrieval performance

**Techniques:**

* Regex scrubbing
* NER-based masking
* Pre-processing before embedding

**Example:**
Mask `user_id` before embedding → improves generalization.

---

### 2️⃣ Guardrails Against Malicious Input

**Input filtering:** block prompt injections
**System prompts:** restrict behavior
**Rate limiting:** detect abuse
**Output validation:** regex/classifiers to reject unsafe content

🔐 Combine filtering + prompts + sanitization.

---

### 3️⃣ Problematic Text Mitigation in Source Data

**Issues:** toxicity, misinformation, PII
**Solutions:** classify, redact, filter at ingest time

✅ Prefer pre-ingestion cleanup.

---

### 4️⃣ Licensing and Legal Considerations

* Avoid copyrighted content
* Prefer open-license datasets (CC, Apache 2.0)
* Track source licenses and attribution
* GDPR compliance for PII

---

### 🎯 Key Rules of Thumb

* Mask sensitive info early
* Block injection and toxicity
* Pre-filter data sources
* Track data lineage via Unity Catalog

---

## 📈 Section 6: Evaluation and Monitoring

### Measuring, improving, and maintaining LLM performance in production RAG systems

---

### 1️⃣ LLM Selection Based on Metrics

| Metric     | Use Case                 |
| ---------- | ------------------------ |
| MMLU       | Knowledge                |
| TruthfulQA | Hallucination resistance |
| GSM8K      | Math reasoning           |
| ARC        | Multi-step QA            |
| HELM       | Holistic evaluation      |

**Rule of thumb:**

* Small models → simple, fast tasks
* Large models → complex reasoning

---

### 2️⃣ Key Monitoring Metrics

| Scenario       | Metric                       |
| -------------- | ---------------------------- |
| General health | Token usage, latency, errors |
| RAG            | Hit rate, hallucination %    |
| Chatbots       | User ratings, success rate   |
| Safety         | Blocked %, PII detections    |

Use MLflow or inference tables for logging.

---

### 3️⃣ Using MLflow for Evaluation

Track:

* Prompt templates
* Context retrieved
* Outputs
* Eval scores

Example:

```python
mlflow.log_metrics({'bleu': 0.89, 'rouge': 0.76})
mlflow.log_params({'model_name': 'llama2', 'chunk_size': 512})
```

---

## 🧠 Additional Study Notes

🔍 RAG Pipelines & Optimization
End-to-End Flow: Ingest → preprocess → chunk → embed → index → retrieve → prompt → generate.
Chunking & Evaluation:
Choose chunk strategy (semantic, sliding window) based on Recall@k or NDCG.
Use LLM-as-a-judge or human evals to tune retrieval performance.
Indexing: Use Delta tables + Databricks Vector Search to manage embeddings.
Retrieval Tip: Use metadata filters (e.g. book="1") to narrow scope in dense indexes.

🧠 Databricks Vector Search & Index Management
Databricks Vector Search:
Indexes Delta tables with embedded chunks.
Query with .search() or SQL ai_query().
Supports hybrid search: keyword + embedding for better relevance.
Similarity Metric: Default is L2 via HNSW. Normalize embeddings for cosine similarity.
Filtering: Add section, doc_type, or user_role as metadata filters.

⚡ Real-Time Data with Feature Store
Use Databricks Feature Store + Feature Serving to supply live, structured inputs.
Example: Serve up-to-date delivery times, account balances, or session state.
Complements RAG by grounding with real-time facts.
Secure via Unity Catalog + auto-scaling endpoints.

🚀 Model Serving & AI SQL Functions
Model Deployment Options:
MLflow PyFunc → wrap chains or preprocessors.
Hosted foundation models via MosaicML → use dbx.ChatCompletion().
AI Functions:
SQL-native: ai_query() or llm_query() for inline LLM inference.
Useful for dashboards and reporting.

🧰 MLflow PyFunc + Secrets Best Practices
Use PyFunc for chaining LLM calls with custom logic.
Avoid spark.conf.set() for secrets — use environment variables or Databricks Secrets.
Track prompt templates, metrics, input/output examples in MLflow runs.

🔐 Unity Catalog Governance
Store all data (chunks, features, logs) in Unity Catalog-backed tables.
Use UC to manage ACLs, model promotion, and audit trails.
Enable table and model lineage for traceability.

🤖 Agents, Tools, & Multi-Step Reasoning
Agent Use Case: When an LLM needs to decide dynamically between tools (e.g., RAG vs SQL vs API call).
Tool list is passed as part of system prompt.
Use LangChain or Databricks Agents to build agents with memory and tool use.

✏️ Prompt Engineering Enhancements
Input Augmentation: Dynamically insert user-specific context (e.g. plan_type, locale, account_status).
Output Formatting: Guide model with structure (e.g. "Respond in JSON with fields X, Y, Z").
Pre-processing: Use PyFunc to sanitize inputs, enforce format, or append extra context before prompt hits LLM.

📊 Model Selection & Trade-offs
Model Type	Use Case
Small Open Models	Low-latency tasks (e.g. classification, tagging)
Large Open Models	Reasoning-heavy RAG with self-hosted options
Proprietary Models	Complex summarization, conversational agents
Check context length, MMLU scores, architecture type when comparing.
Use proprietary APIs only if compliance risk is acceptable.

🧪 Monitoring & Evaluation Metrics
Before deployment (eval):

BLEU/ROUGE for summarization
MMLU for general model strength
Retrieval: Precision@k, Recall@k, NDCG
Post-deployment (monitoring):

Token count, latency, success/failure rate
Retrieval relevance score
Output rejection rate (e.g. via safety guardrails)

⚖️ Guardrails: Types & Use Cases
Type	Purpose
Safety	Block offensive/harmful inputs
Compliance	Enforce business or legal constraints (e.g. no politics)
Contextual	Ensure outputs align with user scope/intended use
Evaluation	Human or model-in-the-loop scoring & audits
Use input filters + output format checks + system prompt constraints.
Guardrails can wrap the entire app or live inside a chain.

🧠 DatabricksIQ, LakehouseIQ, & Assistant Features
DatabricksIQ: LLM assistant for code, SQL, and UI help. Not exposed via API.
LakehouseIQ: Natural language interface over structured data. Supports business-specific querying (e.g. “show me sales by channel for Q2”).
SQL AI Functions:
Use ai_query() in dashboards or notebooks for lightweight inference.
Ideal for summarization, classification, or content tagging in batch.

🧠 Additional GenAI Study Notes (Cleaned & Expanded)
✅ Prompt Design & Output Control
Neutralizing Tone: Instruct the LLM to rephrase emotionally charged input into neutral, professional language.

Example: "Rewrite this message in a neutral, factual tone."
Summarization Task: Condensing paragraphs into 1–2 sentences = summarization. Evaluate with ROUGE (coverage) or BLEU (precision).

Output Structuring: Use clear format instructions (e.g., “Respond in JSON with fields X, Y, Z”) and few-shot examples for reliable formatting.

🔁 LLM Workflows & Agents
Multi-step LLM Workflow: Used when tasks require chaining multiple steps (e.g., retrieve → reason → act).

Use LangChain or Databricks Agent Framework to orchestrate.
ReAct Framework: Combines reasoning (thoughts) with tool actions — ideal for agentic LLMs making decisions and using tools.

Agent Frameworks: Let LLMs select from registered tools (e.g., API lookup, SQL query, RAG retriever). Enables dynamic, goal-directed workflows.

💲 Cost Management & Serving Patterns
Pay-per-token: Ideal for low-volume use cases — no idle costs, scales with usage, no provisioning needed.

Low-Cost RAG Setup: Efficient stack = Prompt + Retriever + LLM (no fine-tuning or agent logic unless required).

📚 Feature Store & Real-Time Context
Databricks Feature Serving: Serves structured, per-query data (e.g., user balance, delivery time) in real-time.
Use when data can't be embedded in advance.
Complements unstructured RAG by grounding LLMs in current state.

🧱 Chunking & Semantic Context
Section Headers in Chunks: Boost semantic clarity in RAG. Helps retriever/embedding models infer context.

🧰 Tooling Overview
Orchestration & Reasoning
LangChain – Build chains, agents, tool use
ReAct – Reasoning + acting loop for tool-using agents
Evaluation & Monitoring
MLflow – Track experiments, register models, deploy with PyFunc
PyFunc lets you wrap preprocessing + postprocessing logic
Evaluation Metrics:
BLEU – Translation accuracy
ROUGE – Summarization recall
MMLU – General LLM benchmark (academic + reasoning)
NDCG – Normalized Discounted Cumulative Gain, ranks relevance in retrieval
LLMs-as-a-judge – LLMs evaluate outputs for quality/consistency

🧠 Databricks-Specific Features
LakehouseIQ – Natural language interface to structured data; understands metadata and lineage.
MosaicML – Databricks-hosted open models; scalable + customizable foundation models.
Unity Catalog Volume – Managed volume-based storage with access control + lineage (e.g., for models, training data).
Inference Tables – Auto-log requests/responses from model serving endpoints — enables live debugging and monitoring.

🧪 Embedding & Model Ecosystem
Key Python Libraries
unstructured – Extract text from PDFs, DOCX, HTML
pytesseract – OCR for image-to-text
langchain – Chain components, agents, tool use
mlflow – Model tracking, serving, versioning
sentence-transformers – Generate embeddings for retrieval
transformers – Load/tokenize Hugging Face models
pandas – Manipulate structured data
openai – Interface with OpenAI models (e.g., GPT-4)
faiss – Self-hosted vector search engine
scrapy – Web scraping
PyMuPDF – Extract structured text from PDFs (preserves layout)
pdfplumber – Extract text from PDFs with table support
doctr – Deep learning OCR for scanned PDFs/images (DocTR = Document Text Recognition)
datasets (Hugging Face) – Load benchmark datasets for evaluation/fine-tuning

🧠 Databricks LLMs – Simplified Reference
Model Name	Notes
DBRX	Databricks' flagship LLM; strong at summarization and general reasoning
MPT (7B / 30B)	MosaicML models; efficient for chat/instruction-tuned tasks
LLaMA (2, 3.1, 3.3, 4)	Meta’s open models; high performance for reasoning & chat
CodeLLaMA	Optimized for code generation and understanding
Claude (3.7, 4, Opus)	Anthropic models; excellent for deep reasoning, summarization
Whisper-large-v3	OpenAI’s speech-to-text model; used for audio transcription
BGE / GTE	Embedding models for RAG; fast and effective in dense retrieval
Dolly (1.0 / 2.0)	Early Databricks models; not production-ready but good for learning
DistilBERT	Lightweight transformer for classification or embedding

