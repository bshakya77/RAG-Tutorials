# Retrieval-Augmented Generation (RAG)

A Python RAG example that loads documents from `data/`, splits them into chunks,
embeds the chunks, stores the vectors locally with FAISS, retrieves relevant
text for a query, and asks a Groq-hosted language model to summarize the
retrieved context.

## Main components

| Component | Library or model | Purpose |
|---|---|---|
| Document loading | LangChain Community loaders, `pypdf`, `pymupdf` | Loads supported documents into LangChain `Document` objects. The loader scans recursively for PDF, TXT, CSV, XLSX, DOCX, and JSON files. |
| Text splitting | `langchain-text-splitters` | Splits documents into chunks of up to 1,000 characters with 200 characters of overlap, preserving document metadata. |
| Embeddings | `sentence-transformers` with `all-MiniLM-L6-v2` | Converts document chunks and queries into 384-dimensional vectors for semantic search. |
| Vector database | FAISS (`faiss-cpu`) | Performs local nearest-neighbor search using an `IndexFlatL2` index. The index and chunk metadata are stored under `faiss_store/`. |
| Language model | Groq through `langchain-groq`; default model `openai/gpt-oss-120b` | Generates a response using the retrieved text as context. |
| Configuration | `python-dotenv` | Loads `GROQ_API_KEY` from a local `.env` file. |

`chromadb` is also listed in `requirements.txt`, but the current `src/`
implementation uses FAISS.

## Setup

Use Python 3.10 or newer. From the project root in PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Create a `.env` file in the project root and add your Groq API key:

```text
GROQ_API_KEY=your_groq_api_key
```

Keep `.env` private; it is excluded by `.gitignore`. You can create an API key
in the [Groq Console](https://console.groq.com/).

## Add documents and run

Put source files anywhere under `data/` using one of the supported file types
(PDF, TXT, CSV, XLSX, DOCX, or JSON). Run the example from the project root:

```powershell
python src\search.py
```

On the first run, the search pipeline loads the documents, chunks and embeds
them, then saves the FAISS index and metadata in `faiss_store/`. Later runs load
that saved index. If you change the source documents and want to rebuild the
index, remove the generated `faiss_store/` directory before running again.

## Source layout

- `src/data_loader.py` — reads supported files from the data directory.
- `src/embedding.py` — chunks documents and creates text embeddings.
- `src/vectorstore.py` — builds, persists, loads, and searches the FAISS index.
- `src/search.py` — combines retrieval with Groq response generation.

The default embedding and chat model IDs are configured in `src/embedding.py`
and `src/search.py`, respectively. Ensure the Groq model ID is currently
available to your account.
