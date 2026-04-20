# 🤖 Field Technician AI Assistant

> An AI-powered RAG chatbot that lets field technicians ask troubleshooting questions in plain English and get instant, context-aware answers — no support calls, no manual digging.

![Java](https://img.shields.io/badge/Java-17-orange?logo=java)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5-brightgreen?logo=springboot)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--3.5-412991?logo=openai)
![OpenSearch](https://img.shields.io/badge/OpenSearch-2.11-blue?logo=opensearch)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker)
![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20S3-FF9900?logo=amazonaws)

---

## 📌 Problem Statement

Field technicians often face time-sensitive issues — weak signal, modem failures, cable errors — and have to either call support or search through stacks of PDF manuals. This chatbot eliminates that friction by acting as an intelligent, always-available assistant trained on your technical knowledge base.

---

## 🏗️ Architecture — RAG (Retrieval Augmented Generation)

```
┌─────────────────────────────────────────────────────────────┐
│                      INGESTION PIPELINE                      │
│                                                             │
│  Technical Docs (S3) → Chunking → OpenAI Embeddings         │
│                                 → OpenSearch (knn_vector)    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                       QUERY PIPELINE                         │
│                                                             │
│  User Question → Embed → knn Search (top 5 chunks)          │
│               → GPT-3.5-turbo → Natural Language Answer     │
└─────────────────────────────────────────────────────────────┘
```

**Confidence guard:** Answers below similarity threshold (0.45) are rejected to prevent hallucination on out-of-scope questions.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Backend | Java 17, Spring Boot 3.5 |
| Embeddings | OpenAI `text-embedding-3-small` (1536-dim) |
| Vector DB | OpenSearch 2.11 (knn_vector, cosine similarity) |
| LLM | GPT-3.5-turbo |
| Document Storage | AWS S3 |
| Deployment | AWS EC2, Docker |

---

## ⚙️ How It Works

1. **Ingest** — Technical PDF/text documents are uploaded to AWS S3
2. **Chunk** — Each document is split into ~115 chunks by paragraph
3. **Embed** — Each chunk is converted to a 1536-dimension vector via OpenAI
4. **Index** — Vectors stored in OpenSearch with `knn_vector` field mapping
5. **Query** — User question → embedded → top-5 similar chunks retrieved via knn search
6. **Answer** — Chunks injected as context → GPT-3.5-turbo generates a precise answer
7. **Guard** — Confidence score threshold (0.45) blocks off-topic responses

---

## 🔌 API

### Ask a question

```
GET /bot/ask?question=what do I do when signal is weak
```

**Response:**
```json
{
  "question": "what do I do when signal is weak",
  "answer": "Check all cable connections first, then re-align the dish toward the satellite. If signal strength remains below 60%, run a diagnostic reset from the modem settings page..."
}
```

---

## 📚 Sample Knowledge Base Structure

| Document | Topic |
|---|---|
| `01_dish_installation` | Satellite dish installation steps |
| `02_signal_alignment` | Dish alignment for optimal signal |
| `03_modem_setup` | Modem setup and activation |
| `04_low_signal_troubleshoot` | Low signal diagnosis and fixes |
| `05_no_internet_troubleshoot` | No internet connection troubleshooting |
| `06_error_codes` | Error codes and resolutions |
| `07_cable_connections` | Cable types and connector guide |
| `08_customer_faq` | Common customer questions |
| `09_equipment_specs` | Technical specifications |
| `10_escalation_guide` | When and how to escalate |

> 💡 **Generic-ready:** Replace these documents with any domain-specific knowledge base (healthcare, telecom, IT support, manufacturing) to adapt the chatbot for a new industry.

---

## 🚀 Running Locally

### Prerequisites

- Java 17+
- Docker
- AWS account with S3 bucket
- OpenAI API key

### Step 1 — Start OpenSearch

```bash
docker run -d --name opensearch -p 9200:9200 \
  --platform linux/amd64 \
  -e "discovery.type=single-node" \
  -e "DISABLE_SECURITY_PLUGIN=true" \
  opensearchproject/opensearch:2.11.0
```

> ⚠️ `--platform linux/amd64` required on Apple Silicon (M1/M2) to avoid ARM/AMD64 mismatch.

### Step 2 — Configure `application.properties`

```properties
aws.region=us-east-1
aws.s3.bucket=your-bucket-name
openai.api.key=your-openai-key
opensearch.endpoint=http://localhost:9200
```

### Step 3 — Run the App

```bash
./mvnw spring-boot:run
```

The app will load documents from S3, embed them, and index into OpenSearch on first startup. An **idempotency guard** prevents re-embedding on subsequent restarts (saving OpenAI costs).

---

## 🧠 Key Engineering Decisions

| Challenge | Solution |
|---|---|
| Prevent repeated OpenAI charges on restart | Idempotency guard — checks if vectors already exist before re-embedding |
| Docker ARM/AMD64 mismatch on EC2 | Added `--platform linux/amd64` flag to Docker run |
| OpenSearch knn_vector serialization | Mapped Java `List<Float>` with custom serializer to match OpenSearch schema |
| EC2 OOM errors | Moved OpenSearch to local Docker; EC2 used for Spring Boot only |
| Hallucination on unknown topics | Confidence threshold (0.45) rejects low-similarity results |

---

## 🌱 Extending This Project

This chatbot is designed to be **domain-agnostic**. To adapt it:

1. Replace the S3 documents with your own knowledge base
2. Update the system prompt in `ChatService.java` for your domain context
3. Adjust chunk size and similarity threshold as needed
4. Redeploy — the ingestion pipeline handles everything automatically

**Potential use cases:** IT helpdesk, HR policy assistant, legal document Q&A, manufacturing SOPs, medical equipment support.

---

## 👨‍💻 Developer

**Harsha Vardhan** — Software Engineer | AI & Backend Systems

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?logo=linkedin)](https://www.linkedin.com/in/harsha-vardhan-dev07)
[![GitHub](https://img.shields.io/badge/GitHub-HarshaVardhan--49-181717?logo=github)](https://github.com/HarshaVardhan-49)

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
