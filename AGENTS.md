# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python learning project demonstrating vector database implementations using two approaches:
1. **SQLite + sqlite-vec**: Low-level vector search using SQLite extension
2. **ChromaDB**: High-level vector database for text and multimodal (image) embeddings

## Architecture

### Three Progressive Notebooks

1. **01_sqlite.ipynb** - SQLite Vector Search Foundation
   - Uses `sqlite-vec` extension for vector operations
   - Implements custom embedding function (`embed_text()`) using SHA256 hash
   - Demonstrates manual vector table management with `vec0` virtual table
   - Stores vectors as JSON, retrieves as numpy arrays
   - Uses L2 distance for similarity search via `MATCH` operator

2. **02_chroma_text.ipynb** - ChromaDB Text Embeddings
   - Uses ChromaDB's default embedding function (automatically embeds text)
   - In-memory client (non-persistent)
   - Query returns: distances, documents, metadatas
   - Supports multi-query batching

3. **03_chroma_img.ipynb** - ChromaDB Multimodal (Image) Embeddings
   - **Persistent client** using `PersistentClient(path=PERSIST_DIR)`
   - Uses `OpenCLIPEmbeddingFunction()` for image embeddings
   - Uses `ImageLoader()` to load images from URIs
   - Collection stores images via file paths (`uris` parameter in `add()`)
   - Cosine similarity via `metadata={"hnsw:space": "cosine"}`

### Key Data Structures

**SQLite schema:**
```sql
-- Document table
docs (id, content)

-- Vector table (virtual table)
vec_docs (rowid, embedding FLOAT[8])
```

**ChromaDB query result structure:**
```python
{
  'ids': [[...]],
  'embeddings': None | [[...]],
  'documents': [[...]] | [[None, ...]],  # None for images without text
  'uris': None | [[...]],
  'data': None | [[...]],  # Image arrays (only if include=['data'])
  'metadatas': [[...]],
  'distances': [[...]]
}
```

## Common Tasks

### Environment Setup
```bash
# Activate virtual environment
source .venv/bin/activate  # Linux/Mac
.venv\Scripts\activate     # Windows

# Install dependencies
pip install chromadb sqlite-vec numpy matplotlib jupyter
```

### Running Notebooks
```bash
jupyter notebook
# Open 01_sqlite.ipynb, 02_chroma_text.ipynb, or 03_chroma_img.ipynb
```

### Database Files
- `demo.db` - SQLite database (created by 01_sqlite.ipynb)
- `chroma_multimodal_local_dataloader/` - ChromaDB persistent storage (03_chroma_img.ipynb)

## Important Patterns & Gotchas

### ChromaDB Image Queries - The `include` Parameter
**CRITICAL:** When querying images with ChromaDB, the `include` parameter determines what data is returned:

```python
# ❌ WRONG - data field will be None
res = collection.query(
    query_texts=["dog"],
    include=["distances", "documents", "metadatas"]  # Missing 'data'
)
# res['data'] is None → TypeError when accessing res['data'][0]

# ✅ CORRECT - includes actual image arrays
res = collection.query(
    query_texts=["dog"],
    include=["distances", "documents", "metadatas", "data"]  # Add 'data'
)
# res['data'][0] contains list of image arrays for matplotlib
```

**When images are stored via `ImageLoader()`:**
- `documents` field returns `None` (no text stored)
- `data` field contains actual image arrays (only if explicitly included)
- Must add `"data"` to `include` parameter to retrieve images for visualization

### SQLite-vec Embedding Storage
- Vectors stored as JSON strings: `json.dumps([0.1, 0.2, ...])`
- Retrieved as binary blobs, convert with: `np.frombuffer(blob, dtype=np.float32)`
- Query requires JSON-encoded vector: `WHERE embedding MATCH ?` with `json.dumps(query_vec)`

### ChromaDB Collection Patterns
- **In-memory:** `chromadb.Client()` - data lost on restart
- **Persistent:** `chromadb.PersistentClient(path="./dir")` - survives restarts
- Use `get_or_create_collection()` for idempotent operations

### Embedding Dimensions
- SQLite example: 8-dimensional vectors (`FLOAT[8]`)
- ChromaDB: Automatic (384-dim for default text, 512-dim for OpenCLIP)
- Must match dimensions between storage and query

## Project Structure
```
.
├── 01_sqlite.ipynb              # SQLite + sqlite-vec demo
├── 02_chroma_text.ipynb         # ChromaDB text embeddings
├── 03_chroma_img.ipynb          # ChromaDB image embeddings
├── demo.db                      # SQLite database file
├── images/                      # Sample images (dog*.jpg, cat*.jpg)
├── chroma_multimodal_local_dataloader/  # ChromaDB persistence
└── .venv/                       # Python virtual environment
```
