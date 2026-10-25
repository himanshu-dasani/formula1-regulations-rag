# Formula 1 Regulations RAG Prototype

A step-by-step Jupyter notebook demonstrating retrieval-augmented generation (RAG) over Formula 1 regulation PDFs. It loads the PDFs, splits their text into chunks, retrieves passages related to a question, and asks a language model to answer from those passages with source PDF and page citations.

This is a working RAG prototype for learning and experimentation, rather than a deployed application. It uses an in-memory index and rebuilds the document embeddings when the embedding cell is run.

## Requirements

- Python 3.13 or newer
- An OpenAI API key with access to the embeddings and chat completion models used by the notebook

## Setup

Install [uv](https://docs.astral.sh/uv/getting-started/installation/) if it is not already installed. Open a terminal in this project directory (the directory containing this README) and sync the project dependencies:

```bash
uv sync
```

`uv sync` creates the project virtual environment and installs the dependencies declared in `pyproject.toml` (using `uv.lock` for reproducible versions).

## Configure the API key

Create a `.env` file in the project directory, next to the notebook, with your key:

```text
OPENAI_API_KEY=your_api_key_here
```

Keep `.env` private. It is ignored by Git; do not commit your API key.

## Add PDFs

Place PDF files in the `documents/` folder. The repository includes six FIA 2026 Formula 1 regulation sections (A–F). LangChain's `PyMuPDFLoader` extracts text page by page; scanned PDFs without a text layer need OCR before they can be searched.

## Run the notebook

Start Jupyter through uv:

```bash
uv run jupyter notebook
```

Open `app.ipynb` and run the cells from top to bottom. The embedding cell sends document text to the OpenAI embeddings API and may take a little while for these regulations. Change the `query` cell to ask a different question. The final cell prints an answer with citations and the retrieved passages with similarity scores.

In VS Code, select the project's `.venv` Python interpreter as the notebook kernel. If it is not listed, run `uv sync` first, then select `.venv/bin/python` (macOS/Linux) or `.venv\\Scripts\\python.exe` (Windows).

## How it works

1. LangChain's `DirectoryLoader` and `PyMuPDFLoader` load each PDF page with its filename and page number as metadata.
2. LangChain's `RecursiveCharacterTextSplitter` creates overlapping chunks while preserving that metadata.
3. The notebook embeds each chunk using OpenAI's `text-embedding-3-small` model.
4. It embeds a question and retrieves the five most similar chunks by cosine similarity.
5. OpenAI's `gpt-4o-mini` model receives those passages and generates an answer with citations to their PDF filenames and pages.

## Scope and limitations

- Extracted documents, chunks, and embeddings live only in notebook memory. Restarting the kernel clears them.
- The notebook creates embeddings again whenever the embedding cell is rerun. This uses the embeddings API and may incur charges.
- Retrieval compares the question against every chunk in memory. There is no persistent vector database or index.
- Answers are grounded in the retrieved passages, but retrieval can miss relevant text and model-generated answers or citations can be inaccurate. Verify important details against the cited PDF pages.
- The notebook is a local learning demo; it does not provide a web interface, user management, or a deployed service.
