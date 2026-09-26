# Multimodal RAG Pipeline for PDF Analysis

A complete **Multimodal Retrieval-Augmented Generation (RAG)** pipeline designed to ingest, index, and query complex PDF reports containing unstructured text, tables, and visual assets (graphs, diagrams, charts, and images).


## 🏗️ Architecture & How It Works

The system extracts text, tables, and images from PDF documents, generates AI visual summaries for images using vision language models, indexes all representations into a vector database (Pinecone), and routes user queries to text or vision LLMs based on retrieved content.
flowchart TD
    subgraph Ingestion["1. Document Processing & Ingestion"]
        PDF["📄 PDF Document (e.g., NovaCore Report)"]
        PDF --> PyMuPDF["PyMuPDF (fitz) Extractor"]
        
        PyMuPDF -->|Text Extraction| TextDocs["📝 Text Chunks (Modality: Text)"]
        PyMuPDF -->|Table Extraction| TableDocs["📊 Markdown Tables (Modality: Table)"]
        PyMuPDF -->|Image Extraction| ImgExtract["🖼️ Extracted Images (PNG/JPG)"]
        
        ImgExtract --> B64["Encode Image to Base64"]
        B64 --> VisionSummary["🤖 Groq Vision LLM (qwen/qwen3.8-27b)"]
        VisionSummary --> VisualDocs["🖼️ Visual Summaries (Modality: Visual)"]
    end

    subgraph Indexing["2. Vector Embedding & Storage"]
        TextDocs --> Aggregate["LangChain Document Store"]
        TableDocs --> Aggregate
        VisualDocs --> Aggregate
        
        Aggregate --> HFEmbed["HuggingFace Embeddings (all-MiniLM-L6-v2)"]
        HFEmbed --> PineconeDB[("🌲 Pinecone Vector Index (Serverless)")]
    end

    subgraph QueryPipeline["3. Query & Retrieval"]
        UserQuery["❓ User Query"] --> QueryEmbed["Query Vector Embedding"]
        QueryEmbed --> PineconeRetriever["Pinecone Retriever (k=5)"]
        PineconeDB -->|Cosine Similarity| PineconeRetriever
        PineconeRetriever --> ContextFormatter["Context & Source Formatter"]
    end

    subgraph Generation["4. Dynamic Multimodal Generation"]
        ContextFormatter --> CheckVisuals{"Retrieved Context Contains Images?"}
        
        CheckVisuals -->|No| TextLLM["🤖 Groq Text LLM (openai/gpt-oss-20b)"]
        CheckVisuals -->|Yes| VisionLLM["🤖 Groq Vision LLM (qwen/qwen3.8-27b) + Raw Images"]
        
        TextLLM --> FinalResponse["💬 Final Grounded Answer + Page Citations"]
        VisionLLM --> FinalResponse
    end


## ✨ Features

- **Multi-Modal Document Extraction**: Handles textual content, structured tabular data (converted to markdown tables), and visual assets seamlessly.
- **Vision Summarization**: Uses Groq Vision LLM (`qwen/qwen3.8-27b`) to convert images, charts, and diagrams into detailed semantic summaries.
- **Serverless Vector Search**: Stores vectors in Pinecone with cosine similarity indexing under custom namespaces.
- **Dynamic Multimodal Routing**: Automatically routes text-only queries to fast text LLMs (`openai/gpt-oss-20b`) and visual-dependent queries to Vision LLMs with attached images.
- **Page-Level Grounding**: Returns explicit page numbers and source metadata for verifiable answers.

---

## 🚀 Setup & Installation

### 1. Prerequisites
- Python 3.10+
- Groq API Key
- Pinecone API Key
