# Financial RAG Chatbot & Document Automation 🤖📈

An automated Retrieval-Augmented Generation (RAG) pipeline built with n8n to dynamically ingest, parse, and vectorize financial reports, featuring an intelligent intent-routing system powered by local LLMs.

## 📖 Overview

This project automates the extraction and semantic analysis of financial documents (like quarterly earnings reports). Instead of manually searching through dense PDFs, users can query a chat interface. An intelligent routing layer classifies the user's intent to either query the vector database for specific financial data or respond conversationally for general inquiries.

## ✨ Key Features

*   **Automated Data Ingestion:** Actively monitors a Google Drive folder for new financial reports and downloads them automatically.
*   **Intelligent Intent Routing:** Uses a custom HTTP routing layer with a lightweight local LLM to classify user queries.
    *   *Search Intent:* Routes to the RAG pipeline for data retrieval.
    *   *Direct Intent:* Routes to a standard conversational AI agent.
*   **Vectorized Knowledge Base:** Processes documents using LangChain (recursive character splitting) and embeds them into a Pinecone vector database for lightning-fast semantic search.
*   **Local AI Integration:** Completely integrated with Ollama, utilizing `qwen2.5:1.5b` for generation and `nomic-embed-text` for embeddings, ensuring data privacy and reducing API costs.
*   **Context-Aware Chat:** Maintains conversational memory and restricts the AI to answer *only* based on the retrieved financial context to prevent hallucinations.

## 🛠️ Tech Stack

*   **Workflow Automation:** [n8n](https://n8n.io/)
*   **Frameworks:** LangChain
*   **LLMs (Local):** Ollama (`qwen2.5:1.5b`)
*   **Embeddings:** `nomic-embed-text`
*   **Vector Database:** Pinecone
*   **Integrations:** Google Drive API, REST APIs

## 🏗️ Architecture Flow

1.  **Ingestion:** A Google Drive trigger detects new file uploads.
2.  **Processing:** Files are downloaded, parsed by LangChain, and split into chunks (100-character overlap).
3.  **Embedding:** Text chunks are vectorized using Ollama and upserted into Pinecone namespaces based on the filename.
4.  **Query Routing:** When a chat message is received, an LLM evaluates the prompt. If it detects a request for financial data, it outputs `SEARCH`. Otherwise, it outputs `DIRECT`.
5.  **Retrieval & Generation (RAG):**
    *   The system searches Google Drive for the relevant filename.
    *   A Pinecone vector search pulls the top 5 most relevant semantic chunks.
    *   The Agent synthesizes the retrieved chunks into a cited, factual response.

## 🚀 Getting Started

### Prerequisites
*   [n8n](https://n8n.io/) installed (local, Docker, or Cloud).
*   [Ollama](https://ollama.com/) installed and running locally with the following models pulled:
    ```bash
    ollama run qwen2.5:1.5b
    ollama pull nomic-embed-text
    ```
*   A [Pinecone](https://www.pinecone.io/) account and API key.
*   Google Cloud Console account with the Google Drive API enabled and OAuth2 credentials generated.

### Installation

1.  Clone this repository.
2.  Open your n8n instance and click **Import from File** (or copy the JSON and click **Import from Clipboard**).
3.  Select the `Rag_Chatbot.json` workflow file.
4.  Configure your credentials inside the n8n nodes:
    *   Authenticate the **Google Drive** nodes.
    *   Add your **Pinecone API Key**.
    *   Ensure the **Ollama** nodes point to your local instance (default: `http://127.0.0.1:11434`).
5.  Set up a Google Drive folder for your source documents and map the Folder ID in the Trigger node.
6.  Activate the workflow!

## 💡 Usage

Once the workflow is active, upload a financial report (e.g., an Apple Q3 Earnings PDF) to your designated Google Drive folder. Wait a moment for the ingestion pipeline to vectorize the document.

Access the n8n chat trigger interface and ask questions like:
*   *"What was the total revenue reported for the quarter?"*
*   *"How did iPhone sales compare to Mac sales?"*

The bot will route the query, fetch the exact data points from Pinecone, and generate an answer grounded entirely in the uploaded report.
