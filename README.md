# LangChain RAG Tutorial: Complete Step-by-Step Guide

## Table of Contents
1. [What is RAG?](#what-is-rag)
2. [Preparing the Data](#preparing-the-data)
3. [Creating Chroma Database](#creating-chroma-database)
4. [What are Vector Embeddings?](#what-are-vector-embeddings)
5. [Querying for Relevant Data](#querying-for-relevant-data)
6. [Crafting a Great Response](#crafting-a-great-response)
7. [Wrapping Up](#wrapping-up)

---

## What is RAG?

RAG (Retrieval-Augmented Generation) is a technique that combines information retrieval with text generation. Instead of relying solely on the knowledge stored in a language model's parameters, RAG retrieves relevant information from external sources and uses it to generate more accurate and contextual responses.

### How RAG Works:
1. **Retrieve**: Find relevant documents from a knowledge base
2. **Augment**: Add retrieved context to the user's query
3. **Generate**: Use an LLM to create a response based on the augmented prompt

### Benefits:
- More accurate and up-to-date information
- Reduces hallucinations
- Allows access to private/domain-specific knowledge
- Cost-effective compared to fine-tuning large models

---

## Preparing the Data

The first step in building a RAG system is preparing your data. This involves loading documents and splitting them into manageable chunks.

### Installation
```python
pip install langchain chromadb openai tiktoken
```

### Loading Documents
```python
from langchain.document_loaders import TextLoader, PyPDFLoader, DirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Load a single text file
loader = TextLoader("document.txt")
documents = loader.load()

# Load multiple files from a directory
loader = DirectoryLoader("./data", glob="*.txt")
documents = loader.load()

# Load PDF files
pdf_loader = PyPDFLoader("document.pdf")
pdf_documents = pdf_loader.load()
```

### Text Splitting
Breaking documents into smaller chunks is crucial for effective retrieval. Smaller chunks ensure more precise matches and fit within token limits.

```python
# Create a text splitter
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,        # Maximum characters per chunk
    chunk_overlap=200,      # Overlap between chunks to maintain context
    length_function=len,    # Function to measure chunk length
    separators=["\n\n", "\n", " ", ""]  # Split hierarchy
)

# Split documents into chunks
chunks = text_splitter.split_documents(documents)

print(f"Split {len(documents)} documents into {len(chunks)} chunks")
```

### Why Chunking Matters:
- **Precision**: Smaller chunks lead to more relevant retrievals
- **Token Limits**: LLMs have input token limitations
- **Context Quality**: Focused chunks provide better context

---

## Creating Chroma Database

Chroma is a vector database that stores document embeddings and enables fast similarity search. It's perfect for RAG applications.

### Setting Up Chroma
```python
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
import os

# Set your OpenAI API key
os.environ["OPENAI_API_KEY"] = "your-api-key-here"

# Initialize embeddings
embeddings = OpenAIEmbeddings()

# Create Chroma database from documents
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"  # Directory to save the database
)

# Persist the database
vectorstore.persist()
```

### Loading Existing Database
```python
# Load existing Chroma database
vectorstore = Chroma(
    persist_directory="./chroma_db",
    embedding_function=embeddings
)
```

### Adding New Documents
```python
# Add new documents to existing database
new_chunks = text_splitter.split_documents(new_documents)
vectorstore.add_documents(new_chunks)
vectorstore.persist()
```

---

## What are Vector Embeddings?

Vector embeddings are numerical representations of text that capture semantic meaning. Words or phrases with similar meanings have similar vector representations.

### How Embeddings Work:
- Text is converted into high-dimensional vectors (typically 1536 dimensions for OpenAI)
- Similar concepts cluster together in vector space
- Distance between vectors indicates semantic similarity

### Example: Understanding Embeddings
```python
from langchain.embeddings import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()

# Create embeddings for sample texts
texts = [
    "The cat sat on the mat",
    "A feline rested on the rug",
    "Python is a programming language",
    "Dogs are loyal animals"
]

# Generate embeddings
vectors = embeddings.embed_documents(texts)

print(f"Each embedding has {len(vectors[0])} dimensions")
print(f"First few dimensions of 'cat' sentence: {vectors[0][:5]}")
```

### Similarity Search Example
```python
# Search for similar documents
query = "What is machine learning?"
similar_docs = vectorstore.similarity_search(query, k=3)

for i, doc in enumerate(similar_docs):
    print(f"Document {i+1}: {doc.page_content[:100]}...")
```

---

## Querying for Relevant Data

This step involves retrieving the most relevant documents based on user queries using similarity search.

### Basic Retrieval
```python
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings

# Initialize retriever
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4}  # Return top 4 most similar documents
)

# Retrieve relevant documents
query = "How does machine learning work?"
relevant_docs = retriever.get_relevant_documents(query)

print(f"Found {len(relevant_docs)} relevant documents")
for doc in relevant_docs:
    print(f"- {doc.page_content[:100]}...")
```

### Advanced Retrieval with MMR
Maximum Marginal Relevance (MMR) balances relevance with diversity to avoid redundant results.

```python
# Use MMR for diverse results
retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 4,
        "fetch_k": 20,    # Fetch more candidates
        "lambda_mult": 0.7  # Balance relevance vs diversity
    }
)

relevant_docs = retriever.get_relevant_documents(query)
```

### Similarity Search with Scores
```python
# Get similarity scores along with documents
docs_with_scores = vectorstore.similarity_search_with_score(query, k=4)

for doc, score in docs_with_scores:
    print(f"Score: {score:.3f}")
    print(f"Content: {doc.page_content[:100]}...")
    print("---")
```

---

## Crafting a Great Response

The final step combines retrieved context with the user's question to generate a comprehensive response using an LLM.

### Setting Up the LLM
```python
from langchain.llms import OpenAI
from langchain.chat_models import ChatOpenAI
from langchain.chains import RetrievalQA
from langchain.prompts import PromptTemplate

# Initialize the language model
llm = ChatOpenAI(
    model_name="gpt-3.5-turbo",
    temperature=0.7
)
```

### Creating a Custom Prompt
```python
# Define a custom prompt template
prompt_template = """
Use the following pieces of context to answer the question at the end. 
If you don't know the answer, just say that you don't know, don't try to make up an answer.

Context:
{context}

Question: {question}

Answer: """

PROMPT = PromptTemplate(
    template=prompt_template,
    input_variables=["context", "question"]
)
```

### Building the RAG Chain
```python
# Create the RetrievalQA chain
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    chain_type_kwargs={"prompt": PROMPT},
    return_source_documents=True
)

# Ask a question
query = "What are the benefits of machine learning?"
result = qa_chain({"query": query})

print("Answer:", result["result"])
print("\nSource documents:")
for i, doc in enumerate(result["source_documents"]):
    print(f"{i+1}. {doc.page_content[:100]}...")
```

### Alternative: Using ConversationalRetrievalChain
```python
from langchain.chains import ConversationalRetrievalChain
from langchain.memory import ConversationBufferMemory

# Add memory for conversation history
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

# Create conversational chain
conversation_chain = ConversationalRetrievalChain.from_llm(
    llm=llm,
    retriever=retriever,
    memory=memory,
    return_source_documents=True
)

# Have a conversation
result1 = conversation_chain({"question": "What is machine learning?"})
result2 = conversation_chain({"question": "What are its main applications?"})
```

---

## Wrapping Up

### Complete RAG Implementation
Here's a complete, minimal RAG system that ties everything together:

```python
import os
from langchain.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.chat_models import ChatOpenAI
from langchain.chains import RetrievalQA

class SimpleRAG:
    def __init__(self, api_key):
        os.environ["OPENAI_API_KEY"] = api_key
        self.embeddings = OpenAIEmbeddings()
        self.llm = ChatOpenAI(temperature=0.7)
        self.vectorstore = None
        self.qa_chain = None
    
    def load_documents(self, file_path):
        loader = TextLoader(file_path)
        documents = loader.load()
        
        text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200
        )
        chunks = text_splitter.split_documents(documents)
        
        self.vectorstore = Chroma.from_documents(
            documents=chunks,
            embedding=self.embeddings
        )
        
        self.qa_chain = RetrievalQA.from_chain_type(
            llm=self.llm,
            retriever=self.vectorstore.as_retriever(k=4),
            return_source_documents=True
        )
    
    def ask(self, question):
        if not self.qa_chain:
            return "Please load documents first!"
        
        result = self.qa_chain({"query": question})
        return result["result"]

# Usage
rag = SimpleRAG("your-api-key")
rag.load_documents("your-document.txt")
answer = rag.ask("Your question here")
print(answer)
```

### Key Takeaways
1. **Data Preparation**: Proper chunking is crucial for good retrieval
2. **Vector Storage**: Chroma provides efficient similarity search
3. **Embeddings**: Convert text to vectors for semantic search
4. **Retrieval**: Find relevant context using similarity search
5. **Generation**: Combine context with queries for accurate responses

### Best Practices
- Experiment with chunk sizes for your specific use case
- Use appropriate embedding models for your domain
- Fine-tune retrieval parameters (k, similarity thresholds)
- Craft clear prompts for better LLM responses
- Monitor and evaluate your RAG system's performance

### Next Steps
- Implement evaluation metrics
- Add conversation memory
- Experiment with different vector databases
- Try advanced retrieval techniques
- Build a user interface for your RAG system