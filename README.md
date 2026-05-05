# my-local-gpt 🚀

A **local RAG-powered API** that answers questions about people using their own documents — built with **FastAPI**, **ChromaDB**, and **Ollama**. No cloud. No cost. Just your data.

## What is this?

**Retrieval-Augmented Generation (RAG)** is a technique that lets AI models answer questions based on your own documents instead of relying solely on their training data. This project implements a complete RAG pipeline running entirely on your machine.

### The Pipeline:

1. **📄 RETRIEVE** - ChromaDB searches your documents using semantic similarity to find relevant context
2. **🔧 AUGMENT** - The question and retrieved context are combined into a smart prompt
3. **🧠 GENERATE** - A local LLM (Ollama) uses the augmented prompt to generate grounded answers

## Features

✨ **Multi-user support** - Store profiles for multiple users and query them individually or together

🔐 **Privacy-first** - All data stays on your machine (zero cloud dependencies)

💰 **Zero cost** - No API subscriptions, no cloud fees

⚡ **Fast & responsive** - Local inference means sub-second responses

🎯 **Semantic search** - Find relevant context based on meaning, not just keywords

## Tech Stack

- **FastAPI** - Modern web framework for building APIs
- **ChromaDB** - Vector database for semantic search
- **Ollama** - Run LLMs locally
- **Pydantic** - Data validation

## Prerequisites

Before getting started, make sure you have:

- **Python 3.10+**
- **Ollama** installed and running (`ollama serve`)
  - Download from: https://ollama.ai
  - Make sure you pull the required models:
    ```bash
    ollama pull nomic-embed-text  # For embeddings
    ollama pull qwen2.5:0.5b      # For text generation
    ```

## Installation & Setup

### 1. Clone the repository
```bash
git clone https://github.com/Jaber192/my-local-gpt.git
cd my-local-gpt
```

### 2. Create a virtual environment
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Make sure Ollama is running
```bash
# In a separate terminal
ollama serve
```

### 5. Run the API
```bash
uvicorn main:app --reload
```

The API will be available at `http://localhost:8000`

## API Endpoints

### 📝 POST `/documents`

Add a user's profile/documents to the knowledge base.

**Request:**
```bash
curl -X POST "http://localhost:8000/documents" \
  -H "Content-Type: application/json" \
  -d '{
    "user_name": "Alice",
    "content": "I am a software engineer with 5 years of experience. I specialize in Python and machine learning. I love building scalable systems.\n\nI graduated from MIT with a degree in Computer Science."
  }'
```

**Response:**
```json
{
  "message": "Added 2 chunks for user 'Alice'.",
  "user_name": "Alice",
  "chunks_added": 2
}
```

### 🤖 GET `/ask`

Ask a question about a user's profile (or all profiles if no user specified).

**Query Parameters:**
- `question` (required): The question to ask
- `user` (optional): Filter results to a specific user

**Examples:**

**Query all profiles:**
```bash
curl "http://localhost:8000/ask?question=What%20programming%20languages%20does%20this%20person%20know?"
```

**Query a specific user:**
```bash
curl "http://localhost:8000/ask?question=What%20is%20their%20background?&user=Alice"
```

**Response:**
```json
{
  "question": "What is their background?",
  "answer": "Alice is a software engineer with 5 years of experience who specializes in Python and machine learning. She graduated from MIT with a degree in Computer Science.",
  "context_used": [
    "I am a software engineer with 5 years of experience. I specialize in Python and machine learning. I love building scalable systems.",
    "I graduated from MIT with a degree in Computer Science."
  ],
  "filtered_by_user": "Alice"
}
```

## How It Works

### Step 1: Document Storage

When you POST to `/documents`, the system:
1. Splits your content into chunks (by paragraph)
2. Converts each chunk into embeddings using `nomic-embed-text`
3. Stores them in ChromaDB with metadata (user name, chunk index)

### Step 2: Semantic Search

When you GET `/ask`, the system:
1. Converts your question into an embedding
2. Searches ChromaDB for the 2 most similar chunks
3. Retrieves them as context

### Step 3: AI Response

The system:
1. Builds a prompt: *"Use this context to answer: [question]"*
2. Sends it to the local LLM (qwen2.5:0.5b)
3. Returns the AI-generated answer

## Example Usage

```bash
# 1. Add a profile
curl -X POST "http://localhost:8000/documents" \
  -H "Content-Type: application/json" \
  -d '{
    "user_name": "Bob",
    "content": "I work as a DevOps engineer. I have experience with Docker, Kubernetes, and AWS. I enjoy automation and CI/CD pipelines."
  }'

# 2. Ask about Bob
curl "http://localhost:8000/ask?question=What%20tools%20do%20they%20use?&user=Bob"

# 3. Query all users
curl "http://localhost:8000/ask?question=Who%20has%20cloud%20experience?"
```

## Project Structure

```
my-local-gpt/
├── main.py              # FastAPI application & RAG logic
├── requirements.txt     # Python dependencies
├── .gitignore          # Git ignore rules
└── README.md           # This file
```

## Data Storage

- **ChromaDB data** is stored in `./chroma_db/` (auto-created)
- **No external databases** - everything is local
- **Persistent storage** - data survives application restarts

## Performance

- **Embedding generation**: ~100-200ms (nomic-embed-text)
- **Semantic search**: <10ms (ChromaDB in-memory)
- **LLM inference**: ~1-3s depending on response length
- **Total response time**: ~1.5-3.5s for a typical query

## Troubleshooting

### Ollama connection error
- Make sure Ollama is running: `ollama serve`
- Check the URL is correct: `http://localhost:11434`

### Model not found
- Pull the required models:
  ```bash
  ollama pull nomic-embed-text
  ollama pull qwen2.5:0.5b
  ```

### ChromaDB permission error
- Make sure the `chroma_db/` directory has write permissions
- Try deleting it and restarting the app

## Future Enhancements

- 🔐 Authentication & user accounts
- 📊 Analytics & query tracking
- 🎯 Advanced retrieval (BM25 + semantic hybrid search)
- 🗂️ Multiple document types (PDFs, web pages, etc.)
- ⚙️ Configurable models & parameters
- 🚀 Production deployment guide

## License

MIT License - feel free to use this for personal or commercial projects!

## Author

Built by [Jaber192](https://github.com/Jaber192)

---

**Made with ❤️ for the open-source community**
