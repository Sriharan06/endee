# Smart Study Buddy — AI Learning Assistant powered by Endee Vector DB

> Ask any question → Endee retrieves the most relevant study notes → AI generates a grounded answer.  
> Built as part of the Endee Internship Application.

---

## Project Overview

Students waste hours searching textbooks for the right explanation.  
**Smart Study Buddy** solves this with semantic search: study material is indexed into the **Endee vector database**, and when a student asks a question, Endee finds the most conceptually relevant notes — not just keyword matches.

A **RAG (Retrieval-Augmented Generation)** pipeline then feeds those notes into an LLM to generate a clear, accurate, hallucination-resistant answer.

---

## Features

| Feature | Description |
|---|---|
| Semantic Q&A | Ask in natural language; Endee finds relevant study chunks by meaning |
| RAG Pipeline | Retrieved notes become LLM context for grounded answer generation |
| Multi-Subject KB | Covers ML, Deep Learning, NLP, Algorithms, Data Structures, Python |
| Quiz Mode | Auto-generated quizzes per subject to test understanding |
| Smart Recommendations | Vector similarity suggests what to study next |
| Session Log | All Q&A results saved to `session_log.json` |
| Zero dependencies | Runs on Python stdlib; swap in real Endee SDK for production |

---

## System Design

```
Student Question (text)
        │
        ▼
  ┌─────────────────────┐
  │  Endee Vector DB    │  ◄── upsert() all study note chunks at startup
  │  (semantic index)   │
  └─────────────────────┘
        │  search(query, top_k=3)
        ▼
  Top-K Relevant Chunks  (by cosine similarity)
        │
        ▼
  RAG Context Builder
  (combine retrieved texts)
        │
        ▼
  LLM (Claude / GPT-4)   ◄── "Answer this question using ONLY the context below"
        │
        ▼
  Grounded Answer  +  Source Citations
        │
        ▼
  session_log.json
```

### How Endee is Used

| Step | Endee Operation | Purpose |
|---|---|---|
| Startup | `db.upsert(chunk_id, text, metadata)` | Index all study notes as vectors |
| Q&A | `db.search(question, top_k=3)` | Retrieve semantically closest notes |
| Recommendation | `db.search(current_topic, top_k=4)` | Find related topics to study next |

Endee's vector similarity search lets the system find relevant notes even when the student's question uses **different words** than the stored material — understanding intent, not just matching tokens.

---

## Project Structure

```
smart-study-buddy/
├── main.py            # Full pipeline: indexing, Q&A, quiz, recommendations
├── requirements.txt   # Dependencies (stdlib only for demo)
└── README.md          # This file
```

---

## How to Run

### Prerequisites
- Python 3.9+
- No external packages needed for the demo

### Steps

```bash
# 1. Clone your forked Endee repo
git clone https://github.com/<your-username>/endee.git
cd endee/smart-study-buddy

# 2. Run the study buddy
python main.py
```

---

## Sample Output

```
══════════════════════════════════════════════════════════════
  Smart Study Buddy — powered by Endee Vector DB
══════════════════════════════════════════════════════════════

[Endee] Indexing study material...
  ✅ 13 chunks indexed into Endee vector store.

══════════════════════════════════════════════════════════════
  Question: What is RAG and how does it reduce hallucination in LLMs?
══════════════════════════════════════════════════════════════

  Top Retrieved Chunks (Endee semantic search):
  1. [NLP → RAG — Retrieval-Augmented Generation]
     Score: 0.6821 ███████████████████████████
  2. [Deep Learning → Transformers & Attention]
     Score: 0.3104 ████████████
  3. [Machine Learning → Supervised Learning]
     Score: 0.1823 ███████

  📚 Answer:
  Based on your study notes:
  RAG combines retrieval and generation: a retriever fetches relevant documents
  from a vector database, and an LLM generates an answer grounded in those documents.
  Steps: embed the query → search vector DB → retrieve top-k chunks → prepend to
  LLM prompt → generate. RAG reduces hallucination and keeps the LLM's knowledge
  current without retraining.

══════════════════════════════════════════════════════════════
  Quiz: Machine Learning  (3 questions)
══════════════════════════════════════════════════════════════

  Q1: What does overfitting mean?
  A:  The model memorises training data but performs poorly on unseen data.

  Q2: Name two regularisation techniques.
  A:  L1 (Lasso) and L2 (Ridge) regularisation add weight penalties to the loss.
```

---

## Upgrading to Production Endee

```python
from endee import Client
import os

client = Client(api_key=os.environ["ENDEE_API_KEY"])

# Index a note chunk
client.upsert(
    collection="study_notes",
    id="nlp_001",
    text="Word embeddings represent words as dense vectors...",
    metadata={"subject": "NLP", "topic": "Word Embeddings"}
)

# Semantic search
results = client.search(
    collection="study_notes",
    query="How do transformers work?",
    top_k=3
)
```

---

## Vector Search Concepts Demonstrated

- **Embeddings**: Study notes → dense vectors capturing conceptual meaning
- **Cosine Similarity**: Ranks notes by angle in vector space (1.0 = identical concept)
- **Nearest-Neighbour Search**: Endee finds notes closest to the student's question
- **RAG**: Retrieved chunks ground the LLM, preventing fabricated answers
- **Semantic Recommendations**: Vector proximity suggests related topics to study next

---

## Roadmap / Future Improvements

- Integrate `sentence-transformers` (all-MiniLM-L6-v2) for richer dense embeddings
- Connect Claude / GPT-4 API for real LLM answer generation
- Add a FastAPI + React frontend for a full web study app
- Upload your own PDF notes → auto-chunk → index into Endee
- Track weak topics per student using session history
- Spaced repetition reminders based on quiz performance

---

## Author

Submitted for the **Endee AI/ML Internship 2024**.  
Demonstrates: Endee vector DB integration, RAG pipeline, semantic search, and practical AI application design.
