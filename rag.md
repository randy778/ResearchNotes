# RAG (Retrieval-Augmented Generation)

## What is it?
RAG is an AI framework that combines large language models (LLMs) with external data sources through a retrieval component. It retrieves relevant documents at query time and feeds them as context to the LLM, enabling more accurate and grounded responses.

## Key Ideas
- **Retrieval + Generation**: Combines information retrieval systems with generative AI to produce contextually informed outputs
- **Vector Databases**: Uses semantic embeddings stored in vector databases to efficiently find relevant documents matching the query intent
- **Reduced Hallucinations**: Grounds responses in actual external data, preventing the model from generating false or made-up information

## Popular Tools
- **LangChain**: Open-source library for chaining LLMs, embeddings, and knowledge bases together
- **Vector Databases**: Tools like Pinecone, Weaviate, and Milvus store document embeddings for fast semantic retrieval
- **Google Gemini with RAG**: Cloud-based RAG implementation leveraging long context windows and vector search

## Why Use It?
RAG solves the problem of LLMs having static, outdated training data by enabling real-time retrieval of current, domain-specific information. It's become the standard architecture for enterprise AI applications, reducing hallucinations and allowing organizations to build reliable systems without expensive fine-tuning.

## References
- [What is Retrieval-Augmented Generation (RAG)? - Google Cloud](https://cloud.google.com/use-cases/retrieval-augmented-generation)
- [What is Retrieval Augmented Generation (RAG)? - Databricks](https://www.databricks.com/blog/what-is-retrieval-augmented-generation)
- [What Is Retrieval-Augmented Generation aka RAG - NVIDIA](https://blogs.nvidia.com/blog/what-is-retrieval-augmented-generation/)
- [Top Use Cases of Retrieval-Augmented Generation - Glean](https://www.glean.com/blog/retrieval-augmented-generation-use-cases)
