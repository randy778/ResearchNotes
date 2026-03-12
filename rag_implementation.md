# RAG Learning Summary

---

## Part 1 — RAG Implementation Progress

**Stack: LangChain + Pinecone + Claude/OpenAI API**

---

### Project Structure

```
rag-step1/
├── loader.py
├── .env
└── data/
    ├── sample.txt
    └── samplePDF.pdf
```

---

### Packages Installed So Far

```bash
pip install langchain-community pypdf
pip install langchain-text-splitters
pip install langchain-openai langchain-pinecone pinecone python-dotenv
```

---

### Current Code — `loader.py`

```python
from langchain_community.document_loaders import TextLoader, PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings
from langchain_pinecone import PineconeVectorStore
from dotenv import load_dotenv
import os

load_dotenv()

# STEP 1 — Load Documents
def load_documents(source_path):
    all_docs = []

    if os.path.isdir(source_path):
        for filename in os.listdir(source_path):
            full_path = os.path.join(source_path, filename)
            if filename.endswith(".txt"):
                all_docs.extend(TextLoader(full_path).load())
            elif filename.endswith(".pdf"):
                all_docs.extend(PyPDFLoader(full_path).load())

    elif source_path.endswith(".pdf"):
        all_docs = PyPDFLoader(source_path).load()

    else:
        all_docs = TextLoader(source_path).load()

    return all_docs


# STEP 2 — Chunk Documents
def chunk_documents(docs):
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=800,
        chunk_overlap=100,
    )
    chunks = splitter.split_documents(docs)
    return chunks


# STEP 3 — Embed & Store in Pinecone
def embed_and_store(chunks):
    embeddings = OpenAIEmbeddings(
        model="text-embedding-3-small",
        api_key=os.getenv("OPENAI_API_KEY"),
    )

    vectorstore = PineconeVectorStore.from_documents(
        documents=chunks,
        embedding=embeddings,
        index_name="rag-index",
    )

    return vectorstore


# TEST
docs = load_documents("data")
print(f"STEP 1 — Total documents loaded: {len(docs)}")

chunks = chunk_documents(docs)
print(f"STEP 2 — Total chunks created: {len(chunks)}")

vectorstore = embed_and_store(chunks)
print(f"STEP 3 — Successfully embedded and stored {len(chunks)} chunks into Pinecone!")
```

---

### Steps Completed

- ✅ Step 1 — Load documents (txt + pdf, single file + folder)
- ✅ Step 2 — Chunk documents (800 chars, 100 overlap)
- ✅ Step 3 — Embed and store in Pinecone (code written, pending API key setup)

### Steps Remaining

- ⏳ Step 4 — Retrieve relevant chunks from Pinecone
- ⏳ Step 5 — Augment prompt with retrieved chunks
- ⏳ Step 6 — Generate answer via Claude
- ⏳ Step 7 — Optional: FastAPI server + Streamlit UI

---

### Environment Variables Needed (`.env`)

```
OPENAI_API_KEY=your_openai_api_key_here
PINECONE_API_KEY=your_pinecone_api_key_here
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

### Pinecone Index Settings

```
Name       : rag-index
Dimensions : 1536
Metric     : cosine
Plan       : Free (Serverless)
```

---

### Key Decisions Made

- Embedding model: `text-embedding-3-small` (1536 dimensions)
- Generation model: `claude-sonnet-4-20250514`
- Chunk size: `800` characters
- Chunk overlap: `100` characters
- Top K retrieval: `5` chunks

---

## Part 2 — Core Concepts Learned

---

### Tokens

- Every input to any AI model gets converted to tokens first — the model never sees raw text
- Tokens are not words — they are fragments based on the model's vocabulary
- Average English word ≈ 1-2 tokens, roughly 4-5 characters per token
- Token count determines cost — more tokens = higher API bill
- Output tokens cost more than input tokens because generation requires multiple forward passes
- Different providers have different tokenizers = different token counts = different costs for same text

---

### Tokenizer & Detokenizer

- Tokenizer converts text → token IDs before the model processes anything
- Detokenizer converts token IDs → text after the model generates a response
- Both are fixed lookup tables built during training — deterministic, no agenda, no optimization
- Tokenizer and model are inseparable — you cannot swap one without breaking the other
- Good analogy: tokenizer is like DNS — just translates human readable input into machine readable IDs
- Open source models like Llama come fully bundled with their own tokenizer — you never build your own unless training from scratch

---

### Embeddings & Vectors

- Embeddings are fixed size numerical representations of text meaning
- `text-embedding-3-small` always outputs exactly 1536 numbers regardless of input length
- 1536 is OpenAI's architecture decision — hardcoded into the model design
- Similar meaning = similar numbers = close together in vector space
- The 1536 numbers stay in Pinecone forever — they never get sent to Claude
- Only the original chunk text gets sent to Claude for generation

---

### Chunk Size

- Chunk size is YOUR decision — not the model's
- It is measured in characters, not tokens
- 800 characters ≈ 150-200 tokens roughly
- Chunk size affects quality and cost in two places — embedding and generation
- Too large = diluted meaning in the vector, unfocused answers
- Too small = incomplete meaning, poor retrieval
- Sweet spot depends on content type:

| Content Type            | Recommended Chunk Size |
| ----------------------- | ---------------------- |
| Short FAQ answers       | 256 - 512              |
| HR / policy documents   | 512 - 800              |
| Legal contracts         | 800 - 1200             |
| Research papers         | 1000 - 1500            |
| Books / long narratives | 1500 - 2000            |

---

### RAG vs Retraining

- RAG never retrains the model — the model stays completely frozen
- RAG just hands the model better information at query time
- Think of it like giving a consultant documents before a meeting — their brain doesn't change, just their available information
- Three ways to customize AI:

| Method      | Complexity    | Cost           | Use Case             |
| ----------- | ------------- | -------------- | -------------------- |
| Prompting   | Simplest      | Free           | General instructions |
| RAG         | Middle ground | Low            | Your own documents   |
| Fine Tuning | Most complex  | Very expensive | Style/tone changes   |

- RAG is always necessary for private data regardless of how many times the base model gets retrained
- You can detect retraining by checking knowledge cutoff date, official release notes, or benchmarking yourself

---

### Electricity & Compute

- Every token processed = GPU computation = electricity consumed
- One A100 GPU consumes ~400 watts at full load
- Generation costs more electricity than embedding because it runs multiple forward passes
- AI industry consumes ~100 TWh per year estimated — comparable to a small country
- Smaller chunks = fewer tokens = less electricity = lower cost + smaller carbon footprint

---

### Model Versions

- Version bump does not always mean retraining — could be bug fix, safety update, or infrastructure change
- Knowledge cutoff date changing is the most reliable sign of retraining
- Always check official release notes for what actually changed
- Private data never enters any model through retraining — RAG remains necessary forever for your own documents

---

### Open Source vs Closed Models

|             | Closed (OpenAI/Anthropic) | Open Source (Llama)   |
| ----------- | ------------------------- | --------------------- |
| Tokenizer   | Built in, hidden          | Built in, open source |
| Cost        | Pay per token             | Free                  |
| Hosting     | Their servers             | Your machine          |
| Privacy     | Data leaves your system   | Fully local           |
| Maintenance | Auto updated              | You manage            |

---

### How to Check Token Count Per Provider

**OpenAI:**

```python
import tiktoken

encoder = tiktoken.encoding_for_model("gpt-4")
tokens = encoder.encode("your text here")
print(len(tokens))
```

**Anthropic (Claude):**

```python
import anthropic

client = anthropic.Anthropic()
response = client.messages.count_tokens(
    model="claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "your text here"}]
)
print(response.input_tokens)
```

---

_Continue from Step 4 — Retrieval in your next chat session._
