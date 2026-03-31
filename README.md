# Hughes Network Systems — Field Technician AI Chatbot

An AI-powered chatbot built for Hughes Network Systems field technicians.
Instead of calling customer support, technicians can ask the chatbot
installation and troubleshooting questions and get instant answers.

## Architecture — RAG (Retrieval Augmented Generation)
```
User Question → Embed (OpenAI) → Search OpenSearch (k=5) → GPT Answer
```

## Tech Stack

- Java 17 + Spring Boot 3.5
- AWS S3 — document storage
- OpenAI text-embedding-3-small — vector embeddings
- AWS OpenSearch — vector database (knn search)
- GPT-3.5-turbo — answer generation
- Docker — local OpenSearch instance

## How it works

1. 10 Hughes technician documents uploaded to AWS S3
2. Each doc split into ~115 chunks by paragraph
3. Each chunk converted to 1536-dimension vector via OpenAI
4. Vectors stored in OpenSearch with knn_vector field
5. User asks question → embedded → top 5 similar chunks found
6. Chunks fed to GPT as context → natural language answer returned
7. Confidence threshold (0.45) prevents hallucination on unknown topics

## API
```
GET /bot/ask?question=what do I do when signal is weak
```

Response:
```json
{
  "question": "what do I do when signal is weak",
  "answer": "Check cable connections first, then re-align the dish..."
}
```

## Knowledge Base

| Doc | Topic |
|-----|-------|
| 01_dish_installation | Satellite dish installation steps |
| 02_signal_alignment | Dish alignment for optimal signal |
| 03_modem_setup | HT2000W modem setup and activation |
| 04_low_signal_troubleshoot | Low signal diagnosis and fixes |
| 05_no_internet_troubleshoot | No internet connection troubleshooting |
| 06_error_codes | Error codes and resolutions |
| 07_cable_connections | Cable types and connector guide |
| 08_customer_faq | Common customer questions |
| 09_equipment_specs | Technical specifications |
| 10_escalation_guide | When and how to escalate |

## Setup

### Prerequisites
- Java 17+
- Docker
- AWS account with S3 bucket
- OpenAI API key

### Run locally
```bash
# Start OpenSearch
docker run -d --name opensearch -p 9200:9200 \
  -e "discovery.type=single-node" \
  -e "DISABLE_SECURITY_PLUGIN=true" \
  opensearchproject/opensearch:2.11.0

# Configure application.properties
aws.region=us-east-1
aws.s3.bucket=your-bucket-name
openai.api.key=your-openai-key
opensearch.endpoint=http://localhost:9200

# Run Spring Boot
./mvnw spring-boot:run
```

## Developer

Harshh — Software Developer at Hughes Network Systems