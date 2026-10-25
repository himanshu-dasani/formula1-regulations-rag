# Document Similarity Search Demo

A small Jupyter notebook demonstrating semantic search with text embeddings. It embeds a collection of Formula 1 sentences and a user query with OpenAI's `text-embedding-3-large` model, compares them using cosine similarity, and returns the sentence with the highest score.

This is a learning demo: the highest similarity is the closest match by meaning, and is not guaranteed to be a factually correct answer.

## Requirements

- Python 3.13 or newer
- An OpenAI API key with access to the embeddings API

## Setup

Install [uv](https://docs.astral.sh/uv/getting-started/installation/) if it is not already installed. Open a terminal in this project directory (the directory containing this README) and sync the project dependencies:

```bash
uv sync
```

`uv sync` creates the project virtual environment and installs the dependencies declared in `pyproject.toml` (using `uv.lock` for reproducible versions).

## Configure the API key

Create a `.env` file next to the notebook with your key:

```text
OPENAI_API_KEY=your_api_key_here
```

Keep `.env` private. It is ignored by Git; do not commit your API key.

## Run the notebook

Start Jupyter through uv:

```bash
uv run jupyter notebook
```

Open `document_similarity_app.ipynb` and run the cells from top to bottom. Edit the `query` cell or the `documents` list to try other searches. The final cell prints the closest matching sentence and its cosine similarity score.

## How it works

1. The notebook creates an embedding for each document sentence and for the query.
2. It calculates cosine similarity between the query vector and each document vector.
3. It selects the index of the largest score and returns the sentence at that index.

The example uses a short in-memory list of Formula 1 sentences; it does not read documents from files or use a vector database.
