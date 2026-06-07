# LMS ChatBot 🤖📚

A local AI chatbot I built for learning management systems. You upload your documents — lecture notes, PDFs, policies, whatever — and it answers questions about them. No cloud, no API keys, everything runs on your machine.

The assistant is named **PY**. It uses RAG (Retrieval-Augmented Generation) under the hood, which basically means it searches your documents first before asking the LLM to generate a response. If your docs don't have the answer, it falls back to DuckDuckGo and Wikipedia automatically.

---

## Why I built this

I wanted a chatbot that could actually read *my* documents — not just generic internet knowledge. Most solutions either require expensive API calls or send your data to some server. This one runs completely offline using [Ollama](https://ollama.ai/), so your notes and files stay on your machine.

---

## What it can do

- Upload PDFs, DOCX, TXT, or Markdown files and chat with them
- Paste raw text directly if you don't have a file
- Streams answers token by token (like ChatGPT) instead of waiting for the full response
- Automatically searches the web when your documents don't have the answer
- Summarize any document just by asking "summarize [doc name]"
- Suggests follow-up questions after each answer
- Delete individual documents or wipe everything and start fresh
- Dark mode / light mode, persisted across sessions

---

## How it works

The pipeline has two parts:

**When you upload a document:**
1. Text is extracted (handles PDF, DOCX, TXT, MD)
2. Split into overlapping chunks of ~400 tokens
3. Each chunk gets embedded using `all-MiniLM-L6-v2`
4. Embeddings are saved to a local JSON store

**When you ask a question:**
1. Your question gets embedded the same way
2. Cosine similarity finds the most relevant chunks
3. If the best match score is too low → falls back to DuckDuckGo + Wikipedia
4. A prompt is built with the retrieved context
5. Ollama streams the response back to your browser

---

## Tech stack

- **FastAPI** — backend + streaming via SSE
- **Ollama** — local LLM (`qwen2.5:1.5b` by default, swap for any model)
- **sentence-transformers** — `all-MiniLM-L6-v2` for embeddings
- **PyMuPDF** — PDF parsing
- **python-docx** — DOCX parsing
- **duckduckgo-search** — web fallback
- **Vanilla JS + Jinja2** — frontend, no frameworks

---

## Setup

You'll need Python 3.10+ and [Ollama](https://ollama.ai/download) installed.

```bash
# Pull the model first
ollama pull qwen2.5:1.5b

# Clone and install
git clone https://github.com/sundhip/LMS_BOT.git
cd LMS_BOT
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

pip install fastapi uvicorn python-multipart pymupdf python-docx \
            sentence-transformers duckduckgo-search numpy jinja2

# Run
uvicorn main:app --reload
```

Then open `http://localhost:8000` in your browser.

**Project structure:**
```
LMS_BOT/
├── main.py
├── templates/
│   ├── chat.html
│   └── train.html
└── static/
    ├── script.js
    └── style.css
```

---

## Usage

Go to `/train` → upload a file or paste your notes → hit Train.

Then go to `/chat` and just ask questions normally:
- *"What are the attendance rules?"* → pulls from your docs
- *"Summarize the lecture 3 notes"* → gives a structured summary
- *"What is gradient descent?"* → falls back to web if not in your docs

---

## Config

A few things you might want to change in `main.py`:

```python
OLLAMA_MODEL = "qwen2.5:1.5b"   # change to llama3.2, mistral, etc.
OLLAMA_URL   = "http://localhost:11434/api/generate"
```

Chunk size and overlap:
```python
chunk_text(pages, chunk_size=400, overlap=80)
```

---

## Known limitations

- The vector store is in-memory and backed by a JSON file — not ideal for very large document collections (1000+ pages). A proper vector DB like Chroma or Qdrant would handle that better.
- No conversation memory between sessions — each reload starts fresh.
- Web search quality depends on DuckDuckGo's results.

---

## What's next

Things I'm planning to add:
- Hybrid search (BM25 + vector) for better retrieval accuracy
- Persistent chat history
- Docker support so setup is one command
- A model picker in the UI so you can switch Ollama models without touching the code
- Support for URLs and CSV files

---

## License

MIT — do whatever you want with it.
