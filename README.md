# Product Recommendation Chatbot

An end-to-end **LLMOps RAG chatbot system** that provides intelligent product recommendations using real customer reviews.  
The application integrates **LangChain, LangGraph, AstraDB vector search, Groq LLM, Docker, Kubernetes, Prometheus, and Grafana** to demonstrate production-ready AI deployment.

---

# Overview

This project implements a **Retrieval-Augmented Generation (RAG)** chatbot that:

- Indexes product reviews into a vector database (AstraDB)
- Retrieves semantically relevant product context
- Generates responses using Groq LLM (Qwen)
- Maintains conversation memory via LangGraph
- Exposes REST + Web UI interfaces
- Deploys via Docker + Kubernetes
- Monitored with Prometheus & Grafana

---

# Features

- Semantic product search across reviews  
- HuggingFace embeddings (BAAI/bge-base-en-v1.5)  
- AstraDB persistent vector storage  
- LangGraph conversational memory  
- Flask REST API + Streamlit UI  
- Prometheus metrics endpoint  
- Kubernetes deployment ready  
- Observability with Grafana dashboards  

---

# Architecture

```
                ┌────────────────────┐
                │   Product Reviews   │
                └─────────┬──────────┘
                          ↓
                  HuggingFace Embeddings
                          ↓
                      AstraDB
                          ↓
                  Retriever Tool
                          ↓
                 LangGraph RAG Agent
                          ↓
                      Groq LLM
                          ↓
              Flask / Streamlit Interface
                          ↓
                 Kubernetes Service
                          ↓
                       User
```

Monitoring flow:

```
Flask Metrics → Prometheus → Grafana
```

---

# Project Structure

```
Product-Recommendation-Chatbot/
│
├── app.py
├── streamlit_app.py
├── main.py
├── requirements.txt
├── Dockerfile
├── flask-deployment.yaml
│
├── data/
│   └── product_review.csv
│
├── products/
│   ├── config.py
│   ├── data_converter.py
│   ├── data_ingestion.py
│   └── rag_agent.py
│
├── utils/
│
├── frontend/
│
├── prometheus/
│   ├── prometheus-configmap.yaml
│   └── prometheus-deployment.yaml
│
└── grafana/
    └── grafana-deployment.yaml
```

---

# Local Installation

## Prerequisites

- Python 3.9+
- AstraDB account
- Groq API key
- HuggingFace token

---

## Setup

```bash
git clone https://github.com/kushaliitm/Product-Recommendation-Chatbot.git
cd Product-Recommendation-Chatbot

python -m venv venv
source venv/bin/activate

pip install -r requirements.txt
```

---

## Environment Variables

Create `.env`:

```
ASTRA_DB_API_ENDPOINT=
ASTRA_DB_APPLICATION_TOKEN=
ASTRA_DB_KEYSPACE=default_keyspace
GROQ_API_KEY=
HF_TOKEN=
HUGGINGFACEHUB_API_TOKEN=
```

---

# Usage

## Run Flask API

```bash
python app.py
```

Open:

```
http://localhost:8000
```

---

## Run Streamlit UI

```bash
streamlit run streamlit_app.py
```

Open:

```
http://localhost:8501
```

---

# Data Ingestion

```bash
python -m products.data_ingestion
```

---

# Deployment (LLMOps Stack)

This project demonstrates full AI system deployment on **Google Cloud VM → Docker → Kubernetes → Monitoring**.

---

# 1. Create Google Cloud VM

Configuration:

- Machine: E2 Standard
- RAM: 16 GB
- Disk: 256 GB
- OS: Ubuntu 24.04
- Enable HTTP/HTTPS

SSH into VM.

---

# 2. Clone Repo on VM

```bash
git clone https://github.com/kushaliitm/Product-Recommendation-Chatbot.git
cd Product-Recommendation-Chatbot
```

---

# 3. Install Docker

```bash
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

---

# 4. Install Minikube & kubectl

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

minikube start --driver=docker

sudo snap install kubectl --classic
```

Verify:

```bash
kubectl get nodes
```

---

# 5. Build Docker Image inside Minikube

```bash
eval $(minikube docker-env)
docker build -t product-chatbot:latest .
```

---

# 6. Create Kubernetes Secrets

```bash
kubectl create secret generic chatbot-secrets   --from-literal=GROQ_API_KEY=YOUR_KEY   --from-literal=ASTRA_DB_APPLICATION_TOKEN=YOUR_TOKEN   --from-literal=ASTRA_DB_KEYSPACE=default_keyspace   --from-literal=ASTRA_DB_API_ENDPOINT=YOUR_ENDPOINT   --from-literal=HF_TOKEN=YOUR_TOKEN   --from-literal=HUGGINGFACEHUB_API_TOKEN=YOUR_TOKEN
```

---

# 7. Deploy App to Kubernetes

```bash
kubectl apply -f flask-deployment.yaml
kubectl get pods
```

---

# 8. Access Application

```bash
kubectl port-forward svc/flask-service 5000:80 --address 0.0.0.0
```

Open:

```
http://VM_EXTERNAL_IP:5000
```

---

# Monitoring (Prometheus + Grafana)

---

# 9. Create Monitoring Namespace

```bash
kubectl create namespace monitoring
```

---

# 10. Deploy Prometheus

```bash
kubectl apply -f prometheus/prometheus-configmap.yaml
kubectl apply -f prometheus/prometheus-deployment.yaml
```

Access:

```bash
kubectl port-forward --address 0.0.0.0 svc/prometheus-service -n monitoring 9090:9090
```

Open:

```
http://VM_IP:9090
```

---

# 11. Deploy Grafana

```bash
kubectl apply -f grafana/grafana-deployment.yaml
```

Access:

```bash
kubectl port-forward --address 0.0.0.0 svc/grafana-service -n monitoring 3000:3000
```

Open:

```
http://VM_IP:3000
```

Login:

```
admin / admin
```

---

# 12. Connect Grafana to Prometheus

Grafana → Settings → Data Sources → Add Prometheus

```
URL:
http://prometheus-service.monitoring.svc.cluster.local:9090
```

Save & Test.

---

# Metrics Exposed

```
/metrics
```

Includes:

- http_requests_total
- model_predictions_total
- request_latency_seconds

---

# Deployment Verification

- Docker running
- Minikube cluster active
- Pod running
- Flask accessible
- Prometheus scraping
- Grafana dashboard visible

---

# Production Considerations

- Kubernetes Secrets for credentials
- Containerized inference API
- Vector DB externalized (AstraDB)
- Observability integrated
- Cloud-portable architecture

---

# Future Improvements

- GKE / EKS deployment
- Helm charts
- Ingress + domain
- CI/CD pipeline
- Autoscaling (HPA)
- Centralized logging

---

# Tech Stack

- LangChain
- LangGraph
- AstraDB
- Groq LLM
- HuggingFace Embeddings
- Flask
- Streamlit
- Docker
- Kubernetes
- Prometheus
- Grafana

---

# License

MIT License

---

