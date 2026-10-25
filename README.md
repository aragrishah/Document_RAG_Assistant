# 🤖 Document RAG Assistant – Chat With Your Company Policy PDF

<p align="center">
  <b>Retrieval • Augmentation • Generation — built from scratch, no frameworks</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.x-blue?style=for-the-badge&logo=python" />
  <img src="https://img.shields.io/badge/Embeddings-MiniLM-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/VectorDB-ChromaDB-purple?style=for-the-badge" />
  <img src="https://img.shields.io/badge/NLP-NLTK-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/LLM-Gemini-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/No-LangChain-critical?style=for-the-badge" />
</p>

---

## 🚀 Project Overview

This project builds a complete **Retrieval-Augmented Generation (RAG)** pipeline that lets you
ask plain-English questions about a private company document and get answers grounded
**only** in that document — with the page number to verify against.

Everything is written **from scratch in a single Jupyter notebook**. No LangChain, no
LlamaIndex, no wrapper libraries. Every step of the pipeline is visible, commented, and
explained.

```
Question:  "How many days of paid annual leave do employees get?"

Answer:    Confirmed full-time employees are entitled to 22 working days
           of paid annual leave per calendar year [1].

Source:    NovaTech_Company_Policy_Handbook.pdf — page 2
```

---

## 🎯 Objectives

This project sets out to answer:

- How do you turn a messy real-world PDF into clean, searchable knowledge?
- How should documents be split so that meaning is never cut in half?
- How does an AI search by **meaning** instead of keywords?
- How do you stop an LLM from making things up when it does not know?
- Can a private RAG system run **free and offline**, without sending data to an API?

---

## 📂 Dataset Information

- **Document:** `NovaTech_Company_Policy_Handbook.pdf`
- **Type:** Company policy handbook (fictional company, realistic content)
- **Size:** 10 pages, ~18,000 characters

### Sections Covered

| Section | Content |
|---------|---------|
| 1. Leave & Time Off | Annual, sick, parental, bereavement, public holidays |
| 2. Remote Working | Hybrid baseline, allowances, core hours |
| 3. Travel & Expenses | Claim windows, per-grade limits table |
| 4. Information Security | MFA, device loss, data classification |
| 5. Code of Conduct | Conflicts of interest, grievance process |
| 6. Product Warranty | Coverage, exclusions, claims |
| 7. Returns & Refunds | Return window, restocking fees |
| 8. Support & SLA | Severity tiers, response targets |

The document deliberately includes **real PDF problems** — a running header on every page,
tables, bullet lists, and sentences broken across lines — so the cleaning step has genuine
work to do.

---

## 🛠️ Tech Stack

| Category | Tools Used |
|----------|------------|
| Language | Python 3.x |
| PDF Extraction | PyPDF |
| NLP Preprocessing | NLTK (tokenization, stop words, lemmatization) |
| Embeddings | Sentence-Transformers (`all-MiniLM-L6-v2`, 384-dim) |
| Vector Database | ChromaDB (cosine similarity) |
| LLM | Google Gemini (free tier) |
| Environment | Jupyter Notebook |

---

## 📊 Pipeline Workflow

### Part 1 — Data Preparation

**1. Document Loading**
- Read the PDF page by page using PyPDF
- Attach metadata (`source`, `page`) to every page so answers can be cited

**2. Text Cleaning**
- Detect and strip the running header/footer repeating across all pages
- Fix unicode issues, curly quotes, and encoding artifacts
- Rejoin words hyphenated across line breaks
- Flatten PDF line-wrapping while preserving real paragraph breaks

**3. NLP Preprocessing**
- Sentence tokenization, word tokenization
- Stop-word removal and lemmatization
- Token counting for cost estimation

**4. Chunking — Sentence-Aware Sliding Window**
- Pack whole sentences up to 800 characters
- Step back ~150 characters so neighbouring chunks overlap
- **No chunk ever starts or ends mid-sentence**

### Part 2 — Retrieval & Generation

**5. Embeddings**
- Convert each chunk into a 384-dimensional vector using MiniLM
- Runs **locally and free** — the document never leaves your machine

**6. Vector Storage**
- Store vectors, text, and metadata in ChromaDB
- Configured for **cosine similarity**, not the L2 default

**7. Retrieval**
- Embed the question with the same model
- Return the **top 3** most similar chunks
- Drop anything below a 0.25 similarity threshold

**8. Generation**
- Build a prompt with numbered context and a strict grounding instruction
- Send to Gemini at `temperature=0.2`
- Return an answer with `[1] [2]` citations

---

## 📈 Key Results

| Metric | Result |
|--------|--------|
| Pages processed | 10 |
| Chunks created | 36 |
| Boilerplate removed | 30 header lines (1,328 characters) |
| Chunks ending on a complete sentence | 35 / 36 (97%) |
| Overlap coverage | 26 / 26 same-page pairs (100%) |
| Embedding dimensions | 384 |
| Retrieval accuracy (top match) | 0.70 similarity on direct questions |

### Semantic Search in Action

| Text Pair | Similarity |
|-----------|------------|
| "sick leave" vs "medical time off" | **0.450** |
| "sick leave" vs "annual holiday" | 0.254 |
| "sick leave" vs "laptop battery warranty" | 0.084 |

The top pair shares **zero words** yet scores highest — proof the system searches by
meaning, not keywords.

---

## 🛡️ Hallucination Control

The system has **two independent defences**, because neither alone is enough.

### 1. Similarity Threshold

> *"Who won the cricket world cup in 2011?"*

Every chunk scores around **0.09**. Nothing passes the filter, so the LLM is **never called**.
Zero cost, zero risk.

### 2. Strict Prompt Instruction

> *"What is NovaTech's policy on pet insurance?"*

Chunks score **0.58** — high, because the question *sounds* like the handbook. The threshold
gives no protection here. The prompt instruction is what produces:

```
I don't have enough information to answer this.
```

**This is the core lesson of the project:** a similarity score tells you whether text *looks*
related, never whether it actually answers the question.

---

## ⚙️ Key Design Decisions

| Decision | Value | Reasoning |
|----------|-------|-----------|
| Chunk size | 800 characters | Large enough for one complete idea |
| Overlap | 150 characters (~19%) | The standard 10–20% rule of thumb |
| Chunking strategy | Sentence-aware sliding window | Prevents mid-sentence cuts |
| Lowercasing stored text | No | Capitals carry meaning for embeddings |
| Embedding model | MiniLM (local) | Free, private, no API cost |
| Distance metric | Cosine | Compares meaning, ignores text length |
| Top-K | 3 | Enough context without adding noise |
| Temperature | 0.2 | Consistent factual answers |

---

## 💻 How to Run

**1. Install dependencies**

```bash
pip install -r requirements.txt
```

**2. Launch the notebook**

```bash
jupyter notebook Document_RAG_Assistant.ipynb
```

**3. Run all cells, top to bottom**

Steps 1–7 work immediately — **no API key, no cost**. The embedding model runs locally.

**4. (Optional) Add a Gemini API key for generated answers**

Get a free key at [aistudio.google.com](https://aistudio.google.com) → **Get API key**, then:

```bash
setx GEMINI_API_KEY "your-key-here"
```

Restart the kernel and re-run. Without a key, the notebook still runs the full pipeline and
displays the retrieved chunks instead of a generated answer.

---

## 📁 Project Structure

```
Document_RAG_Assistant/
│
├── Document_RAG_Assistant.ipynb        # The full pipeline (Parts 1 & 2)
├── NovaTech_Company_Policy_Handbook.pdf # Source document
├── prepared_chunks.json                 # Output of Part 1 — 36 cleaned chunks
├── requirements.txt                     # Dependencies
└── README.md
```

---

## 🔄 Using Your Own Document

1. Place your PDF in the project folder
2. Update `PDF_PATH` near the top of the notebook
3. Run all cells again

Nothing else needs changing. The cleaning step **detects** repeating headers automatically
rather than hard-coding anything about the sample file.

---

## 🧠 Key Learnings

- **Garbage in, garbage out.** RAG quality is decided at the cleaning and chunking stage,
  long before the LLM is involved.
- **Overlap is not optional.** Without it, a sentence split across two chunks becomes
  unanswerable by either.
- **Semantic search beats keywords**, but it has no concept of truth — it only measures
  resemblance.
- **Grounding needs layers.** A score threshold catches obvious misses; only the prompt can
  catch a question that sounds right but has no answer in the document.
- **RAG is not training.** The model learns nothing. Update the PDF, re-run, and the answers
  change immediately.

---

## 🔮 Future Improvements

- Re-ranking with a cross-encoder — retrieve 10 chunks, keep the best 3
- Multi-document support (metadata already carries `source`)
- Persistent ChromaDB so embeddings are computed once
- A retrieval evaluation set to measure accuracy objectively
- A Streamlit interface for non-technical users

---

<p align="center">
  Built as part of the <b>RAGs and LangChain</b> program — implemented from scratch to
  understand every step.
</p>
