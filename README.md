# Vector Database Python Learning Project

This project demonstrates vector database implementations in Python using two different database level:
1. **SQLite + sqlite-vec**: Low-level vector search using an SQLite extension.
2. **ChromaDB**: High-level vector database for text and multimodal (image) embeddings.

## Architecture & Notebooks

The project is structured into three progressive Jupyter Notebooks:

### 1. `01_sqlite.ipynb` - SQLite Vector Search Foundation
- Uses the `sqlite-vec` extension for vector operations.
- Implements a custom embedding function (`embed_text()`) using SHA256 hashes for demonstration.
- Demonstrates manual vector table management with the `vec0` virtual table.
- Vectors are stored as JSON and retrieved as numpy arrays.
- Uses L2 distance for similarity search via the `MATCH` operator.

### 2. `02_chroma_text.ipynb` - ChromaDB Text Embeddings
- Uses ChromaDB's default embedding function, which automatically embeds text.
- Connects to an in-memory client (non-persistent).
- Queries return: distances, documents, and metadatas.
- Supports multi-query batching.

### 3. `03_chroma_img.ipynb` - ChromaDB Multimodal (Image) Embeddings
- Uses a **Persistent client** storing data in `chroma_multimodal_local_dataloader/`.
- Uses `OpenCLIPEmbeddingFunction()` for image embeddings.
- Utilizes `ImageLoader()` to load images from URIs.
- Collections store images via file paths (the `uris` parameter).
- Configured to use Cosine similarity.

## Setup Instructions

1. **Create and activate a virtual environment**:
   ```bash
   python -m venv .venv
   
   # Linux/Mac
   source .venv/bin/activate
   
   # Windows
   .venv\Scripts\activate
   ```

2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Run Jupyter Notebook**:
   ```bash
   jupyter notebook
   ```

## Important Notes on ChromaDB Images
When querying images with ChromaDB in this project, ensure you include the `"data"` field in the `include` parameter to retrieve actual image arrays for visualization, as the `documents` field will be empty.
