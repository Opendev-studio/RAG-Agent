# Personal PDF Chatbot

A small, local-first PDF assistant built with [Chainlit](https://chainlit.io/),
[LangChain](https://python.langchain.com/), and NVIDIA NIM models.

Upload one or more PDFs and ask questions about them. The agent searches the
uploaded documents with a `search_documents` tool, answers only from the
retrieved passages, and cites the filename and page number.

## Features

- PDF upload with per-chat document isolation.
- Retrieval-augmented answers with page-level citations.
- Streaming responses through Chainlit.
- Conversation checkpointing and automatic summarization of long chats.
- Local Qdrant storage; no external vector database required.

## Setup

This project uses [uv](https://docs.astral.sh/uv/).

```bash
uv sync
cp .env.example .env
# Edit .env and set NVIDIA_API_KEY
uv run chainlit run app.py -w
```

Or run the packaged entrypoint:

```bash
uv run personal-pdf-chatbot
```

Get an NVIDIA API key at <https://build.nvidia.com/>. The key starts with
`nvapi-`.

## Configuration

Settings live in `src/personal_pdf_chatbot/config/settings.py` and can be
overridden with environment variables or `.env`. See `.env.example` for the
available options.

| Variable | Default |
| --- | --- |
| `NVIDIA_API_KEY` | required |
| `NVIDIA_CHAT_MODEL` | `stepfun-ai/step-5-preview` |
| `NVIDIA_EMBEDDING_MODEL` | `nvidia/nemotron-3.5-embed-1b` (2048 dimensions) |
| `MAX_COMPLETION_TOKENS` | `4096` |
| `CHUNK_SIZE` / `CHUNK_OVERLAP` | `1000` / `200` |
| `RETRIEVAL_K` | `4` |
| `MAX_UPLOAD_MB` | `50` |
| `QDRANT_PATH` | `.qdrant` |
| `SUMMARIZATION_TRIGGER_TOKENS` | `6000` |
| `SUMMARIZATION_KEEP_MESSAGES` | `20` |

## Architecture

```text
app.py                              Chainlit UI and event streaming
src/personal_pdf_chatbot/
  config/settings.py                Pydantic settings
  config/prompt.py                  System and welcome prompts
  loader/pdf.py                     PDF extraction and chunking (PyMuPDFLoader)
  embeddings/nim.py                 NVIDIA embedding model
  vectors/store.py                  Per-session Qdrant collections
  tools/search_documents.py         Retrieval tool and input schema
  agents/pdf_agent.py               Agent and model factories
  middleware/summarization.py       Conversation summarization
  checkpointers/memory.py           In-memory checkpointer
  streaming.py                      TokenTrimmer for astream_events v3 deltas
```

All I/O-facing helpers (loading, indexing, agent construction, checkpoint
deletion) expose async functions; blocking work runs in worker threads so the
Chainlit event loop never stalls.

`src/personal_pdf_chatbot/main.py` documents which module implemented each TODO
from the original scaffold.
