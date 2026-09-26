# VectorDB — C++ Vector Database & RAG Engine

A C++ vector database built from scratch with a web UI, implementing **HNSW, KD-Tree, and Brute Force** nearest-neighbor search alongside multiple distance metrics. The project also supports real **768-dimensional embeddings** generated locally with Ollama and a document-focused **RAG pipeline** that retrieves relevant chunks with HNSW and passes them to a local LLM for answer generation.

## Key Features

- **Three vector search algorithms:** HNSW, KD-Tree, and Brute Force.
- **Three distance metrics:** Cosine similarity, Euclidean distance, and Manhattan distance.
- **Semantic demo dataset:** 20 pre-loaded 16D vectors across CS, Math, Food, and Sports.
- **2D PCA visualization:** Projects the demo vectors into a 2D scatter plot.
- **Real document embeddings:** Uses Ollama `nomic-embed-text` to generate 768D embeddings.
- **Document chunking:** Splits long documents into overlapping 250-word chunks.
- **RAG pipeline:** Embeds questions, retrieves the top 3 relevant chunks with HNSW, and sends the retrieved context to `llama3.2`.
- **REST API:** Supports search, insert, delete, benchmarking, HNSW inspection, document operations, RAG queries, and status checks.
- **Web interface:** Provides search, algorithm comparison, document insertion, RAG interaction, PCA visualization, and benchmark views.

## Architecture

```text
                         VectorDB
                            │
             ┌──────────────┴──────────────┐
             │                             │
       Demo Vectors                  Documents
             │                             │
             ▼                             ▼
     ┌─────────────────┐          Ollama Embeddings
     │ Search Engine   │             nomic-embed-text
     │                 │                    │
     │ HNSW            │                    ▼
     │ KD-Tree         │              768D vectors
     │ Brute Force     │                    │
     └────────┬────────┘                    ▼
              │                       HNSW Index
              │                             │
              │                             ▼
              │                       Semantic Retrieval
              │                             │
              └─────────────────────────────▼
                                      Ollama llama3.2
                                             │
                                             ▼
                                           Answer
```

### Core Components

```text
BruteForce   → Exact baseline search
KDTree       → Exact axis-aligned space partitioning
HNSW         → Approximate multilayer graph search

VectorDB     → Unified interface over all 3 search algorithms
DocumentDB   → HNSW-only index for real Ollama embeddings
OllamaClient → HTTP client for embedding and generation APIs
```

## Processing Pipeline

### Vector Search

```text
Query Vector
     │
     ▼
Select Algorithm
     │
     ├── HNSW
     ├── KD-Tree
     └── Brute Force
     │
     ▼
Select Distance Metric
     │
     ├── Cosine
     ├── Euclidean
     └── Manhattan
     │
     ▼
Nearest Neighbors
```

### Document RAG

```text
Document
   │
   ▼
250-word overlapping chunks
   │
   ▼
Ollama: nomic-embed-text
   │
   ▼
768D embeddings
   │
   ▼
HNSW document index
   │
   ▼
User question
   │
   ▼
Question embedding
   │
   ▼
Top 3 relevant chunks
   │
   ▼
Ollama: llama3.2
   │
   ▼
Generated answer
```

## Technologies

- **C++17**
- **g++ / GCC**
- **cpp-httplib** — single-header HTTP server/client library
- **Ollama**
  - `nomic-embed-text` for 768D embeddings
  - `llama3.2` for local answer generation
- REST APIs
- HTML / JavaScript frontend
- PCA visualization
- HNSW
- KD-Tree
- Brute Force nearest-neighbor search

## Project Structure

```text
VectorDB/
├── main.cpp        # C++ backend: search algorithms, REST API, RAG
├── httplib.h       # Single-header HTTP library
├── index.html      # Frontend: PCA visualization, search, benchmark, RAG UI
└── README.md
```

## Build Instructions

### Prerequisites

On Windows, install:

1. **MSYS2** with GCC/g++
2. **Git**
3. **Ollama**

Verify the compiler:

```powershell
g++ --version
```

Install the required Ollama models:

```powershell
ollama pull nomic-embed-text
ollama pull llama3.2
```

Verify:

```powershell
ollama list
```

### Clone the Repository

```powershell
git clone https://github.com/Arpit-Phutela/VectorDB.git
cd VectorDB
```

### Compile

```powershell
g++ -std=c++17 -O2 main.cpp -o db -lws2_32
```

This produces `db.exe`.

### Run

Start Ollama if it is not already running:

```powershell
ollama serve
```

In another terminal:

```powershell
./db
```

The server runs at:

```text
http://localhost:8080
```

Expected startup output:

```text
=== VectorDB Engine ===
http://localhost:8080
20 demo vectors | 16 dims | HNSW+KD-Tree+BruteForce
Ollama: ONLINE
  embed model: nomic-embed-text  gen model: llama3.2
```

## Usage

### Search

The Search tab allows you to:

- Search the 20 demo vectors.
- Select HNSW, KD-Tree, or Brute Force.
- Select Cosine, Euclidean, or Manhattan distance.
- Compare all three algorithms.
- View matching vectors on the PCA scatter plot.

### Documents

The Documents tab:

1. Accepts a title and document text.
2. Splits long text into overlapping 250-word chunks.
3. Generates a 768D embedding for each chunk using `nomic-embed-text`.
4. Stores the chunks in an HNSW index.

### Ask AI

The RAG interface:

1. Embeds the question with `nomic-embed-text`.
2. Retrieves the 3 most relevant document chunks using HNSW.
3. Sends those chunks as context to `llama3.2`.
4. Generates an answer based on the retrieved document context.

## REST API

The server exposes its REST API at:

```text
http://localhost:8080
```

### Vector Operations

| Method | Endpoint | Description |
|---|---|---|
| GET | `/search?v=f1,f2,...&k=5&metric=cosine&algo=hnsw` | K-NN search |
| POST | `/insert` | Insert a demo vector |
| DELETE | `/delete/:id` | Delete by ID |
| GET | `/items` | List demo vectors |
| GET | `/benchmark?v=...&k=5&metric=cosine` | Compare all three algorithms |
| GET | `/hnsw-info` | HNSW graph and layer information |
| GET | `/stats` | Database statistics |

### Document & RAG Operations

| Method | Endpoint | Description |
|---|---|---|
| POST | `/doc/insert` | Embed and store a document |
| GET | `/doc/list` | List stored documents |
| DELETE | `/doc/delete/:id` | Delete a document chunk |
| POST | `/doc/ask` | Retrieve context and generate an answer |
| GET | `/status` | Ollama status and model information |

### Search with curl

```powershell
curl "http://localhost:8080/search?v=0.9,0.8,0.7,0.6,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1&k=3&metric=cosine&algo=hnsw"
```

### Ask a RAG Question

```powershell
curl -X POST http://localhost:8080/doc/ask `
  -H "Content-Type: application/json" `
  -d '{"question":"What is dynamic programming?","k":3}'
```

## Example Data

The demo initializes **20 vectors with 16 dimensions** across four semantic categories:

```text
CS
Math
Food
Sports
```

The application also provides a PCA scatter plot to visualize these vectors in two dimensions.

For document search, Ollama generates **768-dimensional embeddings** using `nomic-embed-text`.

## Core Engineering Concepts

### HNSW

HNSW (Hierarchical Navigable Small World) stores vectors in a multilayer graph. Search begins at higher layers and progressively moves toward the relevant neighborhood before performing the detailed search at layer 0.

The implementation uses a beam-search construction process with `ef_construction=200` and bidirectional neighbor connections.

### KD-Tree

The KD-Tree recursively partitions vector space along different dimensions. During search, subtrees can be pruned when their distance bounds cannot improve the current result.

The README notes that KD-Tree effectiveness decreases as dimensionality increases and that it becomes less suitable for the 768D document embeddings.

### Brute Force

Brute Force provides the exact baseline by evaluating the query against the available vectors directly.

### Algorithm Comparison

The application exposes all three algorithms through a common vector-search interface and provides a benchmark endpoint to compare their execution behavior.

## Limitations

- The demo vector dataset contains only 20 pre-loaded 16D vectors.
- KD-Tree performance becomes less suitable for high-dimensional vectors such as the 768D document embeddings.
- RAG generation depends on a locally running Ollama instance and the selected local models.
- LLM response time depends on the local machine; the original project notes that `llama3.2` can take 10–30 seconds on a laptop CPU.
- The application is an educational implementation rather than a production distributed vector database.
- The documented setup targets Windows/MSYS2.

## Future Improvements

Potential extensions include:

- Extending the vector storage layer beyond the current implementation.
- Expanding benchmark and comparison capabilities.
- Supporting additional embedding and generation models through the existing Ollama integration.
- Extending the REST API and frontend around the existing search, document, and RAG functionality.

## Troubleshooting

| Issue | Resolution |
|---|---|
| `Ollama: OFFLINE` | Run `ollama serve` |
| `g++: command not found` | Add `C:\msys64\ucrt64\bin` to Windows PATH |
| Port `8080` already in use | Find and stop the process using `netstat -ano \| findstr 8080` |
| Embedding takes a long time initially | Ollama may be downloading the model on first use |
| LLM response is slow | Local generation performance depends on the machine; the original project suggests `llama3.2:1b` as a smaller alternative |

### Optional Smaller LLM

```powershell
ollama pull llama3.2:1b
```

Then change the generation model in `main.cpp`:

```cpp
std::string genModel = "llama3.2:1b";
```

Recompile and restart the server.

## License

MIT License.
