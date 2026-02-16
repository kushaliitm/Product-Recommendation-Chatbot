# Product Recommendation Chatbot

An AI-powered e-commerce chatbot application that leverages **LangChain**, **LangGraph**, and vector database technology to provide intelligent product recommendations and answer product-related queries based on real product reviews.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [License](#license)
- [Contributing](#contributing)

## Overview

This project implements a **Retrieval-Augmented Generation (RAG)** chatbot that:
- Ingests product review data into a vector database (AstraDB)
- Retrieves relevant product information based on user queries
- Generates contextual responses using LLM (Groq's Qwen-3.2B)
- Provides multiple interfaces for interaction (Flask, Streamlit)
- Maintains conversation memory using LangGraph

## ✨ Features

- **🤖 AI-powered product search** - Semantic search across product reviews
- **📊 Vector embeddings** - HuggingFace embeddings (BAAI/bge-base-en-v1.5)
- **💾 Persistent vector DB** - AstraDB integration for scalable storage
- **🧵 Conversation memory** - Thread-based memory management
- **🌐 Multiple interfaces** - Flask REST API & Streamlit interactive UI
- **📈 Monitoring** - Prometheus metrics and health checks
- **⚡ Auto-summarization** - Conversation summarization middleware

## Architecture

### Core Components

**Data Ingestion Pipeline**
- **DataConverter**: Transforms CSV product reviews into LangChain Documents
- **DataIngestor**: Manages AstraDB vector store operations
- **Embeddings**: BAAI/bge-base-en-v1.5 from HuggingFace

**RAG Agent**
- **RAGAgentBuilder**: LangGraph-based agent with:
  - Product retriever tool (top-3 relevant reviews)
  - Groq Qwen-3.2B LLM
  - Summarization middleware (every 10 messages)
  - In-memory conversation checkpointer

**Web Interfaces**
- **Flask** (`app.py`) - RESTful API with metrics & health checks
- **Streamlit** (`streamlit_app.py`) - Interactive chat UI with session state

### Data Flow

```
CSV Dataset → DataConverter → LangChain Docs → HuggingFace Embeddings → AstraDB
                                                                           ↓
                                                      RAG Agent + Retriever Tool
                                                           ↓
                                                      Groq LLM Response
```

## Installation

### Prerequisites

- Python 3.8 or higher
- AstraDB account and credentials
- Groq API key
- HuggingFace API token

### Setup Steps

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd Product-Recommendation-Chatbot
   ```

2. **Create virtual environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Configure environment variables:**
   Create a `.env` file in the root directory:
   ```env
   ASTRA_DB_API_ENDPOINT=https://your-instance-id-region.apps.astra.datastax.com
   ASTRA_DB_APPLICATION_TOKEN=AstraCS:...
   ASTRA_DB_KEYSPACE=default_keyspace
   GROQ_API_KEY=gsk_...
   HF_TOKEN=hf_...
   HUGGINGFACEHUB_API_TOKEN=hf_...
   ```

5. **Prepare product data:**
   Place your product reviews CSV at `data/product_review.csv` with required columns:
   - `product_title` - Product name
   - `review` - Customer review text

## Usage

### Running Flask Application

```bash
python app.py
```
Accessible at `http://localhost:8000`

**Available Endpoints:**
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Chat interface |
| `/get` | POST | Submit query |
| `/health` | GET | Health status |
| `/metrics` | GET | Prometheus metrics |

### Running Streamlit Application

```bash
streamlit run streamlit_app.py
```
Accessible at `http://localhost:8501`

**Features:**
- Interactive chat interface
- Session state management
- New chat functionality
- Real-time response streaming

### Data Ingestion

Initialize or update the vector database:

```bash
# Use existing vectors
python -m products.data_ingestion

# Process new data
python -c "from products.data_ingestion import DataIngestor; DataIngestor().ingest(load_existing=False)"
```

## Project Structure

```
Product-Recommendation-Chatbot/
├── app.py                      # Flask application
├── streamlit_app.py            # Streamlit chat UI
├── main.py                     # Entry point
├── requirements.txt            # Python dependencies
├── .dockerignore               # Docker ignore patterns
├── README.md                   # This file
├── data/
│   └── product_review.csv      # Product reviews dataset
├── products/
│   ├── __init__.py
│   ├── config.py               # Configuration management
│   ├── data_converter.py       # CSV to Document conversion
│   ├── data_ingestion.py       # Vector DB operations
│   └── rag_agent.py            # RAG agent builder
├── utils/
│   ├── __init__.py
│   ├── logger.py               # Logging utilities
│   └── custom_exception.py     # Custom exceptions
└── frontend/
    ├── static/
    │   └── style.css           # Styling
    └── templates/
        └── index.html          # Flask template
```

## Configuration

Edit `products/config.py` to customize models and settings:

```python
class Config:
    EMBEDDING_MODEL = "BAAI/bge-base-en-v1.5"  # HuggingFace model
    RAG_MODEL = "groq:qwen/qwen3-32b"          # Groq model
    # AstraDB and API keys loaded from .env
```

## Deployment

### Docker Deployment

1. **Build Docker image:**
   ```bash
   docker build -t product-chatbot:latest .
   ```

2. **Run container:**
   ```bash
   docker run -p 8000:8000 \
     --env-file .env \
     product-chatbot:latest
   ```

### Kubernetes Deployment

1. **Create secrets:**
   ```bash
   kubectl create secret generic chatbot-secrets \
     --from-literal=GROQ_API_KEY=<key> \
     --from-literal=ASTRA_DB_APPLICATION_TOKEN=<token> \
     --from-literal=ASTRA_DB_API_ENDPOINT=<endpoint> \
     --from-literal=HF_TOKEN=<token>
   ```

2. **Apply deployment:**
   ```bash
   kubectl apply -f k8s-deployment.yaml
   ```

### Monitoring with Prometheus & Grafana

The application exposes Prometheus metrics at `/metrics`:
- `http_requests_total` - Total HTTP requests
- `model_predictions_total` - Total model predictions

Configure Grafana to visualize these metrics using the Prometheus data source.

## How It Works

1. **User submits query** - E.g., "What's the best laptop?"
2. **Vector search** - Retrieves top-3 relevant reviews from AstraDB
3. **LLM processing** - Groq LLM generates response based on reviews
4. **Memory management** - LangGraph maintains conversation history
5. **Conversation optimization** - Summarizes long conversations automatically

## Error Handling

The chatbot gracefully handles:
- Missing relevant products → Polite fallback message
- Connection issues → Retry logic with timeouts
- Invalid queries → Context-aware error responses

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| langchain | 1.2.1 | LLM orchestration |
| langchain-groq | 1.1.1 | Groq integration |
| langchain-astradb | 1.0.0 | AstraDB vector store |
| langchain-huggingface | 1.2.0 | Embeddings |
| streamlit | 1.52.2 | Web UI |
| flask | 3.1.2 | REST API |
| pandas | 2.3.3 | Data processing |
| prometheus-client | 0.23.1 | Metrics |

## License

MIT License - See LICENSE file for details

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit changes (`git commit -m 'Add feature'`)
4. Push to branch (`git push origin feature/improvement`)
5. Open a Pull Request

## Support

For issues, questions, or suggestions:
- Open an [GitHub Issue](https://github.com/kushaliitm/Product-Recommendation-Chatbot/issues)
- Check existing discussions
- Review the documentation

---

**Built with ❤️ using LangChain, LangGraph, and AstraDB**