# Formula 1 Regulations RAG

An end-to-end retrieval-augmented generation (RAG) prototype in a Jupyter notebook. It answers questions about Formula 1 regulations by retrieving relevant passages from the supplied PDFs and asking an LLM to respond using those passages, with PDF and page citations.

Choose an embedding backend (OpenAI, Hugging Face, or Ollama) and a local vector store (Chroma or FAISS) in the first notebook cell. OpenAI's chat model generates the final answer in every configuration.

## Requirements

- Python 3.13 or newer
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- An OpenAI API key with access to `gpt-4o-mini` for answer generation

## Setup

From this directory, install the default dependencies:

```bash
uv sync
```

Optional backend dependencies can be installed with extras:

```bash
uv sync --extra huggingface  # Hugging Face embeddings
uv sync --extra ollama       # Ollama embeddings
uv sync --extra faiss        # FAISS vector store
```

You can combine extras, for example:

```bash
uv sync --extra huggingface --extra faiss
```

Create a `.env` file next to the notebook:

```text
OPENAI_API_KEY=your_api_key_here
```

Keep the key private. `.env` is ignored by Git. It is still required for answer generation even when using local Hugging Face or Ollama embeddings.

### Ollama setup

Install and start Ollama, then download the embedding model used in the notebook:

```bash
ollama pull nomic-embed-text
```

The Ollama server must be running when the indexing cell executes.

## Source documents

The `documents/` folder contains the six FIA 2026 Formula 1 regulation sections (A–F). To add more regulations, place their PDFs in that folder. The notebook uses LangChain's `PyMuPDFLoader` to extract page text; scanned PDFs without a text layer need OCR before they can be searched.

## Run

Start Jupyter:

```bash
uv run jupyter notebook
```

Open `app.ipynb` and run its cells from top to bottom. In the first code cell, set:

- `EMBEDDING_BACKEND` to `"openai"`, `"huggingface"`, or `"ollama"`.
- `VECTOR_STORE_BACKEND` to `"chroma"` or `"faiss"`.

The notebook loads and splits the PDFs, embeds and indexes the chunks, retrieves the five most relevant chunks for the query, and generates a cited answer. Change the `query` cell to ask a different question. If you change either backend, rerun from the start to rebuild the index with matching embeddings.

In VS Code, use this project's `.venv` as the notebook kernel. If needed, run `uv sync` (with the relevant extras) and select `.venv/bin/python` (macOS/Linux) or `.venv\\Scripts\\python.exe` (Windows), then restart the kernel.

## How it works

1. LangChain's `DirectoryLoader` and `PyMuPDFLoader` load each PDF page with its filename and page number as metadata.
2. `RecursiveCharacterTextSplitter` creates overlapping text chunks and preserves the source metadata.
3. The selected embedding backend converts each chunk into a vector:
   - OpenAI: `text-embedding-3-small` (API; base dependencies).
   - Hugging Face: `sentence-transformers/all-MiniLM-L6-v2` (local model; `huggingface` extra).
   - Ollama: `nomic-embed-text` (local Ollama service; `ollama` extra).
4. The selected vector store saves the vectors, text, and metadata locally:
   - Chroma: `chroma_db/<embedding_backend>/`.
   - FAISS: `faiss_index/<embedding_backend>/`.
5. The vector store retrieves the five most relevant chunks. OpenAI's `gpt-4o-mini` answers from those passages and cites their source PDF and page.

## Persistence and costs

The indexing cell rebuilds the selected index from the PDFs each time it runs, so changes and removals are reflected. Re-indexing with OpenAI embeddings uses the embeddings API and may incur charges. Hugging Face downloads its model on first use; Ollama uses the model already pulled locally. The local `chroma_db/` and `faiss_index/` folders are ignored by Git and can be recreated by running the notebook.

## Limitations

- This is a local notebook prototype; it has no web interface, user accounts, or hosted database.
- The final answer still uses OpenAI's chat API, regardless of the embedding backend.
- Retrieval can miss relevant passages, and generated answers or citations can be inaccurate. Check important details against the cited regulation pages.
