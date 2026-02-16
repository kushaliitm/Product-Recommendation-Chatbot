# Product Recommendation Chatbot

An AI-powered e-commerce chatbot application that leverages **LangChain**, **LangGraph**, and vector database technology to provide intelligent product recommendations and answer product-related queries based on real product reviews.

## Overview

This project implements a **Retrieval-Augmented Generation (RAG)** chatbot that:
- Ingests product review data into a vector database (AstraDB)
- Retrieves relevant product information based on user queries
- Generates contextual responses using LLM (Groq's Qwen-3.2B)
- Provides both web and CLI interfaces for interaction
- Maintains conversation memory using LangGraph

## Features

✨ **Key Capabilities:**
- 🤖 AI-powered product search and recommendations
- 📊 Vector-based semantic search using HuggingFace embeddings (BAAI/bge-base-en-v1.5)
- 💾 Persistent vector database integration with AstraDB
- 🧵 Thread-based conversation memory management
- 📝 Product review-based context for accurate recommendations
- 🌐 Multiple interfaces: Web UI (Flask & Streamlit), REST API
- 📈 Prometheus metrics for monitoring
- ⚡ Conversation summarization middleware

## Architecture

### Core Components

**Data Ingestion Pipeline:**
- `DataConverter`: Converts CSV product reviews into LangChain Documents
- `DataIngestor`: Manages vector database operations with AstraDB
- Embedding Model: BAAI/bge-base-en-v1.5 from HuggingFace

**RAG Agent:**
- `RAGAgentBuilder`: Constructs LangGraph-based agent with:
  - Product retriever tool (retrieves top-3 relevant reviews)
  - LLM: Groq's Qwen-3.2B model
  - Summarization middleware (summarizes every 10 messages, keeps 4)
  - In-memory checkpointer for conversation state

**Web Interfaces:**
- **Flask App** (`app.py`): RESTful API with Prometheus metrics, health checks
- **Streamlit App** (`streamlit_app.py`): Interactive chat UI with session management

### Data Flow

```
Product Reviews (CSV)
         ↓
   DataConverter
         ↓
   LangChain Documents
         ↓
   HuggingFace Embeddings
         ↓
   AstraDB Vector Store
         ↓
   RAG Agent (LLM + Retriever)
         ↓
   User Response
```

## Installation

### Prerequisites

- Python 3.8+
- AstraDB account and credentials
- Groq API key
- HuggingFace API key (for embeddings)

### Setup

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd Product-Recommendation-Chatbot
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure environment variables:**
   Create a `.env` file in the project root:
   ```env
   ASTRA_DB_API_ENDPOINT=<your-astradb-endpoint>
   ASTRA_DB_APPLICATION_TOKEN=<your-astradb-token>
   ASTRA_DB_KEYSPACE=<your-keyspace>
   GROQ_API_KEY=<your-groq-api-key>
   HF_TOKEN=<your-huggingface-token>
   HUGGINGFACEHUB_TOKEN=<your-huggingface-token>
   ```

4. **Prepare data:**
   Place your product review CSV at `data/product_review.csv` with columns:
   - `product_title`: Name of the product
   - `review`: Customer review text

## Usage

### Flask Web Application

Run the Flask server:
```bash
python app.py
```

The application will be available at `http://localhost:8000`

**Available Endpoints:**
- `GET /` - Main chat interface
- `POST /get` - Submit product queries
- `GET /health` - Health check endpoint
- `GET /metrics` - Prometheus metrics

### Streamlit Application

Run the Streamlit chat interface:
```bash
streamlit run streamlit_app.py
```

Access the app at `http://localhost:8501`

### Data Ingestion

Ingest product reviews into the vector database:
```bash
python -m products.data_ingestion
```

Set `load_existing=False` to process new data, or `load_existing=True` to use existing vectors.

## Project Structure

```
Product-Recommendation-Chatbot/
├── app.py                          # Flask web application
├── streamlit_app.py                # Streamlit chat interface
├── main.py                         # Entry point
├── requirements.txt                # Python dependencies
├── data/
│   └── product_review.csv          # Product reviews dataset
├── products/
│   ├── config.py                   # Configuration & environment variables
│   ├── data_converter.py           # CSV to LangChain Documents conversion
│   ├── data_ingestion.py           # Vector database operations
│   └── rag_agent.py                # RAG agent builder & retriever tool
├── utils/
│   ├── logger.py                   # Logging utilities
│   └── custom_exception.py         # Custom exception handling
└── frontend/
    ├── static/
    │   └── style.css               # UI styling
    └── templates/
        └── index.html              # Flask frontend
```

## Dependencies

Key packages:
- **LangChain** (1.2.1): LLM orchestration and agent framework
- **LangGraph**: Graph-based agent with memory management
- **LangChain Community**: Integrations and utilities
- **LangChain HuggingFace** (1.2.0): HuggingFace embeddings integration
- **LangChain Groq** (1.1.1): Groq LLM integration
- **LangChain AstraDB** (1.0.0): AstraDB vector database integration
- **Streamlit** (1.52.2): Interactive web UI
- **Flask** (3.1.2): Web framework
- **Pandas** (2.3.3): Data processing
- **Prometheus Client** (0.23.1): Metrics collection
- **Python-dotenv** (1.2.1): Environment variable management

## Configuration

Edit `products/config.py` to customize:
```python
class Config:
    EMBEDDING_MODEL = "BAAI/bge-base-en-v1.5"  # HuggingFace embedding model
    RAG_MODEL = "groq:qwen/qwen3-32b"          # LLM model
```

## Monitoring

Prometheus metrics are available at `/metrics` endpoint in Flask:
- `http_requests_total`: Total HTTP requests
- `model_predictions_total`: Total model predictions

## How It Works

1. **User Query**: User submits a product-related question
2. **Retrieval**: RAG agent retrieves top-3 relevant product reviews using semantic search
3. **Generation**: Groq LLM generates a response based on retrieved reviews
4. **Memory**: Conversation state is maintained using LangGraph's thread-based memory
5. **Summarization**: Long conversations are automatically summarized to maintain context

## Sample Query Examples

- "What are the best laptops under $1000?"
- "Tell me about the battery life of iPhone 15"
- "Compare features of Samsung and Sony TVs"
- "What do customers say about this product?"

## Error Handling

The chatbot gracefully handles edge cases:
- If no relevant products are found, it returns a helpful message
- Missing environment variables are caught during configuration
- Vector database connection errors are handled with fallback responses

## Deployment

### 1. Initial Setup

- **Push code to GitHub**  
  Push your project code to a GitHub repository.

- **Create a Dockerfile**  
  Write a `Dockerfile` in the root of your project to containerize the app.

- **Create Kubernetes Deployemtn file**  
  Make a file named 'llmops-k8s.yaml' 

- **Create a VM Instance on Google Cloud**

  - Go to VM Instances and click **"Create Instance"**
  - Name: ``
  - Machine Type:
    - Series: `E2`
    - Preset: `Standard`
    - Memory: `16 GB RAM`
  - Boot Disk:
    - Change size to `256 GB`
    - Image: Select **Ubuntu 24.04 LTS**
  - Networking:
    - Enable HTTP and HTTPS traffic

- **Create the Instance**

- **Connect to the VM**
  - Use the **SSH** option provided to connect to the VM from the browser.



### 2. Configure VM Instance

- **Clone your GitHub repo**

  ```bash
  git clone https://github.com/d-hackmt/flipkart-product-chatbot.git
  ls
  cd TESTING-9
  ls  # You should see the contents of your project
  ```

- **Install Docker**

  - Search: "Install Docker on Ubuntu"
  - Open the first official Docker website (docs.docker.com)
  - Scroll down and copy the **first big command block** and paste into your VM terminal
  - Then copy and paste the **second command block**
  - Then run the **third command** to test Docker:

    ```bash
    docker run hello-world
    ```

- **Run Docker without sudo**

  - On the same page, scroll to: **"Post-installation steps for Linux"**
  - Paste all 4 commands one by one to allow Docker without `sudo`
  - Last command is for testing

- **Enable Docker to start on boot**

  - On the same page, scroll down to: **"Configure Docker to start on boot"**
  - Copy and paste the command block (2 commands):

    ```bash
    sudo systemctl enable docker.service
    sudo systemctl enable containerd.service
    ```

- **Verify Docker Setup**

  ```bash
  systemctl status docker       # You should see "active (running)"
  docker ps                     # No container should be running
  docker ps -a                 # Should show "hello-world" exited container
  ```


### 3. Configure Minikube inside VM

- **Install Minikube**

  - Open browser and search: `Install Minikube`
  - Open the first official site (minikube.sigs.k8s.io) with `minikube start` on it
  - Choose:
    - **OS:** Linux
    - **Architecture:** *x86*
    - Select **Binary download**
  - Reminder: You have already done this on Windows, so you're familiar with how Minikube works

- **Install Minikube Binary on VM**

  - Copy and paste the installation commands from the website into your VM terminal

- **Start Minikube Cluster**

  ```bash
  minikube start
  ```

  - This uses Docker internally, which is why Docker was installed first

- **Install kubectl**

  - Search: `Install kubectl`
  - Run the first command with `curl` from the official Kubernetes docs
  - Run the second command to validate the download
  - Instead of installing manually, go to the **Snap section** (below on the same page)

  ```bash
  sudo snap install kubectl --classic
  ```

  - Verify installation:

    ```bash
    kubectl version --client
    ```

- **Check Minikube Status**

  ```bash
  minikube status         # Should show all components running
  kubectl get nodes       # Should show minikube node
  kubectl cluster-info    # Cluster info
  docker ps               # Minikube container should be running
  ```

### 4. Interlink your Github on VSCode and on VM

```bash
git config --global user.email "gyrogodnon@gmail.com"
git config --global user.name "data-guru0"

git add .
git commit -m "commit"
git push origin main
```

- When prompted:
  - **Username**: `data-guru0`
  - **Password**: GitHub token (paste, it's invisible)

---


### 5. Build and Deploy your APP on VM

```bash
## Point Docker to Minikube
eval $(minikube docker-env)

docker build -t flask-app:latest .

kubectl create secret generic llmops-secrets \
  --from-literal=GROQ_API_KEY="" \
  --from-literal=ASTRA_DB_APPLICATION_TOKEN="" \
  --from-literal=ASTRA_DB_KEYSPACE="default_keyspace" \
  --from-literal=ASTRA_DB_API_ENDPOINT="" \
  --from-literal=HF_TOKEN="" \
  --from-literal=HUGGINGFACEHUB_API_TOKEN=""


kubectl apply -f flask-deployment.yaml


kubectl get pods

### U will see pods runiing


kubectl port-forward svc/flask-service 5000:80 --address 0.0.0.0

## Now copy external ip and :5000 and see ur app there....


```

### 6. PROMETHEUS AND GRAFANA MONITORING OF YOUR APP

```bash
## Open another VM terminal 

kubectl create namespace monitoring

kubectl get ns


kubectl apply -f prometheus/prometheus-configmap.yaml

kubectl apply -f prometheus/prometheus-deployment.yaml

kubectl apply -f grafana/grafana-deployment.yaml

# check if its running 

kubectl get pods -n monitoring

## Check target health also..
## On IP:9090
kubectl port-forward --address 0.0.0.0 svc/prometheus-service -n monitoring 9090:9090

## Username:Pass --> admin:admin
kubectl port-forward --address 0.0.0.0 svc/grafana-service -n monitoring 3000:3000



Configure Grafana
Go to Settings > Data Sources > Add Data Source

Choose Prometheus

URL: http://prometheus-service.monitoring.svc.cluster.local:9090

Click Save & Test

Green success mesaage shown....


## License

This project is open source and available under the MIT License.

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues for bugs and feature requests.

## Support

For issues or questions, please open an issue in the repository.