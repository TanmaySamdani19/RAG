# RAG Data Ingestion and Parsing Techniques

This project demonstrates comprehensive data ingestion and parsing techniques for building Retrieval-Augmented Generation (RAG) systems. The implementation covers various data formats and sources commonly encountered in enterprise environments.

## Overview

The `0-DataIngestParsing/` directory contains Jupyter notebooks that showcase different approaches to loading, parsing, and processing data from multiple sources. Each notebook focuses on specific data types and demonstrates both standard library approaches and custom implementations.

## Data Ingestion Fundamentals

### Document Structure in LangChain

All data ingestion in this project uses LangChain's `Document` class, which provides a standardized way to represent text content with metadata:

```python
from langchain_core.documents import Document

doc = Document(
    page_content="Main text content",
    metadata={
        "source": "file_path",
        "page": 1,
        "author": "author_name",
        "date_created": "2024-01-01"
    }
)
```

**Why metadata matters:**
- Provides context for retrieval and relevance ranking
- Enables filtering by source, date, author, or custom fields
- Enhances search experience in RAG applications

### Text Splitting Strategies

Before embedding, text documents are split into manageable chunks. The project implements three main strategies:

#### 1. Character Text Splitter
- **Method**: Splits on specified separators (spaces, newlines)
- **Pros**: Simple, predictable, good for structured text
- **Cons**: May break sentences mid-way
- **Use case**: Text with clear delimiters

#### 2. Recursive Character Text Splitter
- **Method**: Tries multiple separators in order of preference
- **Pros**: Respects text structure, best general-purpose splitter
- **Cons**: Slightly more complex
- **Use case**: Default choice for most text types

#### 3. Token Text Splitter
- **Method**: Splits based on token count (not characters)
- **Pros**: Respects model token limits, more accurate for embeddings
- **Cons**: Slower than character-based methods
- **Use case**: Working with token-limited models like GPT

## Data Source Specific Techniques

### 1. Text Files (.txt)

#### Single File Loading
```python
from langchain_community.document_loaders import TextLoader

loader = TextLoader("file.txt", encoding='utf-8')
documents = loader.load()
```

#### Directory Loading
```python
from langchain_community.document_loaders import DirectoryLoader

loader = DirectoryLoader(
    'directory_path',
    glob='**/*.txt',
    loader_cls=TextLoader,
    loader_kwargs={'encoding': 'utf-8'},
    show_progress=True
)
documents = loader.load()
```

**DirectoryLoader Characteristics:**
- ✅ Loads multiple files at once
- ✅ Supports glob patterns
- ✅ Progress tracking
- ✅ Recursive directory scanning
- ❌ All files must be same type
- ❌ Limited error handling per file

### 2. PDF Documents

The project implements multiple PDF parsing approaches:

#### PyPDFLoader
- **Pros**: Simple, reliable, preserves page numbers
- **Cons**: Basic text extraction
- **Use case**: Standard text PDFs

#### PyMuPDFLoader
- **Pros**: Fast processing, good text extraction, image support
- **Use case**: Speed-critical applications

#### Custom SmartPDFProcessor
A custom class that provides:
- Intelligent text cleaning (removes ligatures, excessive whitespace)
- Enhanced metadata (page numbers, chunk counts, character counts)
- Error handling and empty page filtering
- Configurable chunking with RecursiveCharacterTextSplitter

**PDF Processing Challenges Addressed:**
- Ligature conversion (fi, fl)
- Excessive whitespace removal
- Page structure preservation
- Metadata enrichment

### 3. Word Documents (.docx)

#### Docx2txtLoader
- **Pros**: Simple extraction, good for basic documents
- **Cons**: Limited formatting preservation

#### UnstructuredWordDocumentLoader
- **Pros**: Preserves document structure, handles complex formatting
- **Mode**: "elements" for structured extraction
- **Cons**: Requires unstructured library

### 4. Structured Data (CSV/Excel)

#### CSV Processing

**Row-based Approach (CSVLoader):**
```python
from langchain_community.document_loaders import CSVLoader

loader = CSVLoader(
    file_path='data.csv',
    encoding='utf-8',
    csv_args={'delimiter': ',', 'quotechar': '"'}
)
documents = loader.load()  # One document per row
```

**Custom Intelligent Processing:**
- Creates structured content with field descriptions
- Preserves relationships and context
- Rich metadata for better retrieval
- Better suited for Q&A applications

#### Excel Processing

**Pandas-based Processing:**
- Handles multiple sheets
- Preserves sheet structure
- Creates comprehensive table summaries
- Metadata includes row/column counts

**UnstructuredExcelLoader:**
- Handles complex Excel features
- Preserves formatting information
- Requires unstructured library with Excel support

### 5. JSON Data

#### JSONLoader with jq
```python
from langchain_community.document_loaders import JSONLoader

loader = JSONLoader(
    file_path='data.json',
    jq_schema='.employees[]',  # Extract specific paths
    text_content=False  # Get full JSON objects
)
documents = loader.load()
```

#### Custom JSON Processing
- Intelligent flattening of nested structures
- Context preservation
- Structured content creation
- Rich metadata for nested objects

**Supports:**
- Regular JSON files
- JSON Lines (.jsonl) format
- Complex nested structures
- Array extraction with jq queries

### 6. SQL Databases

#### SQLDatabase Utility
```python
from langchain_community.utilities import SQLDatabase

db = SQLDatabase.from_uri("sqlite:///database.db")
print(db.get_usable_table_names())
print(db.get_table_info())
```

#### Custom SQL to Documents
- Table schema extraction
- Sample record inclusion
- Relationship document creation
- Join query results as documents

**Features:**
- Automatic table discovery
- Schema documentation
- Sample data inclusion
- Relationship mapping (e.g., employee-project joins)

## Best Practices

### 1. Error Handling
- Implement try-catch blocks for file operations
- Handle encoding issues (use UTF-8)
- Validate file existence before processing

### 2. Memory Management
- Process large files in chunks
- Use streaming for big datasets
- Monitor memory usage with large directories

### 3. Metadata Enrichment
- Include source information
- Add timestamps and version info
- Preserve original structure hints
- Add custom fields for domain-specific filtering

### 4. Text Cleaning
- Remove excessive whitespace
- Handle special characters and ligatures
- Normalize encoding
- Preserve important formatting

### 5. Chunking Strategy
- Choose splitter based on content type
- Set appropriate chunk sizes (1000-2000 characters)
- Use overlap for context preservation
- Consider token limits for embedding models

## Dependencies

Key libraries used:
- `langchain-core`: Document structure
- `langchain-community`: Loaders and utilities
- `langchain-text-splitters`: Text splitting
- `PyPDF2` or `pypdf`: PDF processing
- `python-docx`: Word document handling
- `pandas`: Structured data processing
- `sqlite3`: Database operations
- `unstructured`: Advanced document parsing

## Usage Examples

See individual notebooks in `0-DataIngestParsing/` for complete working examples:

1. `1-dataingestion.ipynb`: Text files and splitting
2. `2-dataparsingpdf.ipynb`: PDF processing
3. `3-dataparsingdoc.ipynb`: Word documents
4. `4-csvexcelparsing.ipynb`: Structured data
5. `5-jsonparsing.ipynb`: JSON processing
6. `6-databaseparsing.ipynb`: SQL databases

Each notebook includes:
- Sample data creation
- Multiple processing approaches
- Comparison of methods
- Best practice recommendations

## Next Steps

After data ingestion and parsing, the processed documents can be:
1. Embedded using vector databases (see `1-VectorEmbeddingAndDatabases/`)
2. Indexed for retrieval
3. Used in RAG pipelines for question-answering

This foundation enables building robust RAG systems that can handle diverse enterprise data sources effectively.

# Vector Embedding and Vector Databases

The `1-VectorEmbeddingAndDatabases/` directory contains implementations and demonstrations of vector embeddings and vector database technologies essential for modern RAG (Retrieval-Augmented Generation) systems.

## What are Vector Embeddings?

Vector embeddings are numerical representations of text, images, or other data in a high-dimensional vector space. They capture semantic meaning and relationships between different pieces of content.

### Key Concepts

#### 1. Semantic Similarity
- **Definition**: Embeddings place similar concepts close together in vector space
- **Example**: "cat" and "kitten" embeddings are closer than "cat" and "car"
- **Application**: Finding relevant documents without exact keyword matches

#### 2. Dimensionality
- **Typical sizes**: 384 (MiniLM), 768 (BERT-base), 1536 (OpenAI text-embedding-3-small)
- **Trade-offs**: Higher dimensions = more semantic information but higher computational cost

#### 3. Distance Metrics
- **Cosine Similarity**: Measures angle between vectors (most common)
- **Euclidean Distance**: Straight-line distance between vectors
- **Dot Product**: Measures alignment and magnitude

## Embedding Models

### OpenAI Embeddings

The project demonstrates OpenAI's embedding models through `openaiembeddings.ipynb`:

```python
from langchain_openai import OpenAIEmbeddings

# Initialize embeddings
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# Single text embedding
single_embedding = embeddings.embed_query("Your text here")

# Multiple texts embedding
multiple_embeddings = embeddings.embed_documents(["text1", "text2", "text3"])
```

**Model Comparison:**
- `text-embedding-3-small`: 1536 dimensions, cost-effective ($0.02/1M tokens)
- `text-embedding-3-large`: 3072 dimensions, highest quality ($0.13/1M tokens)
- `text-embedding-ada-002`: 1536 dimensions, legacy model ($0.10/1M tokens)

### Hugging Face Embeddings

Local embedding models that don't require API keys:

```python
from langchain_huggingface import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)
```

**Advantages:**
- No API costs
- Privacy (data stays local)
- Offline capability
- Customizable for domain-specific tasks

## Vector Databases

Vector databases store and efficiently search high-dimensional vectors. They enable fast similarity search at scale.

### ChromaDB

A local vector database for prototyping and small-scale applications:

```python
import chromadb

# Initialize client
client = chromadb.Client()

# Create collection
collection = client.create_collection(name="my_collection")

# Add documents with embeddings
collection.add(
    embeddings=[[0.1, 0.2, ...], [0.3, 0.4, ...]],
    documents=["Document 1", "Document 2"],
    ids=["id1", "id2"]
)

# Query similar documents
results = collection.query(
    query_embeddings=[[0.1, 0.2, ...]],
    n_results=5
)
```

**Features:**
- ✅ Local storage (no cloud required)
- ✅ Simple API
- ✅ Metadata filtering
- ✅ Persistent storage
- ❌ Limited scalability for large datasets

### FAISS (Facebook AI Similarity Search)

High-performance vector search library optimized for speed:

```python
import faiss
import numpy as np

# Create index
dimension = 1536  # embedding dimension
index = faiss.IndexFlatL2(dimension)  # L2 distance

# Add vectors
vectors = np.array([[0.1, 0.2, ...], [0.3, 0.4, ...]])
index.add(vectors)

# Search
query_vector = np.array([[0.1, 0.2, ...]])
distances, indices = index.search(query_vector, k=5)
```

**Features:**
- ✅ Extremely fast search
- ✅ Multiple index types (IVF, HNSW, PQ)
- ✅ GPU acceleration support
- ✅ Memory-efficient for large datasets
- ❌ More complex API than ChromaDB

## Integration with LangChain

The project uses LangChain's vector store abstractions for unified interface:

```python
from langchain_chroma import Chroma
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

# Initialize embeddings
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# Create vector store with Chroma
vectorstore = Chroma.from_documents(
    documents=documents,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# Create vector store with FAISS
vectorstore = FAISS.from_documents(
    documents=documents,
    embedding=embeddings
)

# Similarity search
results = vectorstore.similarity_search("query", k=5)
```

## Semantic Search Implementation

The notebooks implement semantic search from scratch to demonstrate core concepts:

```python
def semantic_search(query, documents, embeddings_model, top_k=3):
    # Embed query and documents
    query_embedding = embeddings_model.embed_query(query)
    doc_embeddings = embeddings_model.embed_documents(documents)

    # Calculate similarities
    similarities = []
    for i, doc_emb in enumerate(doc_embeddings):
        similarity = cosine_similarity(query_embedding, doc_emb)
        similarities.append((similarity, documents[i]))

    # Sort and return top results
    similarities.sort(reverse=True)
    return similarities[:top_k]
```

## Best Practices

### 1. Embedding Selection
- **Task-specific models**: Use domain-specific embeddings for better performance
- **Dimension trade-offs**: Balance accuracy vs. computational cost
- **Model updates**: Consider newer models as they become available

### 2. Vector Database Choice
- **Development**: ChromaDB for quick prototyping
- **Production**: FAISS or Pinecone for scale
- **Hybrid**: Combine multiple databases for different use cases

### 3. Indexing Strategies
- **Flat indexes**: Exact search, good for small datasets
- **Approximate indexes**: Faster search with small accuracy trade-off
- **Hierarchical indexes**: Best for very large datasets

### 4. Performance Optimization
- **Batch processing**: Embed documents in batches
- **Caching**: Cache embeddings to avoid recomputation
- **Filtering**: Use metadata filters to reduce search space

### 5. Cost Management
- **Model selection**: Balance cost and performance
- **Caching strategy**: Avoid re-embedding unchanged content
- **Batch API usage**: Use batch endpoints for cost efficiency

## Common Use Cases

### 1. Document Retrieval
- Search through large document collections
- Find relevant information for Q&A systems
- Content recommendation

### 2. Semantic Search
- Natural language queries over structured data
- Cross-lingual search
- Multi-modal search (text + images)

### 3. Clustering and Classification
- Group similar documents
- Automated categorization
- Anomaly detection

### 4. Recommendation Systems
- Content-based recommendations
- User preference matching
- Personalized search results

## Dependencies

Vector embedding and database libraries used:
- `langchain-openai`: OpenAI embeddings integration
- `langchain-huggingface`: Hugging Face model support
- `langchain-chroma`: ChromaDB vector store
- `langchain-community`: FAISS and other vector stores
- `chromadb`: Local vector database
- `faiss-cpu`: High-performance similarity search
- `numpy`: Numerical computations
- `scikit-learn`: Additional similarity metrics

## Usage Examples

See individual notebooks in `1-VectorEmbeddingAndDatabases/` for complete working examples:

1. `embedding.ipynb`: Basic embedding concepts and Hugging Face models
2. `openaiembeddings.ipynb`: OpenAI embeddings, similarity calculations, and semantic search

Each notebook includes:
- Model initialization and configuration
- Embedding generation examples
- Similarity computation demonstrations
- Search implementation walkthroughs
- Performance comparisons

## Next Steps

With embeddings and vector databases in place, you can:
1. Build retrieval systems for RAG applications
2. Implement semantic search interfaces
3. Create recommendation engines
4. Develop content analysis tools

This completes the foundation for advanced AI applications that can understand and retrieve information based on meaning rather than just keywords.
- **Method**: Splits based on token count (not characters)
- **Pros**: Respects model token limits, more accurate for embeddings
- **Cons**: Slower than character-based methods
- **Use case**: Working with token-limited models like GPT

## Data Source Specific Techniques

### 1. Text Files (.txt)

#### Single File Loading
```python
from langchain_community.document_loaders import TextLoader

loader = TextLoader("file.txt", encoding='utf-8')
documents = loader.load()
```

#### Directory Loading
```python
from langchain_community.document_loaders import DirectoryLoader

loader = DirectoryLoader(
    'directory_path',
    glob='**/*.txt',
    loader_cls=TextLoader,
    loader_kwargs={'encoding': 'utf-8'},
    show_progress=True
)
documents = loader.load()
```

**DirectoryLoader Characteristics:**
- ✅ Loads multiple files at once
- ✅ Supports glob patterns
- ✅ Progress tracking
- ✅ Recursive directory scanning
- ❌ All files must be same type
- ❌ Limited error handling per file

### 2. PDF Documents

The project implements multiple PDF parsing approaches:

#### PyPDFLoader
- **Pros**: Simple, reliable, preserves page numbers
- **Cons**: Basic text extraction
- **Use case**: Standard text PDFs

#### PyMuPDFLoader
- **Pros**: Fast processing, good text extraction, image support
- **Use case**: Speed-critical applications

#### Custom SmartPDFProcessor
A custom class that provides:
- Intelligent text cleaning (removes ligatures, excessive whitespace)
- Enhanced metadata (page numbers, chunk counts, character counts)
- Error handling and empty page filtering
- Configurable chunking with RecursiveCharacterTextSplitter

**PDF Processing Challenges Addressed:**
- Ligature conversion (fi, fl)
- Excessive whitespace removal
- Page structure preservation
- Metadata enrichment

### 3. Word Documents (.docx)

#### Docx2txtLoader
- **Pros**: Simple extraction, good for basic documents
- **Cons**: Limited formatting preservation

#### UnstructuredWordDocumentLoader
- **Pros**: Preserves document structure, handles complex formatting
- **Mode**: "elements" for structured extraction
- **Cons**: Requires unstructured library

### 4. Structured Data (CSV/Excel)

#### CSV Processing

**Row-based Approach (CSVLoader):**
```python
from langchain_community.document_loaders import CSVLoader

loader = CSVLoader(
    file_path='data.csv',
    encoding='utf-8',
    csv_args={'delimiter': ',', 'quotechar': '"'}
)
documents = loader.load()  # One document per row
```

**Custom Intelligent Processing:**
- Creates structured content with field descriptions
- Preserves relationships and context
- Rich metadata for better retrieval
- Better suited for Q&A applications

#### Excel Processing

**Pandas-based Processing:**
- Handles multiple sheets
- Preserves sheet structure
- Creates comprehensive table summaries
- Metadata includes row/column counts

**UnstructuredExcelLoader:**
- Handles complex Excel features
- Preserves formatting information
- Requires unstructured library with Excel support

### 5. JSON Data

#### JSONLoader with jq
```python
from langchain_community.document_loaders import JSONLoader

loader = JSONLoader(
    file_path='data.json',
    jq_schema='.employees[]',  # Extract specific paths
    text_content=False  # Get full JSON objects
)
documents = loader.load()
```

#### Custom JSON Processing
- Intelligent flattening of nested structures
- Context preservation
- Structured content creation
- Rich metadata for nested objects

**Supports:**
- Regular JSON files
- JSON Lines (.jsonl) format
- Complex nested structures
- Array extraction with jq queries

### 6. SQL Databases

#### SQLDatabase Utility
```python
from langchain_community.utilities import SQLDatabase

db = SQLDatabase.from_uri("sqlite:///database.db")
print(db.get_usable_table_names())
print(db.get_table_info())
```

#### Custom SQL to Documents
- Table schema extraction
- Sample record inclusion
- Relationship document creation
- Join query results as documents

**Features:**
- Automatic table discovery
- Schema documentation
- Sample data inclusion
- Relationship mapping (e.g., employee-project joins)

## Best Practices

### 1. Error Handling
- Implement try-catch blocks for file operations
- Handle encoding issues (use UTF-8)
- Validate file existence before processing

### 2. Memory Management
- Process large files in chunks
- Use streaming for big datasets
- Monitor memory usage with large directories

### 3. Metadata Enrichment
- Include source information
- Add timestamps and version info
- Preserve original structure hints
- Add custom fields for domain-specific filtering

### 4. Text Cleaning
- Remove excessive whitespace
- Handle special characters and ligatures
- Normalize encoding
- Preserve important formatting

### 5. Chunking Strategy
- Choose splitter based on content type
- Set appropriate chunk sizes (1000-2000 characters)
- Use overlap for context preservation
- Consider token limits for embedding models

## Dependencies

Key libraries used:
- `langchain-core`: Document structure
- `langchain-community`: Loaders and utilities
- `langchain-text-splitters`: Text splitting
- `PyPDF2` or `pypdf`: PDF processing
- `python-docx`: Word document handling
- `pandas`: Structured data processing
- `sqlite3`: Database operations
- `unstructured`: Advanced document parsing

## Usage Examples

See individual notebooks in `0-DataIngestParsing/` for complete working examples:

1. `1-dataingestion.ipynb`: Text files and splitting
2. `2-dataparsingpdf.ipynb`: PDF processing
3. `3-dataparsingdoc.ipynb`: Word documents
4. `4-csvexcelparsing.ipynb`: Structured data
5. `5-jsonparsing.ipynb`: JSON processing
6. `6-databaseparsing.ipynb`: SQL databases

Each notebook includes:
- Sample data creation
- Multiple processing approaches
- Comparison of methods
- Best practice recommendations

## Next Steps

After data ingestion and parsing, the processed documents can be:
1. Embedded using vector databases (see `1-VectorEmbeddingAndDatabases/`)
2. Indexed for retrieval
3. Used in RAG pipelines for question-answering

This foundation enables building robust RAG systems that can handle diverse enterprise data sources effectively.